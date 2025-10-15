---
name: Module Design Specification
about: Detailed design for a single module
title: '[DESIGN] Module X.X: '
labels: 'phase-2-design, module-design'
assignees: ''
---

## Module Metadata

- **Module Number:** <!-- e.g., 1.1, 2.3 -->
- **Module Title:**
- **Course:** <!-- Which course does this belong to? -->
- **Estimated Duration:** 15 minutes (target)
- **Prerequisites:**
  - [ ] Prerequisite 1
  - [ ] Prerequisite 2

## Learning Objectives

By the end of this module, learners will be able to:
1. <!-- Specific, measurable objective -->
2. <!-- Specific, measurable objective -->
3. <!-- Specific, measurable objective -->
4. <!-- Optional: 4th objective -->
5. <!-- Optional: 5th objective -->

## Content Outline

### Introduction (2-3 minutes)
**Hook:**
<!-- How do we grab learner attention? Real-world scenario, pain point, or compelling question -->

**Context:**
<!-- Why this topic matters, how it fits in learning journey -->

### Core Concepts (5-7 minutes)
**Concept 1:**
<!-- Key idea, explanation, examples -->

**Concept 2:**
<!-- Key idea, explanation, examples -->

**Concept 3:** (optional)
<!-- Key idea, explanation, examples -->

### Hands-On Lab (5-7 minutes)
**Overview:**
<!-- What will learners do in this lab? -->

**Tasks:**
1. Task 1 with validation
2. Task 2 with validation
3. Task 3 with validation

### Wrap-Up & Assessment (1-2 minutes)
**Key Takeaways:**
- Takeaway 1
- Takeaway 2
- Takeaway 3

**Preview Next Module:**
<!-- Bridge to next topic -->

---

## Lab Exercise Specification

### Lab Environment
**Required Resources:**
- [ ] Hedgehog vlab (default topology)
- [ ] kubectl access
- [ ] Specific CRDs: <!-- list -->
- [ ] Other tools: <!-- hhfab, text editor, etc. -->

**Setup Steps:**
```bash
# Pre-lab validation commands
kubectl cluster-info
kubectl get switches
# etc.
```

### Lab Tasks

#### Task 1: <!-- Title -->
**Objective:** <!-- What learner accomplishes -->

**Steps:**
```bash
# Step 1
kubectl ...

# Expected output:
# <output>

# Step 2
kubectl ...
```

**Validation:**
```bash
# How to confirm success
kubectl get ...
# Expected: <criteria>
```

**Success Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2

#### Task 2: <!-- Title -->
<!-- Same structure as Task 1 -->

#### Task 3: <!-- Title -->
<!-- Same structure as Task 1 -->

### Troubleshooting Hints
**Common Issue 1:**
- **Symptom:**
- **Cause:**
- **Solution:**

**Common Issue 2:**
- **Symptom:**
- **Cause:**
- **Solution:**

---

## Assessment Design

### Quiz Questions (3-5 questions)

**Question 1:** (Multiple Choice)
<!-- Question text -->

- A) <!-- Option -->
- B) <!-- Option -->
- C) <!-- Option -->
- D) <!-- Option -->

**Correct Answer:** <!-- Letter -->
**Explanation:** <!-- Why this is correct, why others are wrong -->

**Question 2:** (Scenario-Based)
<!-- Scenario description -->
<!-- Question -->

**Answer:** <!-- Expected response -->
**Rubric:** <!-- How to grade if open-ended -->

**Question 3:** (True/False or Multiple Choice)
<!-- Question -->

**Answer:**
**Explanation:**

### Practical Assessment
**Task:** <!-- Hands-on task to demonstrate competency -->
**Success Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2

---

## Technical Requirements

### Hedgehog CRDs Used
- [ ] VPC - [Reference](../research/CRD_REFERENCE.md#vpc)
- [ ] VPCAttachment - [Reference](../research/CRD_REFERENCE.md#vpcattachment)
- [ ] <!-- Other CRDs -->

### kubectl Commands
```bash
# Primary commands taught/used
kubectl get vpc
kubectl describe vpc <name>
kubectl apply -f vpc.yaml
# etc.
```

**Reference:** [WORKFLOWS.md](../research/WORKFLOWS.md) - Workflow X

### Other Tools
- [ ] hhfab - <!-- specific commands -->
- [ ] Text editor (vim/nano)
- [ ] <!-- Other tools -->

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized
- [ ] **Train for Reality, Not Rote** - <!-- How? -->
- [ ] **Focus on What Matters Most** - <!-- How? -->
- [ ] **Confidence Before Comprehensiveness** - <!-- How? -->
- [ ] **Abstraction as Empowerment** - <!-- How? -->
- [ ] **Learn by Doing, Not Watching** - <!-- How? -->
- [ ] **Teach the Why Behind the How** - <!-- How? -->
- [ ] **Modularity & Composability** - <!-- How? -->
- [ ] **Bridging Two Worlds** - <!-- How? -->
- [ ] **Support as Part of Learning** - <!-- How? -->
- [ ] **Continuous Learning** - <!-- How? -->

### Target Audience Considerations

**For Cloud-Native Learners:**
<!-- What background knowledge can we assume? What needs explanation? -->

**For Networking Professionals:**
<!-- What background knowledge can we assume? What needs explanation? -->

**Bridge:**
<!-- How does this module help both audiences understand each other's domain? -->

### Common Challenges
**Challenge 1:**
- **Stumbling Block:** <!-- What might trip learners up? -->
- **Mitigation:** <!-- How do we prevent or address this? -->

**Challenge 2:**
- **Stumbling Block:**
- **Mitigation:**

### Confidence-Building Opportunities
<!-- Where in this module do learners get "wins" that build confidence? -->
1.
2.
3.

---

## Dependencies

### Prerequisites (Must Complete First)
- [ ] Module X.X - <!-- Reason -->
- [ ] Module X.X - <!-- Reason -->

### Enables (Unlocks These Modules)
- [ ] Module X.X - <!-- What this module prepares for -->
- [ ] Module X.X

### Related Modules (Complementary, Not Required)
- [ ] Module X.X - <!-- Connection -->

---

## Quality Checklist

### Design Quality
- [ ] Learning objectives are specific and measurable
- [ ] Content outline follows logical progression
- [ ] Lab exercise has clear success criteria
- [ ] Assessment aligns with learning objectives
- [ ] 15-minute timing target is achievable

### Technical Accuracy
- [ ] All CRD examples validated in vlab
- [ ] kubectl commands tested and correct
- [ ] Workflows match CRD_REFERENCE.md and WORKFLOWS.md
- [ ] No technical errors or outdated information

### Learning Philosophy
- [ ] Embodies at least 3 core principles
- [ ] Focuses on high-impact, common tasks
- [ ] Builds confidence through achievable goals
- [ ] Hands-on > passive learning
- [ ] Explains "why" not just "how"

### Accessibility
- [ ] Clear to cloud-native learners
- [ ] Clear to networking professionals
- [ ] Jargon explained or avoided
- [ ] Examples are relevant to both audiences

### Completeness
- [ ] All required sections filled out
- [ ] Lab exercise is detailed and testable
- [ ] Assessment has answers and explanations
- [ ] Dependencies mapped
- [ ] Troubleshooting hints provided

---

## Review & Approval

### Design Review
- [ ] Course lead review (Claude)
- [ ] Technical validation (Dev agent)
- [ ] Timing test (actual 15min run-through)
- [ ] Peer feedback incorporated

### Approval
- [ ] Ready for Phase 3 (Content Development)
- [ ] Approved by: <!-- Name/date -->

---

## Notes & Iterations

<!-- Track design iterations, feedback, decisions made -->
