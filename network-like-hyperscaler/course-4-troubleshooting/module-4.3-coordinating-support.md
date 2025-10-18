---
title: "Coordinating with Support"
slug: "fabric-operations-coordinating-support"
difficulty: "intermediate"
estimated_minutes: 15
version: "v1.0.0"
validated_on: "2025-10-17"
pathway_slug: "network-like-hyperscaler"
pathway_name: "Network Like a Hyperscaler"
tags:
  - hedgehog
  - fabric
  - support
  - escalation
  - troubleshooting
  - communication
description: "Learn to draft effective support tickets, communicate professionally with support engineers, and use escalation as a strategic resource."
order: 403
---

## Introduction

You've diagnosed an issue (Module 4.1). You've tried rollback (Module 4.2). But the problem persists:

- Controller keeps crashing during VPC reconciliation
- Agent disconnects repeatedly despite correct configuration
- Performance degradation with no obvious cause

**It's time to engage Hedgehog support.**

### Support is a Partnership, Not a Failure

Beginners view support escalation as admitting defeat. Experts view it as **strategic resource utilization**:

- **Support has access to internal telemetry and engineering insights**
- **Support can identify product bugs vs. configuration issues**
- **Support provides workarounds while engineering fixes root cause**

**Early escalation with good diagnostics beats endless troubleshooting with incomplete information.**

### What Makes Support Collaboration Effective?

**Poor Escalation:**
```
Subject: VPC broken
Body: VPC not working. Please fix ASAP.
Attachments: None
```

Support response time: 24+ hours (needs clarification, diagnostics, reproduction steps)

**Effective Escalation:**
```
Subject: VPC creation fails - subnet overlap error
Body: Clear problem statement, timeline, diagnostics attached,
      troubleshooting attempted, specific error message
Attachments: hedgehog-diagnostics-20251017.tar.gz (12MB)
```

Support response time: 2-4 hours (has everything needed to investigate)

### Context: Professional Support Collaboration

Module 3.4 taught you **what to collect** (diagnostic checklist).

Module 4.3 teaches you **how to collaborate** with support engineers:
- Writing tickets that get fast responses
- Communicating effectively during investigation
- Providing follow-up diagnostics efficiently
- Understanding support ticket lifecycle

### What You'll Learn

**Effective Support Ticket Structure:**
- Problem statement that support understands in 30 seconds
- Complete diagnostic bundle attached (not just described)
- Clear timeline and recent changes documented
- Business impact and severity stated appropriately

**Communication Best Practices:**
- Responding to follow-up questions promptly (4-hour target)
- Providing context with additional diagnostics
- Structured updates for timeline-sensitive issues
- Confirming resolution and documenting solution

**Working with Support Engineers:**
- Understanding ticket lifecycle and expected timelines
- Knowing when to self-resolve vs. escalate
- Escalating early with confidence when appropriate
- Building collaborative relationship with support team

### Module Scenario

You'll draft a complete support ticket for a real issue encountered in Module 4.1 (VPCAttachment reconciliation problem), including:
- Clear problem statement with specific resource names
- Complete diagnostic bundle
- Business impact assessment
- Documentation of troubleshooting attempted

By the end of this module, you'll have the skills to engage support effectively and build productive partnerships with support engineers.

---

## Learning Objectives

By the end of this module, you will be able to:

1. **Draft effective support tickets** - Write clear, actionable problem statements with complete diagnostic evidence
2. **Communicate timelines and impact** - Articulate incident severity, business impact, and urgency appropriately
3. **Collaborate with support engineers** - Respond to follow-up questions efficiently and provide additional diagnostics
4. **Track escalation lifecycle** - Understand support ticket lifecycle from submission to resolution
5. **Apply support as strategic resource** - Escalate early with confidence when appropriate, not as last resort

---

## Prerequisites

Before starting this module, you should have:

**Completed Modules:**
- Module 3.4: Pre-Support Diagnostic Checklist (diagnostic collection skills)
- Module 4.1: Diagnosing Fabric Issues (systematic troubleshooting)
- Module 4.2: Rollback & Recovery (attempted self-resolution)

**Understanding:**
- How to collect diagnostic evidence (kubectl, Grafana, logs)
- Systematic troubleshooting methodology
- When configuration changes require rollback

**Skills:**
- Using diagnostic collection script (Module 3.4)
- Capturing Grafana screenshots
- Documenting troubleshooting attempts

---

## Scenario

**Incident Background:**

You encountered the issue from Module 4.1: VPCAttachment `customer-app-vpc-server-07` was created successfully (no error events), but server-07 cannot communicate within VPC.

**Troubleshooting Completed:**

You've already:
1. ✅ Diagnosed VLAN mismatch (VPC expects 1025, switch configured with 1020)
2. ✅ Identified root cause: VLAN conflict with another VPC
3. ✅ Attempted fix: Update VPC YAML to use VLAN 1020

**New Problem:**

When you updated the VPC to use VLAN 1020 (matching switch configuration), ArgoCD sync failed with unexpected error:

```
Failed to sync: VPC update rejected - VLAN 1020 reserved for system use
```

This is blocking resolution:
- Can't use VLAN 1025 (conflict with another VPC)
- Can't use VLAN 1020 (rejected as "reserved for system use")
- Unclear which VLANs are actually available

**Your Task:**

Escalate to support with a complete, professional ticket that will enable fast diagnosis and resolution.

---

## Core Concepts & Deep Dive

### Concept 1: Anatomy of an Effective Support Ticket

#### What Makes a Good Support Ticket?

Support engineers handle dozens of tickets daily. Good tickets get faster, more accurate responses.

**Good Ticket Characteristics:**
- **Concise problem statement** (1-2 sentences summary)
- **Specific resource names** (VPC name, switch name, exact error message)
- **Clear timeline** (when started, recent changes)
- **Complete diagnostics attached** (not just described)
- **Business impact stated** (severity, affected users)
- **Troubleshooting attempted documented** (what you've tried)

**Poor Ticket Characteristics:**
- Vague problem ("VPC broken")
- No resource names ("a VPC somewhere")
- No timeline ("recently")
- Diagnostics described but not attached
- No impact statement
- No documentation of self-troubleshooting

#### Support Ticket Structure (Template)

```markdown
# Issue Summary

**Problem:** [One-sentence description of observable symptom]

**Severity:** [P1=Critical/Down | P2=Major/Degraded | P3=Minor/Workaround | P4=Question]

**Impacted Resources:**
- Resource type: resource-name
- Switches: switch-01, switch-02
- Servers: server-05, server-06

**Started:** [Date/time when issue began]

**Recent Changes:** [Configuration changes in last 24-48 hours]

---

# Environment

**Hedgehog Version:**
- Controller: [image version from kubectl get deployment]
- Agents: [version from kubectl get agents]

**Fabric Topology:**
- Spines: X
- Leaves: Y
- Servers: Z

**Kubernetes Version:** [from kubectl version --short]

---

# Symptoms

**Observable Symptoms:**
1. [What you see in kubectl/Grafana/logs]
2. [Specific error messages, copy/paste exactly]
3. [Metrics or behaviors indicating issue]

**Expected Behavior:**
[What should be happening?]

**Actual Behavior:**
[What is actually happening?]

---

# Diagnostics Completed

**Pre-Escalation Checklist:**
- [x] kubectl events checked (attached: events.log)
- [x] Agent status reviewed (attached: agent-leaf-04.yaml)
- [x] BGP health verified (all sessions established)
- [x] Grafana dashboards checked (screenshots attached)
- [x] Controller logs collected (attached: controller.log)
- [x] VPC/VPCAttachment CRDs collected (attached: crds-vpc.yaml)

**Troubleshooting Attempted:**
1. [Action taken]
   - Result: [What happened]
2. [Action taken]
   - Result: [What happened]

**Findings:**
[What did your troubleshooting reveal? Even "no errors found" is useful information.]

---

# Impact

**Users Affected:** [Number of servers/applications/teams]

**Business Impact:** [Production down | Development blocked | Minor inconvenience]

**Workaround:** [If any workaround exists, describe it]

**Urgency:** [Immediate attention needed | Can wait hours | Can wait days]

---

# Attachments

**Diagnostic Bundle:** hedgehog-diagnostics-20251017-103000.tar.gz

**Contents:**
- crds-vpc.yaml, crds-wiring.yaml, crds-agents.yaml
- events.log, events-warnings.log
- controller.log, controller-previous.log
- agent-leaf-04.yaml, agent-leaf-04-bgp.json
- grafana-screenshots/ (3 images)

**Grafana Screenshots:**
- fabric-dashboard.png - Shows overall fabric health
- interfaces-dashboard-leaf04.png - Shows affected interface
- logs-dashboard-errors.png - Shows recent ERROR logs

---

# Additional Context

[Any other relevant information: related tickets, previous incidents, environment-specific details]
```

#### Key Ticket Writing Principles

**1. Problem statement first**

Support should understand the issue in 30 seconds of reading.

**Good:**
```
Problem: VPC customer-app-vpc creation fails with "Subnet 10.20.1.0/24 overlaps
with existing VPC prod-vpc". Created 2025-10-17 11:00 UTC.
```

**Poor:**
```
Problem: VPC not working.
```

**2. Specifics over generalities**

**Good:** "VPC `vpc-prod` on `leaf-04`"
**Poor:** "a VPC somewhere"

**Good:** Error message copy/pasted exactly
**Poor:** "Some error about VLANs"

**3. Attach, don't describe**

**Good:** Attach `controller.log` file
**Poor:** "Controller logs show errors"

**Good:** Attach Grafana screenshot
**Poor:** "Grafana shows the interface is down"

**4. Show your work**

Document troubleshooting attempts and results. This:
- Saves support time (won't suggest things you've tried)
- Shows you're engaged and competent
- Provides additional diagnostic context

**Example:**
```
Troubleshooting Attempted:
1. Verified VPCAttachment connection reference
   - Result: Correct (server-07--unbundled--leaf-04)
2. Checked for VLAN conflicts
   - Result: VLAN 1025 already used by existing-vpc-prod
3. Attempted VPC update to VLAN 1020
   - Result: Rejected - "VLAN 1020 reserved for system use"
```

**5. State impact clearly**

Helps support prioritize appropriately.

**Good:**
```
Impact:
- Users Affected: 1 development server
- Business Impact: Development blocked, production unaffected
- Urgency: Moderate - 24 hour resolution acceptable
```

**Poor:**
```
Impact: This is urgent!!!!
```

---

### Concept 2: Communication Best Practices

#### Best Practice 1: Respond Promptly to Follow-Up Questions

When support asks for additional information, respond within **4 business hours** if possible.

**Example Support Follow-Up:**

> "Thanks for the diagnostics. Can you provide the output of:
> `kubectl get agent leaf-04 -n fab -o jsonpath='{.status.state.platform}' | jq`
>
> This will help us check for hardware sensor issues."

**Good Response:**

```
Output attached: agent-leaf-04-platform.json

Summary: All PSUs operational, fan speeds normal, temperatures within range.
No hardware sensor warnings detected.

Let me know if you need additional diagnostics.
```

**Poor Response:**

```
I'll check that later.
```

**Why prompt response matters:**
- Keeps investigation moving forward
- Shows you're engaged and professional
- Support can maintain context (not context-switching to other tickets)
- Faster overall resolution time

#### Best Practice 2: Provide Context with Additional Diagnostics

When sending requested information, include brief context and interpretation.

**Example:**

Support requests: "Can you check if the issue reproduces after controller restart?"

**Good Response:**

```
Restarted controller at 2025-10-17 14:30 UTC.

Waited 15 minutes for reconciliation to complete.

Result: Issue still reproduces. VPCAttachment still fails to reconcile.

Attached: controller logs from restart (controller-post-restart.log)

Observations: Controller came up cleanly, no panic or crash.
Reconciliation loop running normally but VPCAttachment reconciliation
still failing with same error.
```

**Poor Response:**

```
Still broken.
```

**What context adds:**
- Saves support time (don't need to ask follow-up questions)
- Shows you understand what you're testing
- Provides additional observations that may help diagnosis

#### Best Practice 3: Use Structured Updates for Timeline-Sensitive Issues

For high-severity issues (P1, P2), provide periodic updates even if no progress.

**Example (Every 2 hours for P1):**

```
Update 14:00 UTC: Still investigating. Tried workaround X (did not resolve).
Collecting additional diagnostics per your request.

Update 16:00 UTC: Workaround Y applied. Issue partially resolved - 80% of
servers now working. 20% still affected. Awaiting root cause identification.

Update 18:00 UTC: Support identified root cause as VLAN reservation bug.
Patch being tested. Estimated resolution: End of day.
```

**Why structured updates matter:**
- Shows you're actively engaged
- Helps support understand current state
- Builds confidence that issue is being addressed
- Documents timeline for post-incident review

#### Best Practice 4: Confirm Resolution and Document Solution

When issue is resolved, confirm and document the solution for knowledge base.

**Example:**

```
Confirmed resolved as of 2025-10-17 17:00 UTC.

Solution Applied: Updated VPC VLAN to 1026 per your recommendation.
VLANs 1020-1025 are reserved for system use in our environment
(not documented in VLANNamespace range).

ArgoCD synced successfully. All 50 servers now have connectivity.

Root Cause: VLANNamespace range (1000-2999) includes reserved VLANs
(1020-1025) but doesn't exclude them automatically. This caused
automatic VLAN allocation to select reserved VLAN.

Recommendation: Update VLANNamespace to exclude reserved range or
document reserved VLANs in environment setup guide.

Thank you for the fast diagnosis and clear resolution steps!

Ticket can be closed.
```

**Why resolution confirmation matters:**
- Confirms fix actually worked (not just theory)
- Documents root cause understanding
- Provides feedback to support (what helped)
- Creates knowledge base for future similar issues

#### Best Practice 5: Ask for Clarification When Needed

If support's response is unclear, ask for clarification immediately.

**Example:**

Support says: "Apply the workaround from KB-1234."

If you can't find KB-1234 or don't understand the workaround:

**Good Response:**

```
I searched for KB-1234 but couldn't locate it in the documentation.

Could you provide a link or paste the workaround steps directly?

Want to ensure I apply the correct procedure.
```

**Poor Response:**

Guess at what support meant and apply the wrong workaround, potentially making issue worse.

**Why asking for clarification is professional:**
- Prevents mistakes from misunderstanding
- Shows you want to do it right
- Saves time (vs. applying wrong fix and escalating again)

---

### Concept 3: Support Ticket Lifecycle

#### Understanding Ticket States

**1. Submitted**
- Ticket created by customer
- Assigned to support queue
- **Expected time to first response:**
  - **P1 (Critical):** <1 hour
  - **P2 (Major):** <4 hours
  - **P3 (Minor):** <24 hours
  - **P4 (Question):** <48 hours

**2. Acknowledged**
- Support engineer assigned
- Initial response sent (may include follow-up questions)
- **Customer action:** Respond to questions promptly (4-hour target)

**3. Investigating**
- Support analyzing diagnostics
- May request additional information
- May engage engineering team for complex issues
- **Customer action:** Provide requested diagnostics, test workarounds

**4. Workaround Provided**
- Temporary solution identified
- Root cause investigation continues in parallel
- **Customer action:** Test and confirm workaround works

**5. Root Cause Identified**
- Support determines underlying issue
- May be configuration error (customer-side fix needed)
- May be product bug (requires engineering fix/patch)
- **Customer action:** Apply fix if configuration issue

**6. Resolved**
- Issue fixed (either configuration corrected or patch applied)
- Customer confirms resolution
- Ticket remains open for 48 hours for re-occurrence monitoring

**7. Closed**
- No re-occurrence after 48 hours
- Ticket archived
- Solution documented in knowledge base

#### Typical Timelines by Severity

**P1 (Critical - Production Down):**
- **First response:** <1 hour
- **Workaround target:** <4 hours
- **Resolution target:** <24 hours
- **Engagement:** Continuous until resolved

**P2 (Major - Degraded Service):**
- **First response:** <4 hours
- **Workaround target:** <24 hours
- **Resolution target:** <5 business days

**P3 (Minor - Workaround Exists):**
- **First response:** <24 hours
- **Resolution target:** <10 business days

**P4 (Question - No Outage):**
- **First response:** <48 hours
- **Resolution target:** <15 business days

**Note:** These are typical targets. Actual timelines depend on issue complexity, support plan tier, and current support queue.

#### What to Do if SLA is Missed

**If first response exceeds expected timeline:**

**P1:** Wait 1.5 hours, then follow up on ticket
**P2:** Wait 6 hours, then follow up on ticket
**P3:** Wait 36 hours, then follow up on ticket
**P4:** Wait 72 hours, then follow up on ticket

**Good Follow-Up:**

```
Following up on this P2 ticket submitted 6 hours ago.

Haven't received initial response yet (expected <4 hours for P2).

Issue remains: VPCAttachment connectivity failure due to VLAN conflict.

Please confirm ticket has been assigned to support engineer.

Let me know if any additional information needed to expedite investigation.

Thank you.
```

**Poor Follow-Up:**

```
WHERE IS MY RESPONSE???
```

---

### Concept 4: Escalation as Strategic Resource

#### When to Escalate Early (Don't Wait)

**Scenario 1: Repeated Failures After Rollback**

You've:
- Diagnosed issue (Module 4.1)
- Rolled back configuration (Module 4.2)
- Issue persists or new errors appear

**Action:** Escalate immediately. This suggests product bug or environment-specific issue requiring support insight.

**Why:** Further self-troubleshooting unlikely to help. Support has internal tools and engineering access you don't have.

---

**Scenario 2: Controller or Agent Crashes**

Symptoms:
- Controller CrashLoopBackOff
- Agent repeatedly disconnecting (not network issue)
- Panic messages in logs
- Core dumps

**Action:** Escalate after collecting crash logs. These are rarely configuration issues.

**Why:** Crashes indicate bugs or resource exhaustion issues that need engineering investigation.

---

**Scenario 3: Unexpected Behavior After Upgrade**

Symptoms:
- Fabric worked before upgrade
- Specific features broken after upgrade
- Configuration unchanged

**Action:** Escalate with "before upgrade" and "after upgrade" diagnostics. May be regression.

**Why:** Regressions require engineering team to identify code changes that caused issue.

---

**Scenario 4: Time-Sensitive Production Issue**

Situation:
- Production fabric serving live traffic
- Issue impacting revenue or SLA
- 30+ minutes of troubleshooting without progress

**Action:** Escalate as P1 even if root cause unclear. Support can troubleshoot faster with internal tools.

**Why:** Minimizing downtime is priority over complete self-diagnosis.

---

#### When to Self-Resolve Before Escalating

**Scenario 1: Configuration Error with Clear Error Message**

Example:
```
kubectl events: "Warning VLANConflict: VLAN 1020 already in use"
```

**Action:** Fix VLAN conflict yourself. Don't escalate configuration errors with obvious fixes.

**Why:**
- Clear error message explains problem
- Fix is straightforward (choose different VLAN)
- Documentation covers VLAN selection
- Resolution time: 10-15 minutes self-service vs. 4+ hours waiting for support

---

**Scenario 2: Documentation Covers the Issue**

Example:
- "How do I configure DHCP relay?"
- Answer is in official documentation with examples

**Action:** Check documentation first. Escalate only if documentation is unclear or incorrect.

**Why:**
- Documentation is immediately available (no wait time)
- Likely has examples matching your use case
- Escalate if documentation doesn't work as described

---

**Scenario 3: Issue Resolved by Standard Troubleshooting**

Example:
- Agent disconnected due to network issue
- Network fixed, agent reconnects
- Finalizer cleanup completes automatically

**Action:** No escalation needed if issue self-resolved.

**Why:** Issue was environmental (network), not product-related.

---

#### Strategic Escalation Mindset

**Good Escalation:**
```
I've spent 2 hours troubleshooting, collected diagnostics, tried rollback.
Issue persists with unexpected error message not covered in documentation.
Escalating with complete diagnostic bundle to expedite resolution.
```

**Bad Escalation:**
```
I've been troubleshooting for 3 days. Tried everything. Please help!
[No diagnostics attached]
```

**Better Escalation:**
```
Encountered controller crash after 30 minutes of troubleshooting.
Crash logs attached. Escalating early to minimize downtime and enable
engineering team to investigate root cause.
```

**Strategic Escalation Triggers:**

**Escalate when:**
- Unexpected behavior (error doesn't explain how to fix)
- Product bugs (crashes, panics, data corruption)
- Need internal tools/knowledge (engineering insight required)
- Time-sensitive and no clear fix

**Self-resolve when:**
- Configuration errors with clear error messages
- Issues covered in documentation
- Standard troubleshooting resolved the issue
- Obvious mistakes (typo, wrong field)

---

## Hands-On Lab

### Lab Overview

**Title:** Draft a Complete Support Ticket

**Scenario:**

You're experiencing the issue from Module 4.1: VPCAttachment `customer-app-vpc-server-07` was created successfully (no error events), but server-07 cannot communicate within VPC.

You've completed Module 4.1 troubleshooting and identified VLAN mismatch (VPC expects 1025, switch configured with 1020 due to conflict).

However, when you attempted to fix the VPC YAML in Gitea (change VLAN to 1020), ArgoCD sync failed with a new error:

```
Failed to sync: VPC update rejected - VLAN 1020 reserved for system use
```

This is unexpected behavior. You need to escalate to support.

**Your Task:**
1. Draft a complete support ticket using the template
2. Document diagnostic evidence
3. Practice professional communication
4. Review ticket quality before submission

**Time:** 6-7 minutes

---

### Task 1: Draft Support Ticket Problem Statement

**Estimated Time:** 2 minutes

**Objective:** Write clear, concise problem summary using template structure.

#### Step 1.1: Complete Issue Summary Section

Using the template from Concept 1, fill in these fields:

**Problem:**

VPCAttachment reconciled successfully but server has no connectivity. VLAN mismatch identified (VPC expects 1025, switch has 1020). Attempted fix by updating VPC to VLAN 1020, but ArgoCD sync rejected with "VLAN 1020 reserved for system use" error.

**Severity:**

P2 (Major - Development environment, 1 server affected, no workaround identified)

**Impacted Resources:**
- VPC: `customer-app-vpc`
- VPCAttachment: `customer-app-vpc-server-07`
- Server: `server-07`
- Switch: `leaf-04`, interface `Ethernet8`

**Started:**

2025-10-17 10:00 UTC (VPCAttachment created)

**Recent Changes:**
- Created `customer-app-vpc` VPC with frontend subnet (VLAN 1025)
- Created VPCAttachment for `server-07` to `customer-app-vpc/frontend`
- Attempted fix: Updated VPC YAML to use VLAN 1020 (Git commit `a1b2c3d`)

#### Step 1.2: Document Environment

```
**Hedgehog Version:**
- Controller: ghcr.io/githedgehog/fabric/controller:v0.41.3
- Agents: v0.41.3 (all switches)

**Fabric Topology:**
- Spines: 2 (spine-01, spine-02)
- Leaves: 5 (leaf-01 through leaf-05)
- Servers: 10

**Kubernetes Version:** v1.28.4
```

**Get version info in real environment:**

```bash
# Controller version
kubectl get deployment fabric-controller-manager -n fab -o jsonpath='{.spec.template.spec.containers[0].image}'

# Agent versions
kubectl get agents -n fab -o jsonpath='{range .items[*]}{.metadata.name}: {.status.version}{"\n"}{end}'

# Kubernetes version
kubectl version --short
```

#### Success Criteria

- ✅ Problem statement is 1-3 sentences, specific and clear
- ✅ Severity appropriate for impact (P2 for dev environment, 1 server)
- ✅ All affected resources listed by exact name
- ✅ Timeline clear (when started, when changes made)
- ✅ Environment info complete and accurate

---

### Task 2: Document Symptoms and Diagnostics

**Estimated Time:** 2 minutes

**Objective:** Provide clear symptoms and diagnostic evidence to support.

#### Step 2.1: Symptoms Section

**Observable Symptoms:**

1. VPCAttachment shows "Ready" status in kubectl, no error events
2. server-07 cannot ping gateway 10.20.10.1 ("Destination Host Unreachable")
3. Agent CRD shows VLAN 1020 on leaf-04/Ethernet8 (VPC spec expects 1025)
4. Attempted VPC VLAN update to 1020, ArgoCD sync failed with error:
   ```
   Failed to sync application 'hedgehog-config':
   VPC customer-app-vpc update rejected: VLAN 1020 reserved for system use
   ```

**Expected Behavior:**
- server-07 should have connectivity to VPC gateway and other servers
- OR VLAN update to 1020 should succeed (if 1020 is not actually reserved)

**Actual Behavior:**
- server-07 has no connectivity
- Cannot use VLAN 1025 (conflict with `existing-vpc-prod`)
- Cannot use VLAN 1020 (rejected as "reserved for system use")
- No available VLAN identified to resolve conflict

#### Step 2.2: Diagnostics Completed Section

**Pre-Escalation Checklist:**
- [x] kubectl events checked - VPCAttachment shows no errors
- [x] Agent status reviewed - leaf-04 Ready, VLAN 1020 on Ethernet8
- [x] BGP health verified - all sessions established
- [x] Grafana dashboards checked - interface up, VLAN 1020 configured
- [x] Controller logs collected - no reconciliation errors for VPCAttachment
- [x] VPC/VPCAttachment CRDs collected
- [x] ArgoCD sync logs collected - shows VLAN 1020 rejection error

**Troubleshooting Attempted:**

1. Verified VPCAttachment references correct connection (`server-07--unbundled--leaf-04`)
   - **Result:** Connection reference correct

2. Checked if frontend subnet exists in `customer-app-vpc`
   - **Result:** Subnet exists, VLAN specified as 1025

3. Investigated VLAN mismatch (VPC expects 1025, switch has 1020)
   - **Result:** VLAN conflict with `existing-vpc-prod` caused automatic allocation of 1020

4. Attempted fix: Updated VPC YAML to use VLAN 1020 (matching switch config)
   - **Result:** ArgoCD sync rejected - "VLAN 1020 reserved for system use"

5. Checked VLANNamespace for available VLANs
   - **Result:** Range 1000-2999, many VLANs appear unused
   - **Question:** How to determine which VLANs are actually reserved?

6. Attempted alternate VLAN (1026) in VPC YAML
   - **Result:** Not yet tested (waiting for guidance on VLAN reservation)

**Findings:**
- VLAN mismatch confirmed between VPC spec (1025) and switch config (1020)
- Cannot fix by updating VPC to match switch (VLAN 1020 rejected as reserved)
- Unclear which VLANs are "reserved for system use"
- Need guidance on VLAN selection or understanding of reservation error

#### Success Criteria

- ✅ Symptoms numbered and specific (not vague)
- ✅ Error messages copy/pasted exactly (not paraphrased)
- ✅ Troubleshooting steps documented with results
- ✅ Findings summarize current understanding and questions

---

### Task 3: Document Impact and Reference Diagnostics

**Estimated Time:** 1-2 minutes

**Objective:** State business impact clearly and reference complete diagnostic bundle.

#### Step 3.1: Impact Section

**Users Affected:**

1 development server (server-07 in customer-app workload)

**Business Impact:**

Development environment - blocks deployment testing for customer-app workload. Production not affected.

**Workaround:**

None identified. Cannot use VLAN 1025 (conflict with existing VPC) or VLAN 1020 (rejected as reserved for system use).

**Urgency:**

Moderate - Development team blocked, but production unaffected. Resolution within 24 hours acceptable.

#### Step 3.2: Attachments Section

**Diagnostic Bundle:**

`hedgehog-diagnostics-20251017-140000.tar.gz` (12 MB)

**Contents:**
- `crds-vpc.yaml` (all VPCs including customer-app-vpc)
- `crds-vpcattachments.yaml` (all VPCAttachments)
- `crds-wiring.yaml` (switches, servers, connections)
- `crds-agents.yaml` (all agent status)
- `crds-namespaces.yaml` (IPv4Namespace, VLANNamespace configs)
- `events.log` (all events, last 2 hours)
- `events-warnings.log` (Warning events only)
- `controller.log` (last 2000 lines)
- `argocd-sync-error.log` (sync failure details)
- `agent-leaf-04.yaml` (detailed switch status for affected switch)
- `agent-leaf-04-interfaces.json` (interface state including VLAN config)
- `grafana-screenshots/` (3 images)

**Grafana Screenshots:**
- `fabric-dashboard.png` - Overall fabric health (all BGP up, no alarms)
- `interfaces-leaf04-eth8.png` - Shows interface up, VLAN 1020 configured
- `logs-argocd-error.png` - Shows VLAN rejection error in ArgoCD logs

**How to create diagnostic bundle:**

```bash
# Use diagnostic collection script from Module 3.4
./diagnostic-collect.sh

# Or use kubectl commands to collect manually
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
mkdir -p hedgehog-diagnostics-${TIMESTAMP}

# Collect CRDs, events, logs (see Module 3.4 for complete script)

# Compress
tar czf hedgehog-diagnostics-${TIMESTAMP}.tar.gz hedgehog-diagnostics-${TIMESTAMP}
```

#### Step 3.3: Additional Context

**Additional Context:**

This is the first VPC for customer-app workload in this fabric. Previous VPCs (`vpc-prod`, `vpc-staging`) created without VLAN reservation errors.

**Questions for support:**
- Which VLANs are reserved for system use in default environment?
- Is there a way to query available VLANs programmatically?
- Should VLANNamespace exclude reserved VLANs from allocated range?

**Observation:** VLANNamespace shows range 1000-2999, but error suggests some VLANs within range are reserved. This is not mentioned in documentation or VLANNamespace CRD.

#### Success Criteria

- ✅ Impact clearly stated (severity P2 matches stated impact)
- ✅ Diagnostic bundle referenced with size and complete contents listed
- ✅ Screenshots described (what each shows)
- ✅ Additional context provides useful background
- ✅ Specific questions help guide support investigation

---

### Task 4: Review and Submit

**Estimated Time:** 1 minute

**Objective:** Final quality review before submission.

#### Step 4.1: Review Ticket Quality

Use this checklist before submitting:

- [ ] Problem statement is clear in first 30 seconds of reading
- [ ] All resource names are specific (not "a VPC" or "some server")
- [ ] Timeline documented (when issue started, when changes made)
- [ ] Symptoms are numbered and specific
- [ ] Error messages copy/pasted exactly (not paraphrased)
- [ ] Troubleshooting attempts documented with results for each attempt
- [ ] Impact stated clearly (severity, affected users, urgency match)
- [ ] Diagnostic bundle attached and contents explicitly listed
- [ ] Grafana screenshots attached and described
- [ ] Additional context provides useful background
- [ ] Questions for support are specific and actionable

#### Step 4.2: Simulate Submission

In a real environment, you would:

1. Log into support portal (e.g., `support.hedgehog.com`)
2. Click **"Create New Ticket"**
3. Paste ticket content into ticket description field
4. Attach diagnostic bundle (.tar.gz file)
5. Attach Grafana screenshots (3 PNG files)
6. Select severity: **P2 (Major)**
7. Add tags: `vlan`, `argocd`, `vpc`, `connectivity`
8. Click **"Submit"**

**Expected Response Time:**

P2 = first response within 4 hours (business hours)

#### Success Criteria

- ✅ Ticket passes all items in quality checklist
- ✅ All attachments are ready to upload
- ✅ Severity selected appropriately
- ✅ Ticket is professional and complete
- ✅ Ready for actual submission to support

**What happens next:**

After submission:
1. Ticket auto-assigned to support queue
2. Support engineer picks up ticket (within 4 hours for P2)
3. Initial response may be acknowledgment or follow-up questions
4. You respond to follow-up questions within 4 hours (Best Practice 1)
5. Investigation proceeds until resolution or workaround found

---

### Lab Summary

**What You Accomplished:**

You successfully drafted a professional support ticket with:
- ✅ Clear problem statement with specific resource names and timeline
- ✅ Comprehensive symptoms list with exact error messages
- ✅ Documented troubleshooting attempts (shows your work)
- ✅ Appropriate severity and impact assessment
- ✅ Complete diagnostic bundle ready to attach
- ✅ Specific questions to guide support investigation

**Key Takeaways:**

1. **Support is strategic, not a failure** - Early escalation with good diagnostics is professional
2. **Clarity accelerates resolution** - Specific details help support diagnose faster
3. **Attach, don't describe** - Actual logs and screenshots better than summaries
4. **Show your work** - Document what you've tried (saves support time)
5. **Communication matters** - Professional, prompt responses build effective collaboration

**This ticket would get fast support response because:**
- Problem is immediately clear
- All context is provided upfront
- Diagnostics are complete and attached
- Impact and urgency are appropriately stated
- Questions are specific and guide next steps

---

## Troubleshooting

### Common Lab Challenges

#### Challenge: "I don't know what diagnostic bundle to attach"

**Solution:** Use the diagnostic collection script from Module 3.4.

```bash
# Run collection script
./diagnostic-collect.sh

# Or manually collect minimum essentials:
# - All VPC/VPCAttachment CRDs
# - Events (last 2 hours)
# - Controller logs (last 2000 lines)
# - Agent CRD for affected switch
# - Grafana screenshots showing issue
```

**Reference:** Module 3.4 - Pre-Support Diagnostic Checklist

---

#### Challenge: "What severity should I use?"

**Decision tree:**

```
Is production down or severely degraded?
├─ YES → P1 (Critical)
└─ NO → Is development/testing blocked?
         ├─ YES → P2 (Major)
         └─ NO → Is there a workaround?
                  ├─ YES → P3 (Minor)
                  └─ NO, just a question → P4
```

**P1 Examples:**
- Production fabric down
- >50% capacity loss
- Data corruption

**P2 Examples:**
- Development environment blocked
- Single component degraded
- No workaround available

**P3 Examples:**
- Minor issues with workaround
- Feature request
- Performance optimization

**P4 Examples:**
- "How do I...?" questions
- Clarification requests
- Documentation feedback

---

#### Challenge: "Support asked for something I don't have"

**Example:** Support asks for specific kubectl output you didn't collect.

**Solution:**

```
I don't have that output in my original diagnostic bundle.

Let me collect it now and send it to you:

[Run requested command]
[Attach output]

Summary: [Brief interpretation]

Let me know if you need anything else.
```

**Key points:**
- Acknowledge you don't have it (don't apologize excessively)
- Collect it immediately
- Send with context/summary
- Offer to collect more if needed

---

#### Challenge: "I'm not sure if I should escalate or keep troubleshooting"

**Decision framework:**

**Escalate if:**
- Spent >2 hours with no progress
- Error message doesn't explain how to fix
- Controller/agent crashes (not configuration issue)
- After rollback, issue persists
- Time-sensitive and no clear path forward

**Self-resolve if:**
- Error message is clear (e.g., "VLAN conflict")
- Documentation covers the issue
- Standard troubleshooting is working
- Issue is obvious configuration error

**When in doubt:**
- Collect diagnostics
- Document what you've tried
- Escalate with context
- Better to escalate early with good diagnostics than late with poor diagnostics

---

## Resources

### Reference Documentation

**Related Modules:**
- Module 3.4: Pre-Support Diagnostic Checklist (diagnostic collection procedures)
- Module 4.1: Diagnosing Fabric Issues (systematic troubleshooting methodology)
- Module 4.2: Rollback & Recovery (self-resolution procedures)

**Support Ticket Templates:**
- Full template (provided in Concept 1)
- Minimum required fields (Problem, Severity, Diagnostics, Impact)
- Follow-up response template (Concept 2)
- Resolution confirmation template (Concept 2)

### Quick Reference: Support Ticket Checklist

**Before submitting ticket:**

**Problem Statement:**
- [ ] One-sentence summary of issue
- [ ] Specific resource names (not "a VPC")
- [ ] Timeline (when started)
- [ ] Recent changes documented

**Diagnostics:**
- [ ] kubectl events collected
- [ ] Agent CRD status collected
- [ ] Controller logs collected
- [ ] VPC/VPCAttachment CRDs collected
- [ ] Grafana screenshots captured
- [ ] All files compressed into .tar.gz

**Troubleshooting:**
- [ ] Documented what you tried
- [ ] Documented results of each attempt
- [ ] Documented current understanding

**Impact:**
- [ ] Users affected (number/scope)
- [ ] Business impact (prod/dev/test)
- [ ] Urgency (immediate/hours/days)
- [ ] Workaround (if any)

**Quality:**
- [ ] Error messages copy/pasted exactly
- [ ] Specific questions for support
- [ ] Professional tone
- [ ] All attachments referenced

### Communication Timeline Reference

**Expected First Response:**
- **P1:** <1 hour
- **P2:** <4 hours
- **P3:** <24 hours
- **P4:** <48 hours

**Your Response to Support Questions:**
- **Target:** <4 business hours
- **Maximum:** <24 hours (for non-urgent issues)

**Follow-Up if No Support Response:**
- **P1:** After 1.5 hours
- **P2:** After 6 hours
- **P3:** After 36 hours
- **P4:** After 72 hours

### Escalation Decision Matrix

| Scenario | Self-Resolve | Escalate | Priority |
|----------|--------------|----------|----------|
| Clear error message (VLAN conflict) | ✅ Fix yourself | ❌ | - |
| Controller crash | ❌ | ✅ Collect crash logs | P2 or P1 |
| Documentation covers it | ✅ Follow docs | ❌ | - |
| Unexpected error (no explanation) | ❌ | ✅ With diagnostics | P2 or P3 |
| After rollback, still broken | ❌ | ✅ With before/after diagnostics | P2 |
| Production down | ❌ | ✅ Immediately | P1 |
| Issue self-resolved | ✅ Document in notes | ❌ | - |

---

## Assessment

### Question 1: Effective Support Tickets

**Scenario:** You're writing a support ticket for a VPC creation failure. Which problem statement is BEST?

**A:**
"VPC not working. Tried creating it multiple times. Please fix ASAP."

**B:**
"I created a VPC called customer-vpc and it's not working. The servers can't connect. I think it might be a BGP issue or maybe the switch is down. Can you investigate?"

**C:**
"VPC customer-vpc creation fails with error 'Subnet 10.20.1.0/24 overlaps with existing VPC prod-vpc'. Created 2025-10-17 11:00 UTC. No recent changes to prod-vpc. Diagnostics attached."

**D:**
"VPC customer-vpc has issues. See attached 50MB log dump. Figure out what's wrong."

<details>
<summary>Answer & Explanation</summary>

**Answer:** C

**Explanation:**

**Why C is correct:**

**Specific problem statement provides:**
- Exact resource name (`customer-vpc`)
- Exact error message (copy/pasted: "Subnet 10.20.1.0/24 overlaps with existing VPC prod-vpc")
- Timeline (created 2025-10-17 11:00 UTC)
- Context (no recent changes to prod-vpc)
- References diagnostics

**Support can immediately understand:**
- **What failed:** VPC creation
- **Why it failed:** Subnet overlap with existing VPC
- **When it happened:** Specific date/time
- **Next step:** Check subnet allocation or recommend different subnet

**Estimated time for support to understand issue:** 30 seconds

**Why others are wrong:**

**A) Too vague:**
- "VPC not working" - which VPC? What's not working specifically?
- "Tried multiple times" - when? Same error each time? Different errors?
- "Please fix ASAP" - no severity stated, no context for urgency
- No diagnostics mentioned
- No error message
- Support must ask: "Which VPC? What error? When? Send diagnostics?"
- **Estimated time for support to understand:** Unknown, requires 2-3 follow-up questions

**B) Too verbose, lacks specifics:**
- Problem statement buried in explanation
- "I think it might be..." - speculation, not facts
- No exact error message (most critical piece missing)
- "Can you investigate?" - support needs diagnostics to investigate
- Multiple possible causes listed (BGP, switch down) without evidence
- **Estimated time for support to understand:** 5-10 minutes (after requesting error message and diagnostics)

**D) Dumps work on support:**
- "Has issues" - what issues specifically? What's the error?
- 50MB log dump without context or summary
- Expects support to search through 50MB for the issue
- No summary, no error message highlighted
- "Figure out what's wrong" - confrontational tone, expects support to do all work
- **Estimated time for support to understand:** 30-60 minutes (searching logs) OR immediate request for better summary

**Effective Ticket Principle:**

Support should understand the issue in **30 seconds** of reading the problem statement. Specifics (resource names, exact error messages, timeline) accelerate diagnosis dramatically.

**Time Impact:**
- Good ticket (C): Support starts investigating immediately (30 seconds)
- Poor ticket (A, B, D): Support requests clarification (adds 4-24 hour delay)

**Module 4.3 Reference:** Concept 1 - Anatomy of an Effective Support Ticket
</details>

---

### Question 2: Communication Best Practices

**Scenario:** Support responds to your ticket with:

> "Can you provide the output of `kubectl get agent leaf-04 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq` to check BGP neighbor state?"

What is the BEST response?

**A:**
"I'll get that for you eventually."

**B:**
"Here's the output: [paste raw JSON with no context]"

**C:**
"Output attached: agent-leaf-04-bgp.json. Summary: All BGP neighbors in 'established' state. No down neighbors detected. Let me know if you need additional diagnostics."

**D:**
"I don't have jq installed. Can't run that command."

<details>
<summary>Answer & Explanation</summary>

**Answer:** C

**Explanation:**

**Why C is correct:**

**Professional response includes:**

1. **Requested output attached** (`agent-leaf-04-bgp.json` file) - Gives support exactly what they asked for
2. **Brief summary** ("All BGP neighbors in 'established' state") - Saves support time by highlighting key finding
3. **Interpretation** ("No down neighbors detected") - Shows you understand what you're looking at
4. **Offer to help more** ("Let me know if you need additional diagnostics") - Shows continued engagement

**Benefits:**
- Support gets exact data requested (can verify your interpretation)
- Summary allows support to quickly assess next step
- Shows you're actively engaged and professional
- Prompts next step from support

**Expected support response:** "Thanks! BGP looks good. Let's check VLAN configuration next..."

**Why others are wrong:**

**A) Vague commitment:**
- "Eventually" - no specific timeline, could be hours or days
- Support may wait 24+ hours for response before following up
- Delays resolution significantly
- Unprofessional tone (dismissive)
- **Impact:** Investigation stalls, issue resolution delayed by days

**Better version of A:** "I'll run that command in the next hour and send you the output by [specific time]."

**B) Data without context:**
- Raw JSON paste is hard to read in ticket system (formatting issues)
- No summary means support must analyze from scratch
- Missed opportunity to highlight findings
- Shows you ran command but didn't interpret results
- **Impact:** Support spends 5-10 minutes analyzing data you could have summarized in 10 seconds

**Better version of B:** Include data AND summary like option C.

**D) Focuses on blocker instead of solution:**
- `jq` is optional for this command (output is still JSON, just not pretty-printed)
- Could run without `jq` and paste raw output
- Could ask support for alternative command format
- Presents blocker without attempting workaround
- **Impact:** Support must send another response with alternative command, adding 4+ hour delay

**Better version of D:** "jq not available in my environment. Here's the raw output without formatting: [paste raw JSON]"

**Communication Best Practice:**

Respond to support requests within **4 business hours** with:
- **Requested data** (attached or pasted)
- **Brief summary** (key findings, 1-2 sentences)
- **Interpretation** (what it means)
- **Offer to provide more** if needed

**Time Impact:**
- Good response (C): Investigation continues immediately
- Poor response (A, D): Investigation stalls 4-24 hours for follow-up

**Module 4.3 Reference:** Concept 2 - Communication Best Practices (Best Practice 1 & 2)
</details>

---

### Question 3: Strategic Escalation

**Scenario:** You're troubleshooting a VPCAttachment connectivity issue. You've spent 30 minutes and identified that the VPC subnet VLAN (1025) conflicts with another VPC. You need to choose a different VLAN. Should you escalate to support?

- A) Yes - escalate immediately, VLAN conflicts require support investigation
- B) No - this is a configuration error with a clear fix (choose different VLAN)
- C) Yes - any VLAN issue should be escalated to support
- D) No - but only if you've spent at least 4 hours troubleshooting first

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) No - this is a configuration error with a clear fix

**Explanation:**

**Self-Resolve Before Escalating:**

**VLAN conflict diagnosis:**
- kubectl events show clear error: `"Warning VLANConflict: VLAN 1025 already in use"`
- Root cause is obvious: VPC spec uses same VLAN as another VPC
- Fix is straightforward: Update VPC YAML to use different VLAN from VLANNamespace range
- Documentation covers VLAN selection

**Self-Resolution Steps:**

```bash
# Check available VLANs
kubectl get vpc -A -o yaml | grep "vlan:" | sort

# Identify unused VLAN
# Example: VLANs 1020-1025 in use, choose 1026

# Update VPC YAML in Gitea
# Change: vlan: 1025 → vlan: 1026

# Commit change and trigger ArgoCD sync

# Verify via kubectl and Grafana
```

**Expected time to fix:** 10-15 minutes

**Why escalation is unnecessary:**
- Configuration error with obvious fix (error message explains problem)
- Documentation explains VLAN selection process
- No unexpected behavior (conflict error is expected when VLANs overlap)
- You have tools and access to fix (Gitea access, kubectl, VLANNamespace shows range)
- No product bug suspected

**Why others are wrong:**

**A) Escalate immediately:**
- **Wastes support time** on easily fixable configuration issue
- **Support will respond:** "Choose a different VLAN from VLANNamespace range" (same conclusion you reached)
- **Delays resolution:** Waiting 4+ hours for support vs. fixing in 15 minutes yourself
- **Not strategic:** Escalation should be reserved for issues requiring support's unique knowledge/tools

**C) Any VLAN issue escalated:**
- **Too broad:** Many VLAN issues are simple configuration errors
- **Only escalate VLAN issues when:**
  * "Reserved for system use" error without explanation (which VLANs reserved?)
  * VLAN configured but not working despite correct config (unexpected behavior)
  * VLANNamespace corruption or allocation bug suspected
  * Conflict persists after choosing different VLAN (shouldn't happen)

**Example escalatable VLAN issue:** "Updated VPC to use VLAN 1026 (unused per kubectl check), but ArgoCD sync rejected with 'VLAN 1026 reserved for system use'. VLANNamespace shows range 1000-2999 with no documented reservations." ← This is unexpected behavior requiring support insight.

**D) 4 hours minimum:**
- **Arbitrary time limit** with no logical basis
- **Should escalate based on issue type, not time spent**
- **4 hours on obvious config error is wasted time**
- **Strategic escalation is about issue complexity, not duration**

**Strategic Escalation Triggers:**

**Escalate when:**
- Unexpected behavior (error doesn't explain how to fix)
- Product bugs (crashes, panics, data loss)
- Need internal tools/knowledge (engineering insight required)
- Time-sensitive production issue with no clear fix
- After rollback, issue persists with new errors

**Self-Resolve when:**
- Configuration errors with clear error messages
- Issues covered in documentation with examples
- Standard troubleshooting resolved issue quickly
- Obvious mistakes (typo, wrong field value, duplicate resource name)

**Decision Framework:**

Ask yourself:
1. Is the error message clear about what's wrong? (YES for VLAN conflict)
2. Do I know how to fix it? (YES - choose different VLAN)
3. Do I have tools/access to fix it? (YES - Gitea, kubectl)
4. Is fix covered in documentation? (YES - VLAN selection)

If all YES → Self-resolve before escalating.

**Module 4.3 Reference:** Concept 4 - Escalation as Strategic Resource
</details>

---

### Question 4: Support Ticket Lifecycle

**Scenario:** You submitted a P2 support ticket 6 hours ago. You haven't received a response yet. What should you do?

- A) Submit a new ticket with higher severity (P1)
- B) Wait another 6 hours (12 hours total) before following up
- C) Reply to the ticket asking for status update (P2 first response target is <4 hours, but may be delayed)
- D) Call your account manager and complain

<details>
<summary>Answer & Explanation</summary>

**Answer:** C) Reply to the ticket asking for status update

**Explanation:**

**P2 Ticket Timeline:**
- **Expected first response:** <4 hours (business hours)
- **Your wait time:** 6 hours (exceeds target by 2 hours)
- **Appropriate action:** Polite follow-up on ticket

**Appropriate Follow-Up:**

Reply to ticket with:

```
Following up on this P2 ticket submitted 6 hours ago.

Haven't received initial response yet (expected <4 hours for P2).

Issue remains: VPCAttachment connectivity failure due to VLAN conflict.

Please confirm ticket has been assigned to support engineer.

Let me know if any additional information needed to expedite investigation.

Thank you.
```

**Why this is appropriate:**
- **Polite status request** (not demanding or aggressive)
- **References expected SLA** (4 hours for P2) - Shows you know what to expect
- **Re-states issue briefly** (in case ticket lost in queue or needs re-prioritization)
- **Offers to provide more info** (shows willingness to help)
- **Professional tone** (builds good relationship with support)

**Expected support response:** Acknowledgment + assignment confirmation + timeline OR immediate investigation if ticket was missed.

**Why others are wrong:**

**A) Submit new ticket with P1:**
- **Wrong: Severity inflation** - P1 is for production down/critical outages
- **Your issue:** Development environment, 1 server - Not P1
- **Creates duplicate tickets** (confuses support queue, may delay both tickets)
- **May get flagged** for inappropriate severity use (damages credibility)
- **Doesn't actually expedite response** (support will notice duplicate and likely consolidate)
- **Better action:** Follow up on existing ticket first

**When to escalate to P1:** Only if issue impact changes (e.g., now affecting production, not just dev).

**B) Wait another 6 hours (total 12 hours):**
- **Total 12 hours wait for P2 is excessive** (3× the expected response time)
- **SLA already missed by 2 hours** (4-hour target)
- **Proactive follow-up is appropriate** after exceeding SLA
- **Silent waiting doesn't help** (ticket may be stuck, needs nudge)
- **Industry best practice:** Follow up after 1.5× expected response time (6 hours for P2)

**When waiting is appropriate:** P3 or P4 tickets within expected response window.

**D) Call account manager:**
- **Too escalated for 6-hour delay** (not yet an emergency)
- **Account manager will likely say:** "Follow up on the ticket first"
- **Reserve account manager escalation for:**
  * P1 with no response after 2 hours
  * Repeated SLA misses across multiple tickets (systemic issue)
  * Support unresponsive after 2-3 follow-ups on same ticket
  * Relationship or contractual issues
- **Current situation:** First ticket, first SLA miss - Not yet account manager territory

**Best Practice Follow-Up Timeline:**

| Priority | Expected Response | Follow-Up After | Escalate to Manager After |
|----------|------------------|-----------------|--------------------------|
| **P1 (Critical)** | <1 hour | 1.5 hours | 2 hours + 2 follow-ups |
| **P2 (Major)** | <4 hours | 6 hours ← **YOU ARE HERE** | 24 hours + 2 follow-ups |
| **P3 (Minor)** | <24 hours | 36 hours | 72 hours + 2 follow-ups |
| **P4 (Question)** | <48 hours | 72 hours | N/A (not critical) |

**Professional Follow-Up Characteristics:**
- **Polite, not demanding** ("Following up" not "WHERE IS MY RESPONSE??")
- **References SLA** (shows you know expectations)
- **Restates issue** (helps support if they need to re-prioritize)
- **Offers to help** ("Let me know if you need more info")
- **Single follow-up every 4-6 hours** (not spam every hour)

**After your follow-up:**

If still no response after another 6 hours (12 hours total):
- Send second follow-up, slightly more urgent tone
- Consider escalating within support team (supervisor)
- If P2 reaches 24 hours with no response, consider account manager escalation

**Module 4.3 Reference:** Concept 3 - Support Ticket Lifecycle
</details>

---

## Conclusion

You've completed Module 4.3: Coordinating with Support!

### What You Learned

**Effective Support Ticket Structure:**
- Problem statement that support understands in 30 seconds
- Complete diagnostic bundle attached (not just described)
- Clear timeline and recent changes documented
- Business impact and severity stated appropriately
- Troubleshooting attempts documented with results

**Communication Best Practices:**
- Respond to follow-up questions within 4 hours
- Provide context with additional diagnostics
- Use structured updates for timeline-sensitive issues
- Confirm resolution and document solution
- Ask for clarification when needed

**Working with Support Engineers:**
- Understanding ticket lifecycle (Submitted → Acknowledged → Investigating → Resolved → Closed)
- Expected timelines by severity (P1: <1hr, P2: <4hrs, P3: <24hrs, P4: <48hrs)
- When to self-resolve vs. escalate
- Escalating early with confidence when appropriate

**Strategic Escalation:**
- Support is a partnership, not a failure
- Early escalation with good diagnostics is professional
- Escalation triggers: unexpected behavior, crashes, post-rollback issues, time-sensitive situations
- Self-resolve triggers: clear error messages, documented solutions, obvious configuration errors

### Key Takeaways

1. **Support is strategic, not a failure** - Engaging support early with good diagnostics is professional practice

2. **Clarity accelerates resolution** - Specific problem statements and complete diagnostics enable fast support response

3. **Attach, don't describe** - Actual log files and screenshots are far more valuable than summaries

4. **Show your work** - Documenting troubleshooting attempts saves support time and demonstrates competence

5. **Communication matters** - Professional, prompt responses build effective collaborative relationships with support

### Support Collaboration Mindset

As you operate Hedgehog fabrics:

- **Plan for escalation** - Collect diagnostics during troubleshooting (makes escalation faster if needed)
- **Escalate with confidence** - Support has tools and knowledge you don't have access to
- **Communicate professionally** - Build long-term relationship with support team
- **Document everything** - Timeline, changes, troubleshooting, resolution (helps future incidents)
- **Learn from support** - Pay attention to how support diagnoses issues (builds your skills)

### Course 4 Progress

**Completed:**
- ✅ Module 4.1: Diagnosing Fabric Issues (systematic troubleshooting)
- ✅ Module 4.2: Rollback & Recovery (safe undo procedures)
- ✅ Module 4.3: Coordinating with Support (effective escalation)

**Up Next:**
- Module 4.4: Post-Incident Review (final module - blameless reviews, lessons learned, continuous improvement)

**Overall Pathway Progress:** 15/16 modules complete (93.75%)

---

**You're now equipped to escalate effectively and collaborate professionally with support engineers. One more module to go - see you in Module 4.4!**
