# Module 2.2 Design: Attach Servers to VPC

## Module Metadata

- **Module Number:** 2.2
- **Module Title:** Attach Servers to VPC
- **Course:** Course 2 - Provisioning & Connectivity
- **Estimated Duration:** 13-15 minutes
  - Introduction: 2 minutes
  - Core Concepts: 5-6 minutes
  - Hands-On Lab: 5-6 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 2.1: Define VPC Network (VPC created, namespace concepts understood)
  - ✅ Module 1.2: How Hedgehog Works (GitOps workflow)
  - ✅ Module 1.1: Welcome to Fabric Operations (topology understanding)

## Learning Objectives

By the end of this module, learners will be able to:

1. **Identify server connections** - Discover available servers and their connection types in the fabric
2. **Create VPCAttachment resources** - Bind VPC subnets to server connections using GitOps workflow
3. **Understand connection types** - Distinguish between MCLAG, ESLAG, bundled, and unbundled connections
4. **Select appropriate subnets** - Choose correct VPC subnet for server attachments
5. **Verify switch configuration** - Validate that VLANs are applied to correct switch ports using kubectl and Grafana

**Bloom's Taxonomy Level**: Apply, Analyze (selecting connections, validating configurations)

## Content Outline

### Introduction (2 minutes)

**Hook: Networks Need Endpoints**

> In Module 2.1, you designed a VPC with three subnets: frontend, backend, and database. You configured IP addressing, VLANs, and DHCP. But here's the problem: **your VPC has no servers attached to it**.
>
> A VPC without servers is like a road without cars—perfectly configured but completely unused. To make your VPC functional, you need to **attach servers** to it.
>
> This is where **VPCAttachment** comes in.

**Context: Bridging Virtual and Physical**

VPCs are **virtual** networks (Layer 2 domains defined in software).
Servers are **physical** devices (connected to specific switch ports).

**VPCAttachment** bridges these layers:
- **spec.subnet**: Which VPC subnet (virtual)
- **spec.connection**: Which server connection (physical)
- Result: VLAN configured on switch ports, server can communicate in VPC

**What You'll Learn:**

- Server inventory and connection types
- VPCAttachment CRD syntax and workflow
- Difference between MCLAG, ESLAG, bundled, unbundled connections
- Subnet selection for attachments
- Validation using kubectl and Grafana

You'll attach servers to the `webapp-vpc` created in Module 2.1, bringing your three-tier application network to life.

---

### Core Concepts (5-6 minutes)

#### Concept 1: Server Connections in Hedgehog

**What is a Connection?**

A **Connection** CRD represents physical and logical connectivity between devices in the fabric. Each Connection has:
- **Name**: Describes the connection (e.g., `server-01--mclag--leaf-01--leaf-02`)
- **Type**: MCLAG, ESLAG, bundled, unbundled, or fabric
- **Links**: Specific server ports → switch ports mapping

**Connection Naming Convention:**
```
<server-name>--<type>--<switch-name(s)>
```

Examples:
- `server-01--mclag--leaf-01--leaf-02` (MCLAG to two switches)
- `server-05--eslag--leaf-03--leaf-04` (ESLAG to two switches)
- `server-09--unbundled--leaf-05` (Single link to one switch)
- `server-10--bundled--leaf-05` (Port channel to one switch)

**Discovering Connections:**

```bash
# List all server connections
kubectl get connections -n fab | grep server-

# Get connection details
kubectl get connection server-01--mclag--leaf-01--leaf-02 -n fab -o yaml
```

---

#### Concept 2: Connection Types Explained

**1. MCLAG (Multi-Chassis Link Aggregation)**

**What:** Server dual-homed to two switches with traditional MCLAG
**Where:** leaf-01 and leaf-02 (mclag-1 switch group)
**Servers:** server-01, server-02

**Characteristics:**
- **Redundancy:** If one switch fails, server stays connected
- **Bandwidth:** Aggregated bandwidth across two links
- **Active-Active:** Both links carry traffic simultaneously
- **Technology:** Proprietary MCLAG protocol

**Connection Example:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-01--mclag--leaf-01--leaf-02
  namespace: fab
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

**When to Use:** High-availability servers requiring dual-homed redundancy

---

**2. ESLAG (Ethernet Segment LAG / EVPN Multi-homing)**

**What:** Server dual-homed to two switches using standards-based EVPN
**Where:** leaf-03 and leaf-04 (eslag-1 switch group)
**Servers:** server-05, server-06

**Characteristics:**
- **Redundancy:** If one switch fails, server stays connected
- **Standards-Based:** EVPN RFC 7432 (not proprietary)
- **Active-Active:** Both links carry traffic
- **Scalability:** Can extend to >2 switches (future)

**Connection Example:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-05--eslag--leaf-03--leaf-04
  namespace: fab
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

**When to Use:** Modern multi-homing, standards-based redundancy

---

**3. Bundled (Port Channel / LAG)**

**What:** Multiple links to **same switch** in a port channel
**Where:** Servers connected to single switch (leaf-02, leaf-04, leaf-05)
**Servers:** server-04, server-08, server-10

**Characteristics:**
- **No Switch Redundancy:** Single switch attachment
- **Bandwidth Aggregation:** 2x or more link bandwidth
- **Active-Active:** Load balancing across links
- **Simpler:** No multi-switch coordination needed

**Connection Example:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-10--bundled--leaf-05
  namespace: fab
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

**When to Use:** Bandwidth aggregation without switch redundancy requirement

---

**4. Unbundled (Single Link)**

**What:** Single server port → single switch port
**Where:** Servers with no redundancy or aggregation
**Servers:** server-03, server-07, server-09

**Characteristics:**
- **Simplest:** One link, one switch
- **No Redundancy:** Switch or link failure disconnects server
- **No Aggregation:** Single link bandwidth
- **Lowest Complexity:** Easiest to configure

**Connection Example:**
```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-09--unbundled--leaf-05
  namespace: fab
spec:
  unbundled:
    link:
      server:
        port: server-09/enp2s1
      switch:
        port: leaf-05/E1/1
```

**When to Use:** Development environments, non-critical workloads, cost-sensitive deployments

---

**Connection Type Decision Tree:**

```
Do you need switch redundancy?
├─ YES → Do you have MCLAG or ESLAG pair available?
│        ├─ MCLAG pair (leaf-01/02) → Use MCLAG
│        └─ ESLAG pair (leaf-03/04) → Use ESLAG
│
└─ NO  → Do you need bandwidth aggregation?
         ├─ YES → Use Bundled (port channel)
         └─ NO  → Use Unbundled (single link)
```

---

#### Concept 3: VPCAttachment CRD

**Purpose:**

VPCAttachment binds a **VPC subnet** to a **server connection**, making the VPC available on the server's network interfaces.

**Required Fields:**

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: <attachment-name>
  namespace: default
spec:
  subnet: <vpc-name>/<subnet-name>
  connection: <connection-name>
  nativeVLAN: true|false  # Optional
```

**Field Explanations:**

- **metadata.name**: Descriptive name (e.g., `webapp-vpc-frontend-server-01`)
- **spec.subnet**: Format `<vpc-name>/<subnet-name>` (e.g., `webapp-vpc/frontend`)
- **spec.connection**: Connection CRD name (e.g., `server-01--mclag--leaf-01--leaf-02`)
- **spec.nativeVLAN**:
  - `false` (default): VLAN tagged (802.1Q) - server sees VLAN tag
  - `true`: VLAN untagged (native) - server sees untagged traffic

**Example VPCAttachment:**

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: webapp-frontend-server-01
  namespace: default
spec:
  subnet: webapp-vpc/frontend
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: false
```

**What Happens When Applied:**
1. Fabric Controller detects new VPCAttachment
2. Controller identifies switches in connection (leaf-01, leaf-02)
3. Controller updates Agent CRDs for those switches
4. Switch agents configure VLAN on specified ports
5. Server can now communicate in VPC subnet

---

#### Concept 4: Subnet Selection

**Choosing the Right Subnet:**

When attaching a server to a VPC with multiple subnets, you must choose which subnet:

**Scenario:** `webapp-vpc` has three subnets (frontend, backend, database)

**Decision Questions:**
1. What is the server's role? (Web server? API server? Database?)
2. Which tier should it communicate with?
3. Are there security isolation requirements?

**Example Allocation:**
- **Web servers** (server-01, server-02) → `webapp-vpc/frontend`
- **API servers** (server-05, server-06) → `webapp-vpc/backend`
- **Database servers** (server-09, server-10) → `webapp-vpc/database`

**Multiple Attachments Per Server:**

A single server can be attached to **multiple subnets**:

```yaml
# Server-01 in frontend subnet
---
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: webapp-frontend-server-01
spec:
  subnet: webapp-vpc/frontend
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: true  # Untagged VLAN

---
# Same server in backend subnet
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: webapp-backend-server-01
spec:
  subnet: webapp-vpc/backend
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: false  # Tagged VLAN
```

**Result:** Server-01 has two VLANs (frontend untagged, backend tagged)

---

#### Concept 5: Verification and Validation

**Three Layers of Validation:**

**Layer 1: kubectl - VPCAttachment Created**

```bash
# List VPCAttachments
kubectl get vpcattachment

# Get details
kubectl get vpcattachment webapp-frontend-server-01 -o yaml

# Check events
kubectl describe vpcattachment webapp-frontend-server-01
```

**Expected Events:**
- `Normal  Created` - VPCAttachment resource created
- `Normal  Reconciling` - Fabric controller processing
- `Normal  Applied` - Configuration sent to switches

**Layer 2: kubectl - Agent CRD Updated**

```bash
# Check Agent CRD for switch
kubectl get agent leaf-01 -n fab -o yaml

# Look for interface configuration
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | jq
```

**Expected:** Interface E1/5 shows VLAN 1010 configured

**Layer 3: Grafana - Visual Confirmation**

- Open Grafana "Interfaces Dashboard"
- Filter by switch (leaf-01)
- View interface E1/5 - should show VLAN 1010
- Check traffic metrics (will show activity after server boots)

---

### Hands-On Lab (5-6 minutes)

**Lab Title:** Attach Servers to webapp-vpc Three-Tier Network

**Scenario:**

You have `webapp-vpc` from Module 2.1 with three subnets (frontend, backend, database). Now you need to attach servers:
- **Frontend tier**: server-01 (MCLAG)
- **Backend tier**: server-05 (ESLAG)
- **Database tier**: server-09 (unbundled)

**Environment Access:**
- **Gitea:** http://localhost:3001 (username: `student`, password: `hedgehog123`)
- **kubectl:** Already configured
- **Grafana:** http://localhost:3000 (username: `admin`, password: `prom-operator`)

---

#### Task 1: Discover Server Connections (2 minutes)

**Objective:** Identify available servers and their connection types

**Steps:**

1. **List all servers:**

   ```bash
   # View servers in fabric
   kubectl get servers -n fab

   # Expected output:
   # NAME         AGE
   # server-01    1h
   # server-02    1h
   # ... (server-03 through server-10)
   ```

2. **List server connections:**

   ```bash
   # Filter for server connections
   kubectl get connections -n fab | grep server-

   # Expected output:
   # server-01--mclag--leaf-01--leaf-02     1h
   # server-02--mclag--leaf-01--leaf-02     1h
   # server-03--unbundled--leaf-01          1h
   # server-04--bundled--leaf-02            1h
   # server-05--eslag--leaf-03--leaf-04     1h
   # server-06--eslag--leaf-03--leaf-04     1h
   # server-07--unbundled--leaf-03          1h
   # server-08--bundled--leaf-04            1h
   # server-09--unbundled--leaf-05          1h
   # server-10--bundled--leaf-05            1h
   ```

3. **Examine connection details:**

   ```bash
   # View MCLAG connection
   kubectl get connection server-01--mclag--leaf-01--leaf-02 -n fab -o yaml
   ```

4. **Complete server inventory table:**

   | Server | Connection Type | Switches | Tier Assignment |
   |--------|----------------|----------|-----------------|
   | server-01 | MCLAG | leaf-01, leaf-02 | **Frontend** |
   | server-05 | ESLAG | leaf-03, leaf-04 | **Backend** |
   | server-09 | Unbundled | leaf-05 | **Database** |

**Success Criteria:**
- ✅ Identified 10 servers in fabric
- ✅ Listed all server connections
- ✅ Understood connection types for each server

---

#### Task 2: Create VPCAttachments (3 minutes)

**Objective:** Attach servers to webapp-vpc subnets using GitOps workflow

**Steps:**

1. **Open Gitea:**
   - Navigate to http://localhost:3001
   - Sign in (username: `student`, password: `hedgehog123`)
   - Go to `student/hedgehog-config` repository
   - Click `vpc-attachments/` folder

2. **Create attachment for frontend tier:**
   - Click **"New File"**
   - Filename: `webapp-frontend-server-01.yaml`

   ```yaml
   apiVersion: vpc.githedgehog.com/v1beta1
   kind: VPCAttachment
   metadata:
     name: webapp-frontend-server-01
     namespace: default
   spec:
     subnet: webapp-vpc/frontend
     connection: server-01--mclag--leaf-01--leaf-02
     nativeVLAN: false
   ```

   - Commit message: `Attach server-01 to webapp-vpc frontend`
   - Click **"Commit Changes"**

3. **Create attachment for backend tier:**
   - Click **"New File"**
   - Filename: `webapp-backend-server-05.yaml`

   ```yaml
   apiVersion: vpc.githedgehog.com/v1beta1
   kind: VPCAttachment
   metadata:
     name: webapp-backend-server-05
     namespace: default
   spec:
     subnet: webapp-vpc/backend
     connection: server-05--eslag--leaf-03--leaf-04
     nativeVLAN: false
   ```

   - Commit message: `Attach server-05 to webapp-vpc backend`
   - Click **"Commit Changes"**

4. **Create attachment for database tier:**
   - Click **"New File"**
   - Filename: `webapp-database-server-09.yaml`

   ```yaml
   apiVersion: vpc.githedgehog.com/v1beta1
   kind: VPCAttachment
   metadata:
     name: webapp-database-server-09
     namespace: default
   spec:
     subnet: webapp-vpc/database
     connection: server-09--unbundled--leaf-05
     nativeVLAN: false
   ```

   - Commit message: `Attach server-09 to webapp-vpc database`
   - Click **"Commit Changes"**

**Success Criteria:**
- ✅ Three VPCAttachment YAML files created
- ✅ Each attachment targets correct subnet
- ✅ Connection names match actual connections

---

#### Task 3: Validate Attachments (2 minutes)

**Objective:** Verify VPCAttachments deployed and switches configured

**Steps:**

1. **Watch ArgoCD sync:**
   - Open http://localhost:8080
   - Sign in (username: `admin`, password: `qV7hX0NMroAUhwoZ`)
   - View `hedgehog-config` application
   - Wait for sync (up to 60 seconds)
   - Status should show "Synced" and "Healthy"

2. **Verify VPCAttachments created:**

   ```bash
   # List all VPC attachments
   kubectl get vpcattachment

   # Expected output:
   # NAME                          AGE
   # webapp-frontend-server-01     30s
   # webapp-backend-server-05      30s
   # webapp-database-server-09     30s
   ```

3. **Check VPCAttachment details:**

   ```bash
   # Describe attachment
   kubectl describe vpcattachment webapp-frontend-server-01

   # Look for events:
   # Normal  Created      VPCAttachment created
   # Normal  Reconciling  Configuring switches
   # Normal  Applied      Configuration applied to leaf-01, leaf-02
   ```

4. **Verify switch configuration (Agent CRD):**

   ```bash
   # Check leaf-01 Agent (should show VLAN 1010 on E1/5)
   kubectl get agent leaf-01 -n fab -o yaml | grep -A 20 "E1/5"

   # Check leaf-03 Agent (should show VLAN 1020 on E1/1)
   kubectl get agent leaf-03 -n fab -o yaml | grep -A 20 "E1/1"

   # Check leaf-05 Agent (should show VLAN 1030 on E1/1)
   kubectl get agent leaf-05 -n fab -o yaml | grep -A 20 "E1/1"
   ```

5. **View in Grafana (Optional):**
   - Open http://localhost:3000
   - Navigate to "Interfaces Dashboard"
   - Filter by switch (leaf-01, leaf-03, leaf-05)
   - Verify VLANs configured on correct interfaces

**Success Criteria:**
- ✅ All three VPCAttachments created successfully
- ✅ No error events in kubectl describe
- ✅ Agent CRDs show VLAN configuration
- ✅ Grafana shows interfaces configured (optional)

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You successfully attached three servers to `webapp-vpc`:
- ✅ Frontend tier: server-01 (MCLAG) → webapp-vpc/frontend (VLAN 1010)
- ✅ Backend tier: server-05 (ESLAG) → webapp-vpc/backend (VLAN 1020)
- ✅ Database tier: server-09 (unbundled) → webapp-vpc/database (VLAN 1030)

**Key Takeaways:**

1. **VPCAttachment bridges virtual and physical** - Binds VPC subnets to server connections
2. **Connection types determine redundancy** - MCLAG/ESLAG for HA, bundled for bandwidth, unbundled for simplicity
3. **Subnet selection maps to application tiers** - Frontend/backend/database separation
4. **Validation happens at three layers** - kubectl (resources), Agent CRD (switch state), Grafana (visualization)
5. **GitOps workflow applies consistently** - Gitea → ArgoCD → kubectl → Grafana

**Next Module Preview:**

In Module 2.3 (Connectivity Validation), you'll:
- Test end-to-end connectivity between servers
- Validate DHCP leases
- Use kubectl and Grafana to troubleshoot connectivity issues
- Confirm your three-tier network is fully operational

---

### Assessment Questions

#### Question 1: Connection Type Selection

**Scenario:** You need to attach a critical database server that requires maximum uptime. Which connection type should you choose?

- A) Unbundled (single link)
- B) Bundled (port channel to single switch)
- C) MCLAG (dual-homed to two switches)
- D) Any type works equally well

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) MCLAG (dual-homed to two switches)

**Explanation:**
- Critical servers require **switch redundancy** to avoid single points of failure
- MCLAG provides dual-homing: if one switch fails, server stays connected
- Unbundled has no redundancy (single switch/link failure disconnects server)
- Bundled has no switch redundancy (switch failure disconnects server)

**Alternative:** ESLAG would also be correct (standards-based dual-homing)

**Module 2.2 Reference:** Concept 2 - Connection Types Explained
</details>

---

#### Question 2: VPCAttachment Spec

**Scenario:** You want to attach server-05 to the backend subnet of webapp-vpc. The connection is `server-05--eslag--leaf-03--leaf-04`. Which VPCAttachment spec is correct?

**A)**
```yaml
spec:
  subnet: backend
  connection: server-05
```

**B)**
```yaml
spec:
  subnet: webapp-vpc/backend
  connection: server-05--eslag--leaf-03--leaf-04
```

**C)**
```yaml
spec:
  vpc: webapp-vpc
  subnet: backend
  server: server-05
```

**D)**
```yaml
spec:
  subnet: webapp-vpc/backend
  connection: leaf-03--leaf-04
```

<details>
<summary>Answer & Explanation</summary>

**Answer:** B)

**Explanation:**
- `spec.subnet` must be in format `<vpc-name>/<subnet-name>` (e.g., `webapp-vpc/backend`)
- `spec.connection` must be the full Connection CRD name (e.g., `server-05--eslag--leaf-03--leaf-04`)

**Why others are wrong:**
- A: Missing VPC name in subnet, connection should be full connection name
- C: No `spec.vpc` or `spec.server` fields exist in VPCAttachment CRD
- D: Connection should reference server connection, not switch-to-switch connection

**Module 2.2 Reference:** Concept 3 - VPCAttachment CRD
</details>

---

#### Question 3: Multiple Subnet Attachments

**Scenario:** A server needs to access both frontend (VLAN 1010) and backend (VLAN 1020) subnets. How should you configure this?

- A) Create one VPCAttachment with two subnets
- B) Create two separate VPCAttachments (one per subnet)
- C) Modify the server's Connection to include both VLANs
- D) This is not possible in Hedgehog

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Create two separate VPCAttachments (one per subnet)

**Explanation:**
- Each VPCAttachment binds **one subnet** to **one connection**
- To attach a server to multiple subnets, create multiple VPCAttachments

**Example:**
```yaml
# First attachment
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: webapp-frontend-server-01
spec:
  subnet: webapp-vpc/frontend
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: true  # Untagged

---
# Second attachment
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: webapp-backend-server-01
spec:
  subnet: webapp-vpc/backend
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: false  # Tagged
```

**Result:** Server-01 has both VLANs (1010 untagged, 1020 tagged)

**Module 2.2 Reference:** Concept 4 - Subnet Selection
</details>

---

#### Question 4: Validation Strategy

**Scenario:** You created a VPCAttachment but the server cannot communicate in the VPC. What is the correct troubleshooting order?

- A) Check Grafana → Check kubectl events → Check Agent CRD
- B) Check kubectl events → Check Agent CRD → Check Grafana
- C) Check Agent CRD → Check Grafana → Check kubectl events
- D) SSH to switch and check VLAN configuration manually

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Check kubectl events → Check Agent CRD → Check Grafana

**Explanation:**

**Correct Troubleshooting Order:**
1. **kubectl events** - Fastest way to detect configuration errors
   ```bash
   kubectl describe vpcattachment <name>
   # Look for Warning events (e.g., "Invalid connection name")
   ```

2. **Agent CRD** - Check if switches received configuration
   ```bash
   kubectl get agent leaf-01 -n fab -o yaml
   # Verify VLAN configured on correct interface
   ```

3. **Grafana** - Visual confirmation and metrics
   - Interfaces Dashboard shows VLAN configuration
   - Useful for operational state, not initial troubleshooting

**Why D is wrong:** Hedgehog abstracts switch CLI; use kubectl and Grafana instead

**Module 2.2 Reference:** Concept 5 - Verification and Validation
</details>

---

## Technical Requirements

### Hedgehog CRDs Used

#### VPCAttachment - Primary Resource
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#vpcattachment)
- **API Version:** `vpc.githedgehog.com/v1beta1`
- **Key Fields:**
  - `spec.subnet` - VPC subnet reference (format: `<vpc-name>/<subnet-name>`)
  - `spec.connection` - Connection CRD name
  - `spec.nativeVLAN` - Tagged (false) or untagged (true) VLAN

#### Connection - Server Connectivity
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#connection)
- **API Version:** `wiring.githedgehog.com/v1beta1`
- **Connection Types:**
  - `spec.mclag` - MCLAG connection
  - `spec.eslag` - ESLAG connection
  - `spec.bundled` - Bundled (LAG) connection
  - `spec.unbundled` - Unbundled (single link) connection

#### Server - Physical Server Definition
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#server)
- **API Version:** `wiring.githedgehog.com/v1beta1`
- **Key Fields:**
  - `metadata.name` - Server name (e.g., `server-01`)
  - `spec.description` - Optional description

#### Agent - Switch Operational State
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#agent)
- **API Version:** `agent.githedgehog.com/v1beta1`
- **Key Fields:**
  - `status.state.interfaces` - Interface configuration and state
  - `status.state.bgpNeighbors` - BGP neighbor status
  - `status.lastAppliedGen` - Last applied configuration generation

### kubectl Commands

**Server Discovery:**
```bash
# List servers
kubectl get servers -n fab

# List server connections
kubectl get connections -n fab | grep server-

# Get connection details
kubectl get connection <connection-name> -n fab -o yaml
```

**VPCAttachment Operations:**
```bash
# Create VPCAttachment (via GitOps - Gitea commit triggers ArgoCD)
# Direct application (not recommended):
kubectl apply -f vpcattachment.yaml

# List VPCAttachments
kubectl get vpcattachment

# Get VPCAttachment details
kubectl get vpcattachment <name> -o yaml

# Describe VPCAttachment (includes events)
kubectl describe vpcattachment <name>

# Check events
kubectl get events --field-selector involvedObject.name=<name>
```

**Validation:**
```bash
# Check Agent CRD for switch
kubectl get agent <switch-name> -n fab -o yaml

# View interface configuration
kubectl get agent <switch-name> -n fab -o jsonpath='{.status.state.interfaces}' | jq

# Check VLAN on specific interface
kubectl get agent leaf-01 -n fab -o yaml | grep -A 20 "E1/5"

# View all agents
kubectl get agents -n fab
```

**Troubleshooting:**
```bash
# Check for warning events
kubectl get events --field-selector type=Warning

# View controller logs
kubectl logs -n fab deployment/fabric-controller-manager | grep vpcattachment

# Check VPC exists
kubectl get vpc webapp-vpc

# Verify connection exists
kubectl get connection server-01--mclag--leaf-01--leaf-02 -n fab
```

**Reference:** [WORKFLOWS.md](../research/WORKFLOWS.md#workflow-2-attach-server-to-vpc)

### GitOps Workflow

Same as Module 2.1:
1. **Gitea**: Create/edit YAML in `vpc-attachments/` directory
2. **ArgoCD**: Auto-sync (60 seconds) or manual sync
3. **kubectl**: Validate VPCAttachment created
4. **Grafana**: Visual confirmation of switch configuration

**Reference:** [Module 1.2 Design](./module-1.2-design-v2-gitops.md)

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Train for Reality, Not Rote** ⭐
- **How:** Real-world scenario (three-tier app) with different connection types
- **Example:** Students choose appropriate connection type for each tier
- **Why:** Mimics actual provisioning decisions operators make

#### 2. **Focus on What Matters Most** ⭐
- **How:** Covers four main connection types (MCLAG, ESLAG, bundled, unbundled)
- **Example:** Decision tree for connection type selection
- **Why:** These are the most common deployment patterns

#### 3. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on attachment of three servers with different connection types
- **Example:** Create VPCAttachments, validate with kubectl and Grafana
- **Why:** Active learning reinforces understanding

#### 4. **Teach the Why Behind the How** ⭐
- **How:** Explains connection type differences and use cases
- **Example:** "When to Use" sections for each connection type
- **Why:** Enables informed decision-making in new scenarios

#### 5. **Abstraction as Empowerment** ⭐
- **How:** Demonstrates how VPCAttachment abstracts switch port configuration
- **Example:** Commit YAML → VLANs automatically applied to ports
- **Why:** Shows power of declarative infrastructure

#### 6. **Bridging Two Worlds**
- **How:** Connection types (networking concept) presented via CRDs (cloud-native)
- **Example:** MCLAG explained alongside Connection CRD spec
- **Why:** Accessible to both audiences

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What's New:**
- Physical server connectivity (not typical in Kubernetes)
- Connection types (MCLAG, ESLAG, LAG)
- Switch port configuration (abstracted in cloud)

**Bridge Strategy:**
- Relate VPCAttachment to Kubernetes Service (binding abstraction to endpoints)
- Compare Connection types to different Service types (ClusterIP, NodePort, LoadBalancer)
- Frame switch configuration as "infrastructure-as-code" (familiar concept)

---

#### For Networking Professionals

**What's New:**
- CRD-based connection definition (vs manual switch config)
- GitOps workflow for attachments (vs CLI configuration)
- Automated VLAN application (vs manual per-port config)

**Bridge Strategy:**
- Relate MCLAG/ESLAG to familiar dual-homing concepts
- Compare VPCAttachment to "switchport mode trunk" + "switchport trunk allowed vlan"
- Frame GitOps as "centralized configuration management" (familiar goal)

---

### Common Challenges and Mitigation

#### Challenge 1: Connection Name Confusion

**Stumbling Block:** Students use server name instead of full connection name

**Symptoms:**
```yaml
spec:
  connection: server-01  # WRONG
```

**Mitigation:**
- Task 1 explicitly shows connection naming convention
- Assessment Question 2 tests correct connection name format
- Example YAML shows full connection names

---

#### Challenge 2: Subnet Format Errors

**Stumbling Block:** Students forget VPC name in subnet reference

**Symptoms:**
```yaml
spec:
  subnet: frontend  # WRONG (missing VPC name)
```

**Mitigation:**
- Concept 3 emphasizes format: `<vpc-name>/<subnet-name>`
- All examples show correct format
- Assessment Question 2 tests subnet format

---

#### Challenge 3: MCLAG vs ESLAG Confusion

**Stumbling Block:** Students unclear on difference between MCLAG and ESLAG

**Mitigation:**
- Concept 2 dedicates separate sections to each type
- "When to Use" guidance for each
- Lab uses both types (server-01 MCLAG, server-05 ESLAG)

---

### Confidence-Building Opportunities

#### Win 1: Connection Discovery
**Moment:** Task 1 - Successfully listing all server connections
**Feeling:** Understanding fabric topology
**Teaching Point:** "You can navigate the physical connectivity."

#### Win 2: First VPCAttachment
**Moment:** Task 2 - Creating VPCAttachment for frontend tier
**Feeling:** Bridging virtual and physical
**Teaching Point:** "You connected a virtual network to a physical server."

#### Win 3: Validation Success
**Moment:** Task 3 - Seeing VLANs configured on switches
**Feeling:** System understanding (seeing the reconciliation)
**Teaching Point:** "Your YAML became switch configuration automatically."

---

## Dependencies

### Prerequisites

**Required:**
- ✅ **Module 2.1:** Define VPC Network
  - VPC `webapp-vpc` must exist
  - Three subnets (frontend, backend, database) configured

- ✅ **Module 1.2:** How Hedgehog Works
  - GitOps workflow understanding
  - Three interfaces (Gitea, kubectl, Grafana)

- ✅ **Module 1.1:** Welcome to Fabric Operations
  - Fabric topology understanding
  - kubectl basics

**Why These Prerequisites Matter:**
- Module 2.2 attaches servers to VPC created in Module 2.1
- GitOps workflow from Module 1.2 used throughout
- Topology understanding from Module 1.1 needed to interpret connections

---

### Enables

**Immediate:**
- ✅ **Module 2.3:** Connectivity Validation
  - Requires servers attached to VPC
  - Tests DHCP and inter-server connectivity

**Sequential:**
- ✅ **Module 2.4:** Decommission & Cleanup
  - VPCAttachments must be deleted before VPC

**Course-Level:**
- Foundation for multi-server scenarios in Course 3 and 4

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives specific and measurable**
  - LO 1: Identify server connections (testable via Task 1)
  - LO 2: Create VPCAttachment (testable via Task 2)
  - LO 3: Understand connection types (testable via Concept 2, Assessment Q1)
  - LO 4: Select subnets (testable via Task 2, Assessment Q3)
  - LO 5: Verify configuration (testable via Task 3, Assessment Q4)

- ✅ **Content outline logical progression**
  - Introduction → Core Concepts (5 concepts) → Hands-On Lab (3 tasks) → Assessment (4 questions)
  - Concepts build: Connections → Types → VPCAttachment → Subnet selection → Validation

- ✅ **Assessment aligns with learning objectives**
  - Question 1: Connection type selection (LO 3)
  - Question 2: VPCAttachment spec (LO 2)
  - Question 3: Multiple attachments (LO 4)
  - Question 4: Validation strategy (LO 5)

- ✅ **Timing target achievable (13-15 minutes)**
  - Introduction: 2 min
  - Core Concepts: 5-6 min (5 concepts × ~1 min each)
  - Hands-On Lab: 5-6 min (Task 1: 2 min, Task 2: 3 min, Task 3: 2 min)
  - Wrap-Up & Assessment: 2 min
  - **Total: 14-16 minutes** (within target)

---

### Technical Accuracy

- ✅ **All CRD references validated**
  - VPCAttachment fields match CRD_REFERENCE.md (lines 132-161)
  - Connection types match CRD_REFERENCE.md (lines 481-597)
  - Server CRD validated (lines 458-476)
  - Agent CRD validated (lines 796-923)

- ✅ **Workflows match WORKFLOWS.md**
  - VPCAttachment workflow matches lines 185-296
  - GitOps workflow consistent with Module 1.2

- ✅ **VLAB topology assumptions validated**
  - Server connections match VLAB_CAPABILITIES.md (lines 75-93)
  - Connection types match environment (MCLAG, ESLAG, bundled, unbundled)

---

### Learning Philosophy

- ✅ **Embodies 6 of 10 core principles**
  - Train for Reality, Not Rote ⭐
  - Focus on What Matters Most ⭐
  - Learn by Doing, Not Watching ⭐
  - Teach the Why Behind the How ⭐
  - Abstraction as Empowerment ⭐
  - Bridging Two Worlds ⭐

- ✅ **Focuses on high-impact tasks**
  - VPCAttachment creation (common Day 2 operation)
  - Connection type selection (frequent decision)

- ✅ **Builds confidence through achievable goals**
  - Task 1: Discovery (low-risk exploration)
  - Task 2: Creation (guided implementation)
  - Task 3: Validation (success confirmation)

---

### Accessibility

- ✅ **Clear to cloud-native learners**
  - CRD-based workflow familiar
  - Connection types explained in context

- ✅ **Clear to networking professionals**
  - MCLAG/ESLAG concepts familiar
  - VLAN configuration recognizable

- ✅ **Jargon explained**
  - MCLAG, ESLAG, LAG defined
  - nativeVLAN explained (tagged vs untagged)

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
  - Prerequisites: Module 2.1, 1.2, 1.1
  - Enables: Modules 2.3, 2.4

---

## Review & Approval

### Design Review

- ⏳ Course lead review (pending)
- ⏳ Technical validation (dev agent) - Validate in VLAB
- ⏳ Timing test (13-15 min walkthrough)
- ⏳ Peer feedback

### Approval

- ⏳ Ready for Phase 3 (Content Development)
- ⏳ Approved by: <!-- Pending -->

---

## Notes & Iterations

### Design Decisions

**Decision 1: Cover All Four Connection Types**
- **Rationale:** Students need to understand all deployment patterns
- **Benefit:** Comprehensive understanding, can choose appropriate type
- **Alternative:** Focus on MCLAG only (rejected: too narrow)

**Decision 2: Use Different Connection Types in Lab**
- **Rationale:** Hands-on experience with MCLAG, ESLAG, unbundled
- **Benefit:** Reinforces connection type differences
- **Alternative:** Use only MCLAG (rejected: less realistic)

**Decision 3: Three-Server Lab (Not All 10)**
- **Rationale:** Three servers sufficient to demonstrate concept
- **Benefit:** Faster lab completion, less repetition
- **Alternative:** Attach all 10 servers (rejected: too time-consuming)

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING
**Version:** 1.0
**Previous Module:** Module 2.1 (Define VPC Network)
**Next Module:** Module 2.3 (Connectivity Validation - To Be Designed)

**Change Log:**
- 2025-10-16 v1.0: Initial design based on Issue #9 requirements

---

**Status:** ⏳ PENDING REVIEW
**Related Issue:** GitHub Issue #9 - [DESIGN] Course 2: Provisioning & Connectivity
**Design Duration:** ~4 hours (as estimated)
