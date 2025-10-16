---
name: Module Validation Testing
about: Technical validation and timing test for module design
title: '[VALIDATION] Module X.X: '
labels: 'phase-2-design, validation, testing'
assignees: ''
---

## Validation Metadata

- **Module Number:** <!-- e.g., 1.3 -->
- **Module Title:**
- **Design Document:** <!-- Link to design doc -->
- **Design Version:** <!-- e.g., v1.0 -->
- **Validator:** <!-- Dev agent or engineer name -->
- **Date:**

## Validation Objectives

This validation ensures:
- [ ] All technical commands execute successfully
- [ ] Lab timing is accurate (target vs actual)
- [ ] Environment requirements are met
- [ ] Assessment questions are clear and answerable
- [ ] Lab instructions are complete and unambiguous
- [ ] Success criteria are achievable

---

## Environment Validation

### Pre-Flight Checks
```bash
# Verify all required services are running
# Document results below
```

**Required Infrastructure:**
- [ ] Gitea: http://localhost:3001 - Status: <!--  -->
- [ ] ArgoCD: http://localhost:8080 - Status: <!--  -->
- [ ] Grafana: http://localhost:3000 - Status: <!--  -->
- [ ] kubectl: CLI access - Status: <!--  -->
- [ ] VLAB: Switches operational - Status: <!--  -->

**Pre-existing Resources:**
- [ ] List required pre-existing resources
- [ ] Verify each exists

---

## Lab Validation

### Part/Task 1: <!-- Name -->
**Target Duration:** <!-- X minutes -->

**Commands Tested:**
```bash
# Command 1
# Result:

# Command 2
# Result:
```

**Actual Duration:** <!-- X.X minutes -->
**Status:** ✅ PASS / ⚠️ PASS with notes / ❌ FAIL
**Issues Found:**
- Issue 1 (if any)
- Issue 2 (if any)

**Recommendations:**
- Recommendation 1
- Recommendation 2

---

### Part/Task 2: <!-- Name -->
**Target Duration:** <!-- X minutes -->

**Commands Tested:**
```bash
# Commands and results
```

**Actual Duration:** <!-- X.X minutes -->
**Status:** ✅ PASS / ⚠️ PASS with notes / ❌ FAIL
**Issues Found:**
**Recommendations:**

---

### Part/Task 3: <!-- Name -->
<!-- Same structure -->

---

## Timing Analysis

| Part/Task | Target | Actual | Variance | Status |
|-----------|--------|--------|----------|--------|
| Part 1    | X min  | X min  | +/- X    | ✅/⚠️/❌ |
| Part 2    | X min  | X min  | +/- X    | ✅/⚠️/❌ |
| Part 3    | X min  | X min  | +/- X    | ✅/⚠️/❌ |
| **Total** | **X**  | **X**  | **+/-X** | **Status** |

**Timing Assessment:**
- [ ] Within 15-minute module target
- [ ] Realistic for target audience
- [ ] Buffer time for troubleshooting

**Revised Timing Estimate:** <!-- If different from design -->

---

## Technical Validation

### Commands Validation
**All kubectl commands:**
- [ ] Command 1 - Status:
- [ ] Command 2 - Status:
- [ ] Command 3 - Status:

**Issues Found:**
- Issue 1: Description, fix recommendation
- Issue 2: Description, fix recommendation

### Expected vs Actual Output
**Discrepancies:**
- [ ] None - all outputs match design
- [ ] Minor differences (document below)
- [ ] Critical differences requiring design update

**Details:**
<!-- Document any output mismatches -->

### CRD/Resource Validation
**Resources created/modified:**
- [ ] Resource 1 - Status:
- [ ] Resource 2 - Status:

**Validation commands:**
```bash
# Verification commands run
```

---

## Assessment Validation

### Question Testing
**Question 1:**
- [ ] Clear and unambiguous
- [ ] Answerable from module content
- [ ] Correct answer verified

**Question 2:**
- [ ] Clear and unambiguous
- [ ] Answerable from module content
- [ ] Correct answer verified

<!-- Repeat for all questions -->

**Issues Found:**
- Question X: Issue description

---

## Critical Findings

### 🔴 CRITICAL Issues (Block approval)
1. **Issue Title**
   - **Problem:** Description
   - **Impact:** Why this is critical
   - **Fix Required:** What needs to change

### 🟡 Warnings (Non-blocking, should fix)
1. **Issue Title**
   - **Problem:** Description
   - **Impact:** Impact on student experience
   - **Recommendation:** Suggested fix

### ✅ Observations (Nice to have)
1. **Observation**
   - **Note:** Description
   - **Suggestion:** Optional improvement

---

## Infrastructure Status

### System Health
- [ ] All services remained stable during testing
- [ ] No unexpected errors or crashes
- [ ] Network connectivity consistent

**Issues Encountered:**
- Issue 1 (if any)

---

## Required Design Updates

### Critical Updates (Must fix before content development)
- [ ] Update 1: Description
- [ ] Update 2: Description

### Recommended Updates (Should fix)
- [ ] Update 1: Description
- [ ] Update 2: Description

### Optional Enhancements
- [ ] Enhancement 1: Description

---

## Validation Decision

**Overall Result:** ✅ APPROVE / ⚠️ APPROVE with modifications / ❌ NEEDS REVISION

**Rationale:**
<!-- Why this decision? What were the key factors? -->

**What Works Well:**
- ✅ Strength 1
- ✅ Strength 2
- ✅ Strength 3

**What Needs Work:**
- ⚠️ Issue 1
- ⚠️ Issue 2

**Readiness Assessment:**
- Technical foundation: <!-- ✅/⚠️/❌ + explanation -->
- Design document: <!-- ✅/⚠️/❌ + explanation -->
- Infrastructure: <!-- ✅/⚠️/❌ + explanation -->
- Workflow: <!-- ✅/⚠️/❌ + explanation -->

---

## Next Steps

### Immediate Actions
1. Action 1
2. Action 2

### Recommendations
1. Recommendation 1
2. Recommendation 2

### Ready for:
- [ ] Design document updates
- [ ] Phase 3 Content Development
- [ ] Re-validation (if major issues found)

---

## Validation Artifacts

**Test Evidence:**
- [ ] Commands logged
- [ ] Screenshots captured (if applicable)
- [ ] Timing recorded
- [ ] All outputs documented

**Files Created During Testing:**
<!-- List any test files/resources created -->

**Cleanup Completed:**
- [ ] Test resources removed
- [ ] Environment reset for next module

---

## Validator Sign-Off

**Validator:** <!-- Name -->
**Date:** <!-- YYYY-MM-DD -->
**Status:** <!-- PASS/FAIL/NEEDS REVISION -->
**Recommendation:** <!-- APPROVE/APPROVE WITH CHANGES/REJECT -->

**Course Lead Review Requested:** <!-- Yes/No -->
