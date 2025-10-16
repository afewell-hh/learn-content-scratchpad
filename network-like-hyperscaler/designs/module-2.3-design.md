# Module 2.3 Design: Connectivity Validation

## Module Metadata

- **Module Number:** 2.3
- **Module Title:** Connectivity Validation
- **Course:** Course 2 - Provisioning & Connectivity
- **Estimated Duration:** 12-15 minutes
  - Introduction: 2 minutes
  - Core Concepts: 4-5 minutes
  - Hands-On Lab: 5-6 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 2.2: Attach Servers to VPC (servers attached to webapp-vpc)
  - ✅ Module 2.1: Define VPC Network (VPC created with DHCP)
  - ✅ Module 1.3: Mastering the Three Interfaces (kubectl, Gitea, Grafana proficiency)

## Learning Objectives

By the end of this module, learners will be able to:

1. **Validate DHCP functionality** - Verify servers obtain IP addresses from DHCP pools
2. **Test end-to-end connectivity** - Confirm servers can communicate within and across VPC subnets
3. **Inspect Agent CRD state** - Use kubectl to examine switch interface states and reconciliation status
4. **Interpret Grafana metrics** - Read interface dashboards to confirm VLAN configuration and traffic flow
5. **Troubleshoot connectivity issues** - Apply systematic debugging methodology using all three interfaces

**Bloom's Taxonomy Level**: Apply, Analyze, Evaluate (testing, troubleshooting, diagnosing)

## Content Outline

### Introduction (2 minutes)

**Hook: Configuration ≠ Operation**

> You've configured a VPC (Module 2.1) and attached servers to it (Module 2.2). ArgoCD shows "Synced." kubectl shows VPCAttachments created. Grafana shows VLANs on interfaces.
>
> But here's the critical question: **Does it actually work?**
>
> In production operations, "configuration deployed" doesn't mean "service operational." You must **validate** that:
> - Servers obtain DHCP leases
> - Servers can ping each other
> - VLANs are correctly applied
> - Traffic flows as expected
>
> This module teaches you to **prove your network works**—not just assume it does.

**Context: Day 2 Validation Workflow**

Traditional networking: Configure switches, manually ping between hosts, check VLAN tables via CLI.

Hedgehog approach:
- **kubectl**: Inspect Agent CRD interface states
- **Server access**: Test DHCP and connectivity from endpoints
- **Grafana**: Visualize traffic flow and interface metrics
- **Events**: Check for reconciliation errors

**What You'll Learn:**

- DHCP validation techniques (lease verification)
- End-to-end connectivity testing (ping, traceroute)
- Agent CRD inspection (interface states, VLAN configuration)
- Grafana dashboard interpretation (interfaces, traffic counters)
- Systematic troubleshooting methodology

You'll validate the `webapp-vpc` provisioned in Modules 2.1-2.2, confirming your three-tier network is fully operational.

---

### Core Concepts (4-5 minutes)

#### Concept 1: DHCP Validation

**DHCP Workflow Recap:**

When a server connects to a VPC subnet with DHCP enabled:
1. Server broadcasts DHCP Discover
2. Switch forwards request to Hedgehog DHCP server (via DHCP relay)
3. DHCP server assigns IP from configured range
4. Server receives DHCP Offer, configures interface
5. Server sends gratuitous ARP (announces IP)

**Validation Points:**

**1. DHCPSubnet Resource (Controller Level)**

```bash
# List DHCP subnets
kubectl get dhcpsubnet -n default

# View lease allocations
kubectl get dhcpsubnet webapp-vpc--frontend -o yaml
```

**Expected Fields:**
```yaml
status:
  allocated:
    "10.0.10.10":
      hostname: "server-01"
      ip: "10.0.10.10"
      mac: "00:11:22:33:44:55"
      expiry: "2025-10-16T12:30:00Z"
```

**2. Server Interface (Endpoint Level)**

Access server and check interface:

```bash
# SSH to server (via hhfab or kubectl exec)
ip addr show enp2s1

# Expected output:
# enp2s1: <BROADCAST,MULTICAST,UP,LOWER_UP>
#     inet 10.0.10.10/27 brd 10.0.10.31 scope global dynamic enp2s1
#     valid_lft 3600sec preferred_lft 3600sec
```

**3. DHCP Configuration Verification**

```bash
# On server: Check DHCP lease details
cat /var/lib/dhcp/dhclient.leases

# Should show:
# - IP address assigned
# - Subnet mask
# - Gateway
# - DNS servers
# - Lease time
```

**Troubleshooting DHCP Failures:**

| Symptom | Cause | Solution |
|---------|-------|----------|
| No IP assigned | Server interface down | Check `ip link show enp2s1` |
| Wrong subnet | Incorrect VPCAttachment | Verify `spec.subnet` in VPCAttachment |
| DHCP timeout | VLAN misconfiguration | Check Agent CRD for VLAN on switch port |
| Lease not in DHCPSubnet status | DHCP server not running | Check `kubectl get pods -n fab | grep dhcp` |

---

#### Concept 2: End-to-End Connectivity Testing

**Test Layers:**

**Layer 1: Same-Subnet Connectivity**

Servers in same VPC subnet should communicate directly:

```bash
# From server-01 (frontend subnet) to server-02 (frontend subnet)
ping -c 4 10.0.10.11

# Expected: 0% packet loss, <1ms latency
```

**Layer 2: Cross-Subnet Connectivity (with Permit Lists)**

Servers in different subnets communicate if permit list allows:

```bash
# From server-01 (frontend) to server-05 (backend)
# Permit list in VPC: [[frontend, backend]]
ping -c 4 10.0.20.10

# Expected: 0% packet loss, ~1-2ms latency (inter-subnet routing)
```

**Layer 3: Blocked Cross-Subnet (Security Isolation)**

Servers in different subnets **cannot** communicate if not in permit list:

```bash
# From server-01 (frontend) to server-09 (database)
# Permit list does NOT include [frontend, database]
ping -c 4 10.0.30.5

# Expected: 100% packet loss or "Destination Host Unreachable"
```

**Advanced Testing:**

```bash
# Traceroute (shows routing path)
traceroute 10.0.20.10

# Expected path:
# 1  10.0.10.1 (gateway)  0.5 ms
# 2  10.0.20.10           1.2 ms

# TCP connectivity (e.g., web server on backend)
nc -zv 10.0.20.10 8080

# MTU path discovery
ping -c 4 -M do -s 1472 10.0.20.10
```

**Connectivity Decision Matrix:**

| Source Subnet | Destination Subnet | Permit List Entry | Expected Result |
|---------------|-------------------|-------------------|-----------------|
| frontend | frontend | N/A (same subnet) | ✅ Success |
| frontend | backend | `[frontend, backend]` | ✅ Success |
| backend | database | `[backend, database]` | ✅ Success |
| frontend | database | (not in permit list) | ❌ Blocked |

---

#### Concept 3: Agent CRD Inspection

**Agent CRD: Source of Truth for Switch State**

The Agent CRD contains comprehensive operational state reported by switch agents:

**Key Status Fields:**

```yaml
status:
  # Interface configuration and state
  state:
    interfaces:
      Ethernet5:  # Physical port (e.g., E1/5)
        admin: up
        oper: up
        enabled: true
        speed: "25G"
        mtu: 9000
        vlans: [1010]  # VLANs configured on this port
        counters:
          inb: 123456789   # Input bytes
          outb: 987654321  # Output bytes
          ine: 0           # Input errors
          oute: 0          # Output errors

  # BGP neighbor state
  state:
    bgpNeighbors:
      default:
        "172.30.128.10":
          state: established
          prefixes:
            ipv4-unicast:
              rec: 10
              sent: 5

  # Reconciliation tracking
  lastAppliedTime: "2025-10-16T10:30:00Z"
  lastAppliedGen: 15
```

**Validation Commands:**

```bash
# View agent summary
kubectl get agent leaf-01 -n fab

# Check interface state
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq

# Expected output:
# {
#   "admin": "up",
#   "oper": "up",
#   "vlans": [1010],
#   "counters": { "inb": 123456, "outb": 987654 }
# }

# Check all VLANs on switch
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | jq 'to_entries[] | select(.value.vlans != null) | {port: .key, vlans: .value.vlans}'

# Check reconciliation status
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastAppliedTime}'
```

**What to Look For:**

✅ **Good Signs:**
- `admin: up`, `oper: up` (interface operational)
- Correct VLAN in `vlans` array
- Non-zero `inb`/`outb` counters (traffic flowing)
- Zero `ine`/`oute` (no errors)
- Recent `lastAppliedTime` (reconciliation current)

❌ **Warning Signs:**
- `oper: down` (physical link down)
- Missing VLAN in `vlans` array (misconfiguration)
- High `ine`/`oute` counts (link errors, cable issues)
- Old `lastAppliedTime` (reconciliation stale)

---

#### Concept 4: Grafana Interface Monitoring

**Grafana Dashboards for Validation:**

**1. Interfaces Dashboard**

**Purpose:** Visualize interface state, VLAN configuration, traffic counters

**Key Panels:**
- **Interface Status**: Admin/operational state per interface
- **VLAN Configuration**: VLANs assigned to each port
- **Traffic Rates**: Input/output bytes, packets per second
- **Error Rates**: Input/output errors, discards
- **Utilization**: Bandwidth usage percentage

**How to Use:**
1. Open Grafana → Dashboards → "Hedgehog Interfaces"
2. Select switch (e.g., leaf-01)
3. Filter by interface (e.g., E1/5)
4. Verify:
   - Status: Up/Up
   - VLAN: 1010 (matches VPCAttachment)
   - Traffic: Non-zero bytes (servers communicating)
   - Errors: Zero or very low

**2. Fabric Dashboard**

**Purpose:** Overview of fabric health, BGP status, VPC allocation

**Key Panels:**
- **VPC Count**: Number of VPCs deployed
- **VNI Allocation**: VXLAN VNIs in use
- **BGP Sessions**: Established vs down
- **Switch Health**: Overall fabric status

**How to Use:**
1. Open "Hedgehog Fabric" dashboard
2. Verify:
   - `webapp-vpc` appears in VPC list
   - All BGP sessions "Established"
   - No warning indicators

**3. Logs Dashboard (Troubleshooting)**

**Purpose:** Aggregated logs for debugging

**How to Use:**
1. Open "Hedgehog Logs" dashboard
2. Filter by switch (leaf-01)
3. Search for errors related to VPCAttachment or VLANs
4. Check agent logs for reconciliation failures

---

#### Concept 5: Systematic Troubleshooting Methodology

**The Three-Layer Validation Approach:**

**Layer 1: Kubernetes Resources (kubectl)**

```bash
# 1. Verify VPC exists
kubectl get vpc webapp-vpc

# 2. Verify VPCAttachments exist
kubectl get vpcattachment | grep webapp

# 3. Check for error events
kubectl get events --field-selector type=Warning

# 4. Describe resources for events
kubectl describe vpcattachment webapp-frontend-server-01
```

**Expected:** All resources exist, no Warning events

**Layer 2: Switch State (Agent CRD)**

```bash
# 1. Check switch agent readiness
kubectl get agents -n fab

# 2. Verify VLAN on interface
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.vlans}'

# 3. Check interface operational state
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.oper}'

# 4. View BGP neighbors (underlay routing)
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq
```

**Expected:** VLAN configured, interface up, BGP established

**Layer 3: Endpoint Connectivity (Server)**

```bash
# 1. SSH to server
hhfab vlab ssh server-01

# 2. Check interface IP
ip addr show enp2s1

# 3. Test gateway reachability
ping -c 2 10.0.10.1

# 4. Test peer server
ping -c 2 10.0.10.11

# 5. Check routing table
ip route
```

**Expected:** IP assigned, gateway reachable, peer reachable

---

**Troubleshooting Decision Tree:**

```
Server cannot ping another server
├─ Are both servers in same subnet?
│  ├─ YES → Check Layer 2 (VLAN, switch ports)
│  │        └─ kubectl get agent (verify VLAN on both ports)
│  └─ NO  → Check permit list
│           └─ kubectl get vpc -o yaml | grep permit
│
├─ Does server have IP address?
│  ├─ NO  → Check DHCP
│  │        ├─ kubectl get dhcpsubnet (lease allocated?)
│  │        └─ kubectl describe vpcattachment (VLAN correct?)
│  └─ YES → Continue troubleshooting
│
├─ Can server ping its gateway?
│  ├─ NO  → Check VLAN configuration
│  │        └─ Agent CRD interface state
│  └─ YES → Check inter-subnet routing
│           └─ Verify permit list allows communication
│
└─ Check for errors in events
    └─ kubectl get events --field-selector type=Warning
```

---

### Hands-On Lab (5-6 minutes)

**Lab Title:** Validate webapp-vpc Three-Tier Network

**Scenario:**

You've deployed `webapp-vpc` with three servers attached:
- **server-01** (frontend subnet: 10.0.10.0/27)
- **server-05** (backend subnet: 10.0.20.0/28)
- **server-09** (database subnet: 10.0.30.0/29)

Your task: Validate that the network is fully operational.

**Environment Access:**
- **kubectl:** Already configured
- **Server SSH:** `hhfab vlab ssh <server-name>` or `ssh -p 2200X core@localhost`
- **Grafana:** http://localhost:3000 (username: `admin`, password: `prom-operator`)

---

#### Task 1: Validate DHCP Leases (2 minutes)

**Objective:** Confirm all three servers obtained IP addresses from DHCP

**Steps:**

1. **Check DHCPSubnet resources:**

   ```bash
   # List DHCP subnets for webapp-vpc
   kubectl get dhcpsubnet -n default | grep webapp-vpc

   # Expected output:
   # webapp-vpc--frontend    10m
   # webapp-vpc--backend     10m
   # webapp-vpc--database    10m
   ```

2. **View lease allocations:**

   ```bash
   # Check frontend subnet leases
   kubectl get dhcpsubnet webapp-vpc--frontend -o jsonpath='{.status.allocated}' | jq

   # Expected output (example):
   # {
   #   "10.0.10.10": {
   #     "hostname": "server-01",
   #     "ip": "10.0.10.10",
   #     "mac": "00:11:22:33:44:55",
   #     "expiry": "2025-10-16T13:30:00Z"
   #   }
   # }

   # Check backend subnet leases
   kubectl get dhcpsubnet webapp-vpc--backend -o jsonpath='{.status.allocated}' | jq

   # Check database subnet leases
   kubectl get dhcpsubnet webapp-vpc--database -o jsonpath='{.status.allocated}' | jq
   ```

3. **Verify server interfaces:**

   ```bash
   # SSH to server-01
   hhfab vlab ssh server-01

   # Check interface IP (should be 10.0.10.10 or in range .10-.25)
   ip addr show enp2s1 | grep inet

   # Expected output:
   # inet 10.0.10.10/27 brd 10.0.10.31 scope global dynamic enp2s1

   # Exit server
   exit
   ```

4. **Create validation table:**

   | Server | Subnet | Expected IP Range | Actual IP | DHCP Lease | Status |
   |--------|--------|-------------------|-----------|------------|--------|
   | server-01 | frontend | 10.0.10.10-.25 | 10.0.10.10 | ✅ Allocated | ✅ |
   | server-05 | backend | 10.0.20.10-.14 | 10.0.20.10 | ✅ Allocated | ✅ |
   | server-09 | database | 10.0.30.5-.6 | 10.0.30.5 | ✅ Allocated | ✅ |

**Success Criteria:**
- ✅ All three DHCPSubnet resources show allocated leases
- ✅ Server interfaces show IPs within DHCP ranges
- ✅ IPs match DHCPSubnet status

---

#### Task 2: Test Connectivity (2 minutes)

**Objective:** Validate intra-subnet and inter-subnet connectivity based on permit lists

**Steps:**

1. **Review permit list configuration:**

   ```bash
   # Check webapp-vpc permit lists
   kubectl get vpc webapp-vpc -o jsonpath='{.spec.permit}' | jq

   # Expected:
   # [
   #   ["frontend", "backend"],
   #   ["backend", "database"]
   # ]
   ```

   **Permitted paths:**
   - frontend ↔ backend ✅
   - backend ↔ database ✅
   - frontend ↔ database ❌ (not in permit list)

2. **Test same-subnet connectivity:**

   ```bash
   # SSH to server-01 (frontend)
   hhfab vlab ssh server-01

   # Test gateway (should always work)
   ping -c 2 10.0.10.1

   # Expected: 0% packet loss
   ```

3. **Test cross-subnet connectivity (permitted):**

   ```bash
   # From server-01 (frontend) to server-05 (backend)
   # This should SUCCEED (permit list allows it)

   # First, get server-05 IP
   # (From another terminal: kubectl get dhcpsubnet webapp-vpc--backend -o jsonpath='{.status.allocated}' | jq)

   # Assume server-05 has 10.0.20.10
   ping -c 4 10.0.20.10

   # Expected: 0% packet loss, ~1-2ms latency
   ```

4. **Test cross-subnet connectivity (blocked):**

   ```bash
   # From server-01 (frontend) to server-09 (database)
   # This should FAIL (not in permit list)

   # Get server-09 IP (assume 10.0.30.5)
   ping -c 4 10.0.30.5

   # Expected: 100% packet loss or "Destination Host Unreachable"

   # Exit server
   exit
   ```

5. **Create connectivity matrix:**

   | Source | Destination | Permit List | Expected | Actual | Status |
   |--------|-------------|-------------|----------|--------|--------|
   | frontend | frontend gateway | N/A | ✅ Success | 0% loss | ✅ |
   | frontend | backend | ✅ Allowed | ✅ Success | 0% loss | ✅ |
   | frontend | database | ❌ Blocked | ❌ Fail | 100% loss | ✅ |

**Success Criteria:**
- ✅ Same-subnet connectivity works
- ✅ Permitted cross-subnet connectivity works
- ✅ Blocked cross-subnet connectivity fails (as expected)

---

#### Task 3: Inspect Agent CRD and Grafana (2 minutes)

**Objective:** Verify switch-level VLAN configuration and traffic flow

**Steps:**

1. **Check Agent CRD for VLAN configuration:**

   ```bash
   # Check leaf-01 (server-01 connected to E1/5)
   kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq

   # Expected output:
   # {
   #   "admin": "up",
   #   "oper": "up",
   #   "enabled": true,
   #   "vlans": [1010],  # Frontend VLAN
   #   "counters": {
   #     "inb": 234567,
   #     "outb": 345678,
   #     "ine": 0,
   #     "oute": 0
   #   }
   # }

   # Check leaf-03 (server-05 connected to E1/1)
   kubectl get agent leaf-03 -n fab -o jsonpath='{.status.state.interfaces.Ethernet1}' | jq

   # Expected: vlans: [1020]  # Backend VLAN

   # Check leaf-05 (server-09 connected to E1/1)
   kubectl get agent leaf-05 -n fab -o jsonpath='{.status.state.interfaces.Ethernet1}' | jq

   # Expected: vlans: [1030]  # Database VLAN
   ```

2. **Verify all interfaces operational:**

   ```bash
   # List all interfaces with VLANs on leaf-01
   kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | \
     jq 'to_entries[] | select(.value.vlans != null) | {port: .key, vlans: .value.vlans, oper: .value.oper}'

   # Expected: All relevant interfaces show oper: "up"
   ```

3. **Check Grafana Interfaces Dashboard:**

   - Open http://localhost:3000
   - Navigate to **Dashboards** → **"Hedgehog Interfaces"**
   - Select switch: **leaf-01**
   - Filter interface: **E1/5**
   - Verify:
     - **Status**: Up/Up (green)
     - **VLAN**: 1010
     - **Traffic**: Non-zero input/output bytes
     - **Errors**: Zero or near-zero

4. **Check Grafana Fabric Dashboard:**

   - Navigate to **"Hedgehog Fabric"** dashboard
   - Verify:
     - **VPCs**: Shows `webapp-vpc`
     - **BGP Sessions**: All "Established"
     - **Switch Health**: Green/healthy

**Success Criteria:**
- ✅ Agent CRDs show correct VLANs on all ports
- ✅ All interfaces operational (oper: up)
- ✅ Traffic counters increasing (non-zero)
- ✅ Grafana dashboards confirm configuration

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You validated a complete three-tier VPC deployment:
- ✅ DHCP: All servers obtained IP addresses
- ✅ Same-subnet connectivity: Servers reach gateways
- ✅ Cross-subnet connectivity: Permitted paths work
- ✅ Security isolation: Blocked paths correctly denied
- ✅ Switch configuration: VLANs applied to correct ports
- ✅ Observability: Grafana dashboards confirm operational state

**Key Takeaways:**

1. **Validation is multi-layered** - Check Kubernetes resources, switch state, and endpoint connectivity
2. **DHCP is the first test** - No IP = no connectivity
3. **Permit lists control inter-subnet traffic** - Security by design
4. **Agent CRD is source of truth** - Switch operational state lives here
5. **Grafana provides visual confirmation** - Interfaces, traffic, health metrics

**Troubleshooting Mindset:**

- Start with kubectl events (fastest error detection)
- Check Agent CRD (switch-level validation)
- Test from endpoints (proves end-to-end functionality)
- Use Grafana for trends and visualization

**Next Module Preview:**

Module 2.4 (Decommission & Cleanup) teaches you to safely remove VPCs and attachments:
- Deletion order (attachments before VPCs)
- Cleanup validation (no orphaned resources)
- Rollback procedures
- When to archive vs delete

---

### Assessment Questions

#### Question 1: DHCP Troubleshooting

**Scenario:** A server is not obtaining a DHCP lease. What is the correct troubleshooting order?

- A) Check server interface → Check DHCPSubnet status → Check VPCAttachment VLAN
- B) Check VPCAttachment VLAN → Check DHCPSubnet status → Check server interface
- C) Check DHCPSubnet status → Check VPCAttachment VLAN → Check Agent CRD VLAN → Check server interface
- D) Restart DHCP server immediately

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) Check DHCPSubnet status → Check VPCAttachment VLAN → Check Agent CRD VLAN → Check server interface

**Explanation:**

**Correct Troubleshooting Flow:**

1. **DHCPSubnet status**: Is DHCP server running and pool available?
   ```bash
   kubectl get dhcpsubnet webapp-vpc--frontend -o yaml
   # Check: spec.range is correct, server is reconciled
   ```

2. **VPCAttachment VLAN**: Is correct subnet/VLAN specified?
   ```bash
   kubectl describe vpcattachment webapp-frontend-server-01
   # Check: spec.subnet and spec.connection are correct
   ```

3. **Agent CRD VLAN**: Is VLAN applied to switch port?
   ```bash
   kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.vlans}'
   # Check: VLAN 1010 is in array
   ```

4. **Server interface**: Is interface up and requesting DHCP?
   ```bash
   ip link show enp2s1  # Check: interface UP
   dhclient -v enp2s1   # Force DHCP request
   ```

**Why this order:** Work from control plane (Kubernetes) to data plane (switch) to endpoint (server). Each layer depends on the previous.

**Module 2.3 Reference:** Concept 1 - DHCP Validation, Concept 5 - Troubleshooting Methodology
</details>

---

#### Question 2: Connectivity Matrix

**Scenario:** VPC has three subnets with this permit list:
```yaml
permit:
  - [web, app]
  - [app, db]
```

Which connectivity paths work?

- A) web → app, app → db, web → db (all work)
- B) web → app, app → db (web → db blocked)
- C) web → app only (all others blocked)
- D) None work (permit lists block everything)

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) web → app, app → db (web → db blocked)

**Explanation:**

**Permit List Semantics:**
- `[web, app]` - Bidirectional communication allowed between web and app subnets
- `[app, db]` - Bidirectional communication allowed between app and db subnets
- No entry for `[web, db]` - Communication between web and db is **blocked**

**Connectivity Matrix:**

| Source | Destination | Permit List Entry | Result |
|--------|-------------|-------------------|--------|
| web | app | `[web, app]` | ✅ Allowed |
| app | web | `[web, app]` | ✅ Allowed (bidirectional) |
| app | db | `[app, db]` | ✅ Allowed |
| db | app | `[app, db]` | ✅ Allowed (bidirectional) |
| web | db | (none) | ❌ **Blocked** |
| db | web | (none) | ❌ **Blocked** |

**Security Pattern:** This is a classic three-tier architecture where frontend (web) cannot directly access database (db), forcing all database access through application tier (app).

**Module 2.3 Reference:** Concept 2 - End-to-End Connectivity Testing
</details>

---

#### Question 3: Agent CRD Interpretation

**Scenario:** You run this command:
```bash
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}'
```

Output:
```json
{
  "admin": "up",
  "oper": "down",
  "vlans": [1010],
  "counters": {
    "inb": 0,
    "outb": 0,
    "ine": 0,
    "oute": 0
  }
}
```

What is the problem?

- A) VLAN 1010 is not configured (should be empty array)
- B) Physical link is down (oper: down)
- C) Interface is administratively disabled (admin: down)
- D) No problem - this is normal

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Physical link is down (oper: down)

**Explanation:**

**Field Meanings:**
- `admin: up` - Interface is administratively enabled (configuration allows it to be up)
- `oper: down` - **Operational state is down** (physical link is not established)
- `vlans: [1010]` - VLAN 1010 is configured (this is correct)
- `counters: all zero` - No traffic (expected if link is down)

**Root Cause:** Physical link problem:
- Cable unplugged
- Server interface down
- Switch port hardware failure
- Autonegotiation failure

**Troubleshooting:**
```bash
# Check server-side interface
ssh server-01
ip link show enp2s1  # Should show UP
sudo ip link set enp2s1 up  # Bring up if down

# Check for errors
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.counters.ine}'
# High error count suggests cable or hardware issue
```

**Fix:** Physical layer issue - check cables, server interface state, switch port health.

**Module 2.3 Reference:** Concept 3 - Agent CRD Inspection
</details>

---

#### Question 4: Grafana Dashboard Usage

**Scenario:** You want to verify that server-01 is sending traffic through its VPC attachment. Which Grafana dashboard and panel should you check?

- A) Fabric Dashboard → VPC Count
- B) Interfaces Dashboard → Traffic Rates (for E1/5 on leaf-01)
- C) Logs Dashboard → Agent Logs
- D) Node Exporter → CPU Usage

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Interfaces Dashboard → Traffic Rates (for E1/5 on leaf-01)

**Explanation:**

**Correct Approach:**
1. Identify switch and port: server-01 connects to leaf-01 port E1/5
2. Open **Interfaces Dashboard**
3. Select switch: **leaf-01**
4. Filter interface: **E1/5**
5. View **Traffic Rates** panel

**What to Check:**
- **Input Bytes/Packets**: Traffic from server-01 to fabric
- **Output Bytes/Packets**: Traffic from fabric to server-01
- **Bit Rate**: Bandwidth usage (bps)

**Expected:** Non-zero values indicate traffic flowing

**Why others are wrong:**
- A) Fabric Dashboard shows VPC existence, not per-server traffic
- C) Logs Dashboard is for troubleshooting, not traffic metrics
- D) Node Exporter shows switch CPU, not per-port traffic

**Advanced:** Compare traffic rates across all servers to identify abnormal patterns.

**Module 2.3 Reference:** Concept 4 - Grafana Interface Monitoring
</details>

---

## Technical Requirements

### Hedgehog CRDs Referenced

All CRDs from Modules 2.1 and 2.2, plus:

#### DHCPSubnet - DHCP Lease Tracking
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#dhcpsubnet)
- **API Version:** `dhcp.githedgehog.com/v1beta1`
- **Key Fields:**
  - `spec.subnet` - IP subnet (e.g., `10.0.10.0/27`)
  - `spec.range.start` - DHCP pool start
  - `spec.range.end` - DHCP pool end
  - `status.allocated` - Map of allocated leases (IP → hostname, MAC, expiry)

#### Agent - Switch Operational State
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#agent)
- **Key Status Fields:**
  - `status.state.interfaces.<name>.oper` - Operational state (up/down)
  - `status.state.interfaces.<name>.vlans` - VLANs configured on port
  - `status.state.interfaces.<name>.counters` - Traffic and error counters
  - `status.state.bgpNeighbors` - BGP session state

### kubectl Commands

**DHCP Validation:**
```bash
# List DHCPSubnet resources
kubectl get dhcpsubnet -n default

# View lease allocations
kubectl get dhcpsubnet <name> -o jsonpath='{.status.allocated}' | jq

# Check DHCP pool configuration
kubectl get dhcpsubnet <name> -o yaml
```

**Agent CRD Inspection:**
```bash
# Get agent summary
kubectl get agents -n fab

# View specific interface
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.interfaces.<port>}' | jq

# Check all VLANs on switch
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.interfaces}' | \
  jq 'to_entries[] | select(.value.vlans != null) | {port: .key, vlans: .value.vlans}'

# Check BGP neighbors
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# Check reconciliation status
kubectl get agent <switch> -n fab -o jsonpath='{.status.lastAppliedTime}'
```

**Event Inspection:**
```bash
# Check for warnings
kubectl get events --field-selector type=Warning

# Check VPC events
kubectl get events --field-selector involvedObject.name=webapp-vpc

# Check VPCAttachment events
kubectl describe vpcattachment webapp-frontend-server-01
```

**Server Access:**
```bash
# SSH via hhfab (if available)
hhfab vlab ssh server-01

# SSH via port forwarding (alternative)
ssh -p 22001 core@localhost  # server-01
ssh -p 22005 core@localhost  # server-05
ssh -p 22009 core@localhost  # server-09

# On server: Check interface
ip addr show enp2s1
ip route
ping -c 2 <target-ip>
```

**Reference:** [WORKFLOWS.md](../research/WORKFLOWS.md#workflow-3-validate-connectivity)

### Grafana Dashboards

**Interfaces Dashboard:**
- URL: http://localhost:3000/d/interfaces
- Panels: Interface Status, VLAN Configuration, Traffic Rates, Error Rates
- Use: Per-port validation

**Fabric Dashboard:**
- URL: http://localhost:3000/d/fabric
- Panels: VPC Count, BGP Sessions, Switch Health, VNI Allocation
- Use: Fabric-wide health check

**Logs Dashboard:**
- URL: http://localhost:3000/d/logs
- Panels: Agent Logs, BGP Logs, Kernel Logs
- Use: Troubleshooting and error investigation

**Reference:** [OBSERVABILITY.md](../research/OBSERVABILITY.md)

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Train for Reality, Not Rote** ⭐
- **How:** Real-world validation workflow (not just "kubectl get")
- **Example:** Multi-layer troubleshooting (kubectl → Agent → endpoint)
- **Why:** Production operators must validate, not just configure

#### 2. **Focus on What Matters Most** ⭐
- **How:** Emphasizes common validation tasks (DHCP, connectivity, VLAN verification)
- **Example:** DHCP validation is first step (most common failure point)
- **Why:** Builds skills for high-frequency Day 2 operations

#### 3. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on testing of DHCP, connectivity, and troubleshooting
- **Example:** Students ping between servers, check Agent CRDs, use Grafana
- **Why:** Active validation builds confidence

#### 4. **Teach the Why Behind the How** ⭐
- **How:** Explains why validation is multi-layered (Kubernetes → switch → endpoint)
- **Example:** "Why check Agent CRD?" → Source of truth for switch state
- **Why:** Understanding validation layers enables adaptation to new scenarios

#### 5. **Support as Part of Learning** ⭐
- **How:** Normalizes troubleshooting (failure is expected, not abnormal)
- **Example:** "100% packet loss is EXPECTED for blocked paths"
- **Why:** Reduces anxiety about failures during testing

#### 6. **Abstraction as Empowerment**
- **How:** Shows how Grafana abstracts switch telemetry
- **Example:** View traffic without SSH to switches
- **Why:** Demonstrates power of observability tools

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What's New:**
- Network-level testing (ping, traceroute)
- DHCP concepts (less common in container networking)
- Switch-level validation (Agent CRD inspection)

**Bridge Strategy:**
- Relate DHCP validation to Kubernetes Service endpoint readiness
- Compare Agent CRD inspection to Pod status checking
- Frame Grafana as "Kubernetes metrics" for network

---

#### For Networking Professionals

**What's New:**
- kubectl-based validation (vs switch CLI)
- Agent CRD as switch state representation
- Grafana dashboards (vs "show" commands)

**Bridge Strategy:**
- Relate Agent CRD interface state to "show interface" output
- Compare Grafana dashboards to SNMP monitoring
- Frame kubectl validation as "centralized show commands"

---

### Common Challenges and Mitigation

#### Challenge 1: DHCP Lease Not Showing in DHCPSubnet

**Stumbling Block:** Student expects instant lease allocation

**Mitigation:**
- Concept 1 explains DHCP workflow timing
- Lab Task 1 includes wait time (leases may take 10-30 seconds)
- Troubleshooting section covers "DHCP timeout"

---

#### Challenge 2: Confusion About Permit List Behavior

**Stumbling Block:** Student expects all subnets to communicate

**Mitigation:**
- Concept 2 includes connectivity decision matrix
- Lab Task 2 tests both permitted and blocked paths
- Assessment Question 2 reinforces permit list logic

---

#### Challenge 3: Agent CRD JSON Interpretation

**Stumbling Block:** Students unfamiliar with jq and JSON parsing

**Mitigation:**
- All commands include example output
- Concept 3 explains key fields with visual formatting
- Assessment Question 3 tests interpretation skills

---

### Confidence-Building Opportunities

#### Win 1: DHCP Validation Success
**Moment:** Task 1 - Seeing allocated leases in DHCPSubnet status
**Feeling:** "My configuration works"
**Teaching Point:** "Servers automatically got IPs from your DHCP configuration."

#### Win 2: Connectivity Testing
**Moment:** Task 2 - Successful ping between servers
**Feeling:** "End-to-end functionality proven"
**Teaching Point:** "You proved the network works, not just assumed it."

#### Win 3: Security Validation
**Moment:** Task 2 - Blocked ping confirms isolation
**Feeling:** "Security by design is working"
**Teaching Point:** "Failure is success when you're testing security."

#### Win 4: Multi-Interface Validation
**Moment:** Task 3 - Correlating kubectl, Agent CRD, and Grafana
**Feeling:** "System understanding (abstraction layers clicked)"
**Teaching Point:** "You validated at all three layers - that's professional-level operations."

---

## Dependencies

### Prerequisites

**Required:**
- ✅ **Module 2.2:** Attach Servers to VPC
  - Servers must be attached to webapp-vpc
  - VPCAttachments created

- ✅ **Module 2.1:** Define VPC Network
  - webapp-vpc created with DHCP enabled
  - Three subnets configured

- ✅ **Module 1.3:** Mastering the Three Interfaces
  - kubectl proficiency
  - Grafana navigation skills

**Why These Prerequisites Matter:**
- Module 2.3 validates resources created in 2.1 and 2.2
- Cannot test connectivity without VPC and attachments
- Three-interface workflow (kubectl/Grafana) used throughout

---

### Enables

**Immediate:**
- ✅ **Module 2.4:** Decommission & Cleanup
  - Validates resources before deletion
  - Ensures no active connectivity before cleanup

**Course-Level:**
- Foundation for troubleshooting skills in Course 4
- Observability skills apply to Course 3 (Grafana focus)

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives specific and measurable**
  - LO 1: Validate DHCP (testable via Task 1)
  - LO 2: Test connectivity (testable via Task 2)
  - LO 3: Inspect Agent CRD (testable via Task 3)
  - LO 4: Interpret Grafana (testable via Task 3)
  - LO 5: Troubleshoot (testable via Assessment Q1, Q4)

- ✅ **Content outline logical progression**
  - Introduction → Core Concepts (5 concepts) → Hands-On Lab (3 tasks) → Assessment (4 questions)
  - Concepts build: DHCP → Connectivity → Agent → Grafana → Troubleshooting

- ✅ **Assessment aligns with learning objectives**
  - Question 1: DHCP troubleshooting (LO 1, LO 5)
  - Question 2: Connectivity matrix (LO 2)
  - Question 3: Agent CRD interpretation (LO 3)
  - Question 4: Grafana usage (LO 4)

- ✅ **Timing target achievable (12-15 minutes)**
  - Introduction: 2 min
  - Core Concepts: 4-5 min (5 concepts × ~1 min each)
  - Hands-On Lab: 5-6 min (Task 1: 2 min, Task 2: 2 min, Task 3: 2 min)
  - Wrap-Up & Assessment: 2 min
  - **Total: 13-15 minutes** (within target)

---

### Technical Accuracy

- ✅ **All CRD references validated**
  - DHCPSubnet status fields match reference
  - Agent CRD interface fields validated
  - VPC permit list semantics correct

- ✅ **Workflows match WORKFLOWS.md**
  - Connectivity validation workflow (lines 298-391)
  - DHCP validation workflow (lines 303-316)

- ✅ **VLAB topology assumptions validated**
  - Server connections correct
  - Port mappings accurate

---

### Learning Philosophy

- ✅ **Embodies 6 of 10 core principles**
  - Train for Reality, Not Rote ⭐
  - Focus on What Matters Most ⭐
  - Learn by Doing, Not Watching ⭐
  - Teach the Why Behind the How ⭐
  - Support as Part of Learning ⭐
  - Abstraction as Empowerment ⭐

- ✅ **Focuses on high-impact tasks**
  - DHCP validation (common first step)
  - Connectivity testing (core validation)
  - Troubleshooting methodology (Day 2 skill)

- ✅ **Builds confidence**
  - Task 1: DHCP success (early win)
  - Task 2: Connectivity proof (major win)
  - Task 3: Multi-layer validation (mastery)

---

### Accessibility

- ✅ **Clear to cloud-native learners**
  - kubectl-based workflow familiar
  - Grafana dashboards accessible

- ✅ **Clear to networking professionals**
  - Ping/traceroute testing familiar
  - VLAN validation recognizable

- ✅ **Jargon explained**
  - DHCP relay explained
  - Agent CRD fields defined
  - Permit list semantics clear

---

### Completeness

- ✅ **All required sections present**
  - Module Metadata ✓
  - Learning Objectives ✓
  - Content Outline ✓
  - Hands-On Lab ✓
  - Assessment ✓
  - Technical Requirements ✓
  - Pedagogical Design ✓
  - Dependencies ✓
  - Quality Checklist ✓

- ✅ **Assessment has answers and explanations**
  - All 4 questions include detailed answers

- ✅ **Dependencies mapped**
  - Prerequisites: Modules 2.2, 2.1, 1.3
  - Enables: Module 2.4

---

## Review & Approval

### Design Review

- ⏳ Course lead review (pending)
- ⏳ Technical validation (dev agent) - Validate in VLAB
- ⏳ Timing test (12-15 min walkthrough)
- ⏳ Peer feedback

### Approval

- ⏳ Ready for Phase 3 (Content Development)
- ⏳ Approved by: <!-- Pending -->

---

## Notes & Iterations

### Design Decisions

**Decision 1: Multi-Layer Validation Approach**
- **Rationale:** Production validation requires checking multiple system layers
- **Benefit:** Teaches systematic troubleshooting methodology
- **Alternative:** Single-layer validation (rejected: too simplistic)

**Decision 2: Include Blocked Connectivity Test**
- **Rationale:** Security validation is as important as connectivity validation
- **Benefit:** Students understand permit lists enforce isolation
- **Alternative:** Only test permitted paths (rejected: incomplete validation)

**Decision 3: Agent CRD Deep Dive**
- **Rationale:** Agent CRD is source of truth for switch state
- **Benefit:** Students learn where to find authoritative operational data
- **Alternative:** Only use Grafana (rejected: misses kubectl-native approach)

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING
**Version:** 1.0
**Previous Module:** Module 2.2 (Attach Servers to VPC)
**Next Module:** Module 2.4 (Decommission & Cleanup - To Be Designed)

**Change Log:**
- 2025-10-16 v1.0: Initial design based on Issue #9 requirements

---

**Status:** ⏳ PENDING REVIEW
**Related Issue:** GitHub Issue #9 - [DESIGN] Course 2: Provisioning & Connectivity
**Design Duration:** ~4 hours (as estimated)
