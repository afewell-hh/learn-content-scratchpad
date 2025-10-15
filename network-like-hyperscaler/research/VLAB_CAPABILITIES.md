# Hedgehog Vlab Environment Capabilities

**Status**: Initial Research - In Progress
**Last Updated**: 2025-10-15
**Research Phase**: Day 1 - Environment Discovery

## Executive Summary

This document captures the complete capabilities and architecture of the Hedgehog vlab environment used for "Network Like a Hyperscaler" curriculum development. The vlab provides a comprehensive spine-leaf fabric topology with multiple redundancy patterns, showcasing real-world data center network architectures.

**Key Findings:**
- Full-featured spine-leaf topology with 7 switches and 10 servers
- Demonstrates MCLAG, ESLAG, bundled, and unbundled connection patterns
- Kubernetes-native management via CRDs (Custom Resource Definitions)
- Fabric control plane operational with fabric-proxy for API access
- Switches still initializing at time of initial documentation
- Rich observability planned via Grafana dashboards and metrics

---

## Topology Architecture

### Overview

The vlab implements a **spine-leaf architecture** in collapsed-core mode with the following components:

- **2 Spine Switches**: spine-01, spine-02
- **5 Leaf Switches**: leaf-01, leaf-02, leaf-03, leaf-04, leaf-05
- **10 Servers**: server-01 through server-10
- **1 Control Node**: control-1 (running k3s Kubernetes)

### Topology Diagram (ASCII)

```
                    ┌─────────────┐      ┌─────────────┐
                    │  spine-01   │      │  spine-02   │
                    └──────┬──────┘      └──────┬──────┘
                           │                    │
           ┌───────────────┼────────────────────┼───────────────┐
           │               │                    │               │
    ┌──────┴──────┐  ┌─────┴─────┐      ┌─────┴─────┐  ┌──────┴──────┐
    │   leaf-01   │  │  leaf-03  │      │  leaf-04  │  │   leaf-05   │
    │  (MCLAG-1)  │  │ (ESLAG-1) │      │ (ESLAG-1) │  │  (Standalone)│
    └──────┬──────┘  └─────┬─────┘      └─────┬─────┘  └──────┬──────┘
           │               │                    │               │
    ┌──────┴──────┐        │                    │               │
    │   leaf-02   │        │                    │               │
    │  (MCLAG-1)  │        │                    │               │
    └──────┬──────┘        │                    │               │
           │               │                    │               │
     [10 Servers with various redundancy patterns]
```

### Detailed Component Inventory

#### Switches

| Switch Name | Role | Redundancy Type | Switch Group | Profile | MAC Address |
|-------------|------|-----------------|--------------|---------|-------------|
| spine-01 | spine | none | - | vs (Virtual Switch) | 0c:20:12:ff:05:00 |
| spine-02 | spine | none | - | vs | 0c:20:12:ff:06:00 |
| leaf-01 | server-leaf | MCLAG | mclag-1 | vs | 0c:20:12:ff:00:00 |
| leaf-02 | server-leaf | MCLAG | mclag-1 | vs | 0c:20:12:ff:01:00 |
| leaf-03 | server-leaf | ESLAG | eslag-1 | vs | 0c:20:12:ff:02:00 |
| leaf-04 | server-leaf | ESLAG | eslag-1 | vs | 0c:20:12:ff:03:00 |
| leaf-05 | server-leaf | none | - | vs | 0c:20:12:ff:04:00 |

**Key Observations:**
- All switches run SONiC NOS (Network Operating System)
- MCLAG pair (leaf-01/leaf-02) uses traditional Multi-Chassis LAG
- ESLAG pair (leaf-03/leaf-04) uses EVPN-based Ethernet Segment LAG
- leaf-05 demonstrates standalone operation without redundancy

#### Servers

| Server Name | Connection Type | Connected Switches | Interfaces | Description |
|-------------|----------------|-------------------|------------|-------------|
| server-01 | MCLAG | leaf-01, leaf-02 | enp2s1, enp2s2 | Dual-homed with MCLAG |
| server-02 | MCLAG | leaf-01, leaf-02 | enp2s1, enp2s2 | Dual-homed with MCLAG |
| server-03 | Unbundled | leaf-01 | enp2s1 | Single link, no redundancy |
| server-04 | Bundled (LAG) | leaf-02 | enp2s1, enp2s2 | Port channel to single switch |
| server-05 | ESLAG | leaf-03, leaf-04 | enp2s1, enp2s2 | Dual-homed with ESLAG |
| server-06 | ESLAG | leaf-03, leaf-04 | enp2s1, enp2s2 | Dual-homed with ESLAG |
| server-07 | Unbundled | leaf-03 | enp2s1 | Single link, no redundancy |
| server-08 | Bundled (LAG) | leaf-04 | enp2s1, enp2s2 | Port channel to single switch |
| server-09 | Unbundled | leaf-05 | enp2s1 | Single link, no redundancy |
| server-10 | Bundled (LAG) | leaf-05 | enp2s1, enp2s2 | Port channel to single switch |

**Connection Patterns Demonstrated:**
1. **MCLAG** (server-01, server-02): Traditional multi-chassis link aggregation for dual-homing
2. **ESLAG** (server-05, server-06): EVPN-based ESI LAG for modern dual-homing
3. **Unbundled** (server-03, 07, 09): Single links without aggregation
4. **Bundled/LAG** (server-04, 08, 10): Port channels to single switch for bandwidth aggregation

### Fabric Interconnections

**Spine-to-Leaf Links:**
- Each leaf has **4 fabric uplinks total**: 2 to spine-01, 2 to spine-02
- Each spine connects to all 5 leaf switches
- Full mesh connectivity provides path redundancy and ECMP

**MCLAG Domain (leaf-01 ↔ leaf-02):**
- **Peer Links**: E1/3, E1/4 (4 links total for MCLAG peer data)
- **Session Links**: E1/1, E1/2 (control plane connectivity)

### Resource Allocation

**Network Addressing:**
- **Management Subnet**: 172.30.0.0/21
- **Fabric Subnet**: 172.30.128.0/17
- **VTEP Subnet**: 172.30.12.0/22
- **Protocol Subnet**: 172.30.8.0/22
- **VPC Workaround Subnet**: 172.30.96.0/19

**VLAN Allocation:**
- **Default VLAN Namespace**: 1000-2999 (for VPC subnets)
- **VPC IRB VLANs**: 3000-3999
- **VPC Workaround VLANs**: 100-3999

**IPv4 Namespace:**
- **Default**: 10.0.0.0/16 (available for VPC subnets)

**BGP AS Numbers:**
- **Spine ASN**: 65100
- **Leaf ASN Range**: 65101-65533
- **Gateway ASN**: 65534

---

## Hedgehog Architecture

### Kubernetes-Native Management

The entire fabric is managed via Kubernetes Custom Resource Definitions (CRDs). This provides:
- **Declarative configuration**: Desired state specified via YAML
- **GitOps-friendly**: All config can be version controlled
- **API-driven**: Programmatic access via kubectl and REST APIs
- **Event-driven**: Kubernetes reconciliation loops ensure actual state matches desired state

### CRD Hierarchy

```
API Groups:
├── agent.githedgehog.com/v1beta1
│   ├── Agent (switch agent status)
│   └── Catalog (hardware/software catalog)
│
├── dhcp.githedgehog.com/v1beta1
│   └── DHCPSubnet (DHCP configuration)
│
├── fabricator.githedgehog.com/v1beta1
│   ├── Fabricator (fabric-wide configuration)
│   ├── ControlNode (control plane nodes)
│   └── FabNode (registered fabric switches)
│
├── vpc.githedgehog.com/v1beta1
│   ├── VPC (virtual private cloud definition)
│   ├── VPCAttachment (bind VPC to server connections)
│   ├── VPCPeering (VPC-to-VPC connectivity)
│   ├── IPv4Namespace (IP address pool management)
│   ├── External (external connectivity)
│   ├── ExternalAttachment (bind external to switches)
│   └── ExternalPeering (external BGP peers)
│
└── wiring.githedgehog.com/v1beta1
    ├── Switch (physical switch definition)
    ├── SwitchGroup (switch grouping for redundancy)
    ├── SwitchProfile (switch hardware profiles)
    ├── Server (physical server definition)
    ├── ServerProfile (server hardware profiles)
    ├── Connection (physical connectivity)
    └── VLANNamespace (VLAN pool management)
```

### Control Plane Components

**Running in `fab` namespace:**

| Component | Purpose | Status |
|-----------|---------|--------|
| fabricator-ctrl | Main fabric controller, reconciles fabric state | Running |
| fabric-ctrl | VPC/wiring controller | Running |
| fabric-boot | Switch PXE boot service | Running |
| fabric-dhcpd | DHCP service for VPC subnets | Running |
| fabric-proxy | HTTP proxy for switch internet access | Running (NodePort 31028) |
| cert-manager | Certificate management | Running |
| zot | OCI image registry (airgap mode) | Running (NodePort 31000) |
| ntp | NTP time service | Running (NodePort 30123) |
| reloader | Config reload automation | Running |

### Fabric Proxy

- **Service**: `fabric-proxy` (NodePort 31028)
- **Purpose**: Provides internet access for switches in controlled manner
- **Use Case**: Enables switches to download updates and access external resources
- **Access**: http://control-1:31028

---

## Connection Types Deep Dive

### 1. MCLAG (Multi-Chassis Link Aggregation)

**What**: Traditional proprietary dual-homing technology
**Where**: leaf-01 and leaf-02 (mclag-1 switch group)

**Components:**
- **Peer Links** (E1/3-4): Carry both control and data traffic between MCLAG peers
- **Session Links** (E1/1-2): Dedicated control plane connectivity
- **Server Links**: Dual-homed servers connect one link to each switch

**Example Connection CRD:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-01--mclag--leaf-01--leaf-02
spec:
  mclag:
    links:
    - server:
        port: server-01/enp2s1
      switch:
        port: leaf-01/E1/5
    - server:
        port: server-01/enp2s2
      switch:
        port: leaf-02/E1/5
```

### 2. ESLAG (Ethernet Segment LAG / EVPN Multi-homing)

**What**: Standards-based EVPN multi-homing (RFC 7432)
**Where**: leaf-03 and leaf-04 (eslag-1 switch group)

**Advantages over MCLAG:**
- Standards-based (EVPN)
- No dedicated peer links required
- More scalable (>2 switches possible)
- Better for modern data centers

**Example Connection CRD:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-05--eslag--leaf-03--leaf-04
spec:
  eslag:
    links:
    - server:
        port: server-05/enp2s1
      switch:
        port: leaf-03/E1/1
    - server:
        port: server-05/enp2s2
      switch:
        port: leaf-04/E1/1
```

### 3. Bundled (Port Channel / LAG to single switch)

**What**: Link aggregation to single switch
**Where**: server-04, server-08, server-10

**Use Case**: Bandwidth aggregation without switch redundancy

**Example Connection CRD:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-10--bundled--leaf-05
spec:
  bundled:
    links:
    - server:
        port: server-10/enp2s1
      switch:
        port: leaf-05/E1/2
    - server:
        port: server-10/enp2s2
      switch:
        port: leaf-05/E1/3
```

### 4. Unbundled (Single Link)

**What**: Simple single link connectivity
**Where**: server-03, server-07, server-09

**Use Case**: Simplest connection, no redundancy or aggregation

**Example Connection CRD:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-09--unbundled--leaf-05
spec:
  unbundled:
    link:
      server:
        port: server-09/enp2s1
      switch:
        port: leaf-05/E1/1
```

### 5. Fabric (Spine-Leaf Interconnects)

**What**: Multi-link spine-to-leaf connections
**Where**: All spine-leaf pairs

**Features:**
- Multiple links for bandwidth and redundancy
- ECMP load balancing across links
- eBGP for underlay routing

**Example Connection CRD:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: spine-01--fabric--leaf-01
spec:
  fabric:
    links:
    - leaf:
        port: leaf-01/E1/8
      spine:
        port: spine-01/E1/1
    - leaf:
        port: leaf-01/E1/9
      spine:
        port: spine-01/E1/2
```

---

## Vlab Management and Operations

### hhfab CLI Tool

**Version**: v0.41.3
**Location**: `/usr/local/bin/hhfab`

**Key Commands:**

```bash
# Vlab lifecycle
hhfab vlab up                     # Start vlab (includes build if needed)
hhfab vlab generate               # Generate wiring diagram

# Vlab operations
hhfab vlab ssh <vm-name>          # SSH to any VM
hhfab vlab serial <vm-name>       # Serial console access
hhfab vlab show-tech              # Collect diagnostics from all devices

# VPC automation helpers
hhfab vlab setup-vpcs             # Setup VPCs and attachments for all servers
hhfab vlab setup-peerings         # Setup VPC and External peerings
hhfab vlab test-connectivity      # Test connectivity between servers

# Switch management
hhfab vlab wait-switches          # Wait for all switches to be ready
hhfab vlab inspect-switches       # Inspect all switches
hhfab vlab switch <command>       # Manage switch power/reinstall

# Build and validation
hhfab build                       # Build installer
hhfab validate                    # Validate config and wiring
hhfab diagram                     # Generate topology diagram
```

### Accessing Vlab Resources

**Control Node:**
```bash
# SSH to control node
hhfab vlab ssh control-1

# Access via kubectl (already configured)
kubectl --kubeconfig ~/.kube/config get pods -n fab
```

**Switches:**
```bash
# Wait for switches to boot
hhfab vlab wait-switches

# SSH to switch
hhfab vlab ssh leaf-01

# Serial console
hhfab vlab serial spine-01
```

**Servers:**
```bash
# SSH to server
hhfab vlab ssh server-01

# Servers also accessible via usernet on localhost
ssh -p 22001 core@localhost  # server-01
ssh -p 22002 core@localhost  # server-02
# etc...
```

### Vlab Working Directory Structure

```
/home/ubuntu/afewell-hh/hhfab-default-topo/
├── fab.yaml                    # Fabricator and ControlNode config
├── include/
│   └── vlab.generated.yaml     # Complete wiring diagram (generated)
├── result/
│   └── control--control-1--install/
│       ├── control-usb.iso     # Bootable installer
│       └── fab.yaml            # Rendered fabric config
├── vlab/
│   ├── config.yaml             # VM topology definition
│   ├── kubeconfig              # Kubernetes access
│   ├── sshkey/sshkey.pub       # SSH keys
│   └── vms/                    # Running VMs
│       ├── control-1/
│       ├── leaf-01/
│       ├── server-01/
│       └── ...
└── vlab.hhs                    # Vlab state file
```

---

## Current State and Initialization Status

### What's Running

**Control Plane** ✅
- k3s Kubernetes cluster operational
- All Hedgehog control plane pods running
- Fabricator deployed and configured
- Fabric API and webhooks operational

**VMs** ✅
- All 18 VMs (1 control + 7 switches + 10 servers) running via QEMU
- VMs accessible via serial console and will be accessible via SSH when booted

**Services** ✅
- fabric-proxy: Available for switch internet access
- zot registry: Available for image distribution
- NTP: Available for time synchronization
- DHCP: Ready to serve VPC subnets

### What's Pending

**Switches** ⏳
- Switches are booting but not yet registered as FabNodes
- Switch agents not yet connected to control plane
- Wiring resources (Switch, Server, Connection) not yet applied to cluster
- Expected to complete during boot process

**Observability** ❓
- No Alloy metrics collectors detected
- No Prometheus/Grafana pods found
- May need to be configured separately or enabled in Fabricator config

### Next Steps for Full Initialization

1. Wait for switches to complete boot (~5-15 minutes depending on system)
2. Verify switch registration: `kubectl get agents -n fab`
3. Apply wiring diagram: Likely happens automatically or via hhfab command
4. Configure observability if needed
5. Create test VPCs and validate workflows

---

## Planned Observability Capabilities

### Grafana Dashboards

Based on documentation, the following dashboards should be available:

1. **Switch Critical Resources**
   - ACL usage
   - IPv4 route table capacity
   - IPv4 nexthop usage
   - FDB (forwarding database) usage
   - IPMC table usage

2. **Fabric Dashboard**
   - BGP neighbor status
   - BGP session states
   - BGP updates sent/received
   - Keepalive counters
   - Underlay routing health

3. **Interfaces Dashboard**
   - Operational/admin state
   - Packet counters (input/output)
   - Throughput (PPS, bits/sec)
   - Utilization percentages
   - Error and discard counters
   - Unicast/broadcast/multicast breakdowns

4. **Logs Dashboard**
   - Kernel logs
   - BGP logs
   - Switch agent logs
   - Syslog aggregation

5. **Platform Dashboard**
   - PSU voltage/status
   - Fan speeds
   - Temperature sensors (CPU, PSU, optics)
   - Transceiver DOM (Digital Optical Monitoring)

6. **Node Exporter**
   - Memory/disk usage
   - CPU/system utilization
   - SONiC OS statistics

### Metrics Collection

**Expected Components:**
- **Alloy**: Grafana Agent for metrics scraping
- **Prometheus**: Time-series database (or remote endpoint)
- **Loki**: Log aggregation
- **Grafana**: Visualization frontend

**Status**: Not yet detected in cluster - requires further investigation

---

## Gaps and Research Needed

### High Priority

1. **Switch Initialization**
   - ⏳ Waiting for switches to register as FabNodes
   - ⏳ Need to verify Agent pods come up
   - ⏳ Need to confirm wiring resources applied

2. **Observability Stack**
   - ❓ Grafana installation and access method
   - ❓ Prometheus/Alloy configuration
   - ❓ How to enable metrics collection
   - ❓ Dashboard import procedures

3. **Hands-On Workflow Testing**
   - ⏳ VPC creation end-to-end
   - ⏳ VPCAttachment application
   - ⏳ VPCPeering configuration
   - ⏳ External connectivity setup
   - ⏳ Validation commands for each workflow

### Medium Priority

4. **Diagnostic Commands**
   - No kubectl plugins detected (expected: kubectl hhfab plugins?)
   - Need to document SONiC CLI commands accessible via switch SSH
   - Need to identify Hedgehog-specific diagnostic utilities

5. **Event Streaming**
   - How to tail/follow Kubernetes events for fabric changes
   - How to access switch agent logs
   - BGP event monitoring capabilities

6. **Fault Injection**
   - How to simulate link failures
   - How to test switch failures
   - How to verify redundancy and failover
   - What chaos engineering is possible

### Low Priority

7. **Advanced Features**
   - External connectivity patterns (BGP peers)
   - Gateway configuration (if present)
   - StaticExternal connections
   - Remote VPC peering via border switches

8. **API Reference Details**
   - Full field documentation for each CRD
   - Status field interpretations
   - Validation rules and constraints

---

## References

### Documentation Locations

- **Local Docs**: `/home/ubuntu/afewell-hh/docs/docs/`
- **User Guides**: `/home/ubuntu/afewell-hh/docs/docs/user-guide/`
  - `vpcs.md` - VPC, VPCAttachment, VPCPeering
  - `connections.md` - Connection types
  - `external.md` - External connectivity
  - `devices.md` - Switch and server management
  - `grafana.md` - Observability dashboards
- **API Reference**: `/home/ubuntu/afewell-hh/docs/docs/reference/`
  - `fabric-api.md.gen` - Complete API reference
  - `profiles.md` - Hardware profiles

### Generated Wiring Diagram

- **Source**: `/home/ubuntu/afewell-hh/hhfab-default-topo/include/vlab.generated.yaml`
- **Line Count**: 563 lines
- **Contents**: Complete Switch, Server, Connection, VLANNamespace, IPv4Namespace definitions

### Useful Commands

```bash
# Check fabric status
kubectl get fabricator -n fab
kubectl get agents -n fab
kubectl get switches,servers,connections -n fab

# Check VPCs
kubectl get vpc,vpcattachment,vpcpeering

# Check events
kubectl get events -n fab --sort-by='.lastTimestamp'

# View switch logs (once registered)
kubectl logs -n fab <agent-pod-name>

# Check services
kubectl get svc -n fab
```

---

## Educational Value for Curriculum

### What Makes This Topology Ideal for Learning

1. **Comprehensive Coverage**: Demonstrates all major connection patterns in one environment
2. **Real-World Relevance**: Mirrors production data center architectures
3. **Progressive Complexity**: Can teach simple concepts first, then build up
4. **Hands-On Experimentation**: Safe environment to break and rebuild
5. **Multiple Learning Paths**:
   - Cloud-native learners: Understand Kubernetes-native networking
   - Network engineers: Learn modern data center fabrics
   - Both: Bridge the gap between worlds

### Suggested Learning Modules

Based on this topology, the curriculum could include:

1. **Foundation**: Spine-leaf architecture concepts
2. **Physical Layer**: Understanding connections and redundancy patterns
3. **Control Plane**: Kubernetes CRDs and declarative configuration
4. **Workload Networking**: VPCs and overlay networks
5. **Advanced**: Peering, external connectivity, multi-tenancy
6. **Operations**: Observability, troubleshooting, day-2 operations

---

## Next Research Actions

### Immediate (Next Session)

1. Monitor switch boot process
2. Capture example CRDs once switches register
3. Create test VPC and document workflow
4. Investigate observability stack
5. Update this document with findings

### Short Term (Day 2)

1. Complete CRD_REFERENCE.md with real examples
2. Create WORKFLOWS.md with tested procedures
3. Create OBSERVABILITY.md with confirmed access methods
4. Test all connection types
5. Document validation commands

### Before Curriculum Development (Day 3)

1. Identify any gaps between vlab capabilities and consultant plans
2. Document workarounds or limitations
3. Create mapping of curriculum modules to vlab exercises
4. Propose any vlab enhancements needed
5. Final research report with recommendations

---

**Document Status**: Living Document - Updated as research progresses
**Next Update**: After switch initialization completes
