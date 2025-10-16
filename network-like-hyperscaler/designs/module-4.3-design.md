# Module 4.3 Design: Coordinating with Support

## Module Metadata

- **Module Number:** 4.3
- **Module Title:** Coordinating with Support
- **Course:** Course 4 - Troubleshooting, Recovery & Escalation
- **Estimated Duration:** 13-15 minutes
  - Introduction: 2 minutes
  - Core Concepts: 4-5 minutes
  - Hands-On Lab: 6-7 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 3.4 Complete (Pre-Support Diagnostic Checklist)
  - ✅ Module 4.1 Complete (Diagnosing Fabric Issues)
  - ✅ Module 4.2 Complete (Rollback & Recovery)
  - ✅ Understanding of diagnostic collection and troubleshooting

## Learning Objectives

By the end of this module, learners will be able to:

1. **Draft effective support tickets** - Write clear, actionable problem statements with complete diagnostic evidence
2. **Communicate timelines and impact** - Articulate incident severity, business impact, and urgency appropriately
3. **Collaborate with support engineers** - Respond to follow-up questions efficiently and provide additional diagnostics
4. **Track escalation lifecycle** - Understand support ticket lifecycle from submission to resolution
5. **Apply support as strategic resource** - Escalate early with confidence when appropriate, not as last resort

**Bloom's Taxonomy Level**: Apply, Create (writing tickets, communicating professionally)

## Content Outline

### Introduction (2 minutes)

**Hook: Support is a Partnership, Not a Failure**

> You've diagnosed an issue (Module 4.1). You've tried rollback (Module 4.2). But the problem persists:
>
> - Controller keeps crashing during VPC reconciliation
> - Agent disconnects repeatedly despite correct configuration
> - Performance degradation with no obvious cause
>
> **It's time to engage Hedgehog support.**
>
> Beginners view support escalation as admitting defeat. Experts view it as **strategic resource utilization**:
> - Support has access to internal telemetry and engineering insights
> - Support can identify product bugs vs. configuration issues
> - Support provides workarounds while engineering fixes root cause
>
> **Early escalation with good diagnostics beats endless troubleshooting with incomplete information.**

**Context: Professional Support Collaboration**

Module 3.4 taught you **what to collect** (diagnostic checklist).

Module 4.3 teaches you **how to collaborate** with support engineers:
- Writing tickets that get fast responses
- Communicating effectively during investigation
- Providing follow-up diagnostics efficiently
- Understanding support ticket lifecycle

**What You'll Learn:**

- **Effective support ticket structure** (problem statement, environment, diagnostics, impact)
- **Communication best practices** (clear timelines, structured follow-ups)
- **Working with support engineers** (responding to questions, additional diagnostics)
- **Escalation lifecycle** (ticket states, expected timelines)
- **When escalation is strategic** (early escalation vs. endless self-troubleshooting)

**Scenario for This Module:**

You'll draft a complete support ticket for a real issue encountered in Module 4.1 (VPCAttachment reconciliation problem), including:
- Clear problem statement
- Complete diagnostic bundle
- Business impact assessment
- Timeline documentation

---

### Core Concepts (4-5 minutes)

#### Concept 1: Anatomy of an Effective Support Ticket

**What Makes a Good Support Ticket?**

Support engineers handle dozens of tickets daily. Good tickets get faster, more accurate responses.

**Good Ticket Characteristics:**
- **Concise problem statement** (1-2 sentences summary)
- **Specific resource names** (VPC name, switch name, exact error message)
- **Clear timeline** (when started, recent changes)
- **Complete diagnostics attached** (not just described)
- **Business impact stated** (severity, affected users)
- **Troubleshooting attempted documented** (what you've tried)

---

**Support Ticket Structure (Template):**

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

**Diagnostic Bundle:** hedgehog-diagnostics-20251016-103000.tar.gz

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

---

**Key Principles:**

1. **Problem statement first** - Support should understand the issue in 30 seconds
2. **Specifics over generalities** - "VPC vpc-prod on leaf-04" not "a VPC somewhere"
3. **Attach, don't describe** - Include actual log files, not summaries
4. **Show your work** - Document troubleshooting attempts (saves support time)
5. **State impact clearly** - Helps support prioritize appropriately

---

#### Concept 2: Communication Best Practices

**During Active Support Investigation:**

**Best Practice 1: Respond Promptly to Follow-Up Questions**

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

---

**Best Practice 2: Provide Context with Additional Diagnostics**

When sending requested information, include brief context.

**Example:**

Support requests: "Can you check if the issue reproduces after controller restart?"

**Good Response:**

```
Restarted controller at 2025-10-16 14:30 UTC.

Waited 15 minutes for reconciliation to complete.

Result: Issue still reproduces. VPCAttachment still fails to reconcile.

Attached: controller logs from restart (controller-post-restart.log)
```

**Poor Response:**

```
Still broken.
```

---

**Best Practice 3: Use Structured Updates for Timeline-Sensitive Issues**

For high-severity issues, provide periodic updates even if no progress.

**Example (Every 2 hours for P1):**

```
Update 14:00 UTC: Still investigating. Tried workaround X (did not resolve).
Collecting additional diagnostics per your request.

Update 16:00 UTC: Workaround Y applied. Issue partially resolved - 80% of
servers now working. 20% still affected. Awaiting root cause identification.
```

This shows you're actively engaged and helps support understand current state.

---

**Best Practice 4: Confirm Resolution and Document Solution**

When issue is resolved, confirm and document the solution.

**Example:**

```
Confirmed resolved as of 2025-10-16 17:00 UTC.

Solution applied: Updated VPC VLAN from 1050 to 1020 per your recommendation.
ArgoCD synced successfully. All 50 servers now have connectivity.

Root cause understood: VLAN 1050 conflicted with another VPC in same namespace.

Thank you for the fast diagnosis and clear resolution steps!

Ticket can be closed.
```

---

**Best Practice 5: Ask for Clarification When Needed**

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

**Don't:** Guess at what support meant and apply the wrong workaround.

---

#### Concept 3: Support Ticket Lifecycle

**Understanding Ticket States:**

**1. Submitted**
- Ticket created by customer
- Assigned to support queue
- Expected time to first response:
  - P1 (Critical): <1 hour
  - P2 (Major): <4 hours
  - P3 (Minor): <24 hours
  - P4 (Question): <48 hours

**2. Acknowledged**
- Support engineer assigned
- Initial response sent (may include follow-up questions)
- Customer action: Respond to questions promptly

**3. Investigating**
- Support analyzing diagnostics
- May request additional information
- May engage engineering team
- Customer action: Provide requested diagnostics

**4. Workaround Provided**
- Temporary solution identified
- Root cause investigation continues
- Customer action: Test and confirm workaround

**5. Root Cause Identified**
- Support determines underlying issue
- May be configuration error (customer-side fix)
- May be product bug (requires engineering fix)
- Customer action: Apply fix if configuration issue

**6. Resolved**
- Issue fixed
- Customer confirms resolution
- Ticket remains open for 48 hours for re-occurrence

**7. Closed**
- No re-occurrence after 48 hours
- Ticket archived
- Solution documented in knowledge base

---

**Typical Timelines by Severity:**

**P1 (Critical - Production Down):**
- First response: <1 hour
- Workaround target: <4 hours
- Resolution target: <24 hours
- Continuous engagement until resolved

**P2 (Major - Degraded Service):**
- First response: <4 hours
- Workaround target: <24 hours
- Resolution target: <5 business days

**P3 (Minor - Workaround Exists):**
- First response: <24 hours
- Resolution target: <10 business days

**P4 (Question - No Outage):**
- First response: <48 hours
- Resolution target: <15 business days

**Note:** These are typical targets. Actual timelines depend on issue complexity and support plan.

---

#### Concept 4: Escalation as Strategic Resource

**When to Escalate Early (Don't Wait):**

**Scenario 1: Repeated Failures After Rollback**

You've:
- Diagnosed issue (Module 4.1)
- Rolled back configuration (Module 4.2)
- Issue persists or new errors appear

**Action:** Escalate immediately. This suggests product bug or environment-specific issue requiring support insight.

---

**Scenario 2: Controller or Agent Crashes**

Symptoms:
- Controller CrashLoopBackOff
- Agent repeatedly disconnecting (not network issue)
- Panic messages in logs

**Action:** Escalate after collecting crash logs. These are rarely configuration issues.

---

**Scenario 3: Unexpected Behavior After Upgrade**

Symptoms:
- Fabric worked before upgrade
- Specific features broken after upgrade
- Configuration unchanged

**Action:** Escalate with "before upgrade" and "after upgrade" diagnostics. May be regression.

---

**Scenario 4: Time-Sensitive Production Issue**

Situation:
- Production fabric serving live traffic
- Issue impacting revenue or SLA
- 30+ minutes of troubleshooting without progress

**Action:** Escalate as P1 even if root cause unclear. Support can troubleshoot faster with internal tools.

---

**When to Self-Resolve Before Escalating:**

**Scenario 1: Configuration Error with Clear Error Message**

Example:
```
kubectl events: "Warning VLANConflict: VLAN 1020 already in use"
```

**Action:** Fix VLAN conflict yourself. Don't escalate configuration errors with obvious fixes.

---

**Scenario 2: Documentation Covers the Issue**

Example:
- "How do I configure DHCP relay?"
- Answer is in official documentation

**Action:** Check documentation first. Escalate if documentation is unclear or incorrect.

---

**Scenario 3: Issue Resolved by Standard Troubleshooting**

Example:
- Agent disconnected due to network issue
- Network fixed, agent reconnects
- Finalizer cleanup completes

**Action:** No escalation needed if issue self-resolved.

---

**Strategic Escalation Mindset:**

**Good:**
- "I've spent 2 hours troubleshooting, collected diagnostics, tried rollback. Issue persists. Escalating with complete diagnostic bundle."

**Bad:**
- "I've been troubleshooting for 3 days. Tried everything. Please help!" (No diagnostics attached)

**Better:**
- "Encountered controller crash after 30 minutes of troubleshooting. Crash logs attached. Escalating early to minimize downtime."

---

### Hands-On Lab (6-7 minutes)

**Lab Title:** Draft a Complete Support Ticket

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
2. Attach diagnostic bundle (simulated)
3. Practice professional communication

**Environment:**
- All diagnostic tools available (kubectl, Grafana, Git)

---

#### Task 1: Draft Support Ticket Problem Statement (2 minutes)

**Objective:** Write clear, concise problem summary

**Step 1.1: Complete Issue Summary Section**

Using the template from Concept 1, fill in:

**Problem:** VPCAttachment reconciled successfully but server has no connectivity. VLAN mismatch identified (VPC expects 1025, switch has 1020). Attempted fix by updating VPC to VLAN 1020, but ArgoCD sync rejected with "VLAN 1020 reserved for system use" error.

**Severity:** P2 (Major - Development environment, 1 server affected, no workaround identified)

**Impacted Resources:**
- VPC: customer-app-vpc
- VPCAttachment: customer-app-vpc-server-07
- Server: server-07
- Switch: leaf-04, interface Ethernet8

**Started:** 2025-10-16 10:00 UTC (VPCAttachment created)

**Recent Changes:**
- Created customer-app-vpc VPC with frontend subnet (VLAN 1025)
- Created VPCAttachment for server-07 to customer-app-vpc/frontend
- Attempted fix: Updated VPC YAML to use VLAN 1020 (Git commit a1b2c3d)

---

**Step 1.2: Document Environment**

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

---

**Success Criteria:**
- ✅ Problem statement is 1-3 sentences
- ✅ Severity appropriate for impact
- ✅ All affected resources listed by name
- ✅ Timeline clear (when started, when changes made)

---

#### Task 2: Document Symptoms and Diagnostics (2 minutes)

**Objective:** Provide clear symptoms and diagnostic evidence

**Step 2.1: Symptoms Section**

**Observable Symptoms:**
1. VPCAttachment shows "Ready" status, no error events
2. server-07 cannot ping gateway 10.20.10.1 ("Destination Host Unreachable")
3. Agent CRD shows VLAN 1020 on leaf-04/Ethernet8 (expected: 1025)
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
- Cannot use VLAN 1025 (conflict with another VPC)
- Cannot use VLAN 1020 (rejected as "reserved for system use")
- No available VLAN identified

---

**Step 2.2: Diagnostics Completed Section**

**Pre-Escalation Checklist:**
- [x] kubectl events checked - VPCAttachment shows no errors
- [x] Agent status reviewed - leaf-04 Ready, VLAN 1020 on Ethernet8
- [x] BGP health verified - all sessions established
- [x] Grafana dashboards checked - interface up, VLAN 1020 configured
- [x] Controller logs collected - no reconciliation errors for VPCAttachment
- [x] VPC/VPCAttachment CRDs collected
- [x] ArgoCD sync logs collected - shows VLAN 1020 rejection error

**Troubleshooting Attempted:**
1. Verified VPCAttachment references correct connection (server-07--unbundled--leaf-04)
   - Result: Connection reference correct
2. Checked if frontend subnet exists in customer-app-vpc
   - Result: Subnet exists, VLAN specified as 1025
3. Investigated VLAN mismatch (VPC expects 1025, switch has 1020)
   - Result: VLAN conflict with another VPC caused automatic allocation of 1020
4. Attempted fix: Updated VPC YAML to use VLAN 1020 (matching switch config)
   - Result: ArgoCD sync rejected - "VLAN 1020 reserved for system use"
5. Checked VLANNamespace for available VLANs
   - Result: Range 1000-2999, many VLANs appear unused
6. Attempted alternate VLAN (1026) in VPC YAML
   - Result: Not yet tested (waiting for guidance on VLAN reservation)

**Findings:**
- VLAN mismatch confirmed between VPC spec (1025) and switch config (1020)
- Cannot fix by updating VPC to match switch (VLAN 1020 rejected)
- Unclear which VLANs are actually "reserved for system use"
- Need guidance on VLAN selection or understanding of reservation error

---

**Success Criteria:**
- ✅ Symptoms numbered and specific
- ✅ Error messages copy/pasted exactly
- ✅ Troubleshooting steps documented with results
- ✅ Findings summarize current understanding

---

#### Task 3: Document Impact and Attach Diagnostics (1-2 minutes)

**Objective:** State business impact and reference diagnostic bundle

**Step 3.1: Impact Section**

**Users Affected:** 1 development server (server-07)

**Business Impact:** Development environment - blocks deployment testing for customer-app workload. Production not affected.

**Workaround:** None identified. Cannot use VLAN 1025 (conflict) or VLAN 1020 (reserved).

**Urgency:** Moderate - Development team blocked, but production unaffected. Resolution within 24 hours acceptable.

---

**Step 3.2: Attachments Section**

**Diagnostic Bundle:** hedgehog-diagnostics-20251016-140000.tar.gz (12 MB)

**Contents:**
- crds-vpc.yaml (all VPCs including customer-app-vpc)
- crds-vpcattachments.yaml (all VPCAttachments)
- crds-wiring.yaml (switches, servers, connections)
- crds-agents.yaml (all agent status)
- crds-namespaces.yaml (IPv4Namespace, VLANNamespace)
- events.log (all events, last 2 hours)
- events-warnings.log (Warning events only)
- controller.log (last 2000 lines)
- argocd-sync-error.log (sync failure details)
- agent-leaf-04.yaml (detailed switch status)
- agent-leaf-04-interfaces.json (interface state)
- grafana-screenshots/ (3 images)

**Grafana Screenshots:**
- fabric-dashboard.png - Overall fabric health (all BGP up)
- interfaces-leaf04-eth8.png - Shows interface up, VLAN 1020
- logs-argocd-error.png - Shows VLAN rejection error in logs

---

**Step 3.3: Additional Context**

**Additional Context:**

This is the first VPC for customer-app workload in this fabric. Previous VPCs (vpc-prod, vpc-staging) created without VLAN reservation errors.

Question for support:
- Which VLANs are reserved for system use?
- Is there a way to query available VLANs programmatically?
- Should VLANNamespace exclude reserved VLANs from range?

Related: VLANNamespace shows range 1000-2999, but error suggests some VLANs within range are reserved. Documentation doesn't mention VLAN reservations.

---

**Success Criteria:**
- ✅ Impact clearly stated (severity matches impact)
- ✅ Diagnostic bundle referenced with size and contents listed
- ✅ Screenshots described (what they show)
- ✅ Additional context provides useful background

---

#### Task 4: Review and Submit (1 minute)

**Objective:** Final review before submission

**Step 4.1: Review Ticket Quality**

Use this checklist:

- [ ] Problem statement is clear in first 30 seconds of reading
- [ ] All resource names are specific (not "a VPC" or "some server")
- [ ] Timeline documented (when issue started, when changes made)
- [ ] Symptoms are numbered and specific
- [ ] Error messages copy/pasted exactly (not paraphrased)
- [ ] Troubleshooting attempts documented with results
- [ ] Impact stated clearly (severity, affected users, urgency)
- [ ] Diagnostic bundle attached and contents listed
- [ ] Additional context provides useful background

---

**Step 4.2: Simulate Submission**

In a real environment, you would:

1. Log into support portal (e.g., support.hedgehog.com)
2. Click "Create New Ticket"
3. Paste ticket content
4. Attach diagnostic bundle (.tar.gz file)
5. Attach Grafana screenshots
6. Select severity (P2 in this case)
7. Click "Submit"

**Expected Response Time:** P2 = first response within 4 hours

---

**Success Criteria:**
- ✅ Ticket passes quality checklist
- ✅ All attachments referenced
- ✅ Ready for submission

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You successfully drafted a professional support ticket:
- ✅ Clear problem statement with timeline
- ✅ Comprehensive symptoms and diagnostics
- ✅ Documented troubleshooting attempts
- ✅ Appropriate severity and impact assessment
- ✅ Complete diagnostic bundle attached

**Key Takeaways:**

1. **Support is strategic, not a failure** - Early escalation with good diagnostics is professional
2. **Clarity accelerates resolution** - Specific details help support diagnose faster
3. **Attach, don't describe** - Actual logs better than summaries
4. **Show your work** - Document what you've tried (saves support time)
5. **Communication matters** - Professional, prompt responses build effective collaboration

**Working with Support Engineers:**

**Good Practices:**
- Respond to follow-ups within 4 hours
- Provide context with additional diagnostics
- Confirm resolution and document solution
- Ask for clarification when needed
- Thank support for assistance

**Avoid:**
- One-line tickets with no diagnostics ("It's broken, please help")
- Ignoring follow-up questions for days
- Assuming support knows your environment
- Escalating obvious configuration errors
- Providing partial information

**Escalation Mindset:**

> "Support has tools and knowledge I don't have access to. Early escalation with complete diagnostics is the fastest path to resolution."

**Preview of Module 4.4:**

Next, you'll learn **post-incident review practices**—conducting blameless reviews, documenting lessons learned, and building operational knowledge for continuous improvement.

---

### Assessment Questions

#### Question 1: Effective Support Tickets

**Scenario:** You're writing a support ticket for a VPC creation failure. Which problem statement is BEST?

**A:**
"VPC not working. Tried creating it multiple times. Please fix ASAP."

**B:**
"I created a VPC called customer-vpc and it's not working. The servers can't connect. I think it might be a BGP issue or maybe the switch is down. Can you investigate?"

**C:**
"VPC customer-vpc creation fails with error 'Subnet 10.20.1.0/24 overlaps with existing VPC prod-vpc'. Created 2025-10-16 11:00 UTC. No recent changes to prod-vpc. Diagnostics attached."

**D:**
"VPC customer-vpc has issues. See attached 50MB log dump. Figure out what's wrong."

<details>
<summary>Answer & Explanation</summary>

**Answer:** C

**Explanation:**

**Why C is correct:**

**Specific problem statement:**
- Exact resource name (customer-vpc)
- Exact error message (copy/pasted)
- Timeline (when created)
- Context (no recent changes to prod-vpc)
- References diagnostics

**Support can immediately understand:**
- What failed (VPC creation)
- Why it failed (subnet overlap)
- When it happened (date/time)
- Next step (check subnet allocation)

**Why others are wrong:**

**A) Too vague:**
- "VPC not working" - what VPC? What's not working?
- "Tried multiple times" - when? Same error each time?
- "Please fix ASAP" - no severity stated, no context
- No diagnostics

**B) Too verbose, lacks specifics:**
- Problem statement buried in explanation
- "I think it might be..." - speculation, not facts
- No exact error message
- "Can you investigate?" - support needs diagnostics first

**D) Dumps work on support:**
- "Has issues" - what issues specifically?
- 50MB log dump without context (support must search for issue)
- "Figure out what's wrong" - expects support to do all work

**Effective Ticket Principle:**
Support should understand the issue in **30 seconds** of reading. Specifics accelerate diagnosis.

**Module 4.3 Reference:** Concept 1 - Anatomy of an Effective Support Ticket
</details>

---

#### Question 2: Communication Best Practices

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
1. **Requested output attached** (agent-leaf-04-bgp.json file)
2. **Brief summary** (all neighbors established)
3. **Interpretation** (no down neighbors)
4. **Offer to help more** (additional diagnostics if needed)

**Benefits:**
- Support gets exact data requested
- Summary saves support time (can focus on next step)
- Shows you're actively engaged
- Prompts next step from support

**Why others are wrong:**

**A) Vague commitment:**
- "Eventually" - no timeline
- Support may wait hours/days for response
- Delays resolution
- Unprofessional

**B) Data without context:**
- Raw JSON paste is hard to read in ticket
- No summary (support must analyze from scratch)
- Missed opportunity to highlight findings

**D) Focuses on blocker instead of solution:**
- jq is optional (can run without it)
- Could paste raw output instead
- Could ask support for alternative command
- Better: "jq not available, here's raw output: [paste]"

**Communication Best Practice:**
Respond to support requests within **4 business hours** with:
- Requested data
- Brief summary
- Interpretation or context
- Offer to provide more if needed

**Module 4.3 Reference:** Concept 2 - Communication Best Practices (Best Practice 1 & 2)
</details>

---

#### Question 3: Strategic Escalation

**Scenario:** You're troubleshooting a VPCAttachment connectivity issue. You've spent 30 minutes and identified that the VPC subnet VLAN (1025) conflicts with another VPC. You need to choose a different VLAN. Should you escalate to support?

- A) Yes - escalate immediately, VLAN conflicts require support investigation
- B) No - this is a configuration error with a clear fix (choose different VLAN)
- C) Yes - any VLAN issue should be escalated
- D) No - but only if you've spent at least 4 hours troubleshooting first

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) No - this is a configuration error with a clear fix

**Explanation:**

**Self-Resolve Before Escalating:**

**VLAN conflict diagnosis:**
- kubectl events show: "Warning VLANConflict: VLAN 1025 already in use"
- Root cause clear: VPC spec uses same VLAN as another VPC
- Fix clear: Update VPC YAML to use different VLAN from VLANNamespace range

**Self-Resolution Steps:**
```bash
# Check available VLANs
kubectl get vpc -A -o yaml | grep "vlan:" | sort

# Choose unused VLAN (e.g., 1026)
# Update VPC YAML in Gitea
# Commit change
# Verify via ArgoCD sync
```

**Expected time to fix:** 10-15 minutes

**Why escalation is unnecessary:**
- Configuration error with obvious fix
- Documentation explains VLAN selection
- No unexpected behavior (conflict error expected when VLANs overlap)
- You have the tools to fix (Gitea access, kubectl)

**Why others are wrong:**

**A) Escalate immediately:**
- Wastes support time on fixable issue
- Support will respond: "Choose a different VLAN"
- Delays resolution (waiting for support vs. fixing now)

**C) Any VLAN issue escalated:**
- Too broad - many VLAN issues are configuration errors
- Only escalate VLAN issues when:
  * "Reserved for system use" error (unclear which VLANs reserved)
  * VLAN configured but not working (unexpected behavior)
  * VLANNamespace corruption

**D) 4 hours minimum:**
- Arbitrary time limit
- Should escalate based on **issue type**, not time spent
- 4 hours on obvious config error is wasted time

**Strategic Escalation Triggers:**

**Escalate:**
- Unexpected behavior (error doesn't explain how to fix)
- Product bugs (crashes, panics)
- Need internal tools/knowledge (engineering insight)
- Time-sensitive and no clear fix

**Self-Resolve:**
- Configuration errors with clear error messages
- Issues covered in documentation
- Standard troubleshooting resolved the issue

**Module 4.3 Reference:** Concept 4 - Escalation as Strategic Resource
</details>

---

#### Question 4: Support Ticket Lifecycle

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
- **Expected first response:** <4 hours
- **Your wait time:** 6 hours (exceeds target by 2 hours)

**Appropriate Follow-Up:**

Reply to ticket:

```
Following up on this P2 ticket submitted 6 hours ago.

Haven't received initial response yet (expected <4 hours for P2).

Issue remains: VPCAttachment connectivity failure due to VLAN conflict.

Please confirm ticket has been assigned to support engineer.

Let me know if any additional information needed to expedite investigation.

Thank you.
```

**Why this is appropriate:**
- Polite status request (not demanding)
- References expected SLA (4 hours for P2)
- Re-states issue briefly (in case ticket lost in queue)
- Offers to provide more info (shows willingness to help)
- Professional tone

**Why others are wrong:**

**A) Submit new ticket with P1:**
- **Wrong:** Severity inflation
- P1 is for production down (this is development issue)
- Creates duplicate tickets (confuses support)
- May get flagged for inappropriate severity
- Doesn't actually expedite response

**B) Wait another 6 hours:**
- Total 12 hours wait for P2 is excessive
- SLA already missed (4 hours target)
- Proactive follow-up is appropriate after 6 hours
- Silent waiting doesn't help

**D) Call account manager:**
- Too escalated for 6-hour delay
- Account manager will likely say: "Follow up on the ticket first"
- Reserve account manager escalation for:
  * P1 with no response after 2 hours
  * Repeated SLA misses
  * Support unresponsive after multiple follow-ups

**Best Practice Follow-Up Timeline:**

**P1 (Critical):**
- Expected: <1 hour
- Follow-up if no response: After 1.5 hours

**P2 (Major):**
- Expected: <4 hours
- Follow-up if no response: After 6 hours ← YOU ARE HERE

**P3 (Minor):**
- Expected: <24 hours
- Follow-up if no response: After 36 hours

**P4 (Question):**
- Expected: <48 hours
- Follow-up if no response: After 72 hours

**Professional Follow-Up:**
- Polite, not demanding
- References SLA
- Restates issue
- Offers to help
- Single follow-up every 4-6 hours (not spam)

**Module 4.3 Reference:** Concept 3 - Support Ticket Lifecycle
</details>

---

## Technical Requirements

### Support Ticket Tools

**Support Portal Access:**
- Typical URL: https://support.hedgehog.com (or vendor-specific)
- Authentication: SSO or username/password
- Features: Create ticket, attach files, view ticket history, update tickets

**Diagnostic Collection (From Module 3.4):**
```bash
# Use diagnostic collection script from Module 3.4
./diagnostic-collect.sh

# Or manual collection
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
OUTDIR="hedgehog-diagnostics-${TIMESTAMP}"
mkdir -p ${OUTDIR}

# Collect CRDs, events, logs, agent status
# (See Module 3.4 for complete commands)

# Compress
tar czf ${OUTDIR}.tar.gz ${OUTDIR}
```

**Grafana Screenshot Capture:**
- Navigate to relevant dashboard
- Adjust time range to show issue
- Use browser screenshot tool or Print to PDF
- Save with descriptive names (e.g., `fabric-dashboard-bgp-down.png`)

### Communication Templates

**Initial Ticket Template:**
(See Concept 1 for full template)

**Follow-Up Response Template:**
```markdown
[Support's question or request]

**Response:**

[Requested information or diagnostic output]

**Summary:** [Brief interpretation of findings]

**Additional Context:** [Any relevant observations]

Let me know if you need additional diagnostics.
```

**Resolution Confirmation Template:**
```markdown
Confirmed resolved as of [date/time].

**Solution Applied:** [What was done to fix]

**Root Cause:** [Understanding of why issue occurred]

**Testing Completed:**
- [Test 1 - result]
- [Test 2 - result]

Thank you for [specific help provided by support].

Ticket can be closed.
```

### Reference Documents

- **[Module 3.4 Design](./module-3.4-design.md)** - Pre-Support Diagnostic Checklist
- **[Module 4.1 Design](./module-4.1-design.md)** - Diagnosing Fabric Issues (provides context for example ticket)
- **[OBSERVABILITY.md](../research/OBSERVABILITY.md)** - Diagnostic collection procedures

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Support as Part of Learning** ⭐⭐⭐
- **How:** Entire module normalizes support escalation as professional practice
- **Example:** "Support is strategic, not a failure"
- **Why:** Reduces hesitation to escalate, builds collaborative mindset

#### 2. **Train for Reality, Not Rote** ⭐
- **How:** Realistic support ticket for actual issue (Module 4.1 scenario)
- **Example:** Students draft ticket for VLAN conflict with ArgoCD rejection
- **Why:** Prepares for real-world support interactions

#### 3. **Focus on What Matters Most** ⭐
- **How:** Emphasizes effective communication and clear documentation
- **Example:** Ticket structure optimized for fast support response
- **Why:** High-impact skill for production environments

#### 4. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on ticket drafting exercise
- **Example:** Students write complete ticket from scratch
- **Why:** Practice builds confidence for actual escalations

#### 5. **Teach the Why Behind the How** ⭐
- **How:** Explains why ticket structure matters (support perspective)
- **Example:** Why attach diagnostics vs. describe them
- **Why:** Understanding improves future ticket quality

#### 6. **Continuous Learning Over Static Mastery**
- **How:** Teaches escalation decision-making (when to escalate vs. self-resolve)
- **Example:** Strategic escalation scenarios
- **Why:** Builds judgment for evolving situations

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What They Bring:**
- Experience with ticketing systems (Jira, GitHub Issues)
- Familiarity with attaching logs and diagnostics

**What's New:**
- Fabric-specific diagnostics (Agent CRD, Grafana screenshots)
- Escalation decision-making (when to engage support)

**Bridge Strategy:**
- Relate support tickets to GitHub issues (familiar format)
- Use structured templates (comfort with markdown)

---

#### For Networking Professionals

**What They Bring:**
- Experience working with vendor support
- Understanding of severity levels and SLAs

**What's New:**
- Kubernetes-native diagnostics (kubectl outputs, CRDs)
- GitOps context in tickets (Git commits, ArgoCD sync)

**Bridge Strategy:**
- Relate to traditional vendor support workflows
- Emphasize diagnostic collection as familiar practice

---

### Common Challenges and Mitigation

#### Challenge 1: Hesitation to Escalate

**Stumbling Block:** Students feel escalation is "giving up"

**Mitigation:**
- Emphasize "Support as Strategic Resource" principle
- Assessment questions test escalation decision-making
- Provide clear escalation triggers

---

#### Challenge 2: Incomplete Tickets

**Stumbling Block:** Students submit vague tickets without diagnostics

**Mitigation:**
- Provide detailed template
- Lab requires complete ticket drafting
- Assessment tests ticket quality

---

#### Challenge 3: Poor Follow-Up Communication

**Stumbling Block:** Students don't respond promptly to support questions

**Mitigation:**
- Concept 2 teaches communication best practices
- Assessment tests response quality
- Emphasize 4-hour response target

---

### Confidence-Building Opportunities

**Win 1: Complete Ticket Draft**
- **Moment:** Task 3 - Ticket passes quality checklist
- **Feeling:** "I can write professional support tickets"
- **Teaching Point:** "This ticket would get fast support response"

**Win 2: Assessment Success**
- **Moment:** Correctly answering escalation decision questions
- **Feeling:** "I understand when to escalate vs. self-resolve"
- **Teaching Point:** "You're thinking strategically about support utilization"

---

## Dependencies

### Prerequisites (Must Complete First)

- ✅ **Module 3.4:** Pre-Support Diagnostic Checklist (what to collect)
- ✅ **Module 4.1:** Diagnosing Fabric Issues (troubleshooting skills)
- ✅ **Module 4.2:** Rollback & Recovery (attempted self-resolution)

**Why These Prerequisites Matter:**
- Module 4.3 builds on diagnostic collection from 3.4
- Escalation decisions require troubleshooting knowledge from 4.1
- "Tried rollback" is part of ticket documentation

---

### Enables (Unlocks These Modules)

- ✅ **Module 4.4:** Post-Incident Review (support collaboration documented in reviews)
- ✅ **HCFO Certification:** Complete troubleshooting and escalation competency

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
- ✅ **Content outline follows logical progression**
- ✅ **Assessment aligns with learning objectives**
- ✅ **Timing target is achievable (13-15 minutes)**

### Technical Accuracy

- ✅ **Support ticket template is realistic and complete**
- ✅ **Communication best practices are professional**
- ✅ **Escalation triggers are appropriate**

### Learning Philosophy

- ✅ **Embodies at least 3 core principles (embodies 6 of 10!)**
- ✅ **"Support as Part of Learning" is central theme**

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 4.2 (Rollback & Recovery - DESIGNED)
**Next Module:** Module 4.4 (Post-Incident Review - To Be Designed)

---

**Status:** ⏳ DESIGN COMPLETE - Ready for Review
**Related Issue:** GitHub Issue #11 - [DESIGN] Course 4: Troubleshooting, Recovery & Escalation (Modules 4.1-4.4)

---

**Module 4.3 Design Complete!**
Next: Module 4.4 - Post-Incident Review (Final Module)
