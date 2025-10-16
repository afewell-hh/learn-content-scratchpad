# Student Lab Environment Architecture

**Version:** 1.0
**Date:** 2025-10-15
**Status:** Reference Architecture

---

## Executive Summary

The student lab environment represents a **fully operational, Day 2 production-ready Hedgehog fabric** with complete observability and GitOps tooling. This reflects the reality that new Hedgehog customers receive professionally installed systems with operational tooling already configured by Hedgehog Professional Services.

**Key Principle:** Students learn to **operate** an already-installed fabric, not design or install one. Design and installation are covered conceptually but not hands-on, as new customers do not perform these tasks themselves.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Ubuntu Host (Student VM Appliance)                │
│                       IP: 192.168.88.204 (example)                   │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  EMKC (External Management K3s Cluster)                      │   │
│  │  Single-node k3d cluster: "observability"                    │   │
│  │                                                                │   │
│  │  ┌─────────────────────────────────────────────────────────┐ │   │
│  │  │ Observability Stack (namespace: observability)          │ │   │
│  │  │  • Prometheus: localhost:9090                           │ │   │
│  │  │    - Remote Write API enabled                           │ │   │
│  │  │    - Receiving metrics from 7 switches                  │ │   │
│  │  │  • Grafana: localhost:3000 (admin/prom-operator)        │ │   │
│  │  │    - 6 Hedgehog dashboards pre-configured               │ │   │
│  │  │    - Prometheus data source connected                   │ │   │
│  │  │    - Loki data source connected                         │ │   │
│  │  │  • Loki: localhost:3100                                 │ │   │
│  │  │    - Receiving logs from switches                       │ │   │
│  │  └─────────────────────────────────────────────────────────┘ │   │
│  │                                                                │   │
│  │  ┌─────────────────────────────────────────────────────────┐ │   │
│  │  │ GitOps Stack (namespaces: argocd, gitea)               │ │   │
│  │  │  • ArgoCD: localhost:8080                               │ │   │
│  │  │    - Connected to Gitea                                 │ │   │
│  │  │    - Monitoring hedgehog-config repo                    │ │   │
│  │  │    - Auto-sync enabled for training simplicity         │ │   │
│  │  │  • Gitea: localhost:3001                                │ │   │
│  │  │    - Web-based Git UI                                   │ │   │
│  │  │    - Pre-configured hedgehog-config repo                │ │   │
│  │  │    - In-browser file editing enabled                    │ │   │
│  │  └─────────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  kubectl fabric CLI Plugin                                   │   │
│  │  Location: /usr/local/bin/kubectl-fabric                     │   │
│  │  Purpose: Simplified Hedgehog operations                     │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                              ▲
                              │ Network: --controls-restricted=false
                              │
┌─────────────────────────────┼───────────────────────────────────────┐
│          VLAB (Nested VMs)  │                                        │
│                             │                                        │
│  ┌─────────────────────────┴──────────────────────────────────┐    │
│  │ Hedgehog Control Node (Immutable - DO NOT MODIFY)          │    │
│  │ SSH: localhost:22000 (user: core, key: vlab/sshkey)        │    │
│  │                                                              │    │
│  │  • Hedgehog Fabric k3s (Kubernetes API: 127.0.0.1:6443)    │    │
│  │    - Kubeconfig: /home/ubuntu/afewell-hh/learn_content_    │    │
│  │                  scratchpad/hh-kubeconfig                   │    │
│  │  • fabric-proxy (NodePort 31028)                            │    │
│  │    - Forwards telemetry to EMKC services                    │    │
│  │  • Hedgehog Fabricator (configured with telemetry)          │    │
│  │  • Alloy agents push metrics/logs to fabric-proxy           │    │
│  └──────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Switches (7x) - Default VLAB Topology                       │   │
│  │                                                               │   │
│  │  Spines:                                                      │   │
│  │    • spine-01, spine-02                                       │   │
│  │                                                               │   │
│  │  Server Leaves:                                               │   │
│  │    • leaf-01, leaf-02 (MCLAG pair - "mclag-1")               │   │
│  │    • leaf-03, leaf-04 (ESLAG pair - "eslag-1")               │   │
│  │    • leaf-05 (standalone/orphan leaf)                         │   │
│  │                                                               │   │
│  │  All switches running v0.87.4 with Alloy agents enabled      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Servers (10x) - Pre-wired, Not Running                      │   │
│  │  • server-01, server-02 (MCLAG to leaf-01/02)                │   │
│  │  • server-03 (unbundled to leaf-01)                          │   │
│  │  • server-04 (bundled to leaf-02)                            │   │
│  │  • server-05, server-06 (ESLAG to leaf-03/04)                │   │
│  │  • server-07 (unbundled to leaf-03)                          │   │
│  │  • server-08 (bundled to leaf-04)                            │   │
│  │  • server-09 (unbundled to leaf-05)                          │   │
│  │  • server-10 (bundled to leaf-05)                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Hedgehog VLAB - Default Topology

**Purpose:** Production-realistic Hedgehog fabric for hands-on learning

**Configuration:**
- **Topology:** Spine-leaf (2 spines, 5 leaves, 10 servers)
- **Launch Command:** `hhfab vlab up --controls-restricted=false`
- **Control Node Access:** `ssh -i vlab/sshkey -p 22000 core@localhost`
- **Kubernetes API:** Accessed via kubeconfig at `/home/ubuntu/afewell-hh/learn_content_scratchpad/hh-kubeconfig`

**Wiring Diagram:** Pre-applied from `hhfab-default-topo/include/vlab.generated.yaml`

**Network Resources:**
- **VLAN Namespace:** `default` (ranges: 1000-2999)
- **IPv4 Namespace:** `default` (subnets: 10.0.0.0/16)

**Default State:**
- Switches: All 7 registered and healthy
- Servers: Wired but VMs not running (students create VPCs to activate)
- VPCs: None (students create as needed)
- External Connectivity: None configured

**Important:** Students do NOT modify control node, wiring diagram, or switch configuration. These are immutable infrastructure representing professionally installed systems.

---

### 2. EMKC (External Management K3s Cluster)

**Purpose:** Host student-facing operational tools separate from Hedgehog infrastructure

**Rationale:**
- Hedgehog control node is immutable customer infrastructure
- Operational tools (Grafana, ArgoCD, Gitea) belong in separate management plane
- Mirrors real-world deployment pattern: monitoring/GitOps separate from fabric control

**Implementation:**
```bash
# Created via k3d
k3d cluster create observability \
  --port 3000:30080@server:0 \    # Grafana
  --port 8080:30808@server:0 \    # ArgoCD
  --port 3001:30301@server:0 \    # Gitea
  --port 9090:30090@server:0 \    # Prometheus
  --port 3100:30100@server:0 \    # Loki
  --api-port 6550 \
  --servers 1 --agents 0
```

**Access:**
```bash
# Switch to EMKC context
kubectl config use-context k3d-observability

# Switch to Hedgehog VLAB context
export KUBECONFIG=/home/ubuntu/afewell-hh/learn_content_scratchpad/hh-kubeconfig
```

---

### 3. Observability Stack

**Purpose:** Monitor fabric health, troubleshoot issues, demonstrate operational excellence

#### Prometheus
- **URL:** http://localhost:9090
- **Purpose:** Metrics database for switch/fabric monitoring
- **Configuration:**
  - Remote Write API enabled
  - Receiving metrics from all 7 switches via fabric-proxy
  - 120-second scrape intervals
  - Retention: Default (15 days)

**Metrics Categories:**
- Fabric Agent metrics (BGP, interfaces, fabric state)
- Unix/Node Exporter metrics (CPU, memory, disk, network)
- Labels: `env=vlab`, `cluster=default`, `source=hedgehog-fabric`

#### Grafana
- **URL:** http://localhost:3000
- **Credentials:** `admin` / `prom-operator`
- **Data Sources:**
  - Prometheus: Pre-configured and connected
  - Loki: Pre-configured and connected

**Pre-Configured Dashboards (6 Total):**
1. **Switch Critical Resources** - ASIC utilization, route tables, MAC tables
2. **Fabric** - BGP state, fabric topology, peering status
3. **Interfaces** - Port stats, errors, throughput
4. **Logs** - Aggregated syslog from switches
5. **Platform** - Hardware health (PSU, fans, temperature, sensors)
6. **Node Exporter Full-2** - Linux system metrics

**Dashboard Usage in Training:**
- Module 1.2: Observe CRD reconciliation in real-time
- Module 3.x: Fabric health monitoring and troubleshooting
- Module 4.x: Diagnostic data collection before support escalation

#### Loki
- **URL:** http://localhost:3100
- **Purpose:** Log aggregation from switches
- **Configuration:**
  - Receiving syslog from all switches via fabric-proxy
  - Queryable from Grafana Logs dashboard

---

### 4. GitOps Stack

**Purpose:** Industry-standard configuration management workflow for Hedgehog CRDs

**Philosophy:** Students should learn GitOps as the operational model for Hedgehog fabrics. All configuration changes go through Git → ArgoCD → Kubernetes, never direct `kubectl apply`.

#### Gitea
- **URL:** http://localhost:3001
- **Purpose:** Lightweight, web-based Git server for training
- **Credentials:** TBD (will configure during setup)

**Why Gitea (not GitHub):**
- Fully local (no external dependencies)
- Browser-based file editing (GitHub-like UX)
- Simple for Git beginners
- Easy reset between training sessions

**Pre-Configured Repository:** `hedgehog-config`
```
hedgehog-config/
├── README.md                    # Repository overview
├── vpcs/
│   └── README.md               # Instructions for creating VPCs
├── vpc-attachments/
│   └── README.md               # Instructions for server attachments
└── examples/
    ├── vpc-example-1.yaml      # Sample VPC configuration
    ├── vpc-example-2.yaml      # Sample multi-VLAN VPC
    └── attachment-example.yaml # Sample VPC attachment
```

**Student Workflow:**
1. Browse to Gitea web UI
2. Navigate to `hedgehog-config` repo
3. Create/edit YAML file in browser (e.g., `vpcs/my-vpc.yaml`)
4. Commit via web UI
5. ArgoCD detects change and syncs to Hedgehog
6. Observe reconciliation in Grafana dashboards

**Key Feature:** No Git CLI required. Students can complete entire course using only web browser.

#### ArgoCD
- **URL:** http://localhost:8080
- **Purpose:** GitOps continuous delivery for Kubernetes
- **Credentials:** TBD (will configure during setup)

**Pre-Configured Application:** `hedgehog-fabric`
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hedgehog-fabric
  namespace: argocd
spec:
  project: default
  source:
    repoURL: http://gitea-http.gitea:3000/student/hedgehog-config.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc  # Points to VLAB via multi-cluster setup
    namespace: default
  syncPolicy:
    automated:
      prune: false  # Safety: Don't auto-delete resources
      selfHeal: true
    syncOptions:
    - CreateNamespace=false
```

**Configuration Notes:**
- **Auto-sync enabled:** Simplifies training (commit = deployed)
- **Prune disabled:** Safety feature - prevents accidental deletion
- **Self-heal enabled:** Reverts manual kubectl changes (teaches GitOps discipline)
- **Multi-cluster:** ArgoCD in EMKC manages resources in VLAB cluster

**ArgoCD Setup Details:**
- Namespace: `argocd`
- Access: Via port-forward or LoadBalancer service
- RBAC: Read-only for students (view sync status, logs)

---

### 5. kubectl fabric CLI Plugin

**Purpose:** Simplified, user-friendly interface for common Hedgehog operations

**Location:** `/usr/local/bin/kubectl-fabric`

**Key Commands Students Will Use:**
```bash
# View fabric topology
kubectl fabric inspect

# List VPCs
kubectl fabric vpc list

# Create VPC (simplified syntax)
kubectl fabric vpc create my-vpc --subnet 10.0.1.0/24 --vlan 1001

# View switch connections
kubectl fabric connection list

# View server attachments
kubectl fabric server list

# Check switch status
kubectl fabric switch list
```

**When to Use:**
- **GitOps (preferred):** For configuration changes (VPCs, attachments) - via Gitea → ArgoCD
- **kubectl fabric:** For read-only inspection, troubleshooting, validation

**Installation:** Binary copied from control node to Ubuntu host during environment setup

---

## Network Architecture

### Host Networking
- **Host IP:** 192.168.88.204 (example - will vary per deployment)
- **EMKC Services:** Bound to localhost with port mappings
- **VLAB Access:** Via `--controls-restricted=false` flag (allows control node → host connectivity)

### EMKC → VLAB Communication
```
EMKC (k3d cluster)
  ↓
localhost ports (3000, 8080, 9090, etc.)
  ↓
Host IP (192.168.88.204)
  ↓
VLAB control node (via usernet bridge)
  ↓
fabric-proxy (NodePort 31028)
  ↓
Switches (Alloy agents)
```

**ArgoCD → VLAB (GitOps Flow):**
```
Gitea (EMKC)
  ↓
ArgoCD (EMKC)
  ↓
Multi-cluster config pointing to VLAB Kubernetes API
  ↓
VLAB control node k3s API (via host IP:6443 or similar)
  ↓
Hedgehog CRDs applied
```

### Hedgehog Telemetry Flow
```
Switch Alloy Agent (172.28.x.x)
  ↓
fabric-proxy (172.29.163.50:3128 - ClusterIP in VLAB)
  ↓
Host IP:9090 (Prometheus Remote Write)
Host IP:3100 (Loki Push API)
  ↓
EMKC Services
```

**Configuration:** Applied via `fabricator/default` patch:
```yaml
spec:
  config:
    fabric:
      defaultAlloyConfig:
        agentScrapeIntervalSeconds: 120
        unixScrapeIntervalSeconds: 120
        unixExporterEnabled: true
        collectSyslogEnabled: true
        prometheusTargets:
          host_prometheus:
            url: http://192.168.88.204:9090/api/v1/write
            labels:
              env: vlab
              cluster: default
            sendIntervalSeconds: 120
            useControlProxy: true
        lokiTargets:
          host_loki:
            url: http://192.168.88.204:3100/loki/api/v1/push
            labels:
              env: vlab
              cluster: default
            useControlProxy: true
```

---

## Credentials & Access

### Default Credentials (Training Environment Only)

| Service | URL | Username | Password | Notes |
|---------|-----|----------|----------|-------|
| Grafana | http://localhost:3000 | admin | prom-operator | Pre-configured dashboards |
| ArgoCD | http://localhost:8080 | TBD | TBD | To be set during deployment |
| Gitea | http://localhost:3001 | student | TBD | Browser-based Git |
| VLAB Control | ssh://localhost:22000 | core | SSH key | Key: `vlab/sshkey` |

**Security Note:** These are **training-only** credentials. Production Hedgehog deployments use proper authentication, RBAC, and secrets management.

---

## Student Environment Boundaries

### What Students CAN Do:
✅ Create/modify VPCs via GitOps (Gitea → ArgoCD)
✅ Create/modify VPC attachments via GitOps
✅ View Grafana dashboards and explore metrics
✅ Query Prometheus for troubleshooting
✅ View ArgoCD sync status and application health
✅ Use `kubectl fabric` CLI for inspection
✅ View switch status via `kubectl get agents`
✅ Read Hedgehog CRDs (`kubectl get vpcs`, etc.)
✅ Collect diagnostic data for support escalation

### What Students CANNOT Do:
❌ Modify wiring diagram (switches, servers, connections)
❌ Modify Fabricator configuration directly
❌ SSH to switches (not part of operational model)
❌ Deploy/modify control node infrastructure
❌ Change network namespaces (VLAN/IPv4)
❌ Create switch groups or switch profiles
❌ Modify observability stack (Prometheus, Grafana, Loki)

**Rationale:** These boundaries reflect real customer operations where:
- Design/install done by Hedgehog Professional Services
- Day 2 operators manage only VPCs and connectivity
- Infrastructure is immutable and managed centrally

---

## Day 1 vs. Day 2 Scope

### Day 1 (Out of Scope for Hands-On Training)
**Covered Conceptually Only:**
- Fabric design (topology selection, port allocation)
- Wiring diagram creation
- Control node installation
- Switch ZTP/bootstrap process
- Network namespace allocation strategy
- Observability stack installation

**Why Conceptual Only:**
- New customers receive fully installed systems from Hedgehog PS
- Design/install requires deep expertise not expected of new operators
- Training appliance arrives pre-configured (simulates post-install state)

### Day 2 (Core Training Focus)
**Hands-On Lab Exercises:**
- Creating VPCs for server connectivity
- Attaching servers to VPCs
- Validating connectivity and troubleshooting
- Monitoring fabric health via Grafana
- Using kubectl fabric CLI for diagnostics
- Collecting logs/metrics for support escalation
- Understanding CRD reconciliation
- GitOps workflow for configuration changes
- Fabric lifecycle operations (upgrades conceptually)

---

## Lab Exercise Starting Point

Every lab module begins with this consistent state:

**Infrastructure:**
- ✅ 7 switches online and healthy
- ✅ 10 servers wired but VMs not running
- ✅ Grafana dashboards showing live metrics
- ✅ ArgoCD syncing empty `hedgehog-config` repo
- ✅ No VPCs created (clean slate)
- ✅ No VPC attachments
- ✅ kubectl fabric CLI available

**Student Tasks Begin:**
1. Browse to Gitea
2. Create VPC YAML file
3. Commit via web UI
4. Observe ArgoCD sync
5. Watch reconciliation in Grafana
6. Validate with `kubectl fabric`
7. Attach servers and test connectivity

**Environment Reset:** Between modules, instructor can:
- Delete VPCs via GitOps (commit deletion)
- Or reset entire Gitea repo to clean state
- Grafana dashboards and telemetry continue uninterrupted

---

## Configuration Management Strategy

### GitOps-First Approach

**Primary Method: Gitea → ArgoCD → Hedgehog**
```
1. Student edits YAML in Gitea web UI
2. Student commits change
3. ArgoCD detects change within ~1 minute
4. ArgoCD applies to VLAB cluster
5. Hedgehog controller reconciles
6. Student observes in Grafana
```

**Benefits:**
- Audit trail (Git history)
- Repeatability (infrastructure as code)
- Industry standard (students learn real practices)
- Safe (rollback via Git revert)
- Collaborative (multiple students can work in branches)

**Fallback: kubectl fabric CLI**
- Used for emergencies or troubleshooting
- Read-only operations preferred
- Imperative changes discouraged (teach GitOps discipline)

### Example GitOps Workflow (Module 2.1)

**Lab Exercise: Create Your First VPC**

1. Navigate to http://localhost:3001 (Gitea)
2. Open `hedgehog-config` repository
3. Create new file: `vpcs/my-first-vpc.yaml`
4. Paste provided YAML:
```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPC
metadata:
  name: my-first-vpc
  namespace: default
spec:
  subnet: 10.0.100.0/24
  vlan: 1100
  dhcp:
    enable: true
    range:
      start: 10.0.100.10
      end: 10.0.100.250
```
5. Commit with message: "Create my first VPC"
6. Switch to ArgoCD UI (http://localhost:8080)
7. Observe `hedgehog-fabric` app sync status
8. Switch to Grafana (http://localhost:3000)
9. Open "Fabric" dashboard
10. Observe VPC creation metrics
11. Validate: `kubectl fabric vpc list`

**Learning Outcomes:**
- GitOps workflow muscle memory
- Commit → Sync → Reconcile pattern
- Observability during configuration changes
- Validation with CLI tools

---

## IP Addressing & Port Allocations

### EMKC Services (localhost)
| Service | Port | Internal Service | Notes |
|---------|------|------------------|-------|
| Grafana | 3000 | 30080 (NodePort) | Observability UI |
| ArgoCD | 8080 | 30808 (NodePort) | GitOps UI |
| Gitea | 3001 | 30301 (NodePort) | Git web UI |
| Prometheus | 9090 | 30090 (NodePort) | Metrics DB |
| Loki | 3100 | 30100 (NodePort) | Logs DB |

### VLAB Services
| Service | Access | Notes |
|---------|--------|-------|
| Control Node SSH | localhost:22000 | User: core, Key: vlab/sshkey |
| Kubernetes API | Via kubeconfig | Embedded in hh-kubeconfig file |
| fabric-proxy | 172.29.x.x:3128 | Internal ClusterIP, NodePort 31028 |

### Hedgehog Network Ranges
| Resource | Range/Subnet | Purpose |
|----------|--------------|---------|
| VLAN Namespace | 1000-2999 | Available VLANs for VPCs |
| IPv4 Namespace | 10.0.0.0/16 | Available IPs for VPC subnets |
| Fabric Underlay | Auto-generated | Switch-to-switch BGP (not user-visible) |

**Student VPC Guidelines (for lab exercises):**
- Use subnets within 10.0.0.0/16
- Use VLANs 1000-2999
- Suggested allocation: 10.0.X.0/24 where X = student number (1-99)
- DHCP ranges: .10 to .250 (reserve .1-.9 for gateways/infra)

---

## Deployment & Lifecycle

### Initial Setup (Instructor/Developer)
```bash
# 1. Start VLAB with connectivity to host
cd /home/ubuntu/afewell-hh/hhfab-default-topo
hhfab vlab up --controls-restricted=false

# 2. Apply wiring diagram
export KUBECONFIG=/home/ubuntu/afewell-hh/learn_content_scratchpad/hh-kubeconfig
kubectl apply -f include/vlab.generated.yaml

# 3. Deploy EMKC
k3d cluster create observability \
  --port 3000:30080@server:0 \
  --port 8080:30808@server:0 \
  --port 3001:30301@server:0 \
  --port 9090:30090@server:0 \
  --port 3100:30100@server:0 \
  --api-port 6550 \
  --servers 1 --agents 0

# 4. Deploy observability stack
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace observability --create-namespace \
  --set prometheus.prometheusSpec.enableRemoteWriteReceiver=true \
  --set grafana.service.type=NodePort \
  --set grafana.service.nodePort=30080 \
  --set prometheus.service.type=NodePort \
  --set prometheus.service.nodePort=30090

helm install loki grafana/loki-stack \
  --namespace observability \
  --set loki.service.type=NodePort \
  --set loki.service.nodePort=30100

# 5. Configure Hedgehog telemetry
kubectl patch -n fab fabricator/default --type merge \
  --patch-file /home/ubuntu/afewell-hh/observability-stack/hedgehog-telemetry.yaml

# 6. Import Grafana dashboards
# (Already imported - verify presence)

# 7. Deploy GitOps stack
# ArgoCD
kubectl create namespace argocd
helm install argocd argo/argo-cd \
  --namespace argocd \
  --set server.service.type=NodePort \
  --set server.service.nodePort=30808

# Gitea
kubectl create namespace gitea
helm install gitea gitea-charts/gitea \
  --namespace gitea \
  --set service.http.type=NodePort \
  --set service.http.nodePort=30301

# 8. Configure Gitea repository and ArgoCD application
# (Detailed in subsequent setup scripts)
```

### Student Appliance Bootstrap
**Goal:** Package entire environment as VM appliance that boots ready-to-use

**Components to Pre-Install:**
- Ubuntu 22.04 LTS or 24.04 LTS
- Docker
- k3d
- kubectl
- hhfab utility
- Hedgehog VLAB OCI images
- EMKC cluster pre-created
- All Helm charts deployed
- Grafana dashboards imported
- Gitea repository initialized
- ArgoCD application configured

**Startup Sequence:**
1. VM boots
2. VLAB auto-starts (systemd service or startup script)
3. EMKC cluster running (k3d auto-start)
4. Student browses to http://localhost:3000 (Grafana) to verify

**Credentials:** Printed in MOTD or desktop README

---

## Validation & Health Checks

### Environment Health Checklist

**VLAB:**
```bash
export KUBECONFIG=/home/ubuntu/afewell-hh/learn_content_scratchpad/hh-kubeconfig

# All switches online
kubectl get agents
# Expected: 7 agents, all with APPLIED column showing recent timestamp

# All fabric pods running
kubectl get pods -n fab
# Expected: All Running, no CrashLoopBackOff

# fabric-proxy operational
kubectl logs -n fab deployment/fabric-proxy --tail=5
# Expected: Successful POST logs to Prometheus/Loki
```

**EMKC Observability:**
```bash
kubectl config use-context k3d-observability

# All observability pods running
kubectl get pods -n observability
# Expected: Prometheus, Grafana, Loki all Running

# Metrics flowing
curl -s "http://localhost:9090/api/v1/query?query=up{source='hedgehog-fabric'}" | jq '.data.result | length'
# Expected: 7 (one per switch)

# Grafana accessible
curl -s http://localhost:3000/api/health | jq .
# Expected: {"database": "ok", "version": "..."}
```

**EMKC GitOps:**
```bash
# ArgoCD healthy
kubectl get pods -n argocd
# Expected: All Running

# Gitea healthy
kubectl get pods -n gitea
# Expected: All Running

# ArgoCD can reach Gitea
kubectl exec -n argocd deployment/argocd-server -- argocd repo list
# Expected: hedgehog-config repo listed
```

**End-to-End Validation:**
1. Create VPC via Gitea web UI
2. Observe ArgoCD sync within 60 seconds
3. Verify VPC exists: `kubectl get vpcs`
4. Check Grafana Fabric dashboard shows VPC
5. Delete VPC via Gitea (commit deletion)
6. Verify removal in Grafana

---

## Troubleshooting Common Issues

### Switches Not Registering
**Symptom:** `kubectl get agents` shows empty or fewer than 7 switches

**Causes:**
- VLAB still booting (wait 10-15 minutes after `hhfab vlab up`)
- Wiring diagram not applied
- Control node not ready

**Resolution:**
```bash
# Check control node
kubectl get nodes
# Should show: control-1 Ready

# Apply wiring if missing
kubectl apply -f /home/ubuntu/afewell-hh/hhfab-default-topo/include/vlab.generated.yaml

# Check fabric-boot logs
kubectl logs -n fab deployment/fabric-boot --tail=50
```

### Metrics Not Flowing to Prometheus
**Symptom:** Grafana dashboards show "No data"

**Causes:**
- Telemetry not configured
- Network connectivity broken (control node → host)
- fabric-proxy not running

**Resolution:**
```bash
# Test connectivity from control node
ssh -i vlab/sshkey -p 22000 core@localhost \
  "curl -s http://192.168.88.204:9090/-/healthy"
# Expected: "Prometheus Server is Healthy."

# Check fabric-proxy logs
kubectl logs -n fab deployment/fabric-proxy --tail=20
# Expected: POST requests to 192.168.88.204:9090 and :3100

# Verify telemetry config
kubectl get fabricators.fabricator.githedgehog.com -n fab -o jsonpath='{.items[0].spec.config.fabric.defaultAlloyConfig}' | jq .
# Expected: prometheusTargets and lokiTargets configured
```

### ArgoCD Not Syncing
**Symptom:** Git commits don't trigger ArgoCD sync

**Causes:**
- Repository connection broken
- Auto-sync disabled
- RBAC/permissions issue

**Resolution:**
```bash
# Check ArgoCD application status
kubectl get application -n argocd hedgehog-fabric -o yaml

# View sync logs
kubectl logs -n argocd deployment/argocd-application-controller -f

# Manually trigger sync
kubectl exec -n argocd deployment/argocd-server -- \
  argocd app sync hedgehog-fabric
```

### Gitea Not Accessible
**Symptom:** http://localhost:3001 not loading

**Causes:**
- Gitea pod not running
- Port mapping incorrect
- k3d cluster down

**Resolution:**
```bash
# Check Gitea pod
kubectl get pods -n gitea

# Check k3d cluster
k3d cluster list
# Expected: observability cluster running

# Restart port forward if needed
kubectl port-forward -n gitea svc/gitea-http 3001:3000
```

---

## File Locations & References

### Critical Paths
| Resource | Path |
|----------|------|
| Hedgehog kubeconfig | `/home/ubuntu/afewell-hh/learn_content_scratchpad/hh-kubeconfig` |
| VLAB SSH key | `/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/sshkey` |
| Wiring diagram | `/home/ubuntu/afewell-hh/hhfab-default-topo/include/vlab.generated.yaml` |
| Telemetry config | `/home/ubuntu/afewell-hh/observability-stack/hedgehog-telemetry.yaml` |
| Grafana dashboards | `/home/ubuntu/afewell-hh/docs/docs/user-guide/boards/*.json` |
| kubectl-fabric CLI | `/usr/local/bin/kubectl-fabric` |

### Documentation References
- Hedgehog Learning Philosophy: `network-like-hyperscaler/hedgehogLearningPhilosophy.md`
- EMKC Architecture Decision: `network-like-hyperscaler/architecture/ADR-001-emkc-separation.md`
- Phase 2a Context: `network-like-hyperscaler/phase-2a-context-brief.md`
- Project Plan: `network-like-hyperscaler/PROJECT_PLAN.md`

---

## Appendix: Design Decisions

### Why Separate EMKC from VLAB?
**Decision:** Deploy observability and GitOps tools in separate k3d cluster (EMKC) rather than inside Hedgehog control node.

**Rationale:**
1. **Immutability:** Hedgehog control node is customer infrastructure, not for student tools
2. **Realism:** Production Hedgehog fabrics have separate monitoring/management infrastructure
3. **Flexibility:** EMKC can be reset/modified without touching Hedgehog fabric
4. **Pedagogy:** Teaches proper separation of concerns

See `ADR-001-emkc-separation.md` for full details.

### Why Gitea Instead of GitHub?
**Decision:** Use Gitea (local Git server) instead of GitHub for student repositories.

**Rationale:**
1. **Offline:** No internet dependency for training
2. **Simple:** No GitHub account management for students
3. **Privacy:** Student work stays local
4. **UX:** GitHub-like browser editing experience
5. **Reset:** Easy to reset state between training sessions

### Why Auto-Sync in ArgoCD?
**Decision:** Enable auto-sync by default in ArgoCD training environment.

**Rationale:**
1. **Simplicity:** Commit = deploy (no manual sync step)
2. **Immediate Feedback:** Students see results quickly
3. **GitOps Discipline:** Self-heal prevents drift from manual changes

**Trade-off:** Less realistic than production (prod often uses manual sync gates), but better for learning flow.

---

**Document Owner:** Course Lead (Claude)
**Last Updated:** 2025-10-15
**Version:** 1.0 - Initial architecture definition
