# Hedgehog Fabric Observability and Monitoring

**Status**: Documented - Telemetry Not Configured in Current Vlab
**Last Updated**: 2025-10-15
**Research Phase**: Day 1 - Architecture Documentation

## Executive Summary

Hedgehog Fabric provides comprehensive observability through **Grafana Alloy** agents running on switches that collect metrics and logs, forwarding them to external monitoring systems. The current vlab has telemetry **disabled by default** - it must be explicitly configured to push to an external Prometheus/Loki/Grafana stack.

**Key Findings:**
- Observability is opt-in, configured via `defaultAlloyConfig` in Fabricator
- Alloy runs ON SWITCHES, not the control plane
- Metrics/logs pushed through fabric-proxy to external endpoints
- Six pre-built Grafana dashboards available as JSON
- No built-in Grafana/Prometheus - expects external LGTM stack
- Current vlab: `defaultAlloyConfig: {}` (disabled)

---

## Architecture Overview

### Telemetry Flow

```
┌─────────────────────────────────────────────────────────────┐
│                      SONiC Switch                           │
│  ┌──────────────┐         ┌──────────────┐                │
│  │ Fabric Agent │◄────────┤    Alloy     │                │
│  │  (metrics)   │ scrape  │  (collector) │                │
│  └──────────────┘         └───────┬──────┘                │
│                                    │                        │
│  ┌──────────────┐                  │                        │
│  │ Node Exporter│◄─────────────────┘                        │
│  │  (system)    │ scrape                                    │
│  └──────────────┘                                           │
│                                    │                        │
│  ┌──────────────┐                  │                        │
│  │   Syslog     │◄─────────────────┘                        │
│  │   (logs)     │ collect                                   │
│  └──────────────┘                                           │
│                                    │                        │
│                                    │ push metrics/logs      │
└────────────────────────────────────┼────────────────────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │   Control Node        │
                         │  ┌─────────────────┐  │
                         │  │  fabric-proxy   │  │
                         │  │  (NodePort 31028)│  │
                         │  └────────┬────────┘  │
                         └───────────┼───────────┘
                                     │
                                     │ forward
                                     ▼
                          ┌──────────────────────┐
                          │  External Observability│
                          │  ┌────────────────┐  │
                          │  │  Prometheus    │  │
                          │  │  (metrics)     │  │
                          │  └────────────────┘  │
                          │  ┌────────────────┐  │
                          │  │  Loki          │  │
                          │  │  (logs)        │  │
                          │  └────────────────┘  │
                          │  ┌────────────────┐  │
                          │  │  Grafana       │  │
                          │  │  (visualization)│  │
                          │  └────────────────┘  │
                          └──────────────────────┘
                          (Grafana Cloud or Self-Hosted LGTM)
```

### Components

**On Each Switch:**
- **Alloy Agent**: Grafana's telemetry collector
  - Scrapes Fabric Agent metrics (BGP, interfaces, ASIC resources, etc.)
  - Collects Node Exporter metrics (CPU, memory, disk, etc.)
  - Tails syslog for log aggregation
  - Configured via `defaultAlloyConfig` in Fabricator

**On Control Node:**
- **fabric-proxy**: HTTP proxy service
  - Receives metrics/logs from all switch Alloy agents
  - Forwards to configured external targets
  - Service: NodePort 31028
  - Runs as pod in `fab` namespace

**External (Required for Full Observability):**
- **Prometheus** (or compatible): Time-series metrics database
- **Loki** (or compatible): Log aggregation system
- **Grafana**: Visualization and dashboarding
- **Tempo** (optional): Distributed tracing
- **Mimir** (optional): Long-term Prometheus storage

**LGTM Stack**: Loki + Grafana + Tempo + Mimir (full Grafana observability stack)

---

## Configuration

### Current Vlab State

The default vlab has telemetry **DISABLED**:

```yaml
spec:
  config:
    fabric:
      defaultAlloyConfig: {}  # Empty = disabled
```

### Enabling Telemetry

To enable telemetry, patch the Fabricator with Alloy configuration:

**Option 1: Grafana Cloud (SaaS)**

```yaml
# telemetry-grafana-cloud.yaml
spec:
  config:
    fabric:
      defaultAlloyConfig:
        agentScrapeIntervalSeconds: 120          # How often to scrape Fabric Agent
        unixScrapeIntervalSeconds: 120           # How often to scrape Node Exporter
        unixExporterEnabled: true                # Enable system metrics
        collectSyslogEnabled: true               # Enable log collection

        prometheusTargets:
          grafana_cloud:
            url: https://prometheus-prod-XX.grafana.net/api/prom/push
            basicAuth:
              username: "<your-instance-id>"
              password: "<your-grafana-cloud-token>"
            labels:
              env: vlab
              cluster: default
            sendIntervalSeconds: 120
            useControlProxy: true                # Route through fabric-proxy

        lokiTargets:
          grafana_cloud:
            url: https://logs-prod-XXX.grafana.net/loki/api/v1/push
            basicAuth:
              username: "<your-instance-id>"
              password: "<your-grafana-cloud-token>"
            labels:
              env: vlab
              cluster: default
            useControlProxy: true
```

**Option 2: Self-Hosted LGTM Stack**

```yaml
# telemetry-self-hosted.yaml
spec:
  config:
    fabric:
      defaultAlloyConfig:
        agentScrapeIntervalSeconds: 120
        unixScrapeIntervalSeconds: 120
        unixExporterEnabled: true
        collectSyslogEnabled: true

        prometheusTargets:
          local:
            url: http://prometheus.monitoring.svc:9090/api/v1/write
            labels:
              env: vlab
            sendIntervalSeconds: 120
            useControlProxy: false               # Direct access if in same network

        lokiTargets:
          local:
            url: http://loki.monitoring.svc:3100/loki/api/v1/push
            labels:
              env: vlab
            useControlProxy: false

        unixExporterCollectors:                  # Optional: specific collectors
        - cpu
        - filesystem
        - loadavg
        - meminfo
        - netdev
        - diskstats
```

**Apply Configuration:**

```bash
# After installation
kubectl patch -n fab fabricator/default --type merge --patch-file telemetry.yaml

# Or during initial setup, include in fab.yaml
```

### Configuration Options

**Field Reference:**

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| `agentScrapeIntervalSeconds` | int | How often to scrape Fabric Agent | 120 |
| `unixScrapeIntervalSeconds` | int | How often to scrape system metrics | 120 |
| `unixExporterEnabled` | bool | Enable Node Exporter on switches | true |
| `collectSyslogEnabled` | bool | Collect and forward syslog | true |
| `prometheusTargets` | map | Prometheus Remote Write endpoints | {} |
| `lokiTargets` | map | Loki push endpoints | {} |
| `unixExporterCollectors` | []string | Specific collectors to enable | all |

**Prometheus/Loki Target Fields:**

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `url` | string | Remote write/push endpoint URL | Yes |
| `basicAuth.username` | string | HTTP basic auth username | No |
| `basicAuth.password` | string | HTTP basic auth password | No |
| `labels` | map | Additional labels for all metrics/logs | No |
| `sendIntervalSeconds` | int | Batch send interval (Prometheus only) | 120 |
| `useControlProxy` | bool | Route through fabric-proxy | true |

**Collector Options** (Node Exporter):
- `cpu`, `meminfo`, `loadavg`, `diskstats`, `filesystem`
- `netdev`, `netstat`, `uname`, `vmstat`
- Full list: [Alloy Unix Exporter Docs](https://grafana.com/docs/alloy/latest/reference/components/prometheus.exporter.unix/#collectors-list)

---

## Available Metrics

### Fabric Agent Metrics

**BGP Metrics:**
- BGP neighbor status (up/down)
- Prefixes received/sent
- BGP updates sent/received
- Keepalive timers
- Session state changes

**Interface Metrics:**
- Operational state (up/down)
- Administrative state
- Speed and duplex
- Packet counters (in/out, unicast/broadcast/multicast)
- Byte counters and bandwidth utilization
- Error counters (CRC, frame, overrun, etc.)
- Discard counters

**ASIC Critical Resources:**
- ACL table usage (entries used/available)
- IPv4 route table capacity
- IPv4 nexthop table usage
- IPv4 neighbor (ARP) table usage
- FDB (forwarding database) capacity
- IPMC (IP multicast) table usage

**Platform Metrics:**
- PSU voltage (input/output)
- PSU status (present, operational)
- Fan speeds (RPM)
- Temperature sensors (CPU, ASIC, PSU, optics)
- Transceiver DOM (Digital Optical Monitoring)
  - TX/RX power
  - Temperature
  - Voltage

### System Metrics (Node Exporter)

**CPU:**
- CPU utilization (user, system, idle, iowait)
- Load average (1min, 5min, 15min)
- Per-core statistics

**Memory:**
- Total/available/free memory
- Swap usage
- Buffer/cache utilization

**Disk:**
- Disk space (used/available/total)
- Inode usage
- Disk I/O (reads/writes, throughput)

**Network:**
- Interface traffic (bytes/packets in/out)
- Network errors
- Connection states

---

## Grafana Dashboards

Hedgehog provides **6 pre-built Grafana dashboards** as JSON files. These can be imported into any Grafana instance.

### Dashboard Inventory

| Dashboard | File | Panels | Purpose |
|-----------|------|--------|---------|
| **Switch Critical Resources** | `grafana_crm.json` | ~8 | ASIC resource utilization |
| **Fabric** | `grafana_fabric.json` | ~12 | BGP and underlay monitoring |
| **Interfaces** | `grafana_interfaces.json` | ~15 | Interface stats and health |
| **Logs** | `grafana_logs.json` | ~4 | Syslog and agent logs |
| **Platform** | `grafana_platform.json` | ~10 | Hardware health (PSU, fans, temp) |
| **Node Exporter** | `grafana_node_exporter.json` | ~30 | Linux system metrics |

**Location**: `/home/ubuntu/afewell-hh/docs/docs/user-guide/boards/`

### Dashboard Details

#### 1. Switch Critical Resources (`grafana_crm.json`)

Monitors programmable ASIC resource exhaustion:

**Key Panels:**
- ACL table capacity
- IPv4 route table usage
- IPv4 nexthop usage
- IPv4 neighbor (ARP) table
- FDB capacity
- IPMC table usage

**Use Case**: Identify when switch is approaching hardware limits

#### 2. Fabric Dashboard (`grafana_fabric.json`)

BGP underlay and external peering health:

**Key Panels:**
- BGP neighbor status (up/down count)
- BGP session state per neighbor
- Prefixes received/advertised
- BGP update messages
- Keepalive intervals
- Session flap history

**Use Case**: Ensure fabric underlay is stable and routing correctly

#### 3. Interfaces Dashboard (`grafana_interfaces.json`)

Comprehensive interface monitoring:

**Key Panels:**
- Admin/oper state overview
- Traffic rate (bits/sec, packets/sec)
- Interface utilization percentage
- Unicast/broadcast/multicast breakdowns
- Error counters (CRC, frame, etc.)
- Discard counters
- Per-interface detailed view

**Use Case**: Identify congestion, errors, or capacity issues

#### 4. Logs Dashboard (`grafana_logs.json`)

Log aggregation and filtering:

**Key Panels:**
- Syslog stream
- Kernel logs
- BGP daemon logs
- Fabric agent logs
- Error rate visualization
- Log level breakdown

**Use Case**: Troubleshooting and audit trail

#### 5. Platform Dashboard (`grafana_platform.json`)

Hardware health monitoring:

**Key Panels:**
- PSU voltage (input/output)
- PSU operational status
- Fan speeds (all trays)
- Temperature sensors (CPU, ASIC, PSU, ambient)
- Transceiver optics temperature
- Threshold violations

**Use Case**: Prevent hardware failures, capacity planning

#### 6. Node Exporter (`grafana_node_exporter.json`)

Full Linux system metrics (based on community dashboard #1860):

**Key Panels:**
- CPU utilization
- Memory usage
- Disk space
- Disk I/O
- Network throughput
- System load
- Process counts
- And 20+ more

**Use Case**: SONiC OS health and resource monitoring

### Importing Dashboards

**Method 1: Grafana UI**
1. Navigate to Grafana → Dashboards → Import
2. Upload JSON file or paste contents
3. Select Prometheus data source
4. Import

**Method 2: Provisioning (GitOps)**
```yaml
# grafana-dashboards-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hedgehog-dashboards
  namespace: monitoring
data:
  grafana-fabric.json: |
    <paste JSON contents>
```

---

## Kubernetes Event Monitoring

Hedgehog resources emit Kubernetes events for state changes, errors, and informational messages.

### Viewing Events

**All fabric events:**
```bash
kubectl get events -n fab --sort-by='.lastTimestamp'
```

**Watch events live:**
```bash
kubectl get events -n fab --watch
```

**Events for specific resource:**
```bash
kubectl get events -n fab --field-selector involvedObject.name=leaf-01
```

**Filter by type:**
```bash
# Warnings only
kubectl get events -n fab --field-selector type=Warning

# Normal events
kubectl get events -n fab --field-selector type=Normal
```

### Event Types

**Normal Events:**
- Switch registered successfully
- Agent connected
- Configuration applied
- VPC created/updated
- Connection established

**Warning Events:**
- Agent disconnected
- Configuration validation failed
- Resource conflict detected
- Reconciliation errors

**Example Event:**
```
LAST SEEN   TYPE     REASON               OBJECT                MESSAGE
2m          Normal   AgentConnected       switch/leaf-01        Agent connected successfully
5m          Warning  ConfigApplyFailed    switch/leaf-02        Failed to apply config: BGP peer not reachable
```

---

## Agent and Switch Logs

### Viewing Agent Logs

Once switches register, Agent pods appear in the `fab` namespace:

**List agent pods:**
```bash
kubectl get pods -n fab -l app=agent
```

**View logs for specific switch:**
```bash
kubectl logs -n fab agent-leaf-01
```

**Follow logs live:**
```bash
kubectl logs -n fab agent-leaf-01 -f
```

**Previous container logs (if crashed):**
```bash
kubectl logs -n fab agent-leaf-01 --previous
```

### Switch Console Access

**SSH to switch (once registered):**
```bash
# Via hhfab
hhfab vlab ssh leaf-01

# Manual
ssh admin@leaf-01.fabric.local
```

**Serial console:**
```bash
hhfab vlab serial leaf-01
```

**Serial console log:**
```bash
hhfab vlab seriallog leaf-01
```

### SONiC CLI Commands

**System status:**
```bash
show system status
show platform summary
show version
```

**BGP status:**
```bash
show bgp summary
show bgp neighbors
show ip route
```

**Interfaces:**
```bash
show interfaces status
show interfaces counters
show interfaces description
```

**Logs:**
```bash
show logging
tail -f /var/log/syslog
journalctl -u bgp -f
```

---

## Diagnostic Commands

### Fabric-Wide Health Check

```bash
# Wait for all switches to be ready
hhfab vlab wait-switches

# Inspect all switches
hhfab vlab inspect-switches

# Collect diagnostics from all devices
hhfab vlab show-tech
```

### Resource Status

**Fabricator status:**
```bash
kubectl get fabricator -n fab -o wide
```

**All fabric resources:**
```bash
kubectl get switches,servers,connections,agents -n fab
```

**VPC resources:**
```bash
kubectl get vpc,vpcattachment,vpcpeering -A
```

**Detailed resource inspect:**
```bash
kubectl describe switch leaf-01 -n fab
kubectl describe agent leaf-01 -n fab
```

### Connectivity Testing

**Test connectivity between servers:**
```bash
hhfab vlab test-connectivity
```

**Manual ping test:**
```bash
hhfab vlab ssh server-01
ping <server-02-ip>
```

---

## Troubleshooting Guide

### Switch Not Registering

**Symptoms:**
- No Agents pods in `fab` namespace
- `kubectl get switches -n fab` shows no resources

**Debugging:**
1. Check switch VMs are running:
   ```bash
   ps aux | grep qemu | grep leaf
   ```

2. Check serial console for boot errors:
   ```bash
   hhfab vlab serial leaf-01
   ```

3. Check fabric-boot logs:
   ```bash
   kubectl logs -n fab deployment/fabric-boot
   ```

4. Verify management network DHCP:
   ```bash
   kubectl logs -n fab deployment/fabric-dhcpd
   ```

### Agent Connection Issues

**Symptoms:**
- Agent pod exists but in CrashLoopBackOff
- Events show "Agent disconnected"

**Debugging:**
1. Check agent logs:
   ```bash
   kubectl logs -n fab agent-leaf-01
   ```

2. Verify switch is reachable:
   ```bash
   ping leaf-01.fabric.local
   ```

3. Check gNMI connectivity from agent:
   ```bash
   kubectl exec -n fab agent-leaf-01 -- gnmic -a leaf-01:9339 get --path /
   ```

### Configuration Not Applied

**Symptoms:**
- Resource created but status shows errors
- Switch behavior doesn't match desired state

**Debugging:**
1. Check resource status:
   ```bash
   kubectl describe vpc vpc-1
   ```

2. Check fabric controller logs:
   ```bash
   kubectl logs -n fab deployment/fabric-ctrl
   ```

3. Check agent logs for apply errors:
   ```bash
   kubectl logs -n fab agent-leaf-01 | grep ERROR
   ```

4. Verify on switch:
   ```bash
   hhfab vlab ssh leaf-01
   show running-config
   ```

### No Metrics in Grafana

**Symptoms:**
- Dashboards show "No Data"
- Grafana can't connect to Prometheus

**Debugging:**
1. Verify Alloy is configured:
   ```bash
   kubectl get fabricator -n fab -o yaml | grep -A 20 defaultAlloyConfig
   ```

2. Check if Alloy is running on switches (when registered):
   ```bash
   hhfab vlab ssh leaf-01
   systemctl status alloy
   ```

3. Verify fabric-proxy is accessible:
   ```bash
   kubectl get svc -n fab fabric-proxy
   curl http://control-1:31028
   ```

4. Check Prometheus/Loki endpoints are reachable from control node

---

## Curriculum Implications

### Current Vlab Limitations

**No Built-in Observability:**
- Vlab does not include Grafana/Prometheus/Loki
- Students cannot see metrics without external setup
- Telemetry configuration required for any observability

### Options for Curriculum

**Option 1: External Grafana Cloud (Easiest)**
- Pros: No infrastructure setup, free tier available
- Cons: Requires internet access, cloud account per student
- Setup time: 5 minutes per student

**Option 2: Shared LGTM Stack**
- Pros: Single infrastructure for all students, full control
- Cons: Requires deployment and maintenance
- Setup time: 2-4 hours one-time

**Option 3: Per-Student LGTM Stack**
- Pros: Isolated environments, teaches observability deployment
- Cons: Resource intensive, complex
- Setup time: 30-60 minutes per student

**Option 4: No Observability (Kubernetes Events Only)**
- Pros: No setup required, works immediately
- Cons: Limited visibility, misses key learning outcomes
- Setup time: 0 minutes

### Recommended Approach

For "Network Like a Hyperscaler" curriculum:

1. **Foundation Modules**: Use Kubernetes events and kubectl commands (no telemetry needed)
2. **Advanced Modules**: Provide Grafana Cloud setup guide + dashboards
3. **Optional Lab**: Deploy self-hosted LGTM stack as extended learning

This balances "confidence before comprehensiveness" with realistic hyperscaler practices.

---

## Summary

**Observability Status in Current Vlab:**
- ❌ Grafana/Prometheus/Loki: Not installed
- ❌ Alloy: Not configured (defaultAlloyConfig empty)
- ✅ fabric-proxy: Running and ready
- ✅ Grafana dashboards: Available as JSON
- ✅ Kubernetes events: Available immediately
- ✅ Agent logs: Will be available when switches register
- ✅ SONiC CLI: Available via switch SSH

**Key Takeaways:**
1. Observability is opt-in, requires configuration
2. External Prometheus/Loki/Grafana required for full metrics
3. Kubernetes events provide immediate visibility
4. Agent logs accessible via kubectl once switches register
5. Six production-ready dashboards provided
6. Curriculum needs to account for observability setup or omit metrics-based exercises

---

**Next Research Steps:**
- Document example telemetry configurations for curriculum use
- Evaluate Grafana Cloud free tier suitability
- Consider deploying test LGTM stack for full workflow validation
- Create "Observability Setup Guide" for students/instructors

**Document Status**: Complete - Architecture and capabilities documented
**Curriculum Impact**: Moderate - Requires decision on observability approach
