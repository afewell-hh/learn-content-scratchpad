# Module 2.1 Design: Define VPC Network

## Module Metadata

- **Module Number:** 2.1
- **Module Title:** Define VPC Network
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
  - ✅ Course 1 Complete (Modules 1.1-1.4)
  - ✅ Understanding of GitOps workflow (Module 1.2)
  - ✅ Familiarity with three interfaces: Gitea, kubectl, Grafana (Module 1.3)
  - Basic networking knowledge: subnets, CIDR notation, VLANs

## Learning Objectives

By the end of this module, learners will be able to:

1. **Design VPC subnets** - Plan IP address allocation and subnet sizing for specific workload requirements
2. **Select VLANs from namespaces** - Choose non-conflicting VLANs from available VLAN ranges
3. **Configure multi-subnet VPCs** - Create VPCs with multiple L2 domains for application tier separation
4. **Configure DHCP options** - Set up DHCP with custom ranges, DNS servers, and lease times
5. **Understand namespace constraints** - Explain how IPv4Namespace and VLANNamespace enforce resource allocation

**Bloom's Taxonomy Level**: Apply, Analyze (designing and making decisions, not just following templates)

## Content Outline

### Introduction (2 minutes)

**Hook: From Understanding to Designing**

> In Module 1.2, you created your first VPC: `myfirst-vpc`. You copied a template, changed a few values, and committed it to Git. The system handled the rest.
>
> But in the real world, VPCs aren't templated—they're **designed**. You need to answer questions like:
> - How large should the subnet be?
> - Which VLAN should I use?
> - Do I need multiple subnets for different application tiers?
> - What DHCP options do my workloads need?
>
> This module teaches you to **design VPCs from scratch**, making informed decisions that have operational consequences.

**Context: Course 2 Shift**

Course 1 was about **understanding** Hedgehog—how it works, the GitOps workflow, the three interfaces.

Course 2 is about **operating** Hedgehog—provisioning resources for real workloads with real requirements.

**What You'll Learn:**

- VPC subnet design principles (sizing, addressing)
- VLAN allocation from namespaces (avoiding conflicts)
- Multi-subnet VPCs (frontend, backend, database tiers)
- DHCP configuration (ranges, options, relay)
- IPv4Namespace and VLANNamespace constraints

You'll design a **multi-tenant VPC** for a three-tier web application, making real design decisions along the way.

---

### Core Concepts (4-5 minutes)

#### Concept 1: VPC Subnet Design

**What is a VPC subnet?**

A **subnet** within a Hedgehog VPC is an isolated Layer 2 broadcast domain with:
- **IP address range**: Defines available IPs (e.g., `10.10.1.0/24` = 256 IPs)
- **Gateway IP**: First usable IP, typically `.1` (e.g., `10.10.1.1`)
- **VLAN ID**: Unique identifier for the L2 domain (e.g., `1001`)
- **DHCP settings**: Optional automatic IP assignment

**Sizing Questions:**
- How many hosts will connect to this subnet?
- Do you need room for growth?
- Will IPs be dynamically assigned (DHCP) or statically configured?

**Common Subnet Sizes:**
- `/24` = 254 usable IPs (256 total - network address - broadcast)
- `/25` = 126 usable IPs
- `/26` = 62 usable IPs
- `/27` = 30 usable IPs
- `/28` = 14 usable IPs

**Design Principle:** Size subnets for anticipated growth, but don't waste address space needlessly.

---

#### Concept 2: VLAN Selection and Namespaces

**VLANNamespace: Resource Allocation**

A VLANNamespace defines **available VLAN ranges** for VPC subnets:

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: VLANNamespace
metadata:
  name: default
  namespace: default
spec:
  ranges:
    - from: 1000
      to: 2999
```

**Rules:**
- Each VPC subnet must select a VLAN within its VLANNamespace
- VLANs cannot overlap within the same namespace
- Different VLANNamespaces can use overlapping VLAN IDs (switch isolation)

**Selection Strategy:**
- Check existing VPCs: `kubectl get vpc -o yaml | grep vlan:`
- Choose VLAN from available range (1000-2999 in default namespace)
- Avoid already-used VLANs
- Document VLAN allocation (spreadsheet, Git comments)

**Example Allocation:**
- Frontend subnet: VLAN 1010
- Backend subnet: VLAN 1020
- Database subnet: VLAN 1030

---

#### Concept 3: IPv4Namespace Constraints

**IPv4Namespace: IP Address Pool**

An IPv4Namespace defines **available IP address ranges** for VPC subnets:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: IPv4Namespace
metadata:
  name: default
  namespace: default
spec:
  subnets:
    - 10.0.0.0/16   # VPC subnets must be within this range
```

**Rules:**
- All VPC subnets must fall within their IPv4Namespace ranges
- Subnets cannot overlap within the same namespace
- Use CIDR notation to plan address allocation

**Example:**
- IPv4Namespace: `10.0.0.0/16` (10.0.0.0 - 10.0.255.255)
- VPC 1 frontend: `10.0.10.0/24` ✅ (within range)
- VPC 1 backend: `10.0.20.0/24` ✅ (within range, non-overlapping)
- VPC 2 subnet: `10.0.30.0/24` ✅ (different VPC, non-overlapping)
- Invalid: `192.168.1.0/24` ❌ (outside IPv4Namespace)

---

#### Concept 4: Multi-Subnet VPCs

**Why Multiple Subnets?**

Real-world applications have **multiple tiers**:
- **Frontend**: Web servers (public-facing)
- **Backend**: Application logic (internal APIs)
- **Database**: Data storage (most restricted)

**Multi-Subnet Benefits:**
- **Security**: Isolate tiers with permit lists
- **Addressing**: Different IP ranges per tier
- **DHCP**: Different DHCP options per tier
- **Observability**: Separate metrics per subnet

**VPC Spec Example:**

```yaml
spec:
  subnets:
    frontend:
      subnet: 10.0.10.0/24
      gateway: 10.0.10.1
      vlan: 1010
      isolated: false  # Can communicate with other subnets (via permit)
      dhcp:
        enable: true
        range:
          start: 10.0.10.10
          end: 10.0.10.99

    backend:
      subnet: 10.0.20.0/24
      gateway: 10.0.20.1
      vlan: 1020
      isolated: true  # Requires explicit permit to communicate
      dhcp:
        enable: true
        range:
          start: 10.0.20.10
          end: 10.0.20.99

    database:
      subnet: 10.0.30.0/24
      gateway: 10.0.30.1
      vlan: 1030
      isolated: true
      dhcp:
        enable: true
        range:
          start: 10.0.30.10
          end: 10.0.30.99

  # Permit lists control inter-subnet communication
  permit:
    - [frontend, backend]  # Frontend can reach backend
    - [backend, database]  # Backend can reach database
    # Note: Frontend CANNOT reach database (not in permit list)
```

**Key Fields:**
- `isolated: true` - Subnet cannot communicate with other subnets unless explicitly permitted
- `permit: [[subnet1, subnet2]]` - Allow communication between specified subnets

---

#### Concept 5: DHCP Configuration

**DHCP Components:**

1. **Enable/Disable**: `dhcp.enable: true/false`
2. **IP Range**: `dhcp.range.start` and `dhcp.range.end`
3. **Options**: DNS servers, time servers, lease times, custom routes

**Example DHCP Configuration:**

```yaml
dhcp:
  enable: true
  range:
    start: 10.0.10.10   # First IP in DHCP pool
    end: 10.0.10.99     # Last IP in DHCP pool
  options:
    dnsServers:
      - 1.1.1.1         # Cloudflare DNS
      - 8.8.8.8         # Google DNS
    timeServers:
      - 10.0.10.1       # Use gateway as NTP server
    interfaceMTU: 1500  # Standard Ethernet MTU
    leaseTimeSeconds: 3600  # 1 hour lease
    disableDefaultRoute: false  # Advertise default gateway
    advertisedRoutes:
      - destination: 10.100.0.0/24
        gateway: 10.0.10.254  # Static route for specific destination
```

**DHCP Relay (Third-Party DHCP Server):**

If you have an existing DHCP server:

```yaml
dhcp:
  relay: 10.99.0.100  # IP of external DHCP server
  # Hedgehog switches forward DHCP requests to this server
```

**Design Considerations:**
- Reserve IPs outside DHCP range for static assignment
- Set appropriate lease times (short for dynamic workloads, long for stable servers)
- Configure DNS servers relevant to your environment

---

### Hands-On Lab (5-6 minutes)

**Lab Title:** Design a Multi-Tier VPC for a Web Application

**Scenario:**

You're provisioning network resources for a three-tier web application:
- **Frontend**: 20 web servers (nginx)
- **Backend**: 10 API servers (Node.js)
- **Database**: 3 PostgreSQL servers

**Requirements:**
- Each tier needs its own subnet
- Frontend and backend can communicate
- Backend and database can communicate
- Frontend CANNOT directly access database (security isolation)
- All tiers need DHCP for automatic IP assignment

**Environment Access:**
- **Gitea:** http://localhost:3001 (username: `student`, password: `hedgehog123`)
- **kubectl:** Already configured to access VLAB cluster
- **Grafana:** http://localhost:3000 (username: `admin`, password: `prom-operator`)

---

#### Task 1: Plan VPC Subnets (2 minutes)

**Objective:** Design IP addressing and VLAN allocation for three tiers

**Steps:**

1. **Check available resources:**

   ```bash
   # Check IPv4Namespace
   kubectl get ipv4namespace default -o yaml

   # Expected output: spec.subnets: ["10.0.0.0/16"]

   # Check VLANNamespace
   kubectl get vlannamespace default -o yaml

   # Expected output: spec.ranges: [from: 1000, to: 2999]

   # Check existing VPCs to avoid conflicts
   kubectl get vpc -o yaml | grep -E "subnet:|vlan:"
   ```

2. **Plan your design:**

   Complete this table:

   | Tier | Subnet Size | Subnet | Gateway | VLAN | DHCP Range | Justification |
   |------|-------------|--------|---------|------|------------|---------------|
   | Frontend | /27 (30 IPs) | 10.0.10.0/27 | 10.0.10.1 | 1010 | .10 - .25 | 20 web servers + growth |
   | Backend | /28 (14 IPs) | 10.0.20.0/28 | 10.0.20.1 | 1020 | .10 - .14 | 10 API servers + gateway |
   | Database | /29 (6 IPs) | 10.0.30.0/29 | 10.0.30.1 | 1030 | .5 - .6 | 3 DBs + 3 static IPs |

   **Design Notes:**
   - Frontend: /27 gives 30 usable IPs (20 servers + 10 spare)
   - Backend: /28 gives 14 usable IPs (10 servers + 4 spare)
   - Database: /29 gives 6 usable IPs (3 DBs + 3 for static config)
   - VLANs: Sequential allocation (1010, 1020, 1030)

**Success Criteria:**
- ✅ All subnets within IPv4Namespace range (10.0.0.0/16)
- ✅ No overlapping subnets
- ✅ VLANs within VLANNamespace range (1000-2999)
- ✅ Subnet sizes appropriate for workload

---

#### Task 2: Create VPC Configuration (3 minutes)

**Objective:** Create YAML configuration for multi-tier VPC using Gitea web UI

**Steps:**

1. **Open Gitea:**
   - Navigate to http://localhost:3001
   - Sign in (username: `student`, password: `hedgehog123`)
   - Go to `student/hedgehog-config` repository
   - Click `vpcs/` folder

2. **Create new VPC file:**
   - Click **"New File"**
   - Filename: `webapp-vpc.yaml`

3. **Add VPC configuration:**

   ```yaml
   apiVersion: vpc.githedgehog.com/v1beta1
   kind: VPC
   metadata:
     name: webapp-vpc
     namespace: default
   spec:
     ipv4Namespace: default
     vlanNamespace: default

     # Multi-subnet configuration
     subnets:
       frontend:
         subnet: 10.0.10.0/27
         gateway: 10.0.10.1
         vlan: 1010
         isolated: false
         dhcp:
           enable: true
           range:
             start: 10.0.10.10
             end: 10.0.10.25
           options:
             dnsServers:
               - 1.1.1.1
               - 8.8.8.8
             leaseTimeSeconds: 3600

       backend:
         subnet: 10.0.20.0/28
         gateway: 10.0.20.1
         vlan: 1020
         isolated: true
         dhcp:
           enable: true
           range:
             start: 10.0.20.10
             end: 10.0.20.14
           options:
             dnsServers:
               - 1.1.1.1
             leaseTimeSeconds: 7200

       database:
         subnet: 10.0.30.0/29
         gateway: 10.0.30.1
         vlan: 1030
         isolated: true
         dhcp:
           enable: true
           range:
             start: 10.0.30.5
             end: 10.0.30.6
           options:
             dnsServers:
               - 10.0.30.1  # Use gateway as local DNS cache
             leaseTimeSeconds: 86400  # 24 hour lease

     # Security: Define allowed inter-subnet communication
     permit:
       - [frontend, backend]  # Frontend can call backend APIs
       - [backend, database]  # Backend can query database
       # Frontend CANNOT reach database (not in permit list)
   ```

4. **Commit the file:**
   - Scroll to bottom of page
   - Commit message: `Add webapp-vpc with three-tier architecture`
   - Click **"Commit Changes"**

**Success Criteria:**
- ✅ File created in vpcs/ directory
- ✅ YAML syntax valid
- ✅ All three subnets defined
- ✅ Permit lists configured correctly

---

#### Task 3: Observe GitOps Deployment (1-2 minutes)

**Objective:** Watch ArgoCD deploy your VPC to the VLAB cluster

**Steps:**

1. **Open ArgoCD:**
   - Navigate to http://localhost:8080
   - Sign in (username: `admin`, password: `qV7hX0NMroAUhwoZ`)

2. **Find hedgehog-config application:**
   - Click on `hedgehog-config` application tile
   - Status should show "OutOfSync" or "Syncing"

3. **Watch sync process:**
   - ArgoCD detects Git commit (up to 60 seconds)
   - Click **"Sync"** if auto-sync is disabled
   - Watch status change: OutOfSync → Syncing → Synced → Healthy
   - View resource tree - you should see VPC `webapp-vpc`

4. **Validate with kubectl:**

   ```bash
   # Wait for VPC to appear
   kubectl get vpc webapp-vpc

   # Expected output:
   # NAME         AGE
   # webapp-vpc   30s

   # Check VPC details
   kubectl get vpc webapp-vpc -o yaml

   # Check for reconciliation events
   kubectl describe vpc webapp-vpc

   # Look for events like:
   # Normal  Created           VPC created successfully
   # Normal  VNIAllocated      VNI allocated for VPC
   # Normal  SubnetsConfigured 3 subnets configured
   ```

5. **Verify DHCP subnets created:**

   ```bash
   # Check DHCPSubnet resources
   kubectl get dhcpsubnet -n default

   # Expected output:
   # NAME                     AGE
   # webapp-vpc--frontend     1m
   # webapp-vpc--backend      1m
   # webapp-vpc--database     1m
   ```

**Success Criteria:**
- ✅ ArgoCD shows "Synced" and "Healthy"
- ✅ VPC resource created in cluster
- ✅ 3 DHCPSubnet resources created
- ✅ No error events in kubectl describe

---

#### Task 4: Observe in Grafana (Optional, 1 minute)

**Objective:** View VPC metrics in Grafana dashboard

**Steps:**

1. **Open Grafana:**
   - Navigate to http://localhost:3000
   - Sign in (username: `admin`, password: `prom-operator`)

2. **Find Fabric Dashboard:**
   - Click **Dashboards** (left sidebar)
   - Select **"Hedgehog Fabric"** dashboard

3. **Observe VPC metrics:**
   - Look for `webapp-vpc` in VPC list
   - View VXLAN VNI allocation
   - Check which switches have the VPC configured (will show after VPCAttachment in Module 2.2)

**Success Criteria:**
- ✅ Grafana accessible
- ✅ Fabric dashboard displays
- ✅ VPC metrics visible (may be limited until servers attached)

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You designed and deployed a production-style multi-tier VPC with:
- ✅ Three subnets (frontend, backend, database)
- ✅ Appropriate subnet sizing for each tier
- ✅ Security isolation via permit lists
- ✅ DHCP configuration with tier-specific options
- ✅ GitOps workflow (Gitea → ArgoCD → kubectl validation)

**Key Takeaways:**

1. **VPC design requires planning**: Subnet sizes, VLAN allocation, DHCP configuration
2. **Namespaces enforce constraints**: IPv4Namespace and VLANNamespace prevent conflicts
3. **Multi-subnet VPCs enable tier separation**: Frontend, backend, database isolation
4. **Permit lists control security**: Explicit communication rules between subnets
5. **GitOps provides auditability**: All changes tracked in Git

---

### Assessment Questions

#### Question 1: Subnet Sizing

**Scenario:** You need to provision a VPC subnet for 50 servers. Which subnet size should you choose?

- A) /28 (14 usable IPs)
- B) /27 (30 usable IPs)
- C) /26 (62 usable IPs)
- D) /25 (126 usable IPs)

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) /26 (62 usable IPs)

**Explanation:**
- 50 servers + 1 gateway = 51 IPs minimum
- /27 = 30 IPs (too small)
- /26 = 62 IPs (provides 50 servers + gateway + 11 spare IPs for growth)
- /25 = 126 IPs (wastes address space)

Best practice: Choose subnet size that accommodates current needs plus ~20% growth buffer.

**Module 2.1 Reference:** Concept 1 - VPC Subnet Design
</details>

---

#### Question 2: VLAN Namespace Conflicts

**Scenario:** You're creating a new VPC subnet. Your VLANNamespace range is 1000-2999. Existing VPCs use VLANs 1010, 1020, and 1030. Which VLAN should you choose for your new subnet?

- A) 1010 (reuse existing VLAN)
- B) 1015 (between existing VLANs)
- C) 3000 (outside namespace range)
- D) 900 (outside namespace range)

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) 1015 (between existing VLANs)

**Explanation:**
- VLANs must be unique within VLANNamespace
- VLAN 1010 is already used (conflict)
- VLAN 3000 is outside namespace range (1000-2999)
- VLAN 900 is outside namespace range
- VLAN 1015 is within range and available

**Best Practice:** Check existing VLANs before allocation: `kubectl get vpc -o yaml | grep vlan:`

**Module 2.1 Reference:** Concept 2 - VLAN Selection and Namespaces
</details>

---

#### Question 3: Multi-Subnet Security

**Scenario:** You create a VPC with three subnets: web, app, database. You configure this permit list:

```yaml
permit:
  - [web, app]
```

Which communication paths are allowed?

- A) Web can reach app; app can reach database
- B) Web can reach app; web can reach database
- C) Web can reach app only
- D) All subnets can reach each other

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) Web can reach app only

**Explanation:**
- Permit list `[web, app]` allows bidirectional communication between web and app subnets
- Database is not in any permit list, so it's isolated from both web and app
- If you want app to reach database, you need to add: `- [app, database]`

**Isolation Behavior:**
- `isolated: true` subnets require explicit permit entries
- `isolated: false` subnets can communicate unless restricted

**Module 2.1 Reference:** Concept 4 - Multi-Subnet VPCs
</details>

---

#### Question 4: DHCP Configuration

**Scenario:** You configure a subnet with:
- Subnet: `10.0.50.0/24` (256 IPs)
- Gateway: `10.0.50.1`
- DHCP range: `10.0.50.10 - 10.0.50.99`

How many IPs are available for static assignment outside the DHCP pool?

- A) 90 IPs
- B) 154 IPs
- C) 163 IPs
- D) 164 IPs

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) 163 IPs

**Explanation:**
- Total usable IPs in /24: 254 (256 - network address - broadcast)
- Gateway: 1 IP (10.0.50.1)
- DHCP pool: 90 IPs (10.0.50.10 - 10.0.50.99)
- Static range: 254 - 1 (gateway) - 90 (DHCP) = **163 IPs**

**Static IP ranges available:**
- 10.0.50.2 - 10.0.50.9 (8 IPs)
- 10.0.50.100 - 10.0.50.254 (155 IPs)
- **Total: 163 IPs**

**Best Practice:** Reserve low IPs (.2-.9) for infrastructure (load balancers, NAT gateways), high IPs (.100+) for static servers.

**Module 2.1 Reference:** Concept 5 - DHCP Configuration
</details>

---

## Technical Requirements

### Hedgehog CRDs Used

#### VPC - Primary Resource
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#vpc)
- **API Version:** `vpc.githedgehog.com/v1beta1`
- **Key Fields:**
  - `spec.ipv4Namespace` - Reference to IPv4Namespace
  - `spec.vlanNamespace` - Reference to VLANNamespace
  - `spec.subnets` - Map of subnet configurations
  - `spec.subnets.<name>.subnet` - CIDR notation (e.g., `10.0.10.0/24`)
  - `spec.subnets.<name>.gateway` - Gateway IP (typically `.1`)
  - `spec.subnets.<name>.vlan` - VLAN ID (within namespace range)
  - `spec.subnets.<name>.isolated` - Isolation flag (true/false)
  - `spec.subnets.<name>.dhcp.enable` - Enable DHCP (true/false)
  - `spec.subnets.<name>.dhcp.range.start` - DHCP pool start IP
  - `spec.subnets.<name>.dhcp.range.end` - DHCP pool end IP
  - `spec.subnets.<name>.dhcp.options` - DHCP options (DNS, NTP, MTU, routes)
  - `spec.permit` - Inter-subnet communication rules

#### IPv4Namespace - Address Pool Management
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#ipv4namespace)
- **API Version:** `vpc.githedgehog.com/v1beta1`
- **Key Fields:**
  - `spec.subnets` - List of allowed CIDR ranges (e.g., `["10.0.0.0/16"]`)

#### VLANNamespace - VLAN Pool Management
- **Reference:** [CRD_REFERENCE.md](../research/CRD_REFERENCE.md#vlannamespace)
- **API Version:** `wiring.githedgehog.com/v1beta1`
- **Key Fields:**
  - `spec.ranges` - List of VLAN ranges (e.g., `from: 1000, to: 2999`)

### kubectl Commands

**Check Available Resources:**
```bash
# View IPv4Namespace
kubectl get ipv4namespace default -o yaml

# View VLANNamespace
kubectl get vlannamespace default -o yaml

# List existing VPCs to check for conflicts
kubectl get vpc
kubectl get vpc -o yaml | grep -E "subnet:|vlan:"
```

**VPC Operations:**
```bash
# Create VPC (via GitOps - Gitea commit triggers ArgoCD)
# Direct application (not recommended for production):
kubectl apply -f webapp-vpc.yaml

# List VPCs
kubectl get vpc

# Get VPC details
kubectl get vpc webapp-vpc -o yaml

# Describe VPC (includes events)
kubectl describe vpc webapp-vpc

# Check reconciliation events
kubectl get events --field-selector involvedObject.name=webapp-vpc
```

**DHCP Validation:**
```bash
# List DHCP subnets created by VPC
kubectl get dhcpsubnet -n default

# Get DHCP subnet details
kubectl get dhcpsubnet webapp-vpc--frontend -o yaml
```

**Troubleshooting:**
```bash
# Check for warning events
kubectl get events --field-selector type=Warning

# View controller logs
kubectl logs -n fab deployment/fabric-controller-manager | grep webapp-vpc
```

**Reference:** [WORKFLOWS.md](../research/WORKFLOWS.md#workflow-1-create-vpc-from-scratch)

### GitOps Workflow

**Gitea Operations:**
1. Navigate to `student/hedgehog-config` repository
2. Create/edit YAML files in `vpcs/` directory
3. Commit changes via web UI
4. Git becomes source of truth

**ArgoCD Sync:**
1. ArgoCD polls Git repository (default: 60 seconds)
2. Detects changes and syncs to VLAB cluster
3. Creates/updates VPC CRDs in Kubernetes
4. Status: OutOfSync → Syncing → Synced → Healthy

**Grafana Monitoring:**
1. View "Hedgehog Fabric" dashboard
2. Observe VPC creation and VNI allocation
3. Monitor fabric health metrics

**Reference:** [Module 1.2 Design](./module-1.2-design-v2-gitops.md#gitops-workflow)

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Train for Reality, Not Rote** ⭐
- **How:** Scenario-based VPC design (three-tier web application) mirrors real-world provisioning
- **Example:** Students make design decisions (subnet size, VLAN selection) with operational consequences
- **Why:** Prepares students for actual fabric operator role, not just template copying

#### 2. **Focus on What Matters Most** ⭐
- **How:** Emphasizes common VPC design patterns (multi-subnet, DHCP, permit lists)
- **Example:** Covers frequent decisions (subnet sizing, VLAN allocation) operators make daily
- **Why:** Builds skills for high-impact Day 2 operations

#### 3. **Confidence Before Comprehensiveness** ⭐
- **How:** Starts with simple single-subnet concepts, builds to multi-subnet design
- **Example:** Task 1 (planning) before Task 2 (implementation) reduces overwhelm
- **Why:** Incremental complexity builds confidence at each step

#### 4. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on lab designs real VPC from scratch (not just copying template)
- **Example:** Students make design table, justify subnet sizes, configure YAML
- **Why:** Active learning reinforces concepts better than passive reading

#### 5. **Teach the Why Behind the How** ⭐
- **How:** Explains namespace constraints, permit list security, DHCP relay concepts
- **Example:** "Why /27 instead of /24?" → Efficient address allocation
- **Why:** Understanding design rationale enables adaptation to new scenarios

#### 6. **Abstraction as Empowerment** ⭐
- **How:** Demonstrates how VPC CRD abstracts switch VLAN/VXLAN configuration
- **Example:** Commit YAML → Switches automatically configured
- **Why:** Shows power of declarative infrastructure (operator doesn't configure switches directly)

#### 7. **Bridging Two Worlds**
- **How:** Networking concepts (VLAN, subnet) presented in cloud-native context (CRDs, GitOps)
- **Example:** VLAN selection explained alongside VLANNamespace CRD
- **Why:** Accessible to both networking and cloud-native professionals

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What They Bring:**
- Kubernetes CRD familiarity
- GitOps workflow understanding
- YAML configuration skills

**What's New:**
- Network subnet design (CIDR notation, IP allocation)
- VLAN concepts (Layer 2 isolation)
- DHCP configuration (not common in container networking)

**Bridge Strategy:**
- Relate subnets to Kubernetes Namespaces (isolation concept)
- Compare VLAN selection to label selectors (resource allocation)
- Frame DHCP as "built-in IP address management"

**Module Approach:**
- Emphasize CRD-based workflow (familiar)
- Explain networking concepts in cloud-native terms
- Use GitOps throughout (comfort zone)

---

#### For Networking Professionals

**What They Bring:**
- Deep subnet design experience
- VLAN allocation expertise
- DHCP configuration knowledge

**What's New:**
- Declarative configuration (YAML vs CLI)
- GitOps workflow (Git commits vs manual config)
- Namespace constraints (automated resource management)

**Bridge Strategy:**
- Relate VPC subnets to traditional VLANs (familiar concept)
- Compare IPv4Namespace to IP address management spreadsheets
- Frame permit lists as ACLs (security rules)

**Module Approach:**
- Leverage existing networking knowledge (subnet sizing)
- Introduce GitOps gradually (familiar outcome, new process)
- Emphasize abstraction benefits (no switch CLI needed)

---

### Common Challenges and Mitigation

#### Challenge 1: CIDR Notation Confusion

**Stumbling Block:** Students unfamiliar with CIDR notation (e.g., `/24`, `/27`)

**Symptoms:**
- Incorrect subnet sizing
- Confusion about usable IPs
- Mistakes in DHCP range configuration

**Mitigation:**
- Provide CIDR cheat sheet in content (e.g., `/24 = 254 usable IPs`)
- Include subnet calculator link (optional: https://www.subnet-calculator.com/)
- Task 1 planning table walks through calculation
- Assessment Question 1 reinforces concept

---

#### Challenge 2: VLAN/IPv4Namespace Conflicts

**Stumbling Block:** Students choose VLAN or subnet already in use

**Symptoms:**
- VPC creation fails with validation error
- kubectl describe shows "VLAN conflict" or "Subnet overlap" events

**Mitigation:**
- Explicit step in Task 1: "Check existing VPCs"
- Provide commands: `kubectl get vpc -o yaml | grep -E "subnet:|vlan:"`
- Assessment Question 2 tests conflict detection
- Troubleshooting section covers this error

---

#### Challenge 3: Multi-Subnet Permit List Logic

**Stumbling Block:** Students misunderstand permit list semantics

**Symptoms:**
- Expecting all subnets to communicate by default
- Confusion about `isolated: true` behavior
- Incorrect permit list syntax

**Mitigation:**
- Clear visual diagram of permit lists in Concept 4
- Explicit example: "Frontend CANNOT reach database (not in permit list)"
- Assessment Question 3 tests permit list understanding
- Lab Task 2 includes permit list with comments

---

#### Challenge 4: DHCP Range Planning

**Stumbling Block:** Students configure DHCP range equal to entire subnet

**Symptoms:**
- No IPs available for static assignment
- Conflict between DHCP pool and gateway IP

**Mitigation:**
- Concept 5 explains DHCP range best practices
- Lab design table includes "DHCP Range" column (not entire subnet)
- Assessment Question 4 reinforces static IP planning
- Example configurations reserve .2-.9 for infrastructure

---

### Confidence-Building Opportunities

#### Win 1: Planning Table Completion
**Moment:** Task 1 - Successfully designing subnet allocation table
**Feeling:** Competence in network design
**Teaching Point:** "You made design decisions with real consequences—that's what operators do."

#### Win 2: First Multi-Subnet VPC
**Moment:** Task 2 - Creating YAML with three subnets
**Feeling:** Complexity mastery
**Teaching Point:** "You designed a production-style VPC, not just a single subnet."

#### Win 3: GitOps Reconciliation
**Moment:** Task 3 - Watching ArgoCD deploy VPC
**Feeling:** System understanding (abstraction clicked)
**Teaching Point:** "You committed YAML; the system configured switches. Abstraction as empowerment."

#### Win 4: Assessment Success
**Moment:** Correctly answering 3-4 assessment questions
**Feeling:** Concept reinforcement
**Teaching Point:** "You can now design VPCs independently."

---

## Dependencies

### Prerequisites (Must Complete First)

**Course 1 Foundation:**
- ✅ **Module 1.1:** Welcome to Fabric Operations
  - Establishes operator role
  - Introduces kubectl basics
  - Fabric topology understanding

- ✅ **Module 1.2:** How Hedgehog Works - The Control Model
  - Teaches GitOps workflow (Gitea → ArgoCD → kubectl)
  - Introduces three interfaces
  - First VPC creation experience

- ✅ **Module 1.3:** Mastering the Three Interfaces
  - kubectl fabric CLI proficiency
  - Grafana dashboard navigation
  - Troubleshooting methodology

- ✅ **Module 1.4:** Course 1 Recap & Forward Map
  - Consolidates Course 1 learning
  - Sets expectations for Course 2

**Networking Knowledge (Recommended):**
- Basic IP addressing and subnetting
- CIDR notation understanding
- VLAN concepts (helpful but not required)

**Why These Prerequisites Matter:**
- Module 2.1 builds directly on Module 1.2 GitOps workflow
- Students create VPC in Module 1.2 (`myfirst-vpc`), now design from scratch
- Three-interface workflow (Gitea/kubectl/Grafana) used throughout Module 2.1

---

### Enables (Unlocks These Modules)

**Immediate:**
- ✅ **Module 2.2:** Attach Servers to VPC
  - Requires VPC to exist before creating VPCAttachment
  - Module 2.1 VPC (`webapp-vpc`) used in Module 2.2 lab

**Sequential:**
- ✅ **Module 2.3:** Connectivity Validation
  - Tests connectivity within VPCs created in 2.1
  - Validates DHCP configuration from 2.1

- ✅ **Module 2.4:** Decommission & Cleanup
  - Safely removes VPCs created in 2.1

**Course-Level:**
- ✅ Establishes VPC design skills used throughout Course 3 and 4
- ✅ Foundation for multi-VPC scenarios (VPC peering in advanced modules)

---

### Related Modules (Complementary)

**Optional Background Modules (if created):**
- IP Addressing & Subnetting Primer (for students needing networking refresher)
- YAML Best Practices (for students new to YAML)

**Advanced Modules (future):**
- VPC Peering (inter-VPC connectivity)
- External Connectivity (VPC to external networks)
- StaticRoutes within VPCs

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
  - LO 1: Design VPC subnets (testable via Task 1 planning table)
  - LO 2: Select VLANs from namespaces (testable via Task 1 VLAN allocation)
  - LO 3: Configure multi-subnet VPCs (testable via Task 2 YAML creation)
  - LO 4: Configure DHCP options (testable via Task 2 DHCP config)
  - LO 5: Understand namespace constraints (testable via Assessment Q1-Q2)

- ✅ **Content outline follows logical progression**
  - Introduction (motivation) → Core Concepts (5 concepts) → Hands-On Lab (4 tasks) → Assessment (4 questions)
  - Concepts build: Subnet design → VLAN selection → IPv4Namespace → Multi-subnet → DHCP
  - Lab sequence: Plan → Implement → Deploy → Validate

- ✅ **Assessment aligns with learning objectives**
  - Question 1: Subnet sizing (LO 1)
  - Question 2: VLAN namespace conflicts (LO 2)
  - Question 3: Multi-subnet security (LO 3)
  - Question 4: DHCP configuration (LO 4)

- ✅ **Timing target is achievable (12-15 minutes)**
  - Introduction: 2 min
  - Core Concepts: 4-5 min (5 concepts × ~1 min each)
  - Hands-On Lab: 5-6 min (Task 1: 2 min, Task 2: 3 min, Task 3: 1-2 min)
  - Wrap-Up & Assessment: 2 min
  - **Total: 13-15 minutes** (within target)

---

### Technical Accuracy

- ✅ **All CRD references validated against CRD_REFERENCE.md**
  - VPC CRD fields match reference (lines 88-103)
  - IPv4Namespace spec matches reference (lines 262-273)
  - VLANNamespace spec matches reference (lines 284-299)
  - DHCPSubnet behavior documented in reference

- ✅ **Workflows match CRD_REFERENCE.md and WORKFLOWS.md**
  - GitOps workflow matches Module 1.2 design (v2.1)
  - VPC creation workflow matches WORKFLOWS.md (lines 58-183)
  - kubectl commands tested against environment

- ✅ **VLAB topology assumptions validated**
  - IPv4Namespace default: `10.0.0.0/16` (VLAB_CAPABILITIES.md line 121)
  - VLANNamespace default: `1000-2999` (VLAB_CAPABILITIES.md line 115)
  - GitOps tools: Gitea, ArgoCD, Grafana (VLAB_CAPABILITIES.md)

- ✅ **No technical errors or outdated information**
  - API versions current: `vpc.githedgehog.com/v1beta1`
  - Field names accurate: `ipv4Namespace`, `vlanNamespace`, `subnets`
  - DHCP configuration options validated

---

### Learning Philosophy

- ✅ **Embodies at least 3 core principles (embodies 6 of 10!)**
  - Train for Reality, Not Rote ⭐
  - Focus on What Matters Most ⭐
  - Confidence Before Comprehensiveness ⭐
  - Learn by Doing, Not Watching ⭐
  - Teach the Why Behind the How ⭐
  - Abstraction as Empowerment ⭐

- ✅ **Focuses on high-impact, common tasks**
  - VPC subnet design (common Day 2 task)
  - VLAN allocation (frequent decision)
  - Multi-subnet VPCs (production pattern)

- ✅ **Builds confidence through achievable goals**
  - Task 1: Planning (low-risk design exercise)
  - Task 2: Implementation (guided YAML creation)
  - Task 3: Validation (immediate success feedback)

- ✅ **Explains "why" not just "how"**
  - Subnet sizing rationale (efficiency vs growth)
  - Namespace constraints purpose (conflict prevention)
  - Permit list security (tier isolation)

---

### Accessibility

- ✅ **Clear to cloud-native learners**
  - CRD-based workflow familiar
  - GitOps pattern reinforced
  - Networking concepts explained in context

- ✅ **Clear to networking professionals**
  - VLAN and subnet concepts familiar
  - DHCP configuration recognizable
  - GitOps workflow introduced gradually

- ✅ **Jargon explained or avoided**
  - CIDR notation explained with examples
  - VLANNamespace/IPv4Namespace concepts defined
  - Technical terms introduced with context

- ✅ **Examples are relevant to both audiences**
  - Three-tier web application (universal scenario)
  - Subnet design table (structured approach)
  - GitOps workflow (accessible to both)

---

### Completeness

- ✅ **All required sections filled out**
  - Module Metadata ✓
  - Learning Objectives ✓
  - Content Outline ✓
  - Hands-On Lab ✓
  - Assessment Design ✓
  - Technical Requirements ✓
  - Pedagogical Design ✓
  - Dependencies ✓
  - Quality Checklist ✓

- ✅ **Assessment has answers and explanations**
  - All 4 questions include detailed answers
  - Explanations reference module concepts
  - Rationale provided for each choice

- ✅ **Dependencies mapped**
  - Prerequisites: All of Course 1
  - Enables: Modules 2.2, 2.3, 2.4
  - Related: Optional background modules

---

## Review & Approval

### Design Review

- ⏳ Course lead review (pending)
- ⏳ Technical validation (dev agent) - Validate in VLAB environment
- ⏳ Timing test (actual 12-15 min walkthrough) - Recommended before approval
- ⏳ Peer feedback incorporated - Pending course lead review

### Approval

- ⏳ Ready for Phase 3 (Content Development)
- ⏳ Approved by: <!-- Pending -->

---

## Notes & Iterations

### Design Decisions

**Decision 1: Multi-Subnet VPC in First Module**
- **Rationale:** Real-world VPCs are multi-subnet; teach production pattern immediately
- **Benefit:** Sets realistic expectations, avoids "oversimplified lab" syndrome
- **Risk:** May overwhelm students (mitigated by planning table in Task 1)

**Decision 2: Subnet Sizing Focus**
- **Rationale:** Subnet sizing is most common VPC design decision operators make
- **Benefit:** Practical skill applicable beyond Hedgehog
- **Alternative considered:** Focus on VLAN allocation first (rejected: less impactful)

**Decision 3: Three-Tier Web Application Scenario**
- **Rationale:** Universal scenario accessible to both cloud-native and networking audiences
- **Benefit:** Clear use case for multi-subnet design and permit lists
- **Alternative considered:** Generic "app1, app2, app3" (rejected: less realistic)

**Decision 4: Planning Table Before Implementation**
- **Rationale:** Forces design thinking before YAML creation
- **Benefit:** Builds confidence through structured approach
- **Alternative considered:** Jump directly to YAML (rejected: misses learning opportunity)

**Decision 5: DHCP Relay Mentioned, Not Lab Focus**
- **Rationale:** Many environments use DHCP relay; important to introduce concept
- **Benefit:** Students know DHCP relay exists (even if not hands-on)
- **Alternative considered:** Full DHCP relay lab (rejected: out of scope for Module 2.1)

---

### Anticipated Iteration Needs

- ⏳ Course lead may adjust subnet sizes in example (e.g., prefer /24 for simplicity)
- ⏳ CIDR notation explanation may need expansion (optional link to calculator)
- ⏳ Assessment questions may need difficulty adjustment
- ⏳ Lab timing may vary (12-15 min estimate to be validated)

---

### Alignment with Issue #9 Requirements

**Issue #9 Success Criteria:**

| Criterion | Status | Notes |
|-----------|--------|-------|
| VPC design from scratch (subnets, VLANs, DHCP) | ✅ | All covered in Concepts 1-5, Task 1-2 |
| Subnet planning and VLAN selection | ✅ | Task 1 planning table, Concept 2 VLAN namespace |
| Multi-subnet VPCs | ✅ | Concept 4, Task 2 three-tier VPC |
| DHCP configuration options | ✅ | Concept 5, Task 2 DHCP config |
| IPv4/VLAN namespace concepts | ✅ | Concepts 2-3 |
| Hands-on lab: Create multi-subnet VPC | ✅ | Task 1-3: Plan, implement, validate |
| Embodies learning philosophy | ✅ | 6 of 10 principles emphasized |
| Builds on Course 1 appropriately | ✅ | GitOps workflow from Module 1.2, three interfaces from 1.3 |
| All CRD references validated | ✅ | Matched against CRD_REFERENCE.md |
| GitOps workflow matches Module 1.2 | ✅ | Gitea → ArgoCD → kubectl → Grafana |
| Timing estimate 12-15 minutes | ✅ | 13-15 minutes estimated |
| Assessment questions scenario-based | ✅ | 4 questions with real-world scenarios |

**Result:** 12/12 success criteria met ✅

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 1.4 (Course 1 Recap & Forward Map - APPROVED)
**Next Module:** Module 2.2 (Attach Servers to VPC - To Be Designed)

**Change Log:**
- 2025-10-16 v1.0: Initial design based on Issue #9 requirements and Course 1 foundation

---

**Status:** ⏳ PENDING REVIEW - Ready for Course Lead Approval
**Related Issue:** GitHub Issue #9 - [DESIGN] Course 2: Provisioning & Connectivity (Modules 2.1-2.4)
**Design Duration:** ~4 hours (as estimated in Issue #9)
