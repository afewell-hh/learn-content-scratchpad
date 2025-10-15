# Module 1.1 Design: Welcome & Mindset

## Module Metadata

- **Module Number:** 1.1
- **Module Title:** Welcome to Fabric Operations
- **Course:** Course 1 - Foundations & Interfaces
- **Estimated Duration:** 15 minutes
  - Introduction: 3 minutes
  - Core Concepts: 5 minutes
  - Hands-On Lab: 5 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Prerequisites:**
  - Basic command-line familiarity
  - Understanding of what a network does (general concept)
  - No prior Hedgehog or data center networking experience required

## Learning Objectives

By the end of this module, learners will be able to:

1. **Describe the Hedgehog Fabric Operator role** - Understand day-to-day responsibilities vs. design/architecture tasks
2. **Articulate the confidence-first learning approach** - Explain why we focus on common operations before edge cases
3. **Navigate the Hedgehog vlab environment** - Use kubectl to view fabric topology and resources
4. **Identify the three-tier resource hierarchy** - Recognize Switches/Servers (wiring) → VPCs → VPCAttachments
5. **Demonstrate kubectl basics** - Execute get and describe commands to inspect fabric state

## Content Outline

### Introduction (3 minutes)

**Hook: A Day in the Life**

> You're a Fabric Operator at a growing company. This morning, your team needs to onboard three new application servers to the production network. In traditional networking, this might mean: scheduling a maintenance window, manually configuring VLANs on multiple switches, updating routing tables, testing connectivity, and hoping nothing breaks.
>
> With Hedgehog Fabric, you'll define the desired state in a few lines of YAML, apply it with kubectl, and let the fabric controller handle the rest. The switches configure themselves. The routes populate automatically. The connectivity validates itself.
>
> Sound too good to be true? That's what hyperscalers figured out years ago. Now you can do it too.

**Context: What You'll Learn**

This pathway teaches you to **operate** a Hedgehog fabric with confidence. Not to design fabrics from scratch (that's a different course), but to:

- Provision network resources (VPCs, server attachments)
- Validate connectivity
- Monitor fabric health
- Troubleshoot common issues
- Know when to escalate to support

You'll use **Kubernetes-native tools** (kubectl) to manage a real network fabric. If you know Kubernetes, you'll feel at home. If you come from traditional networking, you'll discover a cleaner, safer way to manage network state.

**Setting Expectations:**

- **Focus on what matters most:** We'll teach the 80% of tasks you'll do daily, not the 20% of edge cases you'll rarely encounter
- **Confidence before comprehensiveness:** You'll master core operations before diving into advanced scenarios
- **Hands-on, always:** Every module includes labs in a real Hedgehog environment
- **Support is strength:** You'll learn when and how to escalate effectively (it's a best practice, not a weakness)

### Core Concepts (5 minutes)

**Concept 1: The Fabric Operator Role**

You're not designing the network from scratch—that's already done. The fabric (switches, servers, connections) is wired and operational.

**Your responsibilities:**
- **Provision:** Create VPCs, attach servers to networks
- **Validate:** Confirm connectivity works as expected
- **Monitor:** Watch fabric health, detect anomalies
- **Troubleshoot:** Diagnose issues using kubectl and events
- **Escalate:** Package diagnostics and engage support when needed

**NOT your job (in this role):**
- Physical switch installation
- Initial fabric wiring diagram design
- Low-level BGP/EVPN configuration
- Hardware troubleshooting

Think of it like this: **You manage applications in Kubernetes without configuring kubelet on every node.** Similarly, you manage network resources without manually configuring SONiC on every switch.

**Concept 2: Kubernetes-Native Network Management**

Hedgehog uses **Custom Resource Definitions (CRDs)** to represent network concepts:

```
Wiring Layer (Physical)
├─ Switch: Physical switch in the fabric
├─ Server: Physical server connected to switches
└─ Connection: How servers connect to switches (MCLAG, ESLAG, etc.)

VPC Layer (Virtual Networks)
├─ VPC: Virtual Private Cloud with isolated subnets
├─ VPCAttachment: Binds a VPC to a server connection
└─ VPCPeering: Connects two VPCs

External Layer (Outside Connectivity)
├─ External: External network definition
├─ ExternalAttachment: Border leaf connection
└─ ExternalPeering: VPC to external routing
```

You'll use **kubectl** just like managing Kubernetes resources:
```bash
kubectl get vpcs                    # List all VPCs
kubectl describe vpc my-vpc         # Get VPC details
kubectl apply -f vpc.yaml           # Create/update VPC
kubectl get events                  # See what happened
```

**Concept 3: Declarative, Self-Healing Infrastructure**

Traditional networking: Log into each switch, type commands, hope you didn't make a typo.

Hedgehog approach:
1. **Declare desired state** (YAML): "I want VPC 'production' with subnet 10.10.1.0/24"
2. **Apply it** (kubectl): The fabric controller receives your intent
3. **Reconciliation happens** (automatic): Controllers configure switches to match desired state
4. **Status reflects reality** (observable): kubectl shows current state and any issues

If a switch reboots? The fabric controller automatically reapplies configuration. No manual intervention.

**Concept 4: The Learning Philosophy**

This pathway follows a proven approach:

1. **Train for reality, not rote:** You'll learn workflows, not memorize commands
2. **Focus on what matters:** Common operations, not rare edge cases
3. **Confidence first:** Small wins build competence over time
4. **Learn by doing:** Hands-on labs in every module
5. **Support as strength:** We'll teach you how to escalate effectively

You don't need to become a networking expert or Kubernetes guru overnight. You need to be **confident and competent** in the core operations. Everything else builds from there.

### Hands-On Lab (5 minutes)

**Lab Title:** Explore Your Fabric Environment

**Overview:**
You'll use kubectl to explore the Hedgehog vlab environment, viewing switches, servers, and connections. This is your first hands-on experience—designed to build confidence through successful exploration.

**Environment:**
- Hedgehog vlab with default spine-leaf topology
- 2 spine switches, 5 leaf switches, 10 servers
- kubectl already configured and ready to use

---

#### Task 1: Verify Environment Access

**Objective:** Confirm kubectl can communicate with the Hedgehog control plane

**Steps:**
```bash
# Check cluster access
kubectl cluster-info

# Expected output (similar to):
# Kubernetes control plane is running at https://...
# CoreDNS is running at https://...
```

**Validation:**
```bash
# Verify you can access the fab namespace
kubectl get pods -n fab

# Expected: Several pods running (fabric-controller-manager, fabric-proxy, etc.)
```

**Success Criteria:**
- ✅ kubectl cluster-info shows control plane URL
- ✅ Pods in fab namespace are Running

---

#### Task 2: View Fabric Topology

**Objective:** Explore the physical fabric resources (switches, servers, connections)

**Steps:**
```bash
# List all switches in the fabric
kubectl get switches

# Expected output (7 switches):
# NAME       PROFILE   ROLE          DESCR           GROUPS        AGE
# leaf-01    vs        server-leaf   VS-01 MCLAG 1   ["mclag-1"]   Xh
# leaf-02    vs        server-leaf   VS-02 MCLAG 1   ["mclag-1"]   Xh
# leaf-03    vs        server-leaf   VS-03 ESLAG 1   ["eslag-1"]   Xh
# leaf-04    vs        server-leaf   VS-04 ESLAG 1   ["eslag-1"]   Xh
# leaf-05    vs        server-leaf   VS-05                         Xh
# spine-01   vs        spine         VS-06                         Xh
# spine-02   vs        spine         VS-07                         Xh
```

**Count the switches:**
- How many spine switches? _______ (Answer: 2)
- How many leaf switches? _______ (Answer: 5)

```bash
# List all servers
kubectl get servers

# Expected output (10 servers):
# NAME        TYPE   DESCR
# server-01          S-01 MCLAG leaf-01 leaf-02
# server-02          S-02 MCLAG leaf-01 leaf-02
# ... (10 total)
```

```bash
# View connections (how servers connect to switches)
kubectl get connections

# Expected output shows various connection types:
# - MCLAG (multi-chassis link aggregation)
# - ESLAG (EVPN-based ESI LAG)
# - Bundled (port channel to single switch)
# - Unbundled (single link)
# - Fabric (spine-to-leaf interconnects)
```

**Success Criteria:**
- ✅ Can list switches and count them correctly
- ✅ Can list servers (10 total)
- ✅ Can view connections showing various types

---

#### Task 3: Inspect a Specific Resource

**Objective:** Use kubectl describe to view detailed information

**Steps:**
```bash
# Get detailed information about leaf-01
kubectl describe switch leaf-01

# The output contains many fields. Focus on these key ones:
# - Role: Shows the switch's function (server-leaf or spine)
# - Groups: Shows redundancy groups (e.g., mclag-1 means paired with leaf-02)
# - Redundancy: Shows the redundancy type (mclag, eslag, or none)
# - ASN: The switch's BGP autonomous system number
# - Profile: vs (virtual switch in this lab)
#
# You don't need to understand every field - we'll explore more in later modules.
```

**Questions to answer:**
1. What is the role of leaf-01? _______ (Answer: server-leaf)
2. Is leaf-01 part of a redundancy group? _______ (Answer: Yes, mclag-1)

```bash
# Inspect server-01
kubectl describe server server-01

# Look for:
# - Description: Shows which switches it connects to
# - Connection type (MCLAG, ESLAG, etc.)
```

**Success Criteria:**
- ✅ Can describe a switch and identify its role
- ✅ Can describe a server and see its connections
- ✅ Understand that describe shows more detail than get

---

#### Task 4: Explore Events (Optional, if time permits)

**Objective:** See Kubernetes events for fabric resources

**Steps:**
```bash
# View recent events in the default namespace
kubectl get events --sort-by='.lastTimestamp' | tail -20

# Events show what the fabric controller is doing:
# - Reconciling resources
# - Configuration changes
# - Status updates
```

**Observation:**
Events are how Hedgehog communicates what's happening. You'll use events extensively for troubleshooting later in the pathway.

**Note on Events:**
> If you don't see recent events, that's normal! Kubernetes events expire after about an hour. In active environments (like when creating VPCs or troubleshooting issues), you'll see events showing what the fabric controller is doing. We'll use events extensively in later modules during hands-on provisioning tasks.

**Success Criteria:**
- ✅ Can view events (even if the list is empty or you don't understand all of them yet)

---

### Lab Summary

**What you did:**
- ✅ Verified kubectl access to Hedgehog control plane
- ✅ Explored fabric topology (switches, servers, connections)
- ✅ Inspected detailed resource information with describe
- ✅ Observed Kubernetes events

**What you learned:**
- kubectl is your primary tool for fabric management
- Hedgehog represents infrastructure as Kubernetes resources
- The fabric has physical resources (switches, servers) and virtual resources (VPCs, which you'll explore next)
- You can inspect state without making any changes (read-only exploration builds confidence)

**Key takeaway:** You just successfully navigated a production-like fabric environment using only kubectl. No switch CLI, no manual configuration, no fear of breaking things. This is how hyperscalers operate at scale.

---

### Wrap-Up & Assessment (2 minutes)

**Key Takeaways:**

1. **Your role:** Fabric Operator managing day-to-day network operations
2. **Your tool:** kubectl to manage Kubernetes-native network resources
3. **Your approach:** Declarative, confidence-building, focused on common tasks
4. **Your support:** Escalation is a strength, not a weakness

**Preview of Module 1.2:**

Next, you'll dive deeper into **how Hedgehog works under the hood**: CRD reconciliation, the controller pattern, and how your kubectl commands become switch configurations. You'll understand the "magic" so it's not magic anymore—just well-designed automation.

---

## Assessment Design

### Quiz Questions

**Question 1:** (Multiple Choice)

What is the primary role of a Hedgehog Fabric Operator?

- A) Design network topologies and select switch hardware
- B) Provision VPCs, validate connectivity, and monitor fabric health
- C) Manually configure BGP peering on each switch
- D) Write custom Kubernetes controllers for network automation

**Correct Answer:** B

**Explanation:**
Fabric Operators manage day-to-day network operations using kubectl to provision resources (VPCs, attachments), validate connectivity, and monitor health. Physical design (A), low-level protocol configuration (C), and controller development (D) are outside the operator role scope. This pathway focuses on **operating** an existing fabric, not designing or implementing it.

---

**Question 2:** (Scenario-Based)

You need to view all switches in the fabric and identify which ones are spines vs. leaves. What kubectl command would you use?

**Answer:**
```bash
kubectl get switches
```

This lists all switches with their ROLE field showing "spine" or "server-leaf".

**Rubric:**
- Full credit: `kubectl get switches` (exact or close variation)
- Partial credit: Mentions kubectl and switches but wrong syntax
- No credit: Suggests logging into switches or using non-kubectl tools

---

**Question 3:** (True/False)

True or False: In Hedgehog, you must manually log into each switch to configure VLANs when creating a new VPC.

**Answer:** False

**Explanation:**
Hedgehog uses declarative management. You define the VPC in YAML and apply it with kubectl. The fabric controller automatically configures all necessary switches to realize the desired state. You never manually configure switches for routine operations—that's the whole point of the abstraction.

---

**Question 4:** (Multiple Choice)

According to the Hedgehog learning philosophy, which statement is correct?

- A) You must master all edge cases before attempting basic operations
- B) Focus on common, high-impact tasks to build confidence and immediate productivity
- C) Avoid using support—figure everything out independently
- D) Memorize all kubectl commands before trying labs

**Correct Answer:** B

**Explanation:**
The learning philosophy emphasizes "Focus on What Matters Most" and "Confidence Before Comprehensiveness." You'll learn the 80% of operations you'll do daily (B), not rare edge cases (A). Support is encouraged when needed (C is wrong). Learning by doing > memorization (D is wrong).

---

**Question 5:** (Practical - Open Ended)

Based on what you explored in the lab, how many total switches are in the vlab environment, and how are they split between spines and leaves?

**Answer:**
7 total switches: 2 spines and 5 leaves

**Rubric:**
- Full credit: Correct numbers (7 total, 2 spines, 5 leaves)
- Partial credit: Correct total but wrong breakdown, or vice versa
- No credit: Incorrect numbers

---

### Practical Assessment

**Task:** Using only kubectl commands, determine which switches server-05 is connected to.

**Success Criteria:**
- ✅ Uses `kubectl describe server server-05` or equivalent
- ✅ Correctly identifies the connected switches from the description
- ✅ Can explain whether it's MCLAG, ESLAG, or another connection type

**Expected Process:**
```bash
kubectl describe server server-05
# Output shows: "S-05 ESLAG leaf-03 leaf-04"
# Answer: Connected to leaf-03 and leaf-04 via ESLAG
```

---

## Technical Requirements

### Hedgehog CRDs Used

- ✅ **Switch** - [Reference](../research/CRD_REFERENCE.md#switch)
  - View switches with `kubectl get switches`
  - Inspect details with `kubectl describe switch <name>`

- ✅ **Server** - [Reference](../research/CRD_REFERENCE.md#server)
  - View servers with `kubectl get servers`
  - See connection types in descriptions

- ✅ **Connection** - [Reference](../research/CRD_REFERENCE.md#connection)
  - View all connections with `kubectl get connections`
  - Shows MCLAG, ESLAG, bundled, unbundled, fabric types

### kubectl Commands

**Primary commands taught:**
```bash
# Cluster access
kubectl cluster-info
kubectl get pods -n fab

# Resource listing
kubectl get switches
kubectl get servers
kubectl get connections

# Resource inspection
kubectl describe switch <name>
kubectl describe server <name>

# Event viewing
kubectl get events --sort-by='.lastTimestamp'
```

**Reference:** All commands are read-only (get, describe) - no modifications in this module

### Other Tools

- ✅ Text editor (for viewing command outputs, optional)
- ✅ Terminal with kubectl access

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

- ✅ **Train for Reality, Not Rote**
  - Day-in-life scenario sets real-world context
  - Focus on actual operator responsibilities vs. theoretical knowledge

- ✅ **Focus on What Matters Most**
  - Read-only exploration (can't break anything)
  - Core kubectl commands used daily
  - Physical topology understanding (foundation for all future work)

- ✅ **Confidence Before Comprehensiveness**
  - **CRITICAL for Module 1.1**
  - First hands-on is exploration, not creation (lower stakes)
  - Every command guaranteed to work
  - Success criteria achievable by all learners
  - Building "I can do this" feeling

- ✅ **Abstraction as Empowerment**
  - Introduction to kubectl as powerful tool
  - Don't need to know SONiC CLI to see fabric state
  - Kubernetes abstractions make network management accessible

- ✅ **Learn by Doing, Not Watching**
  - 5 minutes of hands-on exploration
  - Active discovery rather than passive reading

- ✅ **Teach the Why Behind the How**
  - Explain operator role rationale (why focus here vs. design)
  - Explain learning philosophy rationale (why confidence-first)

- ✅ **Bridging Two Worlds**
  - Kubernetes users: "It's just CRDs and kubectl"
  - Network engineers: "It's like network management, but declarative"

- ✅ **Support as Part of Learning**
  - Explicitly state escalation is strength, not weakness
  - Set expectation that support will be taught later

### Target Audience Considerations

**For Cloud-Native Learners:**
- **Assume:** Kubernetes familiarity (pods, kubectl, CRDs)
- **Need to explain:** What switches/servers/connections are physically
- **Bridge:** "CRDs for network resources, just like you use for apps"

**For Networking Professionals:**
- **Assume:** Understanding of network concepts (VLANs, routing, fabrics)
- **Need to explain:** Kubernetes concepts (CRDs, controllers, declarative state)
- **Bridge:** "Like network management, but you declare intent vs. configure manually"

**Module 1.1 Specific:**
- Keep technical depth low
- Focus on navigation and comfort, not mastery
- Both audiences can successfully list switches—no deep knowledge required
- Success = feeling capable and curious, not expert

### Common Challenges

**Challenge 1: "This seems too simple"**
- **Stumbling Block:** Experienced learners may feel Module 1.1 is beneath them
- **Mitigation:**
  - Acknowledge in content: "This may feel basic—that's intentional"
  - Frame as foundation: "You need to walk this topology before running complex workflows"
  - Emphasize: "Module 1.2+ will deepen quickly"

**Challenge 2: "I don't understand what I'm looking at"**
- **Stumbling Block:** New learners may not grasp switch roles or connection types yet
- **Mitigation:**
  - Say explicitly: "You don't need to understand all details yet"
  - Focus on: "Can you run the commands and see the output? That's success for now."
  - Defer deep understanding to Module 1.2

**Challenge 3: "What if I break something?"**
- **Stumbling Block:** Fear of making mistakes in live environment
- **Mitigation:**
  - All commands are read-only in Module 1.1
  - State explicitly: "You can't break anything—we're just looking"
  - Build confidence through safe exploration

**Challenge 4: kubectl output is overwhelming**
- **Stumbling Block:** Too much information in describe output
- **Mitigation:**
  - Point learners to specific fields: "Look for Role:"
  - Say: "You don't need to understand every field"
  - Focus on key information only

### Confidence-Building Opportunities

**Win 1:** kubectl cluster-info works on first try
- **Moment:** Task 1, first command
- **Feeling:** "I can access the environment successfully"

**Win 2:** Counting switches and getting the right answer
- **Moment:** Task 2, simple counting exercise
- **Feeling:** "I can navigate and understand the output"

**Win 3:** Using describe to find specific information
- **Moment:** Task 3, finding switch role and groups
- **Feeling:** "I can dig deeper and find what I need"

**Win 4:** Completing entire lab without errors
- **Moment:** Lab completion
- **Feeling:** "I successfully explored a real fabric—I can do this!"

**Pedagogical Note:**
Module 1.1 is designed for **four confidence wins in 5 minutes**. These early successes are psychologically critical for learner persistence through the pathway.

---

## Dependencies

### Prerequisites (Must Complete First)
- ✅ None - This is Module 1.1 (pathway entry point)

### Enables (Unlocks These Modules)
- ✅ Module 1.2 - Architecture & Control Model (builds on topology understanding)
- ✅ Module 1.3 - Interfaces: kubectl/YAML (deepens kubectl skills)
- ✅ Module 1.4 - Recap & Forward Map (requires Course 1 context)

### Related Modules (Complementary, Not Required)
- None (this is the entry point)

---

## Quality Checklist

### Design Quality
- ✅ Learning objectives are specific and measurable
- ✅ Content outline follows logical progression (hook → concepts → hands-on → wrap-up)
- ✅ Lab exercise has clear success criteria
- ✅ Assessment aligns with learning objectives
- ✅ 15-minute timing target is achievable (3+5+5+2)

### Technical Accuracy
- ⏳ All CRD examples validated in vlab (PENDING - dev agent validation)
- ⏳ kubectl commands tested and correct (PENDING - dev agent validation)
- ✅ Workflows match CRD_REFERENCE.md and WORKFLOWS.md (read-only exploration)
- ✅ No technical errors or outdated information (design phase)

### Learning Philosophy
- ✅ Embodies at least 3 core principles (embodies 8 of 10!)
- ✅ Focuses on high-impact, common tasks (kubectl basics, topology navigation)
- ✅ Builds confidence through achievable goals (four confidence wins)
- ✅ Hands-on > passive learning (5 min hands-on lab)
- ✅ Explains "why" not just "how" (operator role rationale, learning philosophy)

### Accessibility
- ✅ Clear to cloud-native learners (Kubernetes concepts explained minimally)
- ✅ Clear to networking professionals (network concepts used as bridge)
- ✅ Jargon explained or avoided (CRD, kubectl, reconciliation defined)
- ✅ Examples are relevant to both audiences (kubectl is universal)

### Completeness
- ✅ All required sections filled out
- ✅ Lab exercise is detailed and testable
- ✅ Assessment has answers and explanations
- ✅ Dependencies mapped
- ✅ Troubleshooting hints provided (in pedagogical section)

---

## Review & Approval

### Design Review
- ✅ Course lead self-review (Claude) - COMPLETE
- ✅ Technical validation (Dev agent) - COMPLETE
- ✅ Timing test (actual 15min run-through) - COMPLETE (4-6 min estimate)
- ✅ Validation feedback incorporated - COMPLETE

### Approval Status
- ✅ **APPROVED FOR CONTENT DEVELOPMENT**
- ✅ All validation criteria passed
- ✅ Timing target met (4-6 minutes learner estimate)
- ✅ Confidence-building objectives achieved
- ✅ Minor recommendations incorporated

---

## Notes & Iterations

**Design Decisions:**

1. **Read-only lab:** Deliberate choice to eliminate fear of breaking things. Confidence-building through safe exploration.

2. **No VPCs in Module 1.1:** Defer VPC operations to Module 2.1. This module establishes foundation (topology, kubectl basics) only.

3. **Four confidence wins:** Structured lab for psychological success. Each task builds on previous success.

4. **Explicit permission to not understand everything:** Reduces cognitive load for newcomers. "It's okay to not know everything yet."

5. **Role definition up front:** Sets realistic expectations. Not training architects or engineers—training operators.

**Anticipated Iteration Needs:**

- [ ] Dev agent may find kubectl outputs differ from design—will adjust examples
- [ ] Timing may need refinement based on actual run-through
- [ ] Assessment questions may need adjustment for clarity
- [ ] May need to add troubleshooting hints if validation reveals issues

**Validation History:**

1. ✅ Design completed (2025-10-15)
2. ✅ Dispatched to dev agent for validation (Issue #5)
3. ✅ Technical validation complete - all tests passed
4. ✅ Validation recommendations incorporated:
   - Updated expected outputs to match actual vlab outputs (DESCR, GROUPS columns)
   - Added guidance on key fields in describe output
   - Added note about event TTL in long-running environments
5. ✅ **APPROVED** (2025-10-15)

**Next Steps:**

1. ✅ Module 1.1 approved - ready for Phase 3 content development
2. 📋 Begin Module 1.2 design: Architecture & Control Model
3. 📋 Continue Course 1 module designs (1.3, 1.4)

---

**Status:** ✅ APPROVED - Ready for Content Development
**Validation Report:** `module-1.1-validation-report.md`
**Next:** Module 1.2 design
