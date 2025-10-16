# Module 4.4 Design: Post-Incident Review

## Module Metadata

- **Module Number:** 4.4
- **Module Title:** Post-Incident Review
- **Course:** Course 4 - Troubleshooting, Recovery & Escalation
- **Estimated Duration:** 10-12 minutes
  - Introduction: 2 minutes
  - Core Concepts: 3-4 minutes
  - Hands-On Lab: 4-5 minutes
  - Wrap-Up & Course Completion: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 4.1 Complete (Diagnosing Fabric Issues)
  - ✅ Module 4.2 Complete (Rollback & Recovery)
  - ✅ Module 4.3 Complete (Coordinating with Support)
  - ✅ All previous courses complete (Courses 1-3)

## Learning Objectives

By the end of this module, learners will be able to:

1. **Conduct blameless post-incident reviews** - Facilitate reviews focused on learning, not blaming
2. **Document lessons learned** - Create actionable improvement items from incidents
3. **Update operational runbooks** - Improve documentation based on incident experiences
4. **Identify systemic improvements** - Recognize patterns that require process or tool changes
5. **Build operational knowledge** - Contribute to team learning and continuous improvement

**Bloom's Taxonomy Level**: Evaluate, Create (analyzing incidents, synthesizing learnings, creating documentation)

## Content Outline

### Introduction (2 minutes)

**Hook: Learning From What Went Wrong**

> You've completed an incident:
> - Diagnosed the issue (Module 4.1)
> - Rolled back the problematic change (Module 4.2)
> - Worked with support (Module 4.3)
> - Resolved the outage
>
> **Most teams stop here.** Incident resolved, move on to the next task.
>
> **High-performing teams add one more step:** Post-incident review.
>
> **Why?**
> - Incidents are expensive learning opportunities
> - Same issues recur if root causes aren't addressed
> - Team knowledge improves when insights are shared
> - Process gaps become visible through reflection

**Context: SRE Culture - Blameless Reviews**

Site Reliability Engineering (SRE) teaches:

> **"Failure is inevitable in complex systems. Learning from failure is optional."**

Post-incident reviews (PIRs) embody SRE principles:
- **Blameless culture** - Focus on systems and processes, not individuals
- **Continuous improvement** - Every incident improves operations
- **Shared learning** - Team knowledge grows through documentation
- **Forward-looking** - "How do we prevent this?" not "Who caused this?"

**What You'll Learn:**

- **Blameless post-incident review template** (timeline, root cause, prevention)
- **Facilitation techniques** (asking "why" without blaming)
- **Action item creation** (specific, measurable improvements)
- **Runbook updates** (documenting new procedures)
- **Operational knowledge management** (building team expertise)

**Scenario for This Module:**

You'll conduct a post-incident review for the VLAN conflict issue from Module 4.1/4.2:
- Document timeline
- Identify root cause
- Extract lessons learned
- Create 2-3 actionable improvements

---

### Core Concepts (3-4 minutes)

#### Concept 1: Blameless Culture

**What is a Blameless Culture?**

**Traditional approach:**
- "Who made the mistake?"
- "Why didn't they check before applying the change?"
- "This person needs retraining."

**Blameless approach:**
- "Why did the system allow this mistake?"
- "What process would have caught this earlier?"
- "How do we make the right thing easy to do?"

---

**Key Principles:**

**Principle 1: Systems Thinking**

Incidents result from **system failures**, not individual failures:
- Process gaps (no VLAN conflict checking before VPC creation)
- Tool limitations (VLANNamespace doesn't show which VLANs are in use)
- Documentation gaps (VLAN reservation not documented)
- Design issues (no validation that prevents conflicts)

**Human error is a symptom, not a root cause.**

---

**Principle 2: Forward-Looking Questions**

**Avoid:**
- "Who committed the broken YAML?"
- "Why didn't you test before pushing to prod?"

**Ask instead:**
- "What would have caught this error earlier in the process?"
- "How can we make VLAN selection less error-prone?"
- "What tools or checks would prevent this in the future?"

---

**Principle 3: Psychological Safety**

Teams with blameless culture:
- Report incidents honestly (don't hide issues)
- Share near-misses (learning opportunities)
- Ask for help early (not seen as weakness)
- Experiment with improvements (failure is acceptable)

**Without psychological safety:**
- Incidents get hidden (fear of blame)
- Knowledge isn't shared (protective behavior)
- Improvements don't happen (risk-averse culture)

---

#### Concept 2: Post-Incident Review Template

**PIR Structure (4 Sections):**

**Section 1: What Happened? (Timeline)**

Document the incident chronologically:

```
2025-10-16 10:00 UTC - VPCAttachment customer-app-vpc-server-07 created via Gitea commit
2025-10-16 10:05 UTC - ArgoCD synced VPCAttachment to cluster
2025-10-16 10:07 UTC - Developer reported: server-07 no connectivity
2025-10-16 10:15 UTC - Investigation started (checked kubectl events - no errors)
2025-10-16 10:30 UTC - Agent CRD checked: VLAN 1020 on switch (expected 1025)
2025-10-16 10:45 UTC - Root cause identified: VLAN conflict
2025-10-16 11:00 UTC - Attempted fix: Update VPC YAML to VLAN 1020
2025-10-16 11:05 UTC - ArgoCD sync failed: "VLAN 1020 reserved for system use"
2025-10-16 11:30 UTC - Escalated to support (P2 ticket)
2025-10-16 14:00 UTC - Support responded: VLAN 1020-1029 reserved, use 1030+
2025-10-16 14:15 UTC - Updated VPC to VLAN 1030, ArgoCD synced successfully
2025-10-16 14:20 UTC - Connectivity verified, incident resolved
```

**Goal:** Factual timeline, no interpretation yet.

---

**Section 2: Why Did It Happen? (Root Cause)**

Identify underlying cause (not just proximate cause):

**5 Whys Technique:**

1. **Why did server-07 have no connectivity?**
   - Because VPC VLAN (1025) didn't match switch VLAN (1020)

2. **Why didn't VPC VLAN match switch VLAN?**
   - Because VLAN 1025 was already in use, system auto-allocated 1020

3. **Why wasn't VLAN conflict detected before VPC creation?**
   - Because VLANNamespace doesn't validate VLAN availability at creation time

4. **Why doesn't VLANNamespace validate VLAN conflicts?**
   - Because it defines ranges, not tracks usage (design limitation)

5. **Why wasn't VLAN reservation (1020-1029) documented?**
   - Because system reservations aren't exposed via API or documentation

**Root Cause:** No pre-creation validation for VLAN conflicts. VLANNamespace allows VLAN selection from range without checking current usage or reservations.

**Contributing Factors:**
- Documentation gap: Reserved VLANs not listed
- Operator knowledge gap: Didn't know to check existing VPC VLANs manually
- Error message unclear: "Reserved for system use" but no list of reserved VLANs

---

**Section 3: How Was It Resolved? (Actions Taken)**

Document the resolution path:

**Immediate Actions:**
1. Diagnosed VLAN mismatch using Agent CRD inspection
2. Attempted self-resolution by updating VPC VLAN to 1020
3. Encountered "reserved VLAN" error
4. Escalated to support with complete diagnostics
5. Received clarification from support (VLANs 1020-1029 reserved)
6. Updated VPC to VLAN 1030
7. Verified connectivity restored

**Time to Resolution:** 4 hours 20 minutes (10:00 - 14:20 UTC)

**Mean Time to Detect (MTTD):** 7 minutes (10:00 creation - 10:07 reported)

**Mean Time to Resolve (MTTR):** 4 hours 13 minutes (10:07 detected - 14:20 resolved)

---

**Section 4: How Do We Prevent Recurrence? (Improvements)**

Actionable improvements categorized:

**Immediate (Do This Week):**
1. **Update runbook:** Document reserved VLAN ranges (1020-1029) in operator runbook
2. **Create VLAN selection checklist:**
   ```
   Before creating VPC:
   - Check existing VPCs: kubectl get vpc -A -o yaml | grep "vlan:" | sort
   - Avoid reserved VLANs: 1020-1029 (system use)
   - Choose from available VLANNamespace range: 1000-2999 (excluding above)
   ```
3. **Add VLAN conflict example to troubleshooting guide**

**Short-Term (Do This Month):**
4. **Request documentation update:** File issue for Hedgehog docs to list reserved VLANs
5. **Create kubectl helper script:**
   ```bash
   # show-available-vlans.sh
   kubectl get vpc -A -o yaml | grep "vlan:" | sort | uniq
   echo "Reserved: 1020-1029 (system)"
   echo "Available: Check VLANNamespace range and avoid above"
   ```

**Long-Term (Propose to Product Team):**
6. **Feature request:** VLANNamespace API should expose available VLANs (not just range)
7. **Feature request:** VPC creation should validate VLAN availability and return clear error
8. **Feature request:** Document reserved VLANs in VLANNamespace status field

---

#### Concept 3: Creating Actionable Improvements

**What Makes a Good Action Item?**

**Bad Action Items (Too Vague):**
- "Be more careful with VLAN selection"
- "Check things before creating VPCs"
- "Improve documentation"

**Good Action Items (Specific, Measurable, Owned):**
- "Document reserved VLANs 1020-1029 in operator runbook (Owner: Alice, Due: Oct 20)"
- "Create show-available-vlans.sh script and add to kubectl-fabric-helpers repo (Owner: Bob, Due: Oct 25)"
- "File Hedgehog GitHub issue requesting VLAN validation feature (Owner: Charlie, Due: Oct 18)"

---

**SMART Action Items:**

- **Specific:** What exactly will be done?
- **Measurable:** How will we know it's complete?
- **Actionable:** Can someone actually do this?
- **Relevant:** Does this prevent recurrence?
- **Time-bound:** When will it be done?

**Example:**

```
Action Item: Update operator runbook with reserved VLAN ranges

Owner: Alice
Due Date: 2025-10-20
Success Criteria:
  - Runbook section "VLAN Selection Guidelines" created
  - Reserved VLANs 1020-1029 documented
  - kubectl command to check existing VLANs included
  - Runbook committed to docs repository

Status: In Progress
```

---

#### Concept 4: Operational Knowledge Management

**Building Team Knowledge:**

Post-incident reviews create organizational memory:

**Artifacts:**
1. **PIR Document** - Incident history and lessons learned
2. **Updated Runbooks** - Procedures improved based on experience
3. **Troubleshooting Guides** - Common issues and solutions
4. **Known Issues List** - Workarounds for product limitations

**Knowledge Sharing:**
- Team meeting discussion (15 min PIR review)
- Slack/wiki post summarizing learnings
- New operator onboarding (review past PIRs)
- Quarterly review of recurring incident patterns

---

**Continuous Improvement Cycle:**

```
Incident Occurs
    ↓
Troubleshoot & Resolve
    ↓
Post-Incident Review
    ↓
Document Lessons Learned
    ↓
Update Runbooks & Processes
    ↓
Share Knowledge with Team
    ↓
Implement Improvements
    ↓
Monitor for Recurrence
    ↓
(Fewer incidents over time)
```

**Goal:** Each incident makes the next one less likely or easier to resolve.

---

### Hands-On Lab (4-5 minutes)

**Lab Title:** Conduct Post-Incident Review

**Scenario:**

You'll conduct a PIR for the incident from Module 4.1/4.2:

**Incident Summary:**
- VPCAttachment created, but server had no connectivity
- Root cause: VLAN mismatch (VPC expected 1025, switch had 1020)
- Resolution: Updated VPC to use VLAN 1030 after support clarified reserved range
- Duration: ~4 hours

**Your Task:**
1. Document timeline (Section 1)
2. Identify root cause using 5 Whys (Section 2)
3. Create 2-3 actionable improvements (Section 4)

---

#### Task 1: Document Timeline (1-2 minutes)

**Objective:** Create factual chronological timeline

**Step 1.1: List Key Events**

Using information from Module 4.1/4.2, list events in order:

**Timeline Template:**

```
YYYY-MM-DD HH:MM UTC - [Event description]
```

**Example Events to Include:**
- VPCAttachment created
- Issue reported by user
- Investigation started
- Root cause identified
- Attempted self-resolution (failed)
- Escalated to support
- Support response received
- Resolution applied
- Connectivity verified

**Your Timeline:**

```
2025-10-16 10:00 UTC - [Your event]
2025-10-16 10:05 UTC - [Your event]
...
2025-10-16 14:20 UTC - [Your event]
```

**Success Criteria:**
- ✅ At least 8 timeline entries
- ✅ Events in chronological order
- ✅ Timestamps included (even if estimated)
- ✅ Factual (no blame language)

---

#### Task 2: Root Cause Analysis (1-2 minutes)

**Objective:** Use 5 Whys to identify root cause

**Step 2.1: Ask Five Whys**

Start with the symptom, ask "why" repeatedly:

**1. Why did server-07 have no connectivity?**

Your answer: _______________________________________

**2. Why [answer to #1]?**

Your answer: _______________________________________

**3. Why [answer to #2]?**

Your answer: _______________________________________

**4. Why [answer to #3]?**

Your answer: _______________________________________

**5. Why [answer to #4]?**

Your answer: _______________________________________

---

**Step 2.2: State Root Cause**

Based on your 5 Whys, write root cause statement:

**Root Cause:** _______________________________________

**Contributing Factors:**
- _______________________________________
- _______________________________________

**Success Criteria:**
- ✅ Root cause is systemic (process/tool gap, not "human error")
- ✅ Contributing factors identified
- ✅ Root cause would prevent recurrence if addressed

---

#### Task 3: Create Actionable Improvements (1-2 minutes)

**Objective:** Define 2-3 SMART action items

**Step 3.1: Brainstorm Improvements**

What could prevent this incident from recurring?

**Categories:**
- Documentation updates
- Runbook additions
- Scripts or tools
- Feature requests
- Process changes

**Your Ideas:**
1. _______________________________________
2. _______________________________________
3. _______________________________________

---

**Step 3.2: Make Action Items SMART**

Choose 2-3 improvements and make them SMART:

**Action Item 1:**
```
Title: _______________________________________
Owner: _______________________________________
Due Date: _______________________________________
Description: _______________________________________
Success Criteria: _______________________________________
```

**Action Item 2:**
```
Title: _______________________________________
Owner: _______________________________________
Due Date: _______________________________________
Description: _______________________________________
Success Criteria: _______________________________________
```

**Success Criteria:**
- ✅ Each action item has owner and due date
- ✅ Description is specific (not vague)
- ✅ Success criteria are measurable
- ✅ Action items would prevent recurrence

---

#### Task 4: Update Personal Runbook (Optional, 1 minute)

**Objective:** Document VLAN selection procedure for future use

**Step 4.1: Add to Personal Troubleshooting Runbook**

Create or update your runbook with:

**Section: VPC Creation - VLAN Selection**

```markdown
## VLAN Selection for New VPCs

**Objective:** Choose non-conflicting VLAN from VLANNamespace range

**Procedure:**

1. Check VLANNamespace range:
   ```bash
   kubectl get vlannamespace default -o jsonpath='{.spec.ranges}'
   # Expected: [{"from":1000,"to":2999}]
   ```

2. Check existing VPC VLANs:
   ```bash
   kubectl get vpc -A -o yaml | grep "vlan:" | sort | uniq
   ```

3. Avoid reserved VLANs:
   - 1020-1029: System reserved

4. Choose available VLAN from range (1000-2999, excluding above)

5. Document chosen VLAN in VPC design notes

**Troubleshooting:**
- If "VLAN conflict" error: Choose different VLAN
- If "VLAN reserved" error: Avoid 1020-1029 range
- If unclear which VLANs available: Escalate to support
```

**Success Criteria:**
- ✅ Runbook section created
- ✅ Procedure is step-by-step
- ✅ Troubleshooting tips included

---

### Wrap-Up & Course Completion (2 minutes)

**What You Accomplished in Module 4.4:**

You conducted a blameless post-incident review:
- ✅ Documented factual timeline
- ✅ Identified root cause using 5 Whys
- ✅ Created actionable improvements
- ✅ Updated personal runbook

**What You Accomplished in Course 4:**

- **Module 4.1:** Diagnosed fabric issues using systematic troubleshooting
- **Module 4.2:** Rolled back changes and recovered from failures
- **Module 4.3:** Coordinated with support professionally
- **Module 4.4:** Conducted post-incident review for continuous improvement

**Key Takeaways - Course 4:**

1. **Systematic troubleshooting beats random checking** - Hypothesis-driven investigation
2. **GitOps rollback is safe and auditable** - Git history is source of truth
3. **Support is strategic, not a failure** - Early escalation with good diagnostics
4. **Blameless reviews build better systems** - Focus on process, not people
5. **Every incident is a learning opportunity** - Continuous improvement mindset

---

**What You Accomplished in Entire Pathway:**

**Course 1: Foundations & Interfaces**
- Understood Hedgehog architecture and control model
- Mastered three interfaces: Gitea, kubectl, Grafana
- Experienced GitOps workflow

**Course 2: Provisioning & Connectivity**
- Designed and deployed VPCs
- Attached servers to VPCs
- Validated connectivity
- Decommissioned resources safely

**Course 3: Observability & Fabric Health**
- Queried Prometheus metrics
- Interpreted Grafana dashboards
- Monitored Agent CRD status
- Collected diagnostics for support

**Course 4: Troubleshooting, Recovery & Escalation**
- Diagnosed fabric issues systematically
- Rolled back changes safely
- Coordinated with support effectively
- Conducted post-incident reviews

---

**You Are Now a Hedgehog Certified Fabric Operator (HCFO)**

**What This Means:**

You can **confidently operate** a Hedgehog fabric:
- Provision network resources (VPCs, attachments, peerings)
- Monitor fabric health (Grafana, kubectl, events)
- Troubleshoot issues (diagnosis, rollback, recovery)
- Collaborate with support (escalation, communication)
- Improve operations (post-incident reviews, runbooks)

**You are NOT:**
- A fabric architect (design large fabrics from scratch)
- A Hedgehog developer (contribute to Hedgehog codebase)
- A networking expert (deep BGP/EVPN/VXLAN knowledge)

**But you ARE:**
- **Competent** - Can perform daily operations independently
- **Confident** - Trust your troubleshooting methodology
- **Collaborative** - Know when and how to engage support
- **Continuous learner** - Improve operations with each incident

---

**Next Steps:**

**Immediate:**
1. Practice in lab environment (repeat modules if needed)
2. Review runbooks and create personal troubleshooting guides
3. Set up diagnostic collection scripts
4. Bookmark Grafana dashboards and kubectl cheat sheets

**First 30 Days:**
1. Shadow senior operators during incidents
2. Conduct PIRs for any issues you encounter (even small ones)
3. Build personal runbook with procedures and lessons learned
4. Contribute to team knowledge sharing

**Ongoing:**
1. Stay current with Hedgehog releases (version upgrades)
2. Review Hedgehog documentation updates
3. Participate in team post-incident reviews
4. Share your learnings with new operators

---

**Thank You for Completing This Pathway!**

You've learned to **network like a hyperscaler**—not by memorizing switch commands, but by mastering declarative infrastructure, observability, and operational excellence.

**The Hedgehog Learning Philosophy in Action:**

- ✅ **Trained for reality** - Production-like scenarios throughout
- ✅ **Focused on what matters** - Common, high-impact operations
- ✅ **Built confidence** - Small wins leading to competence
- ✅ **Learned by doing** - Hands-on labs in every module
- ✅ **Support as strength** - Normalized escalation and collaboration
- ✅ **Continuous learning** - Mindset for ongoing improvement

**You're ready to operate Hedgehog fabrics with confidence. Welcome to the community of Hedgehog Certified Fabric Operators!**

---

### Assessment Questions

#### Question 1: Blameless Culture

**Scenario:** During a post-incident review, a team member says: "This outage happened because Alice pushed the wrong YAML to Git. She should have tested it first."

What is the BEST blameless response?

- A) "You're right, Alice should have been more careful."
- B) "Let's focus on Alice's training plan to prevent this in the future."
- C) "This isn't Alice's fault, it's the system's fault for allowing the push."
- D) "Let's discuss what process or validation would have caught this error before it reached production."

<details>
<summary>Answer & Explanation</summary>

**Answer:** D) "Let's discuss what process or validation would have caught this error before it reached production."

**Explanation:**

**Why D is correct:**

**Blameless approach characteristics:**
- **Systems thinking:** Focus on process gap, not individual mistake
- **Forward-looking:** "How do we prevent this?" not "Why did this happen?"
- **Constructive:** Leads to actionable improvements

**Example improvements this question might reveal:**
- Pre-commit validation (git hook to validate YAML syntax)
- Staging environment (test changes before production)
- Peer review (GitOps pull request approval workflow)
- Automated testing (CI/CD pipeline validates YAML)

**Why others are wrong:**

**A) Agrees with blame:**
- Focuses on individual, not system
- "More careful" is vague, not actionable
- Doesn't prevent recurrence
- Creates fear of making mistakes

**B) Focuses on individual training:**
- Implies Alice lacks knowledge (blaming)
- Training doesn't address system gap
- What if Alice was trained but made typo? Still blame?
- Doesn't prevent others from making same mistake

**C) Dismisses human role entirely:**
- "System's fault" is too broad
- Doesn't lead to specific improvements
- Oversimplifies root cause

**Blameless Principle:**

> **"Human error is a symptom of systemic issues. Fix the system, not the human."**

**Effective PIR Discussion:**

1. **Acknowledge:** "Alice pushed incorrect YAML, incident occurred." (factual)
2. **Ask:** "What validation could have caught this earlier?" (system focus)
3. **Identify:** "No pre-commit YAML validation exists." (process gap)
4. **Improve:** "Implement git hook to validate YAML syntax before push." (actionable)

**Module 4.4 Reference:** Concept 1 - Blameless Culture
</details>

---

#### Question 2: Root Cause Analysis

**Scenario:** Using the 5 Whys technique, which of these statements represents a ROOT CAUSE (vs. proximate cause)?

- A) "Server had no connectivity because the VLAN was wrong."
- B) "The VLAN was wrong because the operator chose VLAN 1025."
- C) "The operator chose VLAN 1025 because they didn't check existing VPC VLANs."
- D) "VLANNamespace doesn't validate VLAN conflicts at creation time, allowing operators to select conflicting VLANs."

<details>
<summary>Answer & Explanation</summary>

**Answer:** D) "VLANNamespace doesn't validate VLAN conflicts at creation time"

**Explanation:**

**Why D is the root cause:**

**Root cause characteristics:**
- **Systemic:** System design limitation (not individual action)
- **Actionable:** Can be addressed with feature request or workaround
- **Preventive:** Fixing this prevents recurrence

**If you fix this root cause:**
- VLANNamespace API validates VLANs at creation
- Operators get immediate error if VLAN conflicts
- No need to manually check existing VPCs
- System prevents the mistake

**Why others are proximate causes (symptoms):**

**A) "VLAN was wrong":**
- **What happened** (symptom), not **why it was allowed** (cause)
- Doesn't explain how to prevent

**B) "Operator chose VLAN 1025":**
- Human action (proximate cause)
- Fixing: "Choose different VLAN" doesn't prevent next operator from same mistake

**C) "Didn't check existing VLANs":**
- Human action (proximate cause)
- Fixing: "Check existing VLANs" is manual, error-prone
- Doesn't prevent recurrence (relies on human memory)

**5 Whys Example:**

1. **Why no connectivity?** VLAN mismatch
2. **Why VLAN mismatch?** Operator chose conflicting VLAN
3. **Why choose conflicting VLAN?** Didn't check existing VPCs
4. **Why didn't check?** No process required it
5. **Why no process?** → **ROOT CAUSE: System doesn't enforce validation**

**Root Cause Test:**

> "If we fix this, would the same incident be impossible (or much less likely)?"

- Fix D: Yes, system prevents VLAN conflicts automatically
- Fix A, B, C: No, same mistake can happen again

**Module 4.4 Reference:** Concept 2 - Post-Incident Review Template (Section 2: Root Cause)
</details>

---

#### Question 3: SMART Action Items

**Scenario:** Which of these action items is BEST (most SMART)?

**A:**
"Be more careful when selecting VLANs for new VPCs."

**B:**
"Update documentation about VLANs."

**C:**
"Document reserved VLAN ranges (1020-1029) in operator runbook section 'VLAN Selection', including kubectl command to check existing VLANs. Owner: Alice. Due: Oct 20, 2025."

**D:**
"Someone should probably write down the VLAN stuff somewhere so we don't forget."

<details>
<summary>Answer & Explanation</summary>

**Answer:** C

**Explanation:**

**Why C is SMART:**

**Specific:**
- What: Document reserved VLAN ranges (1020-1029)
- Where: Operator runbook section "VLAN Selection"
- Include: kubectl command to check existing VLANs

**Measurable:**
- Done when runbook section exists with reserved VLANs and kubectl command

**Actionable:**
- Alice can do this (has access to runbook, knows kubectl commands)

**Relevant:**
- Prevents VLAN conflict recurrence (addresses root cause)

**Time-bound:**
- Due: Oct 20, 2025 (clear deadline)

**Verification:**
```bash
# Check if action item is complete:
grep "1020-1029" operator-runbook.md
grep "kubectl get vpc" operator-runbook.md
# If both found in "VLAN Selection" section → complete
```

**Why others are not SMART:**

**A) "Be more careful":**
- ❌ Not specific (how to "be more careful"?)
- ❌ Not measurable (how do you verify "more careful"?)
- ❌ Not actionable (vague advice, not concrete action)
- ❌ Not time-bound (when?)
- ❌ Not owned (who?)

**B) "Update documentation":**
- ❌ Not specific (which documentation? What content?)
- ❌ Not measurable (how much is "updated"?)
- Partially actionable (could update docs, but what specifically?)
- ❌ Not time-bound (when?)
- ❌ Not owned (who?)

**D) "Someone should probably":**
- ❌ Not specific ("VLAN stuff" - what exactly?)
- ❌ Not measurable ("somewhere" - where?)
- ❌ Not actionable ("probably" - not committed)
- ❌ Not time-bound (when?)
- ❌ Not owned ("someone" - who?)

**SMART Action Item Template:**

```
Title: [Specific action verb] [What] [Where/How]
Owner: [Person name]
Due Date: [YYYY-MM-DD]
Description: [Detailed steps]
Success Criteria: [How to verify complete]
```

**Module 4.4 Reference:** Concept 3 - Creating Actionable Improvements
</details>

---

#### Question 4: Continuous Improvement

**Scenario:** Your team has conducted 5 post-incident reviews this month. All 5 incidents were VLAN-related configuration errors. What is the BEST next step?

- A) Accept that VLAN issues are common and continue current process
- B) Recognize pattern and prioritize systemic improvement (e.g., VLAN validation tooling)
- C) Require all operators to re-read documentation about VLANs
- D) Add more warning labels to the Gitea repository

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Recognize pattern and prioritize systemic improvement

**Explanation:**

**Why B is correct:**

**Pattern Recognition:**
- 5 incidents, same root cause → **systemic issue**, not isolated mistakes
- Current process (manual VLAN selection) is error-prone
- Individual training won't prevent recurrence

**Systemic Improvements for VLAN Issues:**

1. **Short-term (do immediately):**
   - Create kubectl helper script: `show-available-vlans.sh`
   - Add VLAN selection checklist to runbook
   - Require peer review for VPC changes

2. **Medium-term (do this month):**
   - Develop pre-commit git hook to validate VLAN conflicts
   - Create CI/CD pipeline to test VPC YAML before merge

3. **Long-term (propose to product team):**
   - Feature request: VLANNamespace API validates conflicts
   - Feature request: VPC creation returns clear error for conflicts
   - Feature request: Web UI for VLAN selection (shows available VLANs)

**Prioritization:**
- 5 incidents/month = high impact
- VLAN validation tooling would prevent most/all recurrences
- Worth investing time in systemic fix vs. repeating same PIR

**Why others are wrong:**

**A) Accept as common:**
- ❌ Defeatist mindset (not continuous improvement)
- ❌ Ignores pattern (5 incidents = systemic issue)
- ❌ Incidents will continue to occur
- ❌ Team morale suffers from repeated failures

**C) Re-read documentation:**
- ❌ Training didn't prevent first 5 incidents, won't prevent next
- ❌ Assumes knowledge gap (but 5 different incidents suggests process gap)
- ❌ Manual process still error-prone
- ❌ Doesn't scale (new operators will make same mistakes)

**D) More warnings:**
- ❌ Warning fatigue (more labels = less attention)
- ❌ Doesn't prevent errors (people still make mistakes despite warnings)
- ❌ Reactive (adds warnings) vs. proactive (prevents errors)

**Continuous Improvement Principle:**

> **"Recurring incidents indicate systemic issues. Fix the system, not the symptoms."**

**SRE Best Practice:**

When you see patterns:
1. **Count incidents by root cause** (track recurring issues)
2. **Prioritize by impact** (frequency × severity)
3. **Invest in systemic fixes** (tooling, automation, validation)
4. **Measure improvement** (track incident reduction)

**Example Metrics:**

- **Before:** 5 VLAN incidents/month
- **After tooling:** 0-1 VLAN incidents/month (80-100% reduction)
- **ROI:** Time invested in tooling < time spent on 5 PIRs

**Module 4.4 Reference:** Concept 4 - Operational Knowledge Management (Continuous Improvement Cycle)
</details>

---

## Technical Requirements

### Post-Incident Review Template

**Complete PIR Template (Markdown):**

```markdown
# Post-Incident Review: [Incident Title]

**Date:** YYYY-MM-DD
**Facilitator:** [Name]
**Participants:** [Names]
**Incident Duration:** [Start time] - [End time] ([X hours Y minutes])

---

## Executive Summary

**What Happened:** [1-2 sentence summary]

**Impact:**
- Users Affected: [Number/description]
- Services Affected: [List]
- Duration: [X hours]
- Severity: [P1/P2/P3/P4]

**Root Cause:** [1 sentence]

**Resolution:** [1 sentence]

---

## Section 1: Timeline (What Happened?)

| Time (UTC) | Event | Who/What |
|------------|-------|----------|
| 10:00 | [Event description] | [Person/System] |
| 10:05 | [Event description] | [Person/System] |
| ... | ... | ... |

**Key Metrics:**
- **Time to Detect (TTD):** [X minutes]
- **Time to Resolve (TTR):** [X hours Y minutes]
- **Mean Time Between Failures (MTBF):** [X days since last similar incident]

---

## Section 2: Root Cause (Why Did It Happen?)

**5 Whys Analysis:**

1. **Why [symptom]?**
   - [Answer]

2. **Why [answer to 1]?**
   - [Answer]

3. **Why [answer to 2]?**
   - [Answer]

4. **Why [answer to 3]?**
   - [Answer]

5. **Why [answer to 4]?**
   - [Answer] ← ROOT CAUSE

**Root Cause Statement:**

[Detailed root cause description]

**Contributing Factors:**
- [Factor 1]
- [Factor 2]
- [Factor 3]

---

## Section 3: Resolution (How Was It Resolved?)

**Immediate Actions Taken:**

1. [Action taken] - [Result] - [Time]
2. [Action taken] - [Result] - [Time]
...

**What Worked Well:**
- [Positive observation]
- [Positive observation]

**What Could Be Improved:**
- [Area for improvement]
- [Area for improvement]

---

## Section 4: Prevention (How Do We Prevent Recurrence?)

**Action Items:**

| ID | Action | Owner | Due Date | Priority | Status |
|----|--------|-------|----------|----------|--------|
| 1  | [Specific action] | [Name] | YYYY-MM-DD | High | Pending |
| 2  | [Specific action] | [Name] | YYYY-MM-DD | Medium | Pending |
| 3  | [Specific action] | [Name] | YYYY-MM-DD | Low | Pending |

**Action Item Details:**

**AI-1: [Title]**
- **Description:** [Detailed description]
- **Success Criteria:** [How to verify complete]
- **Notes:** [Additional context]

**AI-2: [Title]**
- **Description:** [Detailed description]
- **Success Criteria:** [How to verify complete]
- **Notes:** [Additional context]

---

## Lessons Learned

**Technical Learnings:**
- [Learning 1]
- [Learning 2]

**Process Learnings:**
- [Learning 1]
- [Learning 2]

**Documentation Updates Needed:**
- [Runbook update]
- [Troubleshooting guide update]

---

## Appendix

**Related Incidents:**
- [Link to previous similar incident]

**References:**
- [Link to support ticket]
- [Link to GitOps commit]
- [Link to monitoring dashboard]

**Attachments:**
- diagnostic-bundle.tar.gz
- grafana-screenshots/
```

---

### Facilitation Techniques

**Blameless Facilitation Tips:**

**1. Set the Tone (Opening):**

> "This is a blameless review. We're here to learn from the system, not to assign fault. Our goal is to make incidents like this less likely in the future."

**2. Use Neutral Language:**

**Avoid:**
- "Who caused the outage?"
- "Why didn't you check before deploying?"
- "This person should have known better."

**Use instead:**
- "What happened in the system?"
- "What process would have caught this earlier?"
- "How do we make the right thing easy to do?"

**3. Redirect Blame to System:**

If participant says: "Alice made a mistake."

Redirect: "Let's focus on what allowed the mistake. What validation could have prevented this?"

**4. Celebrate Learning:**

> "Great insight! This is exactly the kind of learning that makes us better. Let's document this so the team benefits."

**5. Focus on Specific Actions:**

Vague: "We should be more careful."
Specific: "Let's add pre-commit validation to catch YAML errors."

---

### Knowledge Management Tools

**Runbook Structure:**

```
operator-runbook/
├── vpc-operations/
│   ├── vpc-creation.md
│   ├── vlan-selection.md ← Updated from PIR
│   ├── vpc-deletion.md
│   └── troubleshooting.md
├── connectivity/
│   ├── attachment-creation.md
│   ├── peering-setup.md
│   └── troubleshooting.md
├── diagnostics/
│   ├── kubectl-commands.md
│   ├── grafana-dashboards.md
│   └── agent-crd-inspection.md
└── post-incident-reviews/
    ├── 2025-10-16-vlan-conflict.md ← This PIR
    ├── 2025-10-10-bgp-flap.md
    └── template.md
```

**PIR Repository:**

```bash
# Store PIRs in Git repository
post-incident-reviews/
├── README.md (index of all PIRs)
├── 2025-10-16-vlan-conflict.md
├── 2025-10-15-controller-crash.md
└── recurring-issues.md (tracks patterns)
```

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Continuous Learning Over Static Mastery** ⭐⭐⭐
- **How:** Entire module focuses on learning from incidents
- **Example:** Post-incident review creates organizational memory
- **Why:** Embodies continuous improvement culture

#### 2. **Support as Part of Learning** ⭐
- **How:** Blameless culture encourages psychological safety
- **Example:** "Asking for help early" is celebrated, not stigmatized
- **Why:** Creates environment where escalation is normalized

#### 3. **Focus on What Matters Most** ⭐
- **How:** Pattern recognition identifies high-impact improvements
- **Example:** 5 VLAN incidents → prioritize VLAN validation tooling
- **Why:** Invests effort where it reduces most incidents

#### 4. **Train for Reality, Not Rote** ⭐
- **How:** Realistic PIR for actual incident from Module 4.1
- **Example:** Students practice writing PIR they'd use in production
- **Why:** Transferable skill for real incidents

#### 5. **Teach the Why Behind the How**
- **How:** Explains blameless culture rationale (SRE principles)
- **Example:** Why 5 Whys leads to systemic improvements
- **Why:** Understanding creates culture change, not just process compliance

---

### Common Challenges and Mitigation

#### Challenge 1: Blame Mindset

**Stumbling Block:** Students default to "who caused it" thinking

**Mitigation:**
- Concept 1 explicitly teaches blameless culture
- Assessment Question 1 tests blameless responses
- Lab requires systemic root cause (not individual blame)

---

#### Challenge 2: Stopping at Proximate Cause

**Stumbling Block:** Students identify symptom as root cause

**Mitigation:**
- Concept 2 teaches 5 Whys technique
- Assessment Question 2 tests root cause vs. proximate cause
- Lab requires completing all 5 Whys

---

#### Challenge 3: Vague Action Items

**Stumbling Block:** Students write unmeasurable improvements

**Mitigation:**
- Concept 3 teaches SMART action items
- Assessment Question 3 tests SMART criteria
- Lab requires specific, owned, time-bound items

---

### Confidence-Building Opportunities

**Win 1: Completed PIR**
- **Moment:** Task 3 - All action items are SMART
- **Feeling:** "I can conduct professional PIRs"
- **Teaching Point:** "This PIR would drive real improvements"

**Win 2: Course Completion**
- **Moment:** Wrap-up - Realize full pathway competency
- **Feeling:** "I'm a Hedgehog Certified Fabric Operator"
- **Teaching Point:** "You're ready to operate production fabrics confidently"

---

## Dependencies

### Prerequisites (Must Complete First)

- ✅ **Module 4.1:** Diagnosing Fabric Issues (incident to review)
- ✅ **Module 4.2:** Rollback & Recovery (resolution actions)
- ✅ **Module 4.3:** Coordinating with Support (escalation included in timeline)
- ✅ **All previous courses:** Foundation for operational competency

**Why These Prerequisites Matter:**
- Module 4.4 reviews incidents from 4.1/4.2 (provides context)
- PIR documents entire incident lifecycle (diagnosis → resolution → escalation)
- Requires understanding of all previous troubleshooting and recovery concepts

---

### Enables (Unlocks These Paths)

- ✅ **HCFO Certification:** Completes pathway requirements
- ✅ **Production Operations:** Ready for independent fabric operation
- ✅ **Advanced Pathways:** Foundation for architect or SRE pathways (future)
- ✅ **Continuous Learning:** Mindset for ongoing improvement

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
- ✅ **Content outline follows logical progression**
- ✅ **Assessment aligns with learning objectives**
- ✅ **Timing target is achievable (10-12 minutes)**

### Technical Accuracy

- ✅ **PIR template follows SRE best practices**
- ✅ **5 Whys technique correctly applied**
- ✅ **SMART criteria accurately defined**

### Learning Philosophy

- ✅ **Embodies at least 3 core principles (embodies 5 of 10!)**
- ✅ **"Continuous Learning Over Static Mastery" is central theme**
- ✅ **Forward-looking, reflective tone**

### Course Completion

- ✅ **Closes Course 4 and entire pathway**
- ✅ **Celebrates student achievement**
- ✅ **Provides clear next steps**

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 4.3 (Coordinating with Support - DESIGNED)
**Next Module:** None (Pathway Complete!)

---

**Status:** ⏳ DESIGN COMPLETE - Ready for Review
**Related Issue:** GitHub Issue #11 - [DESIGN] Course 4: Troubleshooting, Recovery & Escalation (Modules 4.1-4.4)

---

**🎉 MODULE 4.4 DESIGN COMPLETE!**

**🎊 COURSE 4 DESIGN COMPLETE!**

**🏆 ENTIRE PATHWAY DESIGN COMPLETE (16 MODULES)!**

---

## Course 4 Summary

**All 4 Modules Designed:**

- ✅ **Module 4.1:** Diagnosing Fabric Issues (15-17 min)
  - Systematic troubleshooting methodology
  - Common failure modes and decision trees
  - Hands-on lab: Diagnose VLAN mismatch

- ✅ **Module 4.2:** Rollback & Recovery (13-15 min)
  - GitOps rollback workflows
  - Safe deletion order and finalizers
  - Hands-on lab: Revert VPC configuration change

- ✅ **Module 4.3:** Coordinating with Support (13-15 min)
  - Effective support ticket writing
  - Communication best practices
  - Hands-on lab: Draft complete support ticket

- ✅ **Module 4.4:** Post-Incident Review (10-12 min)
  - Blameless culture and 5 Whys
  - SMART action items
  - Hands-on lab: Conduct PIR for VLAN incident

**Total Course 4 Duration:** 51-59 minutes (~55 minutes average)

**Course 4 Learning Arc:**
Diagnosis → Recovery → Collaboration → Continuous Improvement

**Ready for Course Lead Review!**
