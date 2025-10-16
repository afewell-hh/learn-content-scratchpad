# Module 1.3: Mastering the Three Interfaces

**Course**: Network Like a Hyperscaler - Course 1: Fabric Operations Foundations
**Module**: 1.3 of 4
**Status**: DESIGN - Updated for Event-Based Reconciliation
**Version**: 1.1
**Date**: 2025-10-16

---

## Executive Summary

**Module Title**: Mastering the Three Interfaces
**Duration**: 12-15 minutes
**Module Type**: Hands-on guided exploration with decision-making exercises
**Prerequisites**: Module 1.2 completion (VPC `myfirst-vpc` deployed)

**Core Concept**: This module teaches students *when*, *why*, and *how* to use each operational interface (kubectl, Gitea, Grafana) for different Day 2 operations tasks. Students learn to select the right tool for the job and correlate information across interfaces for effective troubleshooting.

**Pedagogical Approach**:
- **Learn by Doing**: Hands-on exploration of each interface
- **Decision-Making**: Explicit guidance on tool selection
- **Pattern Recognition**: Correlating data across interfaces
- **Troubleshooting Methodology**: Using all three tools together

---

## Learning Objectives

By the end of this module, students will be able to:

1. **Select the appropriate interface** for specific operational tasks (read, write, observe)
2. **Interpret kubectl CRD output** including events, reconciliation status, and resource relationships
3. **Navigate Gitea for configuration audit** including commit history and diffs
4. **Read Grafana dashboards** for fabric health monitoring (all 6 dashboards)
5. **Correlate information across interfaces** to troubleshoot configuration issues
6. **Apply troubleshooting methodology** using the three-interface approach

**Bloom's Taxonomy Level**: Apply, Analyze (higher than Module 1.2's Understand)

---

## Module Context

### Where We Are
- **Module 1.1**: Introduced Hedgehog Fabric as a pre-installed system
- **Module 1.2**: Demonstrated basic GitOps workflow (create VPC via three interfaces)
- **Module 1.3**: Deep dive into each interface and when to use them ← YOU ARE HERE
- **Module 1.4**: Course recap and forward map to provisioning operations

### What Students Already Know
- ✅ Hedgehog Fabric is a pre-installed, declaratively-managed network
- ✅ Three interfaces exist: Gitea (write), ArgoCD (deploy), Grafana (observe)
- ✅ Basic GitOps workflow: Gitea commit → ArgoCD sync → kubectl validate → Grafana monitor
- ✅ How to create a simple VPC (`myfirst-vpc` from Module 1.2)

### What Students Will Learn Here
- 🎯 **When** to use each interface (decision-making)
- 🎯 **What** information each interface provides (unique capabilities)
- 🎯 **How** to interpret output from each interface (reading the data)
- 🎯 **Why** different tools are used for different tasks (operational context)

---

## The Three Interfaces Framework

### Interface Roles

| Interface | Role | Primary Use Cases | Information Type |
|-----------|------|-------------------|------------------|
| **kubectl** | **Read/Inspect** | Current state, status checks, troubleshooting | Real-time cluster state |
| **Gitea** | **Write/Configure** | Create configs, edit VPCs, audit changes | Declarative desired state |
| **Grafana** | **Observe/Monitor** | Health trends, metrics, alerting | Time-series operational data |

### Decision Matrix for Students

**Use kubectl when you need to:**
- ✅ Check current state of VPCs, switches, connections
- ✅ View events and reconciliation status (errors, warnings)
- ✅ List all resources of a type
- ✅ Troubleshoot why something isn't working
- ✅ Get detailed YAML output for a resource

**Use Gitea when you need to:**
- ✅ Create or modify VPC configurations
- ✅ Review configuration history (who changed what, when)
- ✅ Compare configurations across time (diffs)
- ✅ Understand current desired state
- ✅ Audit configuration changes for compliance

**Use Grafana when you need to:**
- ✅ Monitor fabric health over time
- ✅ View switch metrics (CPU, memory, interfaces)
- ✅ Identify trends (traffic patterns, resource usage)
- ✅ Check if switches are healthy
- ✅ Aggregate logs from multiple switches

---

## Lab Structure

### Lab Overview
**Total Duration**: 12-15 minutes
**Structure**: 3 parts (one per interface) + integrated troubleshooting scenario
**Environment**: Uses `myfirst-vpc` VPC created in Module 1.2

**Part 1**: The Read Interface - kubectl (4-5 min)
**Part 2**: The Write Interface - Gitea (3-4 min)
**Part 3**: The Observe Interface - Grafana (5-6 min)
**Synthesis**: Using All Three Together (2 min - embedded in assessment)

---

## Part 1: The Read Interface - kubectl (4-5 minutes)

### Learning Objective
Use kubectl to inspect current cluster state, understand CRD status fields, and troubleshoot resource issues.

### Task 1.1: Inspect Your VPC (2 minutes)

**Objective**: View the VPC created in Module 1.2 and verify its reconciliation

**Commands**:
```bash
# List all VPCs in the cluster
kubectl get vpcs

# Get detailed information about myfirst-vpc
kubectl get vpc myfirst-vpc -o yaml

# Check reconciliation status via events
kubectl describe vpc myfirst-vpc
```

**Expected Output** (kubectl describe excerpt):
```
Name:         myfirst-vpc
Namespace:    default
Labels:       fabric.githedgehog.com/ipv4ns=default
              fabric.githedgehog.com/vlanns=default
...
Spec:
  Ipv4Namespace:  default
  Subnets:
    Default:
      Dhcp:
        Enable:  true
        Range:
          End:    10.0.10.250
          Start:  10.0.10.10
      Gateway:  10.0.10.1
      Subnet:   10.0.10.0/24
      Vlan:     1010
  Vlan Namespace:  default
Events:            <none>
```

**Key Observations for Students**:
- 📊 **Events: \<none>** - VPC reconciled successfully (no errors)
- 📊 **Spec fields present** - Configuration is stored in cluster
- 📊 **Gateway auto-assigned** - Hedgehog computed gateway IP (10.0.10.1)

> 💡 **Hedgehog's Event-Based Reconciliation**
> VPCs use event-based reconciliation - if `kubectl describe` shows no error events, the VPC is working. VNI and VLAN assignments happen transparently in the switches. This "no news is good news" model simplifies Day 2 operations.

**Student Exercise**:
> **Question**: How can you verify that `myfirst-vpc` was successfully reconciled?
> **How to find it**: Run `kubectl describe vpc myfirst-vpc` and check the Events section
> **Expected Answer**: No error/warning events = successful reconciliation

---

### Task 1.2: Explore Fabric Resources (2 minutes)

**Objective**: Discover what other Hedgehog CRDs exist and their relationships

**Commands**:
```bash
# List all switches (agents)
kubectl get agents -A

# List all connections between switches and servers
kubectl get connections -A

# List all VPC attachments (servers connected to VPCs)
kubectl get vpcattachments -A
```

**Expected Output - Switches**:
```
NAMESPACE   NAME       ROLE          DESCR           APPLIED   VERSION
default     leaf-01    server-leaf   VS-01 MCLAG 1   2m        v0.87.4
default     leaf-02    server-leaf   VS-02 MCLAG 1   3m        v0.87.4
default     leaf-03    server-leaf   VS-03 ESLAG 1   2m        v0.87.4
default     leaf-04    server-leaf   VS-04 ESLAG 1   3m        v0.87.4
default     leaf-05    server-leaf   VS-05           5m        v0.87.4
default     spine-01   spine         VS-06           4m        v0.87.4
default     spine-02   spine         VS-07           3m        v0.87.4
```

**Key Observations**:
- 🔍 7 switches total (2 spines, 5 leaves)
- 🔍 Role field indicates spine vs leaf
- 🔍 APPLIED column shows last successful config push
- 🔍 VERSION shows Hedgehog agent version

**Expected Output - Connections**:
```
NAME                                 TYPE           AGE
leaf-01--mclag-domain--leaf-02       mclag-domain   2h
server-01--mclag--leaf-01--leaf-02   mclag          2h
server-02--mclag--leaf-01--leaf-02   mclag          2h
server-03--unbundled--leaf-01        unbundled      2h
...
```

**Key Observations**:
- 🔍 Connection names describe topology: `server--type--switches`
- 🔍 Types: mclag (dual-homed), unbundled (single), eslag, bundled
- 🔍 Connections are pre-configured (Day 1, not Day 2 operations)

**Student Exercise**:
> **Question**: How many spine switches are in the fabric?
> **How to find it**: `kubectl get agents -A | grep spine | wc -l`
> **Expected Answer**: 2

---

### Task 1.3: Understanding kubectl describe (1 minute)

**Objective**: Use `kubectl describe` for troubleshooting context

**Commands**:
```bash
# Get verbose information about a VPC
kubectl describe vpc myfirst-vpc
```

**Expected Output** (excerpt):
```
Name:         myfirst-vpc
Namespace:    default
Labels:       fabric.githedgehog.com/ipv4ns=default
              fabric.githedgehog.com/vlanns=default
Annotations:  kubectl.kubernetes.io/last-applied-configuration: {...}
API Version:  vpc.githedgehog.com/v1beta1
Kind:         VPC
Metadata:
  Creation Timestamp:  2025-10-16T01:00:00Z
  Generation:          1
  Resource Version:    123456
  UID:                 abc-123-def
Spec:
  Ipv 4 Namespace:  default
  Subnets:
    Default:
      Dhcp:
        Enable:  true
        Range:
          End:    10.0.10.250
          Start:  10.0.10.10
      Gateway:  10.0.10.1
      Subnet:   10.0.10.0/24
      Vlan:     1010
  Vlan Namespace:  default
Status:
  Ipv 4 Namespace:  default
  State:            Active
  Subnets:
    Default:
      Dhcp:
        Range:
          End:    10.0.10.250
          Start:  10.0.10.10
      Gateway:  10.0.10.1
      Subnet:   10.0.10.0/24
      Vlan:     1010
  Vlan Namespace:  default
  Vni:             100010
Events:            <none>
```

**Key Observations**:
- 📖 `kubectl describe` shows human-readable format
- 📖 Labels show namespace membership
- 📖 Creation Timestamp answers "when was this created?"
- 📖 **Events section is key** - errors appear here (none = healthy)
- 📖 Spec shows desired configuration (what you want)

**Teaching Point**:
> 💡 **kubectl get vs kubectl describe**
> - `kubectl get` shows tabular list or raw YAML
> - `kubectl describe` shows formatted, human-readable details
> - Use `describe` when troubleshooting (easier to read)
> - Use `get -o yaml` when you need exact field values

---

### Part 1 Summary

**What Students Learned**:
- ✅ How to inspect VPC state with kubectl
- ✅ Understanding event-based reconciliation (no errors = success)
- ✅ Listing fabric resources (agents, connections)
- ✅ Difference between `kubectl get` and `kubectl describe`

**kubectl Quick Reference**:
```bash
kubectl get vpcs                  # List all VPCs
kubectl get vpc <name> -o yaml    # Detailed VPC info
kubectl describe vpc <name>       # Check events for reconciliation status
kubectl get agents -A             # List switches
kubectl get connections -A        # List connections
```

---

### Understanding Hedgehog's Reconciliation Model

**Why VPCs Have Minimal Status**:

Hedgehog uses different reconciliation patterns for different resource types:

| CRD Type | Status Approach | How to Check | Example |
|----------|----------------|--------------|---------|
| **Agent** (switches) | Rich status fields | `kubectl get agent <name> -o yaml` | `status.conditions`, `status.state.bgpNeighbors` |
| **VPC** (network config) | Event-based | `kubectl describe vpc <name>` | Events section shows errors |

**Why the difference?**
- **Agents** represent physical infrastructure (switches) with complex state to monitor
- **VPCs** represent desired configuration - reconciliation is binary (works or errors)

**For Day 2 Operations**:
- **Agents**: Use `kubectl get agent -o yaml` to see detailed switch status
- **VPCs**: Use `kubectl describe vpc` and check for error events
- **Rule of thumb**: No error events = VPC is operational

**Teaching Moment**:
> 💡 This event-based model is actually *simpler* for troubleshooting:
> - No complex status fields to parse
> - Errors appear immediately as events
> - "No news is good news" - if no errors, it's working
> - VNI/VLAN assignments happen transparently (visible on switches via Agent status)

---

## Part 2: The Write Interface - Gitea (3-4 minutes)

### Learning Objective
Use Gitea to audit configuration history, understand Git-based workflow, and trace configuration changes over time.

### Task 2.1: View Commit History (2 minutes)

**Objective**: Understand when and why configurations changed

**Steps**:
1. Open Gitea: http://localhost:3001
2. Navigate to `student/hedgehog-config` repository
3. Click on "Commits" in the top navigation
4. Review the commit history

**Expected Commit History**:
```
fb8f8fe  Fix VPC name to be within 11 character limit        1 hour ago
9fbaa1c  Create my first VPC for Module 1.2 lab             1 hour ago
392b8de  Add test VPC for GitOps workflow validation         2 hours ago
07bae96  Fix VPCAttachment subnet format                     2 hours ago
7c00656  Fix VPC examples: shorten names, add subnet         2 hours ago
...
```

**Key Observations**:
- 📜 Each commit has a SHA (unique identifier)
- 📜 Commit messages describe what changed
- 📜 Timestamps show when changes were made
- 📜 Author information shows who made the change

**Student Exercise**:
> **Question**: When was `myfirst-vpc` created? (Look for commit message)
> **How to find it**: Look for commit message containing "Create my first VPC" or "myfirst-vpc"
> **Expected Answer**: ~1 hour ago (or specific timestamp)

---

### Task 2.2: View File Changes (Diff) (1-2 minutes)

**Objective**: Compare configurations across commits to understand what changed

**Steps**:
1. In Gitea commit history, click on commit `fb8f8fe` (or latest)
2. View the "Diff" (differences) shown in green/red
3. Observe file changes

**Expected Diff View**:
```diff
vpcs/my-first-vpc.yaml → vpcs/myfirst-vpc.yaml

- name: my-first-vpc     # OLD (12 characters - invalid)
+ name: myfirst-vpc      # NEW (11 characters - valid)
```

**Key Observations**:
- 🔄 Red lines (- prefix) show deleted content
- 🔄 Green lines (+ prefix) show added content
- 🔄 File renames are tracked
- 🔄 Diffs make it easy to see exactly what changed

**Teaching Point**:
> 💡 **Why Git for Network Configuration?**
> - **Audit Trail**: Every change is recorded with who/when/why
> - **Rollback**: Can revert to previous configurations
> - **Code Review**: Changes can be reviewed before deployment
> - **Compliance**: Meets regulatory requirements for change tracking

---

### Task 2.3: Explore Repository Structure (1 minute)

**Objective**: Understand how Hedgehog configurations are organized

**Steps**:
1. Navigate to repository root
2. Browse directory structure

**Expected Structure**:
```
hedgehog-config/
├── README.md                    # Repository documentation
├── vpcs/                        # VPC configurations
│   ├── README.md                # VPC documentation
│   ├── myfirst-vpc.yaml         # Your VPC from Module 1.2
│   ├── test-vpc.yaml            # Example VPC
│   └── vpc-example-1.yaml       # Template VPC
└── vpc-attachments/             # VPC attachment configurations
    ├── README.md                # Attachment documentation
    └── attachment-example.yaml  # Example attachment
```

**Key Observations**:
- 📁 Organized by resource type (vpcs/, vpc-attachments/)
- 📁 README files provide documentation
- 📁 Examples serve as templates
- 📁 Flat structure (no deep nesting)

**Student Exercise**:
> **Question**: Where would you create a new VPC configuration file?
> **Expected Answer**: In the `vpcs/` directory

---

### Part 2 Summary

**What Students Learned**:
- ✅ How to view Git commit history in Gitea
- ✅ Reading diffs to understand configuration changes
- ✅ Repository organization for Hedgehog configs
- ✅ Why Git provides audit trail for compliance

**Gitea Quick Reference**:
- **Commits Tab**: View change history
- **Diff View**: See what changed in each commit
- **File Browser**: Navigate repository structure
- **Blame View**: See who last changed each line (advanced)

---

## Part 3: The Observe Interface - Grafana (5-6 minutes)

### Learning Objective
Navigate all 6 Hedgehog Grafana dashboards, interpret metrics, and understand fabric health monitoring.

### Grafana Dashboard Tour Overview

**Access**: http://localhost:3000 (admin/prom-operator)

**6 Hedgehog Dashboards**:
1. **Fabric Dashboard** - Overall fabric health
2. **Platform Dashboard** - Control plane metrics
3. **Interfaces Dashboard** - Port utilization and link status
4. **Logs Dashboard** - Aggregated syslog from switches
5. **Node Exporter Dashboard** - Switch hardware metrics
6. **Switch CRM Dashboard** - Critical Resource Monitoring (TCAM, routes, etc.)

**Time Allocation**:
- Dashboard 1-2: 1 min each (most important)
- Dashboard 3-6: 30 seconds each (overview)
- Navigation: 1 min

---

### Task 3.1: Fabric Dashboard (1 minute)

**Purpose**: High-level fabric health overview

**URL**: http://localhost:3000/d/ab831ceb-cf5c-474a-b7e9-83dcd075c218/fabric

**Key Panels**:
- **Switch Status**: Green/red indicators for each switch
- **Fabric Topology**: Visual representation of spine-leaf
- **Active VPCs**: Count of deployed VPCs
- **Connection Health**: MCLAG, ESLAG status

**What to Look For**:
- ✅ All switches showing green (healthy)
- ✅ VPC count matches `kubectl get vpcs | wc -l`
- ✅ No red indicators or alerts

**Student Exercise**:
> **Question**: According to the Fabric dashboard, how many switches are in "Up" state?
> **Expected Answer**: 7 (all switches should be up)

**Teaching Point**:
> 💡 **Fabric Dashboard as Your Starting Point**
> - First dashboard to check each day
> - Quick fabric health at-a-glance
> - Red indicators warrant investigation
> - Drill down into specific switches from here

---

### Task 3.2: Platform Dashboard (1 minute)

**Purpose**: Hedgehog control plane health (Kubernetes & Fabricator)

**URL**: http://localhost:3000/d/f8a648b9-5510-49ca-9273-952ba6169b7b/platform

**Key Panels**:
- **Control Node Status**: CPU, memory, disk usage
- **Fabricator Controller**: Reconciliation rate, error count
- **Kubernetes API**: Request rate, latency
- **etcd Health**: Cluster database metrics

**What to Look For**:
- ✅ Control node CPU < 80%
- ✅ Fabricator error count = 0
- ✅ API server responding (low latency)
- ✅ etcd healthy (no leader elections)

**Student Exercise**:
> **Question**: What is the current CPU usage of the control node?
> **How to find it**: Look at "Control Node CPU" panel
> **Expected Answer**: < 50% (varies based on activity)

**Teaching Point**:
> 💡 **Platform = The Brain of Hedgehog**
> - Control plane manages the entire fabric
> - If control plane is unhealthy, fabric changes won't apply
> - Monitor this dashboard during VPC deployments
> - Spikes are normal during reconciliation

---

### Task 3.3: Interfaces Dashboard (1 minute)

**Purpose**: Switch port utilization and link status

**URL**: http://localhost:3000/d/a5e5b12d-b340-4753-8f83-af8d54304822/interfaces

**Key Panels**:
- **Port Status**: Up/down state for all interfaces
- **Traffic Rates**: Ingress/egress bandwidth
- **Error Counters**: CRC errors, drops, discards
- **Interface Types**: Fabric, server, MCLAG peer

**What to Look For**:
- ✅ Fabric links (spine↔leaf) all up
- ✅ Low error rates (< 0.1%)
- ✅ Traffic patterns match expected workload
- ✅ No excessive drops

**Student Exercise**:
> **Question**: Are all fabric links (spine-to-leaf connections) showing "Up" status?
> **Expected Answer**: Yes (all fabric links should be up)

**Teaching Point**:
> 💡 **Interfaces = The Fabric's Nervous System**
> - Real-time link status
> - Early warning for physical issues
> - Traffic patterns reveal workload distribution
> - Error counters indicate cable/transceiver problems

---

### Task 3.4: Logs Dashboard (1 minute)

**Purpose**: Aggregated syslog messages from all switches

**URL**: http://localhost:3000/d/c42a51e5-86a8-42a0-b1c9-d1304ae655bc/logs

**Key Panels**:
- **Recent Logs**: Stream of syslog messages
- **Log Levels**: Count by severity (info, warning, error)
- **Top Log Sources**: Which switches are logging most
- **Search**: Filter logs by keyword

**What to Look For**:
- ✅ Mostly INFO level logs (normal operations)
- ✅ No ERROR level logs (or investigate if present)
- ✅ WARNING logs during config changes (expected)
- ✅ Logs from all 7 switches present

**Student Exercise**:
> **Question**: Search for "VPC" in the logs. Do you see logs related to `myfirst-vpc` creation?
> **How to find it**: Use the search box, enter "myfirst-vpc" or "10.0.10.0"
> **Expected Answer**: Yes, should see VNI assignment and VLAN configuration logs

**Teaching Point**:
> 💡 **Logs = The Fabric's Black Box Recorder**
> - Chronological record of all switch events
> - Essential for troubleshooting intermittent issues
> - Searchable across all switches simultaneously
> - Timestamps correlate with config changes

---

### Task 3.5: Node Exporter Dashboard (30 seconds)

**Purpose**: Switch hardware metrics (CPU, memory, disk, thermals)

**URL**: http://localhost:3000/d/rYdddlPWA/node-exporter-full-2

**Key Panels**:
- **CPU Usage**: Per-switch CPU utilization
- **Memory Usage**: RAM consumption
- **Disk I/O**: Read/write rates
- **Network**: Interface statistics
- **System Load**: 1/5/15 minute load averages

**What to Look For**:
- ✅ CPU usage < 80% (spikes during config push OK)
- ✅ Memory usage stable (not growing)
- ✅ Disk space available (> 20% free)

**Teaching Point**:
> 💡 **Node Exporter = Switch Vital Signs**
> - Linux-level metrics from switch OS
> - Hardware health monitoring
> - Capacity planning data
> - Performance troubleshooting baseline

---

### Task 3.6: Switch CRM Dashboard (30 seconds)

**Purpose**: Critical Resource Monitoring - ASIC resources (TCAM, routes, neighbors)

**URL**: http://localhost:3000/d/fb08315c-cabb-4da7-9db9-2e17278f1781/switch-critical-resources

**Key Panels**:
- **TCAM Utilization**: ACL/route table space used
- **Route Count**: IPv4/IPv6 routes programmed
- **ARP/ND Neighbors**: Neighbor table entries
- **VXLAN Tunnels**: VTEP count

**What to Look For**:
- ✅ TCAM usage < 80% (room for growth)
- ✅ Route count matches expected topology
- ✅ Neighbor entries stable
- ✅ VXLAN tunnels = number of VPCs × leaf count

**Teaching Point**:
> 💡 **CRM = Switch Capacity Monitoring**
> - ASIC has finite resources (TCAM)
> - Monitor to avoid hitting hardware limits
> - Plan capacity based on utilization trends
> - Critical for large-scale deployments

---

### Part 3 Summary

**What Students Learned**:
- ✅ Purpose of each of the 6 Grafana dashboards
- ✅ Key metrics to monitor in each dashboard
- ✅ How to navigate Grafana dashboard structure
- ✅ Interpreting fabric health from metrics

**Grafana Dashboard Quick Reference**:

| Dashboard | Use When | Key Metrics |
|-----------|----------|-------------|
| **Fabric** | Daily health check | Switch status, VPC count |
| **Platform** | Control plane issues | Fabricator errors, API latency |
| **Interfaces** | Link problems | Port status, error rates |
| **Logs** | Troubleshooting | Error logs, event timeline |
| **Node Exporter** | Performance issues | CPU, memory, disk |
| **CRM** | Capacity planning | TCAM usage, route count |

---

## Integrated Troubleshooting Scenario

### Scenario: "My VPC Isn't Working"

**Presented to Students**:
> A colleague tells you: "I created a VPC called `broken-vpc` but servers can't get DHCP addresses. Can you help troubleshoot?"

**Troubleshooting Methodology - Using All Three Interfaces**:

#### Step 1: Check Configuration (Gitea)
**Question**: Is the VPC configured correctly in Git?

**Actions**:
1. Open Gitea → navigate to `vpcs/` directory
2. Look for `broken-vpc.yaml` file
3. Review DHCP configuration in the file

**What to look for**:
```yaml
spec:
  subnets:
    default:
      dhcp:
        enable: true        # ✅ Should be true
        range:
          start: 10.x.x.10  # ✅ Should be valid IP
          end: 10.x.x.250   # ✅ Should be > start
```

**Teaching Point**: If config is wrong in Git, fix it there (source of truth)

---

#### Step 2: Check Deployment Status (kubectl)
**Question**: Did the VPC actually get created in the cluster?

**Actions**:
```bash
# Check if VPC exists
kubectl get vpc broken-vpc

# Check reconciliation via events
kubectl describe vpc broken-vpc

# Look specifically for error events
kubectl describe vpc broken-vpc | tail -20
```

**What to look for**:
- **VPC exists** in `kubectl get` output
- **Events section** shows no errors (healthy) or specific error messages
- **Recent events** at bottom of describe output show reconciliation activity

**Common Error Events**:
- "Subnet overlaps with existing VPC" = configuration conflict
- "Invalid VLAN in namespace" = VLAN allocation problem
- "DHCP range invalid" = IP range configuration error

> 💡 **Event-Based Troubleshooting**
> No error events = VPC reconciled successfully. If you see Warning or Error events, the message tells you exactly what's wrong. This is simpler than parsing status fields!

**Teaching Point**: kubectl events show reconciliation result (errors if any)

---

#### Step 3: Check Fabric Health (Grafana)
**Question**: Are the switches healthy and able to serve DHCP?

**Actions**:
1. Open Grafana Fabric Dashboard
2. Check switch status (all green?)
3. Open Platform Dashboard
4. Check Fabricator errors (should be 0)
5. Open Logs Dashboard
6. Search for "broken-vpc" or "DHCP" errors

**What to look for**:
- Red switches = hardware/connectivity issue
- Fabricator errors = reconciliation loop failing
- DHCP relay logs = switches trying to forward requests
- Error logs = specific failure messages

**Teaching Point**: Grafana shows operational health separate from configuration

---

### Troubleshooting Decision Tree

```
My VPC Isn't Working
    ↓
Is the config in Gitea correct?
    ├─ NO → Fix config in Gitea, commit, wait for sync
    └─ YES → ↓
              Does kubectl show the VPC exists?
                  ├─ NO → Check ArgoCD sync status
                  └─ YES → ↓
                            Are there error events in kubectl describe?
                                ├─ YES → Fix configuration issue (event message explains what)
                                └─ NO → ↓
                                          Are switches healthy in Grafana?
                                              ├─ NO → Check Fabric/Platform dashboards
                                              └─ YES → Check Logs dashboard for DHCP errors
```

**Student Exercise**:
> **Question**: What's the first interface you should check when troubleshooting "VPC not working"?
> **Expected Answer**: Gitea (verify configuration is correct at source)

---

## Assessment Questions

### Question 1: Interface Selection (Multiple Choice)
**Stem**: You need to view historical CPU usage trends for spine-01 over the last 24 hours. Which interface should you use?

A) kubectl
B) Gitea
C) Grafana ✓
D) ArgoCD

**Explanation**: Grafana stores time-series metrics and can display trends. kubectl shows current state only (snapshot). Gitea is for configuration, not metrics.

**Learning Objective**: LO #1 - Select the appropriate interface

---

### Question 2: kubectl Event Interpretation (Multiple Choice)
**Stem**: You run `kubectl describe vpc test-vpc` and see this event:
```
Warning  ReconcileFailed  2m  fabricator  Subnet 10.10.10.0/24 overlaps with existing VPC 'prod-vpc'
```
What does this mean?

A) The VPC is being deployed (this is normal)
B) The VPC configuration has an error that must be fixed ✓
C) The VPC is waiting for switches to become available
D) The Fabricator controller needs to be restarted

**Explanation**: Warning/Error events indicate configuration problems that prevent reconciliation. This specific event shows a subnet conflict - the VPC cannot be created until the configuration is fixed (choose non-overlapping subnet). No error events = successful reconciliation.

**Learning Objective**: LO #2 - Interpret kubectl CRD output

---

### Question 3: Gitea Audit Trail (Multiple Choice)
**Stem**: Your manager asks "When was the production-vpc modified and by whom?" Which Gitea feature should you use?

A) File browser
B) Commit history ✓
C) Branch manager
D) Pull requests

**Explanation**: Commit history shows all changes with timestamps and authors. Each commit records who made the change and when.

**Learning Objective**: LO #3 - Navigate Gitea for configuration audit

---

### Question 4: Grafana Dashboard Selection (Multiple Choice)
**Stem**: You suspect a switch is dropping packets due to a bad cable. Which Grafana dashboard is MOST useful?

A) Fabric Dashboard
B) Platform Dashboard
C) Interfaces Dashboard ✓
D) Node Exporter Dashboard

**Explanation**: Interfaces Dashboard shows error counters (CRC errors, drops, discards) which indicate physical layer issues. Fabric Dashboard shows health but not detailed errors.

**Learning Objective**: LO #4 - Read Grafana dashboards

---

### Question 5: Troubleshooting Methodology (Multiple Choice)
**Stem**: A VPC appears to be configured correctly in Gitea but isn't working. What should you check NEXT?

A) Recreate the VPC in Gitea
B) Use kubectl to verify the VPC was deployed ✓
C) Check Grafana for switch health
D) Contact support immediately

**Explanation**: Follow the troubleshooting flow: Config (Gitea) → Deployment (kubectl) → Health (Grafana). If Gitea config is correct, verify deployment actually happened using kubectl.

**Learning Objective**: LO #5 - Correlate information across interfaces

---

### Question 6: Interface Correlation (Scenario-Based)
**Stem**: You see in Grafana that spine-01 is showing "Down" status. What should you do FIRST?

A) Reboot spine-01
B) Check kubectl for spine-01 agent status ✓
C) Edit spine-01 config in Gitea
D) Check Platform Dashboard

**Explanation**: Grafana shows symptoms (switch down). kubectl can tell you more detail about WHY (agent not reporting, config errors, etc.). Always gather more information before taking action.

**Learning Objective**: LO #6 - Apply troubleshooting methodology

---

## Success Criteria

Students successfully complete Module 1.3 when they can:

### Knowledge Check
- ✅ Name the primary use case for each interface (read, write, observe)
- ✅ Explain when to use kubectl vs Gitea vs Grafana
- ✅ List all 6 Grafana dashboards and their purposes
- ✅ Describe the troubleshooting flow across interfaces

### Skill Demonstration
- ✅ Use kubectl to inspect VPC and check for error events
- ✅ Navigate Gitea commit history to find configuration changes
- ✅ Open and interpret all 6 Grafana dashboards
- ✅ Follow troubleshooting methodology for "VPC not working" scenario

### Assessment Performance
- ✅ Score ≥ 80% on 6 assessment questions (5 of 6 correct)
- ✅ Correctly answer interface selection questions (Q1, Q4)
- ✅ Correctly answer troubleshooting methodology questions (Q5, Q6)

---

## Lab Environment Requirements

### Infrastructure (Same as Module 1.2)
- ✅ Gitea: http://localhost:3001 (student/hedgehog123)
- ✅ ArgoCD: http://localhost:8080 (admin/password)
- ✅ Grafana: http://localhost:3000 (admin/prom-operator)
- ✅ kubectl: CLI access with KUBECONFIG set
- ✅ VLAB: 7 switches (2 spines, 5 leaves) operational

### Pre-existing Resources (From Module 1.2)
- ✅ VPC: `myfirst-vpc` (created in Module 1.2)
- ✅ Git history: At least 2-3 commits in hedgehog-config repo
- ✅ Grafana dashboards: All 6 Hedgehog dashboards imported
- ✅ Metrics: Active time-series data from switches

### New Requirements (None)
- No new infrastructure needed
- No new VPCs created (uses existing `myfirst-vpc`)
- Lab is entirely exploratory/read-only

---

## Timing Breakdown

| Section | Duration | Cumulative |
|---------|----------|------------|
| Introduction & Learning Objectives | 1 min | 1 min |
| Part 1: kubectl Interface | 4-5 min | 5-6 min |
| Part 2: Gitea Interface | 3-4 min | 8-10 min |
| Part 3: Grafana Dashboards | 5-6 min | 13-16 min |
| Troubleshooting Scenario (in text) | 1 min | 14-17 min |
| Assessment Questions | - | - |

**Total Lab Time**: 12-15 minutes (longer than Module 1.2 due to depth)
**Assessment Time**: 3-4 minutes (6 questions)
**Total Module Time**: 15-19 minutes

---

## Instructor Notes

### Common Student Questions

**Q: "Why use three different tools? Isn't that complicated?"**
A: Each tool has a specific purpose. kubectl for current state (what's happening now), Gitea for configuration (what should happen), Grafana for trends (what's been happening over time). Using the right tool for the job is more efficient than trying to make one tool do everything.

**Q: "Can I use kubectl to change VPC configuration?"**
A: Technically yes (`kubectl edit` or `kubectl apply`), but don't. Always change configuration through Gitea (Git) to maintain audit trail and GitOps workflow. Direct kubectl edits bypass Git history and will be overwritten by ArgoCD.

**Q: "Which Grafana dashboard should I check first?"**
A: Start with Fabric Dashboard (overall health), then drill down to specific dashboards based on symptoms. For link issues → Interfaces. For control plane issues → Platform. For capacity → CRM.

**Q: "What if kubectl and Grafana show different information?"**
A: kubectl shows real-time cluster state (authoritative). Grafana shows metrics with slight delay (30-60 seconds). If they differ, kubectl is more current. Grafana is better for trends, kubectl for current state.

### Teaching Tips

1. **Demonstrate Correlation**: Show the same VPC in all three interfaces simultaneously (three browser tabs). Point out how information in one interface complements another.

2. **Use Real Examples**: Reference the `myfirst-vpc` from Module 1.2 throughout. Students already know this VPC, so focus is on tool usage, not VPC concepts.

3. **Emphasize Decision-Making**: Constantly ask "which tool would you use for X?" to build decision-making muscle memory.

4. **Dashboard Tour Pacing**: Don't get bogged down in every Grafana panel. Goal is familiarity, not mastery. Students will see dashboards again in Module 3.

5. **Troubleshooting Practice**: Walk through the troubleshooting scenario slowly, explicitly calling out "Now we use Gitea because..." "Next we use kubectl because..."

### Potential Issues

**Issue**: Students overwhelmed by 6 Grafana dashboards
**Solution**: Focus on Fabric & Platform (most important), skim the rest. Provide cheat sheet of "which dashboard for which problem"

**Issue**: kubectl output verbose and intimidating
**Solution**: Use `grep` to focus on relevant fields. Teach `kubectl get vpc <name> -o jsonpath='{.status.state}'` for specific fields.

**Issue**: Gitea UI slightly different from GitHub
**Solution**: Point out similarities (commits, diffs, files), acknowledge differences. Most Git concepts transfer.

### Validation Checklist (For Testing)

Before module release, validate:
- [ ] All kubectl commands execute successfully
- [ ] `myfirst-vpc` exists and shows correct status
- [ ] Gitea commit history has at least 3 commits
- [ ] All 6 Grafana dashboards accessible
- [ ] Grafana shows metrics from all 7 switches
- [ ] Timing: Lab completes in 12-15 minutes
- [ ] Assessment questions render correctly
- [ ] All URLs in lab instructions work
- [ ] Screenshots/diagrams present (if added during content development)

---

## Next Steps

### After Module 1.3 Completion

Students will have mastered:
- ✅ Three-interface operational model
- ✅ Tool selection for different tasks
- ✅ Basic troubleshooting methodology
- ✅ Reading Grafana dashboards

**Module 1.4 will cover**:
- Course 1 recap (Modules 1.1-1.3 summary)
- Forward map to Course 2 (Provisioning Operations)
- Introduction to upcoming topics (VPC creation, attachments, validation)
- Confidence-building: "You're ready for Day 2 operations!"

### Content Development Priorities

1. **Screenshots**: Capture Grafana dashboard examples for Part 3
2. **Callout Boxes**: Highlight teaching points (already marked with 💡)
3. **Quick Reference Cards**: Create printable kubectl/Gitea/Grafana cheat sheets
4. **Video Demos**: Short clips of navigating each interface (optional)

---

## Document Metadata

**Created**: 2025-10-16
**Author**: Course Design (Claude Code Agent)
**Review Status**: ✅ APPROVED by Course Lead (2025-10-16)
**Version**: 1.0
**Previous Module**: Module 1.2 v2.1 (GitOps VPC Creation)
**Next Module**: Module 1.4 (Course 1 Recap)

**Change Log**:
- 2025-10-16 v1.0: Initial design based on Module 1.2 v2.1 validation success
- 2025-10-16 v1.0: APPROVED by Course Lead - Ready for validation
- 2025-10-16 v1.1: **ARCHITECTURAL UPDATE** - Removed VPC status dependencies, replaced with event-based reconciliation model
  - Updated LO #2: "events, reconciliation status" instead of "status fields"
  - Updated Task 1.1: Event-based VPC validation approach
  - Added new section: "Understanding Hedgehog's Reconciliation Model"
  - Updated troubleshooting Step 2: Event-based kubectl checks
  - Updated Assessment Q2: Event interpretation instead of status fields
  - Updated all 6 Grafana dashboard UIDs (removed <uid> placeholders)
  - Rationale: VPCs use event-based reconciliation (documented in CRD_REFERENCE.md)

---

## Course Lead Approval (2025-10-16)

**Approved by**: Course Lead (Claude)
**Decision**: ✅ APPROVE for Dev Agent Validation

**Strengths**:
1. ⭐ Exceptional pedagogical structure - three parts, one per interface
2. ⭐ Embodies all 10 learning philosophy principles
3. ⭐ Decision-making framework empowers students
4. ⭐ Troubleshooting methodology is practical and reusable
5. ⭐ Builds naturally on Module 1.2's validated success
6. ⭐ All six Grafana dashboards covered systematically
7. ⭐ Assessment questions are high-quality and aligned with LOs
8. ⭐ Instructor notes are comprehensive

**Technical Validation**:
- ✅ All kubectl commands validated in prior modules
- ✅ Gitea workflows tested in Module 1.2
- ✅ Grafana dashboards match OBSERVABILITY.md research
- ✅ VPC status fields accurate per Module 1.2 validation

**Pedagogical Validation**:
- ✅ Learning philosophy alignment: 10/10 principles embodied
- ✅ Bloom's taxonomy: Apply/Analyze (appropriate progression)
- ✅ Confidence-building: Uses familiar VPC from Module 1.2
- ✅ Abstraction as empowerment: Three-interface paradigm

**Timing Assessment**:
- Estimated: 12-15 minutes (lab) + 3-4 min (assessment) = 15-19 min total
- Realistic: Yes (will be validated by dev agent)
- Note: Grafana section is tight (6 dashboards, 5-6 min) but design explicitly prioritizes skim approach

**Content Development Notes**:
- Replace `<uid>` placeholders (line 458, etc.) with actual Grafana dashboard UIDs or navigation instructions
- Screenshots recommended for Grafana dashboard sections
- Quick reference cards mentioned in Next Steps (excellent idea)

**Next Steps**:
1. Dev agent validation (test all commands, verify timing, check environment)
2. Iterate based on validation findings (if any)
3. Proceed to Module 1.4 design (Course 1 Recap)
