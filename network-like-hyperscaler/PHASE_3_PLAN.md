# Phase 3: Content Development Plan

## Overview

**Phase:** 3 - Content Development
**Status:** 🚀 READY TO START
**Started:** TBD
**Goal:** Transform 16 approved module designs into polished, production-ready learning content

## Phase 2 Completion Summary

### Deliverables Inherited from Phase 2
- ✅ All 16 core module designs (Courses 1-4) - 100% approved
- ✅ Ideal environment setup (EMKC with Prometheus/Grafana/ArgoCD/Gitea)
- ✅ Module dependency graph (MODULE_DEPENDENCY_GRAPH.md)
- ✅ Capstone assessment design (CAPSTONE_ASSESSMENT_DESIGN.md)
- ✅ Comprehensive research documentation (Phase 1: 3,600+ lines)

### Quality Standards Established
- **Learning Philosophy:** 10 principles embedded in every module
- **Timing Target:** 15 minutes per module (~4 hours total pathway)
- **Hands-On Focus:** Every module includes tested lab exercises
- **Dual Audience:** Accessible to both cloud-native and networking professionals
- **Technical Accuracy:** All CRDs, commands, and workflows validated against vlab

---

## IMPORTANT: Format Remediation Required

**Status:** 🚨 Course 1 modules (1.1-1.4) completed but in WRONG FORMAT

**Issue Discovered:** 2025-10-16
- Course 1 modules written in incorrect format (flat markdown files)
- hh-learn platform requires specific structure: modules in `content/modules/[slug]/README.md`
- Missing YAML front matter with required fields
- Missing course JSON and pathway JSON files

**Remediation Plan:**
- See detailed analysis: `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/HH-LEARN_FORMAT_ANALYSIS.md`
- Estimated effort: 4-6 hours to convert all 4 Course 1 modules
- Must complete before continuing with Course 2 development

**Next Steps:**
1. Convert Course 1 modules (1.1-1.4) to proper hh-learn format
2. Create course JSON for Course 1: `network-like-hyperscaler-foundations.json`
3. Create/update pathway JSON: `network-like-hyperscaler.json`
4. Test with `npm run sync:content`
5. Resume Course 2 development using correct format from the start

---

## Phase 3 Approach

### Development Methodology

**Iterative, Module-by-Module Content Development**

1. **Select Next Module** (follow critical path from MODULE_DEPENDENCY_GRAPH.md)
2. **Agent Dispatch** (create detailed GitHub issue with full context)
3. **Content Development** (agent transforms design → polished content in CORRECT FORMAT)
4. **Course Lead Review** (quality gates, philosophy alignment, technical accuracy, FORMAT COMPLIANCE)
5. **Approval/Iteration** (approve or request revisions)
6. **Commit & Close** (merge to main, close issue)
7. **Repeat** for next module

**Critical Path Order:**
```
Module 1.1 → 1.2 → 1.3 → 1.4 → 2.1 → 2.2 → 2.3 → 2.4 → 3.1 → 3.2 → 3.3 → 3.4 → 4.1 → 4.2 → 4.3 → 4.4
```

**Rationale:**
- Linear progression ensures prerequisite knowledge is available
- Course boundaries provide natural checkpoints
- Module-by-module approach enables high-quality, focused development
- Agent specialization allows course lead to maintain oversight and context

---

## Content Development Standards

### Input: Module Design Document

Each module has a comprehensive design document in `designs/module-X.X-design.md` containing:

1. **Module Metadata** (title, duration, prerequisites)
2. **Learning Objectives** (5 specific, measurable outcomes)
3. **Content Outline** (introduction, core concepts, hands-on lab, wrap-up)
4. **Assessment Design** (5 quiz questions with explanations + practical assessment)
5. **Technical Requirements** (CRDs used, kubectl commands, tools)
6. **Pedagogical Design** (learning philosophy principles, audience considerations, common challenges)
7. **Dependencies** (prerequisites, enables, related modules)
8. **Quality Checklist** (design quality, technical accuracy, philosophy alignment, accessibility, completeness)

**Location:** `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/designs/`

### Output: Polished Module Content

**IMPORTANT: hh-learn Platform Format Requirements**

The hh-learn platform uses a specific three-tier structure:
- **Pathways** → JSON files in `content/pathways/`
- **Courses** → JSON files in `content/courses/`
- **Modules** → Individual directories in `content/modules/`

**Module Directory Structure (REQUIRED):**
```
hh-learn/content/modules/
├── network-like-hyperscaler-welcome/
│   └── README.md
├── network-like-hyperscaler-how-it-works/
│   └── README.md
├── network-like-hyperscaler-mastering-interfaces/
│   └── README.md
├── network-like-hyperscaler-foundations-recap/
│   └── README.md
├── [... 12 more modules for Courses 2-4]
└── hcfo-certification-capstone/
    └── README.md
```

**Course JSON Files (REQUIRED):**
```
hh-learn/content/courses/
├── network-like-hyperscaler-foundations.json
├── network-like-hyperscaler-provisioning.json
├── network-like-hyperscaler-observability.json
└── network-like-hyperscaler-troubleshooting.json
```

**Pathway JSON File (REQUIRED):**
```
hh-learn/content/pathways/
└── network-like-hyperscaler.json
```

**Module Content Structure (with REQUIRED YAML Front Matter):**

```markdown
---
title: "Module Title"
slug: "lowercase-hyphen-slug"
difficulty: "beginner"  # beginner|intermediate|advanced
estimated_minutes: 15
version: "v1.0.0"
validated_on: "2025-10-16"
pathway_slug: "network-like-hyperscaler"
pathway_name: "Network Like a Hyperscaler"
tags:
  - hedgehog
  - fabric
  - kubernetes
  - operations
description: "120-160 character SEO-friendly description of what learners will achieve."
order: 100
---

# Module Title

## Learning Objectives
- Objective 1 (specific, measurable)
- Objective 2
- Objective 3

## Prerequisites
- Required prerequisite 1
- Required prerequisite 2

## Scenario: [Scenario Title]
Scenario-based introduction (not abstract)

### Step 1: [Step Name]
Step-by-step instructions with commands and expected outputs

### Step 2: [Step Name]
Continue scenario...

## Concepts & Deep Dive
Deeper explanations of concepts introduced in scenario

### Concept 1
Explanation...

### Concept 2
Explanation...

## Troubleshooting
Common issues, symptoms, causes, fixes

## Resources
- [Link to authoritative documentation]
- [Link to related modules]
```

**Critical Format Requirements:**
1. **YAML Front Matter** - MUST be at top of README.md between `---` markers
2. **Slug** - lowercase-hyphen format, used for URLs and references
3. **Description** - 120-160 characters for SEO
4. **Tags** - Array format for categorization
5. **Scenario-based approach** - Not abstract explanations
6. **Code blocks** - Must specify language (```bash, ```yaml)
7. **Section names** - "Scenario:" and "Concepts & Deep Dive" (not "Lab" or "Core Concepts")

**Style Guidelines:**

1. **Tone:** Professional but conversational, approachable, confidence-building
2. **Voice:** Second person ("you'll learn", "you'll use") for engagement
3. **Length:** Concise, scannable, no fluff (respect 15-minute target)
4. **Code Blocks:** Always include expected outputs, validation steps
5. **Explanations:** Always explain "why" not just "how"
6. **Visuals:** Use mermaid diagrams, ASCII art, or descriptions when helpful
7. **Learning Philosophy:** Embed principles naturally, not as explicit callouts

**Quality Gates (must pass before approval):**

1. ✅ **Technical Accuracy** - All commands, CRDs, workflows tested in vlab
2. ✅ **Philosophy Alignment** - Embodies 3+ learning philosophy principles
3. ✅ **Completeness** - All required sections present and detailed
4. ✅ **Accessibility** - Clear to both cloud-native and networking audiences
5. ✅ **Timing Feasibility** - Content fits 15-minute target (estimate)
6. ✅ **Lab Validation** - Hands-on lab steps tested and work correctly
7. ✅ **Assessment Quality** - Questions test learning objectives, answers correct

---

## Agent Dispatch Template

### Issue Creation Checklist

When creating a content development issue for an agent:

- [ ] Issue title: `[CONTENT] Module X.X: [Module Title]`
- [ ] Labels: `phase-3-content`, `course-X`, `module-development`
- [ ] Assigned to: `@dev-agent` (or leave unassigned for dispatch)
- [ ] Include full context in issue body (see template below)

### Issue Body Template

```markdown
## Module Content Development: Module X.X

**Objective:** Transform the approved design for Module X.X into polished, production-ready learning content.

---

## Context for Ephemeral Agent

You're working on Phase 3 (Content Development) of the **Network Like a Hyperscaler with Hedgehog** learning pathway. This pathway teaches engineers to operate Hedgehog Open Network Fabric using Kubernetes-native tools (kubectl, CRDs, GitOps).

### Project Status
- ✅ **Phase 1 Complete:** Research & Discovery (3,600+ lines of technical documentation)
- ✅ **Phase 2 Complete:** Architecture & Design (all 16 modules designed, capstone assessment, dependency graph)
- 🚀 **Phase 3 Starting:** Content Development (transform designs → polished modules)

### Your Task
Convert the approved design document for **Module X.X: [Title]** into a polished, production-ready markdown file for publication.

---

## Input: Module Design

**Design Document Location:**
`/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/designs/module-X.X-design.md`

**Design Status:** ✅ APPROVED - Ready for Content Development

**What's in the design:**
- Complete module metadata (title, duration, prerequisites, learning objectives)
- Content outline (introduction, core concepts, hands-on lab, wrap-up)
- Assessment design (5 quiz questions + practical assessment)
- Technical requirements (CRDs, kubectl commands, tools)
- Pedagogical design notes (learning philosophy principles, audience considerations, common challenges)
- Dependencies (prerequisites, enables, related modules)

**Your job:** Transform this design into a polished, engaging learning module.

---

## Output: Polished Module Content

**Target Location:**
`/home/ubuntu/afewell-hh/hh-learn/content/modules/[module-slug]/README.md`

**Example paths:**
- Module 1.1: `content/modules/network-like-hyperscaler-welcome/README.md`
- Module 2.1: `content/modules/network-like-hyperscaler-vpc-provisioning/README.md`

**REQUIRED: YAML Front Matter**
Every module README.md MUST start with YAML front matter containing:
- title, slug, difficulty, estimated_minutes
- version, validated_on, pathway_slug, pathway_name
- tags (array), description (120-160 chars), order

**Required Sections:**
1. **Learning Objectives** (bulleted list of specific outcomes)
2. **Prerequisites** (required prior knowledge)
3. **Scenario: [Title]** (scenario-based walkthrough with steps)
4. **Concepts & Deep Dive** (deeper explanations of concepts)
5. **Troubleshooting** (common issues and fixes)
6. **Resources** (links to authoritative docs)

**Style Requirements:**
- **Tone:** Professional but conversational, approachable, confidence-building
- **Voice:** Second person ("you'll learn", "you'll use")
- **Length:** Concise, scannable, respect 15-minute target
- **Code Blocks:** Must specify language (```bash, ```yaml) and include expected outputs
- **Explanations:** Explain "why" not just "how" (teach design choices)
- **Learning Philosophy:** Embed principles naturally (confidence-first, hands-on, reality-focused)
- **Scenario-based:** Start with real-world scenario, not abstract concepts

---

## Quality Standards

Your content must meet these quality gates:

1. ✅ **Technical Accuracy** - All commands, CRDs, workflows match design and work in vlab
2. ✅ **Philosophy Alignment** - Embodies 3+ learning philosophy principles from design
3. ✅ **Completeness** - All required sections present and detailed
4. ✅ **Accessibility** - Clear to both cloud-native and networking audiences
5. ✅ **Timing Feasibility** - Content fits 15-minute target (estimate)
6. ✅ **Lab Validation** - Hands-on lab steps tested and work correctly (if you have vlab access)
7. ✅ **Assessment Quality** - Questions test learning objectives, answers correct

---

## Reference Documentation

**Key Project Documents:**

1. **Hedgehog Learning Philosophy:**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/hedgehogLearningPhilosophy.md`
   10 guiding principles for all content (train for reality, confidence first, hands-on, etc.)

2. **Module Design (Input):**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/designs/module-X.X-design.md`
   The approved design you're converting to polished content

3. **CRD Reference:**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/research/CRD_REFERENCE.md`
   Comprehensive reference for all Hedgehog CRDs (VPC, VPCAttachment, Switch, Server, etc.)

4. **Workflow Reference:**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/research/WORKFLOWS.md`
   Common operational workflows (VPC lifecycle, GitOps, diagnostics)

5. **Project Plan:**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/PROJECT_PLAN.md`
   Master project plan (phases, status, quality gates)

6. **Module Dependency Graph:**
   `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/MODULE_DEPENDENCY_GRAPH.md`
   Visual map of all 16 modules and their prerequisites

---

## Deliverables

**When you're done, post to this issue:**

1. ✅ **Polished Module Content** - Markdown file at target location
2. ✅ **Content Quality Report** - Brief summary of:
   - Technical validation status (did you test commands/labs in vlab?)
   - Learning philosophy principles embedded
   - Estimated completion time (15-minute target check)
   - Any deviations from design (with rationale)
3. ✅ **Self-Quality Check** - Confirm all 7 quality gates passed

**Do NOT commit directly.** Post your work to this issue for course lead review. Course lead will review, request changes if needed, then commit on approval.

---

## Important Notes

- **Respect the design:** The design was carefully crafted and approved. Follow it closely unless you find a technical error or improvement.
- **Technical accuracy is critical:** If a command doesn't work or a CRD field is wrong, flag it and suggest a fix.
- **Learning philosophy matters:** This isn't just documentation—it's learning content designed to build confidence and competence.
- **Audience is dual:** Content must be accessible to both Kubernetes/cloud-native learners AND traditional networking professionals.
- **Hands-on is mandatory:** The lab section is the most important part. Make it detailed, testable, and achievable.

---

## Questions or Issues?

If you encounter blockers or have questions:
1. Check reference documentation (listed above)
2. Post questions to this issue
3. Course lead will respond with guidance

---

**Ready to start? Read the design document, review the learning philosophy, and create amazing learning content!**
```

---

## Content Development Workflow

### Step 1: Create Issue
- Use template above
- Assign module-specific details (number, title, course, design path)
- Add labels: `phase-3-content`, `course-X`, `module-development`

### Step 2: Agent Develops Content
- Agent reads design document
- Agent references learning philosophy, CRD reference, workflow docs
- Agent writes polished markdown content
- Agent tests commands/labs in vlab (if access available)
- Agent posts deliverables to issue

### Step 3: Course Lead Review
- Read polished content
- Check against 7 quality gates
- Test commands in vlab (if agent couldn't)
- Verify learning philosophy alignment
- Provide feedback (approve or request revisions)

### Step 4: Iteration (if needed)
- Agent addresses feedback
- Posts updated content
- Repeat review cycle

### Step 5: Approval & Commit
- Course lead commits content to hh-learn repo
- Update PROJECT_PLAN.md with module status
- Close issue with summary

### Step 6: Next Module
- Identify next module on critical path
- Create new issue
- Repeat workflow

---

## Milestones

### Course 1: Foundations & Interfaces (Modules 1.1-1.4)
- **Modules:** 4
- **Target Duration:** ~60 minutes
- **Start:** Issue #13 (Module 1.1)
- **Completion:** TBD

### Course 2: Provisioning & Connectivity (Modules 2.1-2.4)
- **Modules:** 4
- **Target Duration:** ~60 minutes
- **Start:** After Course 1 complete
- **Completion:** TBD

### Course 3: Observability & Fabric Health (Modules 3.1-3.4)
- **Modules:** 4
- **Target Duration:** ~60 minutes
- **Start:** After Course 2 complete
- **Completion:** TBD

### Course 4: Troubleshooting, Recovery & Escalation (Modules 4.1-4.4)
- **Modules:** 4
- **Target Duration:** ~60 minutes
- **Start:** After Course 3 complete
- **Completion:** TBD

### Capstone Assessment (HCFO Certification)
- **Duration:** 75-90 minutes
- **Start:** After all 16 modules complete
- **Completion:** TBD

---

## Success Criteria

### Phase 3 Complete When:
- ✅ All 16 core modules converted to polished content
- ✅ All modules pass 7 quality gates
- ✅ All hands-on labs tested in vlab
- ✅ Capstone assessment content created
- ✅ All content committed to hh-learn repo
- ✅ Pathway ready for Phase 4: Quality Assurance

---

## Risk Management

### Risk 1: Technical Inaccuracies
- **Mitigation:** Test all commands/labs in vlab before approval
- **Owner:** Course lead + dev agent

### Risk 2: Timing Target Misses
- **Mitigation:** Estimate completion time during review, adjust if needed
- **Owner:** Course lead (track timing, adjust in Phase 4)

### Risk 3: Learning Philosophy Drift
- **Mitigation:** Explicit quality gate check on every module
- **Owner:** Course lead (philosophy guardian)

### Risk 4: Content Inconsistency Across Modules
- **Mitigation:** Use standard template, course lead reviews for consistency
- **Owner:** Course lead

### Risk 5: Context Loss for Ephemeral Agents
- **Mitigation:** Comprehensive issue templates with full context
- **Owner:** Course lead (create detailed dispatches)

---

## Next Steps

1. Create Issue #13: Module 1.1 Content Development
2. Dispatch to dev agent with full context
3. Agent develops Module 1.1 content
4. Course lead reviews and approves
5. Commit Module 1.1 to hh-learn repo
6. Repeat for Module 1.2, 1.3, 1.4 (complete Course 1)
7. Continue with Courses 2, 3, 4
8. Develop Capstone Assessment content
9. Transition to Phase 4: Quality Assurance

---

**Status:** 🚀 READY - Issue #13 creation in progress
**Last Updated:** 2025-10-16
**Created By:** Course Lead (Claude)
**Next Milestone:** Module 1.1 Content Development
