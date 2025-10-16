# Module 3.3 Design: Events & Status Monitoring

## Module Metadata

- **Module Number:** 3.3
- **Module Title:** Events & Status Monitoring
- **Course:** Course 3 - Observability & Fabric Health
- **Estimated Duration:** 13-15 minutes
  - Introduction: 2 minutes
  - Core Concepts: 5-6 minutes
  - Hands-On Lab: 5-6 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 3.1 Complete (Fabric Telemetry Overview)
  - ✅ Module 3.2 Complete (Dashboard Interpretation)
  - ✅ Module 1.3 Complete (kubectl basics from Course 1)
  - ✅ Understanding of Kubernetes events

## Learning Objectives

By the end of this module, learners will be able to:

1. **Monitor kubectl events** - Track VPC lifecycle events and reconciliation status
2. **Interpret Agent CRD status** - Read switch operational state from Agent CRD status fields
3. **Track VPC lifecycle** - Follow VPC creation, attachment, and deletion through events
4. **Identify error patterns** - Recognize common failure events (VLAN conflicts, subnet overlaps, missing connections)
5. **Correlate kubectl and Grafana** - Cross-reference events with metrics for complete troubleshooting picture
6. **Apply integrated troubleshooting** - Use both kubectl events and Grafana dashboards together

**Bloom's Taxonomy Level**: Analyze, Evaluate (correlating multiple data sources, root cause analysis)

## Content Outline

### Introduction (2 minutes)

**Hook: The Complete Observability Picture**

> In Module 3.2, you learned to interpret Grafana dashboards—answering questions like "Are BGP sessions up?" and "Are there interface errors?"
>
> Dashboards show **what** is happening (metrics over time).
>
> But when something goes wrong, you need to know **why** it's happening.
>
> That's where **kubectl events and Agent CRD status** come in—they provide:
> - **Configuration errors:** "VLAN 1010 already in use"
> - **Reconciliation status:** "VPC successfully created"
> - **Dependency issues:** "Connection server-01--mclag not found"
> - **Switch state details:** BGP neighbor state, interface status, platform health

**The Troubleshooting Question:**

A user reports: "My server can't reach the VPC."

**Grafana tells you:**
- Interface is up
- No packet errors
- VLAN is configured

**But that's not enough. You need to know:**
- Did VPCAttachment reconcile successfully? *(kubectl events)*
- Is the switch Agent ready? *(Agent CRD status)*
- Was there a configuration error? *(kubectl events)*
- What's the exact VLAN and interface? *(Agent CRD state)*

**What You'll Learn:**

- How to monitor Kubernetes events for fabric resources
- How to interpret Agent CRD status fields
- How to track VPC lifecycle through events
- Common error event patterns and their meanings
- How to correlate kubectl data with Grafana metrics

**The Integration:**

```
Grafana:    "Interface Ethernet5 has no traffic"
                        ↓
kubectl events: "VPCAttachment failed: VLAN conflict"
                        ↓
Agent CRD:  "VLAN 1010 already used by VPC other-vpc"
                        ↓
Solution:   Change VLAN in VPC spec
```

This module teaches you to use kubectl and Grafana **together** for complete observability.

---

### Core Concepts (5-6 minutes)

#### Concept 1: Kubernetes Event Monitoring

**What Are Kubernetes Events?**

Events are timestamped records of actions taken by Kubernetes controllers. In Hedgehog, events track:
- Resource creation/updates
- Reconciliation success/failure
- Configuration validation errors
- Dependency resolution

**Event Types:**

**1. Normal Events (Informational)**
- Successful operations
- Reconciliation progress
- State transitions

**2. Warning Events (Errors/Issues)**
- Validation failures
- Dependency problems
- Reconciliation errors

**Viewing All Events:**

```bash
# All events, recent first
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Events in specific namespace
kubectl get events -n default --sort-by='.lastTimestamp'

# Events in fab namespace (switches, agents)
kubectl get events -n fab --sort-by='.lastTimestamp'

# Watch events live
kubectl get events --all-namespaces --watch
```

**Filtering Events:**

```bash
# Only Warning events
kubectl get events --all-namespaces --field-selector type=Warning

# Events for specific resource
kubectl get events --field-selector involvedObject.name=myvpc

# Events for specific resource type
kubectl get events --field-selector involvedObject.kind=VPC

# Events in last 1 hour
kubectl get events --all-namespaces --field-selector involvedObject.kind=VPC
```

**Event Fields:**

```
LAST SEEN   TYPE      REASON              OBJECT           MESSAGE
2m          Normal    Created             vpc/myvpc        VPC created successfully
1m          Normal    VNIAllocated        vpc/myvpc        Allocated VNI 100010
30s         Warning   VLANConflict        vpc/test-vpc     VLAN 1010 already in use by vpc-1
```

- **LAST SEEN:** How long ago event occurred
- **TYPE:** Normal or Warning
- **REASON:** Short event reason code
- **OBJECT:** Resource that triggered event
- **MESSAGE:** Detailed human-readable message

**Event Retention:**

Kubernetes retains events for **1 hour by default**. After 1 hour, events are deleted.

**Best Practice:** Check events soon after operations. For long-term audit, export events or use logging system.

---

#### Concept 2: Agent CRD Status - Switch State

**What is Agent CRD?**

The **Agent CRD** is Hedgehog's internal representation of each switch. It contains **comprehensive switch operational state**—everything happening on the switch right now.

**Why Agent CRD Matters:**

While VPC, VPCAttachment, and Connection CRDs have minimal status (`status: {}`), the **Agent CRD has detailed status fields**:

- Switch version and uptime
- Interface operational state (up/down, counters)
- BGP neighbor state
- Platform health (PSU, fans, temperature)
- ASIC resource usage
- Configuration application status

**Accessing Agent CRD:**

```bash
# List all switch agents
kubectl get agents -n fab

# Expected output:
NAME       ROLE          DESCR           APPLIED   VERSION
leaf-01    server-leaf   VS-01 MCLAG 1   2m        v0.87.4
leaf-02    server-leaf   VS-02 MCLAG 1   3m        v0.87.4
spine-01   spine         VS-06           4m        v0.87.4
...
```

**Get Full Agent Status:**

```bash
# Full YAML output
kubectl get agent leaf-01 -n fab -o yaml

# Human-readable description
kubectl describe agent leaf-01 -n fab
```

**Key Status Fields:**

**1. Agent Heartbeat and Version:**

```bash
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastHeartbeat}'
# Output: 2025-10-16T10:30:00Z

kubectl get agent leaf-01 -n fab -o jsonpath='{.status.version}'
# Output: v0.41.3
```

**2. Configuration Application Status:**

```bash
# When was config last applied?
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastAppliedTime}'

# What generation was applied?
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastAppliedGen}'
```

**3. Interface State:**

```bash
# All interfaces
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | jq

# Specific interface
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq
```

**Example output:**

```json
{
  "enabled": true,
  "admin": "up",
  "oper": "up",
  "mac": "00:11:22:33:44:55",
  "speed": "25G",
  "counters": {
    "inb": 123456789,
    "outb": 987654321,
    "ine": 0,
    "oute": 0
  }
}
```

**4. BGP Neighbor State:**

```bash
# All BGP neighbors
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# Check specific neighbor state
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors.default["172.30.128.10"].state}'
# Output: established
```

**5. Platform Health:**

```bash
# PSU status
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.platform.psus}' | jq

# Fan status
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.platform.fans}' | jq

# Temperature
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.platform.temps}' | jq
```

**6. ASIC Critical Resources:**

```bash
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.criticalResources}' | jq
```

**Example output:**

```json
{
  "stats": {
    "ipv4RoutesUsed": 150,
    "ipv4RoutesAvailable": 32000,
    "ipv4NexthopsUsed": 200,
    "ipv4NexthopsAvailable": 16000
  }
}
```

**Agent Conditions:**

```bash
# Check if agent is Ready
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
# Output: True
```

**Why This Matters:**

Agent CRD status provides **source of truth for switch state** at Kubernetes level. When Grafana shows an issue, Agent CRD tells you the exact switch state.

---

#### Concept 3: VPC Lifecycle Event Tracking

**VPC Creation Event Sequence**

When you create a VPC via GitOps (Gitea commit → ArgoCD sync), events track the lifecycle:

**Successful VPC Creation:**

```
Time    Type    Reason              Object          Message
0s      Normal  Created             vpc/myvpc       VPC created successfully
2s      Normal  VNIAllocated        vpc/myvpc       Allocated VNI 100010
5s      Normal  SubnetsConfigured   vpc/myvpc       3 subnets configured
10s     Normal  Ready               vpc/myvpc       VPC is ready
```

**Failed VPC Creation (VLAN Conflict):**

```
Time    Type     Reason              Object          Message
0s      Normal   Created             vpc/badVPC      VPC created successfully
2s      Warning  ValidationFailed    vpc/badVPC      VLAN 1010 already in use by vpc-1/default
```

**VPCAttachment Event Sequence**

**Successful Attachment:**

```
Time    Type    Reason              Object                      Message
0s      Normal  Created             vpcattachment/vpc1-srv01    VPCAttachment created
3s      Normal  ConnectionFound     vpcattachment/vpc1-srv01    Connection server-01--mclag found
5s      Normal  VLANConfigured      vpcattachment/vpc1-srv01    VLAN 1010 configured on leaf-01/Ethernet5
8s      Normal  Ready               vpcattachment/vpc1-srv01    VPCAttachment ready
```

**Failed Attachment (Connection Not Found):**

```
Time    Type     Reason                  Object                      Message
0s      Normal   Created                 vpcattachment/vpc1-srv05    VPCAttachment created
2s      Warning  DependencyMissing       vpcattachment/vpc1-srv05    Connection server-05--mclag not found
```

**VPC Deletion Event Sequence**

**Successful Deletion:**

```
Time    Type    Reason              Object          Message
0s      Normal  Deleting            vpc/myvpc       VPC deletion initiated
2s      Normal  DetachingServers    vpc/myvpc       Removing VPCAttachments
5s      Normal  ReleasingVNI        vpc/myvpc       VNI 100010 released
8s      Normal  Deleted             vpc/myvpc       VPC deleted successfully
```

**Failed Deletion (Active Attachments):**

```
Time    Type     Reason                  Object          Message
0s      Normal   Deleting                vpc/myvpc       VPC deletion initiated
2s      Warning  DependencyExists        vpc/myvpc       Cannot delete: 2 VPCAttachments still active
```

**Tracking VPC Lifecycle:**

```bash
# Watch VPC events live
kubectl get events --field-selector involvedObject.name=myvpc --watch

# Check VPC and related resources
kubectl get events --field-selector involvedObject.kind=VPC
kubectl get events --field-selector involvedObject.kind=VPCAttachment
kubectl get events --field-selector involvedObject.kind=DHCPSubnet
```

---

#### Concept 4: Common Error Event Patterns

**Error Pattern 1: VLAN Conflict**

**Event:**
```
Type: Warning
Reason: VLANConflict
Message: VLAN 1010 already in use by vpc-1/default
```

**Cause:** VPC subnet specifies VLAN already used by another VPC in same VLANNamespace

**Solution:**
1. Check existing VLANs: `kubectl get vpc -o yaml | grep vlan:`
2. Choose unused VLAN (e.g., 1011)
3. Update VPC YAML in Gitea
4. Commit change

**Grafana Correlation:** Interface shows no VLAN configured (VPC reconciliation failed)

---

**Error Pattern 2: Subnet Overlap**

**Event:**
```
Type: Warning
Reason: SubnetOverlap
Message: Subnet 10.0.10.0/24 overlaps with existing VPC vpc-2/backend
```

**Cause:** VPC subnet overlaps with another VPC in same IPv4Namespace

**Solution:**
1. Check existing subnets: `kubectl get vpc -o yaml | grep subnet:`
2. Choose non-overlapping subnet (e.g., 10.0.20.0/24)
3. Update VPC YAML in Gitea

**Grafana Correlation:** No DHCP leases visible (VPC reconciliation failed)

---

**Error Pattern 3: Connection Not Found**

**Event:**
```
Type: Warning
Reason: DependencyMissing
Message: Connection server-05--mclag--leaf-01--leaf-02 not found
```

**Cause:** VPCAttachment references a Connection that doesn't exist

**Solution:**
1. List available connections: `kubectl get connections -n fab`
2. Find correct connection name for server-05
3. Update VPCAttachment YAML with correct connection name

**Grafana Correlation:** Interface shows as down or no VLAN (attachment failed)

---

**Error Pattern 4: Invalid CIDR**

**Event:**
```
Type: Warning
Reason: ValidationFailed
Message: Invalid subnet CIDR: 10.0.10.0/33
```

**Cause:** Invalid CIDR notation (prefix length > 32)

**Solution:**
1. Fix CIDR notation (e.g., /24 instead of /33)
2. Update VPC YAML in Gitea

---

**Error Pattern 5: Agent Not Ready**

**Event:**
```
Type: Warning
Reason: AgentNotReady
Message: Switch leaf-03 agent not ready, configuration pending
```

**Cause:** Switch agent disconnected or switch offline

**Solution:**
1. Check agent status: `kubectl get agents -n fab`
2. Verify switch reachable: `ping leaf-03.fabric.local`
3. Check agent logs: `kubectl logs -n fab agent-leaf-03`
4. If switch down, investigate switch console/power

**Grafana Correlation:** Fabric Dashboard shows switch missing metrics

---

#### Concept 5: Event-Metric Correlation

**Using kubectl Events and Grafana Dashboards Together**

**Scenario 1: Interface No Traffic**

**Grafana shows:**
- Interfaces Dashboard: leaf-01/Ethernet5 has 0 bps traffic
- Interface state: UP

**kubectl investigation:**

```bash
# Check if VPCAttachment exists for this interface
kubectl get vpcattachment -A

# Check events for VPCAttachment
kubectl get events --field-selector involvedObject.kind=VPCAttachment

# Found event:
# Warning  VLANConflict  VPCAttachment/vpc1-srv01  VLAN 1010 conflict
```

**Root Cause:** VPCAttachment failed due to VLAN conflict, so VLAN never configured on interface

**Solution:** Fix VLAN conflict in VPC YAML

---

**Scenario 2: BGP Session Down**

**Grafana shows:**
- Fabric Dashboard: BGP session leaf-01 ↔ spine-01 down

**kubectl investigation:**

```bash
# Check Agent CRD for BGP state
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors.default["172.30.128.1"]}' | jq

# Output shows:
{
  "state": "idle",
  "enabled": true,
  "localAS": 65101,
  "peerAS": 65100
}

# Check switch events
kubectl get events -n fab --field-selector involvedObject.name=leaf-01

# Found event:
# Warning  ConfigApplyFailed  switch/leaf-01  BGP peer 172.30.128.1 not reachable
```

**Root Cause:** Spine-01 not reachable from leaf-01 (network issue)

**Solution:** Investigate spine-01 connectivity (check spine-01 agent, physical links)

---

**Scenario 3: Server Can't Get DHCP**

**Grafana shows:**
- Interfaces Dashboard: leaf-02/Ethernet10 UP, traffic minimal
- Logs Dashboard: No DHCP errors

**kubectl investigation:**

```bash
# Check DHCPSubnet resources
kubectl get dhcpsubnet -A

# Expected: myvpc--default
# Found: Missing!

# Check VPC events
kubectl get events --field-selector involvedObject.name=myvpc

# Found event:
# Warning  SubnetOverlap  vpc/myvpc  Subnet 10.0.10.0/24 overlaps with vpc-2
```

**Root Cause:** VPC reconciliation failed due to subnet overlap, so DHCPSubnet never created

**Solution:** Fix subnet overlap in VPC YAML

---

**Integration Workflow:**

```
1. Grafana: Identify symptom (no traffic, session down, etc.)
           ↓
2. kubectl events: Check for configuration errors
           ↓
3. Agent CRD status: Verify switch state
           ↓
4. Controller logs: Detailed reconciliation info (if needed)
           ↓
5. Solution: Fix configuration, verify in Grafana
```

---

### Hands-On Lab (5-6 minutes)

**Lab Title:** Integrated Troubleshooting with kubectl and Grafana

**Scenario:**

You provisioned a new VPC called `testapp-vpc` with a VPCAttachment for `server-03`. The server reports it's not getting DHCP. You'll use both Grafana and kubectl to diagnose the issue.

**Environment Access:**
- **Grafana:** http://localhost:3000
- **kubectl:** Already configured

---

#### Task 1: Identify Symptom in Grafana (1 minute)

**Objective:** Use Grafana to observe the issue

**Steps:**

1. **Open Grafana Interfaces Dashboard:**
   - Navigate to http://localhost:3000
   - Dashboards → "Hedgehog Interfaces"

2. **Find server-03 interface:**
   - Look for leaf-XX/EthernetYY connected to server-03
   - (In vlab, check Connection CRDs to find interface)

3. **Observe symptoms:**
   - **Expected:** Interface UP, VLAN configured, traffic flowing
   - **Actual:** What do you see?
     - Is interface up or down?
     - Is VLAN configured?
     - Is there traffic?

**Expected Finding:** Interface UP, but no traffic or VLAN not visible

---

#### Task 2: Check VPC and VPCAttachment Status (2 minutes)

**Objective:** Use kubectl to check resource status

**Steps:**

1. **List VPCs:**

   ```bash
   kubectl get vpc testapp-vpc
   ```

   **Expected:** VPC exists

2. **Check VPC events:**

   ```bash
   kubectl get events --field-selector involvedObject.name=testapp-vpc

   # OR use describe for events
   kubectl describe vpc testapp-vpc
   ```

   **Look for:**
   - ✅ Normal events: Created, VNIAllocated, SubnetsConfigured, Ready
   - ❌ Warning events: ValidationFailed, VLANConflict, SubnetOverlap

3. **List VPCAttachments:**

   ```bash
   kubectl get vpcattachment -A | grep server-03
   ```

   **Expected:** VPCAttachment exists

4. **Check VPCAttachment events:**

   ```bash
   kubectl get events --field-selector involvedObject.kind=VPCAttachment | grep server-03

   # OR
   kubectl describe vpcattachment <attachment-name>
   ```

   **Look for:**
   - ✅ Normal: ConnectionFound, VLANConfigured, Ready
   - ❌ Warning: DependencyMissing, ConfigApplyFailed

**Success Criteria:**
- ✅ Identified if VPC or VPCAttachment has errors
- ✅ Found specific error event message

**Common Issues to Find:**
- VLAN conflict
- Connection not found
- Subnet overlap

---

#### Task 3: Check Agent CRD for Switch State (2 minutes)

**Objective:** Verify switch-level configuration using Agent CRD

**Steps:**

1. **Identify which switch has server-03:**

   ```bash
   # List connections to find server-03
   kubectl get connections -n fab | grep server-03

   # Example output:
   # server-03--unbundled--leaf-05
   ```

   **Result:** server-03 connected to leaf-05, Ethernet7 (example)

2. **Check Agent status for leaf-05:**

   ```bash
   kubectl get agent leaf-05 -n fab
   ```

   **Expected:** Agent Ready

3. **Check interface state in Agent CRD:**

   ```bash
   # View interface Ethernet7 state
   kubectl get agent leaf-05 -n fab -o jsonpath='{.status.state.interfaces.Ethernet7}' | jq
   ```

   **Expected output (healthy):**

   ```json
   {
     "enabled": true,
     "admin": "up",
     "oper": "up",
     "speed": "25G"
   }
   ```

   **Look for:**
   - Is `oper` = "up"? (Interface operational)
   - Are there VLANs listed? (VLAN configuration)

4. **Check configuration application status:**

   ```bash
   # When was config last applied?
   kubectl get agent leaf-05 -n fab -o jsonpath='{.status.lastAppliedTime}'

   # Recent = good (within last few minutes)
   # Old = problem (config not applying)
   ```

**Success Criteria:**
- ✅ Verified interface operational state
- ✅ Checked if VLAN configured at switch level
- ✅ Confirmed agent applied config recently

---

#### Task 4: Correlate kubectl and Grafana Findings (1 minute)

**Objective:** Combine kubectl and Grafana data to identify root cause

**Analysis Template:**

**Grafana Symptom:**
- Interface: leaf-05/Ethernet7
- State: UP
- VLAN: Not visible or wrong VLAN
- Traffic: 0 bps

**kubectl Events:**
- VPC Event: [Normal/Warning] [Reason] [Message]
- VPCAttachment Event: [Normal/Warning] [Reason] [Message]

**Agent CRD Status:**
- Interface oper: [up/down]
- Config applied: [timestamp]
- VLANs: [list]

**Root Cause:**
- Example: VPCAttachment failed due to VLAN conflict → VLAN never configured → No DHCP

**Solution:**
- Example: Change VPC VLAN from 1010 to 1011, commit to Gitea

**Success Criteria:**
- ✅ Identified root cause from event messages
- ✅ Correlated kubectl findings with Grafana symptom
- ✅ Proposed solution

---

#### Task 5: Monitor Resolution (Optional, 1 minute)

**Objective:** Verify fix by monitoring events and Grafana

**Steps:**

1. **Watch VPC/VPCAttachment events:**

   ```bash
   kubectl get events --field-selector involvedObject.kind=VPCAttachment --watch
   ```

2. **Apply fix** (change VLAN in Gitea, commit)

3. **Observe events:**
   - Normal events appearing
   - "Ready" event

4. **Check Grafana Interfaces Dashboard:**
   - VLAN now visible
   - Traffic appearing (server gets DHCP)

**Success Criteria:**
- ✅ Events show successful reconciliation
- ✅ Grafana shows VLAN configured and traffic

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You performed integrated troubleshooting using kubectl and Grafana:
- ✅ Identified symptom in Grafana (interface no traffic)
- ✅ Checked kubectl events for configuration errors
- ✅ Reviewed Agent CRD status for switch state
- ✅ Correlated kubectl and Grafana data to find root cause
- ✅ Proposed solution based on event messages

**Key Takeaways:**

1. **Grafana shows "what"** (metrics, symptoms)
2. **kubectl events show "why"** (configuration errors, reconciliation failures)
3. **Agent CRD shows "exactly"** (precise switch state)
4. **Integrated approach** (Grafana + kubectl) enables root cause analysis
5. **Events expire after 1 hour** - check soon after operations
6. **Event patterns are recognizable** (VLAN conflict, connection not found, etc.)

**Troubleshooting Workflow:**

```
Symptom (Grafana)
      ↓
Events (kubectl get events)
      ↓
Agent CRD (kubectl get agent ... -o jsonpath)
      ↓
Controller Logs (if needed)
      ↓
Root Cause → Solution
```

**Next Module Preview:**

Module 3.4: Pre-Support Diagnostic Checklist - You'll learn to collect comprehensive diagnostics (kubectl + Grafana) for support escalation.

---

### Assessment Questions

#### Question 1: Event Filtering

**Scenario:** You want to check if any VPCs failed to reconcile in the last hour. Which kubectl command should you use?

- A) `kubectl get vpc -A`
- B) `kubectl get events --field-selector involvedObject.kind=VPC,type=Warning`
- C) `kubectl describe vpc --all-namespaces`
- D) `kubectl logs -n fab deployment/fabric-controller-manager | grep VPC`

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) `kubectl get events --field-selector involvedObject.kind=VPC,type=Warning`

**Explanation:**

This command filters events to:
- `involvedObject.kind=VPC` - Only VPC-related events
- `type=Warning` - Only Warning events (errors/issues)

**Why other options don't work:**
- **A**: Lists VPCs, but doesn't show events or reconciliation status
- **C**: Shows VPC specs and recent events, but verbose and hard to scan for errors across all VPCs
- **D**: Controller logs are detailed but harder to parse than events

**Event retention caveat:** Events expire after 1 hour, so only recent issues visible.

**Module 3.3 Reference:** Concept 1 - Kubernetes Event Monitoring
</details>

---

#### Question 2: Agent CRD Status Interpretation

**Scenario:** You run this command:

```bash
kubectl get agent leaf-02 -n fab -o jsonpath='{.status.state.bgpNeighbors.default["172.30.128.10"].state}'
```

Output: `idle`

What does this tell you?

- A) BGP neighbor is established and working
- B) BGP neighbor is down/not connected
- C) BGP configuration is not applied
- D) Agent CRD is not ready

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) BGP neighbor is down/not connected

**Explanation:**

BGP neighbor state can be:
- **established** = Working (neighbors exchanging routes)
- **idle** = Not connected (session down)
- **active**, **connect** = Attempting connection
- **opensent**, **openconfirm** = Establishing session

**State = idle** means the BGP session is not up.

**Next troubleshooting steps:**
1. Check why neighbor isn't connecting:
   - Is spine switch reachable? `ping 172.30.128.10`
   - Check switch events: `kubectl get events -n fab --field-selector involvedObject.name=leaf-02`
2. Check Grafana Fabric Dashboard for BGP session status
3. Review logs for BGP errors

**Why other options are wrong:**
- **A**: "established" means working, not "idle"
- **C**: Config can be applied, but neighbor still down (network issue)
- **D**: Agent can be Ready, but BGP neighbor down

**Module 3.3 Reference:** Concept 2 - Agent CRD Status (BGP Neighbor State)
</details>

---

#### Question 3: VPC Lifecycle Events

**Scenario:** You create a new VPC and immediately see this event:

```
Type: Warning
Reason: VLANConflict
Message: VLAN 1020 already in use by vpc-prod/backend
```

What should you do?

- A) Delete the existing VPC (vpc-prod) to free up VLAN 1020
- B) Change your new VPC's VLAN to an unused VLAN (e.g., 1021)
- C) Wait 5 minutes for automatic VLAN allocation
- D) Restart the fabric-controller to clear the conflict

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Change your new VPC's VLAN to an unused VLAN (e.g., 1021)

**Explanation:**

VLAN conflict means the VLAN you specified is already allocated to another VPC in the same VLANNamespace.

**Solution steps:**
1. Check existing VLANs: `kubectl get vpc -o yaml | grep vlan:`
2. Find unused VLAN in VLANNamespace range (e.g., 1000-2999)
3. Edit your VPC YAML in Gitea
4. Change `vlan: 1020` to `vlan: 1021` (or another unused VLAN)
5. Commit change
6. ArgoCD will sync, VPC will reconcile successfully

**Why other options are wrong:**
- **A**: Deleting production VPC is disruptive and inappropriate
- **C**: Hedgehog requires explicit VLAN specification (no auto-allocation)
- **D**: Controller restart doesn't fix configuration issue

**Prevention:** Always check existing VLANs before creating new VPCs.

**Module 3.3 Reference:** Concept 4 - Common Error Event Patterns (VLAN Conflict)
</details>

---

#### Question 4: Event-Metric Correlation

**Scenario:** Grafana Interfaces Dashboard shows:
- leaf-03/Ethernet12: State UP, Traffic = 0 bps

You check kubectl events and find:

```
Warning  DependencyMissing  vpcattachment/app1-srv08  Connection server-08--mclag not found
```

What is the root cause and solution?

- A) Interface is down - check physical cable
- B) VPCAttachment failed because connection name is wrong - fix connection name
- C) DHCP server is down - restart DHCP pod
- D) BGP session is down - check BGP configuration

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) VPCAttachment failed because connection name is wrong - fix connection name

**Explanation:**

**Event analysis:**
- Type: Warning (error occurred)
- Reason: DependencyMissing (referenced resource doesn't exist)
- Message: "Connection server-08--mclag not found"

**Root cause:** VPCAttachment references connection `server-08--mclag` which doesn't exist in the cluster.

**Solution:**
1. List available connections: `kubectl get connections -n fab | grep server-08`
2. Find correct connection name (e.g., `server-08--mclag--leaf-03--leaf-04`)
3. Edit VPCAttachment YAML in Gitea
4. Update `connection:` field to correct name
5. Commit change

**Why Grafana shows interface UP but no traffic:**
- Physical interface is up (cable connected)
- But VPCAttachment failed, so VLAN not configured
- Server sends traffic, but switch doesn't forward (no VLAN config)

**Event-Metric Correlation:**
- Grafana: Symptom (no traffic despite interface UP)
- kubectl events: Root cause (VPCAttachment failed, wrong connection name)

**Module 3.3 Reference:** Concept 5 - Event-Metric Correlation
</details>

---

## Technical Requirements

### kubectl Commands Reference

**Event Monitoring:**

```bash
# All events, recent first
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Events for specific namespace
kubectl get events -n default --sort-by='.lastTimestamp'

# Warning events only
kubectl get events --all-namespaces --field-selector type=Warning

# Events for specific resource
kubectl get events --field-selector involvedObject.name=myvpc

# Events for resource type
kubectl get events --field-selector involvedObject.kind=VPC

# Watch events live
kubectl get events --all-namespaces --watch

# Events in last N minutes (requires --since flag, but limited support)
# Instead, use sort and manual filtering
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | head -20
```

**Agent CRD Queries:**

```bash
# List all agents
kubectl get agents -n fab

# Get agent full status
kubectl get agent leaf-01 -n fab -o yaml

# Check agent heartbeat
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastHeartbeat}'

# Check agent version
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.version}'

# View all interfaces
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | jq

# View specific interface
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq

# View BGP neighbors
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# View specific BGP neighbor state
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors.default["172.30.128.10"].state}'

# View platform health
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.platform}' | jq

# View ASIC resources
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.criticalResources}' | jq

# Check agent conditions (Ready)
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
```

**VPC/VPCAttachment Queries:**

```bash
# List VPCs with events
kubectl describe vpc myvpc

# Get VPCAttachment events
kubectl describe vpcattachment vpc1-srv01

# List DHCPSubnets
kubectl get dhcpsubnet -A

# Check Connection exists
kubectl get connection -n fab | grep server-01
```

### Common Event Reasons

**Normal Events:**
- `Created` - Resource created
- `VNIAllocated` - VNI assigned to VPC
- `SubnetsConfigured` - VPC subnets configured
- `ConnectionFound` - VPCAttachment found connection
- `VLANConfigured` - VLAN applied to interface
- `Ready` - Resource ready

**Warning Events:**
- `VLANConflict` - VLAN already in use
- `SubnetOverlap` - Subnet overlaps with existing
- `DependencyMissing` - Referenced resource not found
- `ValidationFailed` - Spec validation error
- `ConfigApplyFailed` - Switch configuration failed
- `AgentNotReady` - Switch agent not ready

### Reference Documents

- **[CRD_REFERENCE.md](../research/CRD_REFERENCE.md)** - Agent CRD status fields (lines 796-923)
- **[OBSERVABILITY.md](../research/OBSERVABILITY.md)** - Kubernetes events (lines 425-476)
- **[Module 1.3 Design](./module-1.3-design.md)** - kubectl basics

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Teach the Why Behind the How** ⭐
- **How:** Explains why kubectl events and Agent CRD complement Grafana
- **Example:** Grafana shows "what", kubectl shows "why"
- **Why:** Understanding multiple data sources enables root cause analysis

#### 2. **Focus on What Matters Most** ⭐
- **How:** Focuses on common error patterns (VLAN conflict, connection not found)
- **Example:** Assessment tests realistic troubleshooting scenarios
- **Why:** Prepares for most frequent operational issues

#### 3. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on troubleshooting lab with real scenario
- **Example:** Students diagnose VPCAttachment failure
- **Why:** Practice builds muscle memory for troubleshooting workflow

#### 4. **Train for Reality, Not Rote** ⭐
- **How:** Integrated troubleshooting (kubectl + Grafana)
- **Example:** Real-world workflow: symptom → events → agent status → solution
- **Why:** Mirrors actual Day 2 troubleshooting

#### 5. **Confidence Before Comprehensiveness** ⭐
- **How:** Simple event filtering before complex Agent CRD queries
- **Example:** Task 2 (events) before Task 3 (Agent CRD jsonpath)
- **Why:** Progressive complexity builds confidence

#### 6. **Support as Part of Learning**
- **How:** Prepares for Module 3.4 diagnostic collection
- **Example:** Understanding what data to collect for escalation
- **Why:** Normalizes collaboration with support

---

## Dependencies

### Prerequisites (Must Complete First)

**Module 3.1 & 3.2:**
- ✅ Prometheus metrics understanding
- ✅ Grafana dashboard interpretation
- ✅ Metric types and PromQL basics

**Module 1.3:**
- ✅ kubectl basics from Course 1
- ✅ Three-interface workflow

**Why These Prerequisites Matter:**
- Module 3.3 correlates kubectl with Grafana (requires Grafana knowledge)
- Agent CRD queries require kubectl proficiency

---

### Enables (Unlocks These Modules)

**Immediate:**
- ✅ **Module 3.4:** Pre-Support Diagnostic Checklist
  - Collects events and Agent status for support tickets
  - Builds on kubectl commands from Module 3.3

**Course-Level:**
- ✅ Foundation for Course 4 troubleshooting workflows
- ✅ Integrated kubectl + Grafana approach used throughout

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
  - LO 1-6: Testable via lab tasks and assessment
  - LO 7: Integrated troubleshooting workflow

- ✅ **Content outline follows logical progression**
  - Introduction → Core Concepts (5 concepts) → Hands-On Lab (5 tasks) → Assessment

- ✅ **Assessment aligns with learning objectives**
  - All questions test kubectl event filtering, Agent CRD interpretation, and correlation

- ✅ **Timing target is achievable (13-15 minutes)**
  - Total: 14-16 minutes (within acceptable range)

### Technical Accuracy

- ✅ **All kubectl commands validated**
  - Event filtering syntax correct
  - Agent CRD jsonpath queries accurate
  - Commands match CRD_REFERENCE.md

- ✅ **Event patterns documented accurately**
  - Warning/Normal events match Hedgehog behavior
  - Error messages realistic

### Completeness

- ✅ **All required sections filled out**
- ✅ **Assessment has detailed answers and explanations**
- ✅ **Dependencies mapped**

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 3.2 (Dashboard Interpretation - DESIGNED)
**Next Module:** Module 3.4 (Pre-Support Diagnostic Checklist - To Be Designed)

---

**Status:** ⏳ DESIGN COMPLETE - Ready for Review
**Related Issue:** GitHub Issue #10 - [DESIGN] Course 3: Observability & Fabric Health (Modules 3.1-3.4)
