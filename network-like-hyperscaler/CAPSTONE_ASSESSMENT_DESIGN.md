# Capstone Assessment Design: Hedgehog Certified Fabric Operator (HCFO)

**Pathway:** Network Like a Hyperscaler with Hedgehog
**Certification:** Hedgehog Certified Fabric Operator (HCFO)
**Assessment Type:** Hands-on practical exam with scenario-based tasks
**Duration:** 75-90 minutes
**Passing Score:** 70% (70/100 points)

---

## Assessment Overview

**Purpose:** Validate that learners have achieved operational competency across all 4 courses through integrated, real-world scenarios.

**Format:** Hands-on lab environment with pre-configured issues and tasks that require applying skills from all 16 modules.

**Assessment Philosophy:**
- **Realistic scenarios** - Tasks mirror actual fabric operator responsibilities
- **Integrated skills** - Requires combining knowledge from multiple modules
- **Confidence validation** - Tests ability to operate independently, not memorization
- **Support-friendly** - Includes escalation scenarios (Module 4.3 competency)

---

## Capstone Scenario

**Context:** You are a newly hired fabric operator at a company running Hedgehog Open Network Fabric. Your manager is going on vacation and needs you to:
1. Provision a new VPC for the platform team
2. Diagnose and fix a connectivity issue in the production VPC
3. Monitor fabric health and escalate appropriately

**Environment:**
- Hedgehog VLAB with default topology (2 spines, 5 leaves, 10 servers)
- Existing `prod-vpc` with known issue (pre-seeded)
- kubectl, Gitea, Grafana, ArgoCD access
- 90 minutes to complete all tasks

---

## Assessment Structure

### Part 1: Provisioning (25 points)
**Duration:** 20-25 minutes
**Competencies Tested:** Course 1 (GitOps), Course 2 (VPC provisioning & attachment)

### Part 2: Troubleshooting (35 points)
**Duration:** 30-35 minutes
**Competencies Tested:** Course 3 (Observability), Course 4 (Diagnosis & rollback)

### Part 3: Observability & Escalation (25 points)
**Duration:** 15-20 minutes
**Competencies Tested:** Course 3 (Grafana dashboards), Course 4 (Support coordination)

### Part 4: Post-Incident Review (15 points)
**Duration:** 10-15 minutes
**Competencies Tested:** Course 4 (PIR, continuous improvement)

---

## Part 1: VPC Provisioning (25 points)

### Task 1.1: Design VPC Configuration (8 points)

**Scenario:** The platform team requests a new VPC with the following requirements:
- **Name:** `platform-vpc`
- **Subnets:**
  - `api-gateway` - 10.30.10.0/24, VLAN 1100, DHCP enabled (range .10-.100), not isolated
  - `app-servers` - 10.30.20.0/24, VLAN 1101, DHCP enabled (range .10-.200), isolated from api-gateway
  - `database` - 10.30.30.0/28, VLAN 1102, DHCP disabled, fully isolated
- **Permit traffic:** api-gateway ↔ app-servers, app-servers ↔ database
- **IPv4Namespace:** `default`
- **VLANNamespace:** `default`

**Task:** Create `platform-vpc.yaml` file with correct VPC CRD specification

**Grading Rubric:**
- [ ] 2 pts - VPC metadata correct (name, namespace, labels)
- [ ] 2 pts - All 3 subnets defined with correct CIDR, gateway, VLAN
- [ ] 2 pts - DHCP configuration correct (enabled/disabled, ranges)
- [ ] 1 pt - Isolation flags correct (isolated: true/false)
- [ ] 1 pt - Permit list allows required traffic flows

**Module Mapping:** Module 2.1 (Define VPC Network)

---

### Task 1.2: Attach Servers to VPC (10 points)

**Scenario:** Attach the following servers to `platform-vpc` subnets:
- `server-01` → `api-gateway` subnet (MCLAG connection via leaf-01/leaf-02)
- `server-03` → `app-servers` subnet (ESLAG connection via leaf-03/leaf-04)
- `server-05` → `database` subnet (unbundled connection via leaf-05)

**Task:**
1. Create 3 VPCAttachment YAML files
2. Commit to Gitea repository
3. Wait for ArgoCD sync
4. Verify attachments are Ready

**Grading Rubric:**
- [ ] 3 pts - All 3 VPCAttachment files created with correct metadata
- [ ] 3 pts - Connection references are accurate (server-01--mclag--leaf-01--leaf-02, etc.)
- [ ] 2 pts - Subnet references use correct format (platform-vpc/api-gateway)
- [ ] 1 pt - nativeVLAN setting appropriate (false for tagged VLANs)
- [ ] 1 pt - GitOps workflow followed (Gitea → ArgoCD sync → verification)

**Module Mapping:** Module 2.2 (Attach Servers to VPC), Module 1.2 (GitOps workflow)

---

### Task 1.3: Validate Connectivity (7 points)

**Task:**
1. Verify DHCP assignments for api-gateway and app-servers
2. Test connectivity: server-01 ping server-03 (should succeed per permit list)
3. Test isolation: server-01 ping server-05 (should fail - no permit)
4. Document validation results

**Grading Rubric:**
- [ ] 2 pts - DHCP validation using kubectl or server inspection
- [ ] 2 pts - Successful connectivity test (api-gateway ↔ app-servers)
- [ ] 2 pts - Successful isolation test (api-gateway ✗ database)
- [ ] 1 pt - Results documented clearly (commands + outputs)

**Module Mapping:** Module 2.3 (Connectivity Validation)

---

## Part 2: Troubleshooting (35 points)

### Pre-Seeded Issue:

**Environment State:**
- Existing `prod-vpc` with 3 servers attached
- **Issue:** `server-07` in `prod-vpc/frontend` subnet cannot ping gateway (10.10.10.1)
- kubectl events show no errors
- VPCAttachment shows Ready status
- **Root Cause (hidden):** VLAN mismatch - VPC spec says VLAN 1050, Agent CRD shows VLAN 1051

---

### Task 2.1: Diagnose Connectivity Issue (15 points)

**Scenario:** Your manager left a note: "server-07 in prod-vpc can't reach the gateway. Please investigate and fix."

**Task:** Use systematic troubleshooting (Module 4.1) to identify the root cause

**Grading Rubric:**
- [ ] 3 pts - Check kubectl events for prod-vpc and VPCAttachment (hypothesis testing)
- [ ] 3 pts - Inspect Agent CRD for switch connected to server-07 (find VLAN mismatch)
- [ ] 3 pts - Compare VPC spec VLAN vs. Agent CRD interface VLAN (identify discrepancy)
- [ ] 2 pts - Check Grafana Interface Dashboard to validate findings
- [ ] 2 pts - Document root cause clearly ("VLAN mismatch: VPC expects 1050, switch has 1051")
- [ ] 2 pts - Identify correct solution approach (update VPC VLAN to 1051 or investigate why mismatch)

**Module Mapping:** Module 4.1 (Diagnosing Fabric Issues), Module 3.3 (Agent CRD inspection)

---

### Task 2.2: Execute Rollback (12 points)

**Scenario:** Investigation reveals that yesterday someone manually changed the VPC VLAN from 1050 to 1051 via kubectl (bypassing GitOps), but Gitea still shows 1050. The switch reconciled to 1051, but now there's drift.

**Task:**
1. Restore GitOps alignment by updating Gitea to match actual state (VLAN 1051)
2. Verify ArgoCD sync completes
3. Validate server-07 connectivity restored

**Grading Rubric:**
- [ ] 3 pts - Identify configuration drift (Gitea shows 1050, cluster has 1051)
- [ ] 3 pts - Update prod-vpc.yaml in Gitea with VLAN 1051
- [ ] 2 pts - Commit change with clear message
- [ ] 2 pts - Verify ArgoCD sync completes successfully
- [ ] 2 pts - Validate connectivity restored (server-07 pings gateway)

**Module Mapping:** Module 4.2 (Rollback & Recovery - GitOps restoration), Module 1.2 (GitOps workflow)

---

### Task 2.3: Document Troubleshooting (8 points)

**Task:** Create a brief troubleshooting summary documenting:
1. Symptoms observed
2. Diagnostic steps taken
3. Root cause identified
4. Resolution applied

**Grading Rubric:**
- [ ] 2 pts - Symptoms clearly stated (server-07 no gateway connectivity)
- [ ] 2 pts - Diagnostic steps listed in order (events → Agent CRD → Grafana)
- [ ] 2 pts - Root cause accurately described (VLAN mismatch due to kubectl drift)
- [ ] 2 pts - Resolution documented (GitOps restoration)

**Module Mapping:** Module 4.1 (Troubleshooting methodology)

---

## Part 3: Observability & Escalation (25 points)

### Task 3.1: Fabric Health Assessment (12 points)

**Task:** Use Grafana dashboards to assess fabric health and answer questions:

1. **Fabric Dashboard:** Are all BGP sessions established? (2 pts)
2. **Interfaces Dashboard:** Identify any interfaces with errors in last 1 hour (3 pts)
3. **Platform Dashboard:** Are any switches showing high CPU (>80%)? (2 pts)
4. **Logs Dashboard:** Are there any ERROR logs from fabric controller? (3 pts)
5. **Documentation:** Screenshot 2 dashboards showing healthy state (2 pts)

**Grading Rubric:**
- [ ] 2 pts - BGP session health correctly assessed
- [ ] 3 pts - Interface errors identified (if any) with switch/interface names
- [ ] 2 pts - CPU usage assessed for all switches
- [ ] 3 pts - Controller errors identified and logged
- [ ] 2 pts - Screenshots attached showing dashboard state

**Module Mapping:** Module 3.2 (Dashboard Interpretation), Module 3.3 (Status Monitoring)

---

### Task 3.2: Pre-Escalation Diagnostic Collection (13 points)

**Scenario:** While investigating, you notice the fabric controller has logged 3 errors related to "VLANNamespace exhaustion" in the past hour. You've checked VLANNamespace and it shows range 1000-1099 with 95 VPCs created. You're not sure if this is the cause of the earlier issue or a new problem.

**Task:** Using Module 3.4 checklist, prepare diagnostic bundle for support escalation

**Grading Rubric:**
- [ ] 2 pts - Collect kubectl get vpc -A -o yaml > vpcs.yaml
- [ ] 2 pts - Collect kubectl get events --field-selector type=Warning > warnings.log
- [ ] 2 pts - Collect controller logs: kubectl logs -n fab deployment/fabric-controller-manager --tail=500 > controller.log
- [ ] 2 pts - Collect VLANNamespace: kubectl get vlannamespace default -o yaml > vlannamespace.yaml
- [ ] 2 pts - Screenshot Grafana Fabric Dashboard
- [ ] 3 pts - Write brief escalation summary explaining:
   - Issue: VLANNamespace exhaustion warnings
   - Environment: 95 VPCs, range 1000-1099
   - Impact: Unclear if affecting operations
   - Question: Is range exhaustion cause of VLAN mismatch? Should we expand range?

**Module Mapping:** Module 3.4 (Pre-Support Diagnostic Checklist), Module 4.3 (Support coordination)

---

## Part 4: Post-Incident Review (15 points)

### Task 4.1: Conduct Blameless PIR (15 points)

**Scenario:** Now that the server-07 connectivity issue is resolved, conduct a post-incident review using Module 4.4 template.

**Task:** Complete PIR document with:
1. Timeline of events
2. Root cause (5 Whys)
3. Resolution actions
4. 2-3 actionable improvements

**Grading Rubric:**
- [ ] 3 pts - Timeline accurate (issue reported → diagnosis → fix → verification)
- [ ] 4 pts - Root cause uses 5 Whys reaching systemic issue (not just "VLAN wrong")
   - Example systemic cause: "No validation that kubectl changes match Gitea (configuration drift possible)"
- [ ] 2 pts - Resolution clearly documented
- [ ] 4 pts - Action items are SMART (Specific, Measurable, Actionable, Relevant, Time-bound)
   - Example: "Implement git pre-commit hook to validate VLAN ranges (Owner: Student, Due: Next week)"
   - Example: "Document reserved VLANs in operator runbook (Owner: Student, Due: 2 days)"
- [ ] 2 pts - Blameless tone (focuses on system, not individual)

**Module Mapping:** Module 4.4 (Post-Incident Review)

---

## Scoring Summary

| Part | Task | Points | Competencies Tested |
|------|------|--------|---------------------|
| **Part 1** | **Provisioning** | **25** | **GitOps, VPC Design, Attachment, Validation** |
| 1.1 | Design VPC Configuration | 8 | Module 2.1 |
| 1.2 | Attach Servers to VPC | 10 | Modules 2.2, 1.2 |
| 1.3 | Validate Connectivity | 7 | Module 2.3 |
| **Part 2** | **Troubleshooting** | **35** | **Diagnosis, Rollback, Documentation** |
| 2.1 | Diagnose Connectivity Issue | 15 | Modules 4.1, 3.3 |
| 2.2 | Execute Rollback | 12 | Modules 4.2, 1.2 |
| 2.3 | Document Troubleshooting | 8 | Module 4.1 |
| **Part 3** | **Observability & Escalation** | **25** | **Grafana, Diagnostics, Support** |
| 3.1 | Fabric Health Assessment | 12 | Modules 3.2, 3.3 |
| 3.2 | Pre-Escalation Diagnostics | 13 | Modules 3.4, 4.3 |
| **Part 4** | **Post-Incident Review** | **15** | **PIR, Continuous Improvement** |
| 4.1 | Conduct Blameless PIR | 15 | Module 4.4 |
| **TOTAL** | | **100** | **All 16 modules** |

**Passing Score:** 70/100 points (70%)

---

## Certification Levels

**Hedgehog Certified Fabric Operator (HCFO):**
- **Passing (70-84%):** Competent operator, can perform daily tasks with occasional support
- **Proficient (85-94%):** Strong operator, can handle most scenarios independently
- **Exemplary (95-100%):** Expert operator, ready for advanced responsibilities

**Certificate includes:**
- Score percentage
- Completion date
- Competency level (Passing, Proficient, Exemplary)
- Digital badge for LinkedIn/resume

---

## Assessment Environment Setup

### Pre-Assessment Setup (Proctor/System):

1. **VLAB State:**
   - Default topology running
   - Pre-create `prod-vpc` with server-07 VLAN mismatch (seeded issue)
   - Ensure VLANNamespace shows 95 VPCs in range 1000-1099
   - Create "VLANNamespace exhaustion" errors in controller logs

2. **Gitea:**
   - Repository: `student/hedgehog-config`
   - Branch: `main`
   - Contains: existing prod-vpc.yaml (VLAN 1050)
   - Student has write access

3. **ArgoCD:**
   - Application: `hedgehog-config` (auto-sync enabled)
   - Points to student Gitea repository

4. **Grafana:**
   - All 6 Hedgehog dashboards loaded
   - Data shows healthy fabric (BGP up, interfaces up)
   - Accessible at localhost:3000

5. **Student Workspace:**
   - Text editor (vim/nano/VS Code)
   - Terminal with kubectl configured
   - Browser for Gitea/ArgoCD/Grafana
   - 90-minute timer visible

### Assessment Submission:

**Students submit:**
1. All YAML files created (platform-vpc.yaml, 3× VPCAttachment files)
2. Troubleshooting documentation (Task 2.3)
3. Diagnostic bundle (Task 3.2)
4. Post-incident review document (Task 4.1)
5. Screenshots as specified

---

## Grading Guidelines

### Automated Grading (Technical Validation):
- VPC YAML validation (syntax, schema compliance)
- VPCAttachment validation (connection references exist)
- GitOps workflow completion (commits in Gitea, ArgoCD synced)
- Connectivity tests (ping results, DHCP assignments)

### Manual Grading (Instructor/Proctor):
- Troubleshooting documentation quality (clarity, accuracy)
- Diagnostic bundle completeness
- PIR document quality (5 Whys depth, SMART action items, blameless tone)
- Screenshots showing correct dashboard interpretation

---

## Retake Policy

**If score < 70%:**
- **Immediate retake:** Not allowed (must wait 7 days)
- **Remediation:** Review failed modules, complete practice labs
- **Retake limit:** 3 attempts within 6 months
- **Alternate assessment:** Different scenario (same competencies tested)

**Retake scenarios:**
- Different VPC requirements (alternate subnet design)
- Different troubleshooting issue (BGP problem instead of VLAN mismatch)
- Same assessment structure, different technical details

---

## Assessment Validity

**Content Validity:**
- Maps to all 16 modules
- Tests integrated real-world scenarios
- Aligns with HCFO job tasks

**Construct Validity:**
- Measures operational competency, not memorization
- Requires applying knowledge, not recalling facts
- Tests decision-making and problem-solving

**Predictive Validity:**
- Passing score indicates readiness for independent fabric operations
- Tasks mirror actual day-to-day operator responsibilities
- Skills tested are critical for production environments

---

## Accessibility Accommodations

**Time Extensions:**
- Students with documented needs: +30 minutes (total 120 minutes)

**Alternative Formats:**
- Screen reader compatible (all tasks text-based)
- Keyboard-only navigation supported

**Language Support:**
- English (primary)
- Translations available for technical terms

---

## Continuous Improvement

**Assessment Evolution:**
- Review pass rates quarterly
- Update scenarios based on product changes
- Collect student feedback on difficulty
- Adjust scoring rubrics based on common errors

**Version Control:**
- Current version: 1.0
- Next review: After first 50 completions
- Changelog maintained for transparency

---

## Capstone Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Version:** 1.0
**Status:** ✅ DESIGNED - Ready for Development
**Duration:** 75-90 minutes
**Total Points:** 100
**Passing Score:** 70%
**Module Coverage:** All 16 core modules
**Assessment Type:** Hands-on practical with scenario-based tasks

---

**Next Steps:**
1. Develop assessment environment automation (vlab seeding, Gitea setup)
2. Create alternate scenarios for retakes
3. Build automated grading scripts for technical validation
4. Pilot assessment with beta testers
5. Refine rubrics based on pilot feedback
