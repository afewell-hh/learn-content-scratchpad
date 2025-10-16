# Network Like a Hyperscaler - Master Project Plan

## Project Overview

**Pathway Name:** Network Like a Hyperscaler with Hedgehog
**Certification:** Hedgehog Certified Fabric Operator (HCFO)
**Target Duration:** ~4 hours core content + hands-on labs
**Target Audience:** Engineers bridging cloud-native/Kubernetes and networking domains
**Repository:** `afewell-hh/learn-content-scratchpad` (development) → `hh-learn` (publication)

## Guiding Principles

All content MUST embody the **Hedgehog Learning Philosophy** (see `hedgehogLearningPhilosophy.md`):

1. **Train for Reality, Not Rote** - Focus on workflows, not wizards
2. **Focus on What Matters Most** - Prioritize common, high-impact tasks
3. **Confidence Before Comprehensiveness** - Build momentum and psychological safety
4. **Abstraction as Empowerment** - Teach abstractions as tools
5. **Learn by Doing, Not Watching** - Hands-on with live vlab systems
6. **Teach the Why Behind the How** - Understanding design choices builds adaptable engineers
7. **Modularity & Composability** - Short, reusable, combinable modules
8. **Bridging Two Worlds** - Accessible to both cloud-native and networking audiences
9. **Support as Part of Learning** - Normalize escalation and collaboration
10. **Continuous Learning Over Static Mastery** - Teach how to stay current

## Project Phases

### Phase 1: Research & Discovery (CURRENT)
**Goal:** Deep technical validation and mapping

- ✅ Review consultant planning documents
- ✅ Verify vlab environment access
- ✅ Identify available Hedgehog CRDs
- 🔄 Map consultant placeholders to real Hedgehog concepts
- 🔄 Document vlab topology and capabilities
- 🔄 Identify gaps between consultant vision and reality
- 🔄 Create detailed technical mapping document

### Phase 2: Architecture & Design
**Goal:** Translate pedagogy into technically accurate learning path

**PREREQUISITE: Ideal Environment Setup**
Student environment will be an Ubuntu host with:
- Hedgehog VLAB (default topology) - Immutable control node
- **External Management K3s Cluster (EMKC)** - Separate k3d cluster for student tools
- Prometheus + Grafana (with Hedgehog dashboards pre-configured)
- ArgoCD + Gitea (for GitOps workflows)
- kubectl fabric CLI plugin installed and accessible

**Architecture Decision:** After discovering that the Hedgehog control node is an immutable Flatcar Linux appliance (customers cannot modify it), we pivoted to deploying all student-facing tools in a separate single-node k3d cluster on the Ubuntu host (EMKC). This provides clear separation between Hedgehog infrastructure and student tooling, mirrors real-world deployment patterns, and enables easy reset/replication. See ADR-001 for full rationale.

**Phase 2a: Environment Setup (✅ COMPLETE - Oct 15, 2025)**
- ✅ Research kubectl fabric CLI plugin capabilities
- ✅ Set up EMKC (k3d cluster "observability") on Ubuntu host
- ✅ Deploy Prometheus/Grafana/Loki stack in EMKC via Helm
- ✅ Configure Hedgehog telemetry (Alloy → fabric-proxy → EMKC services)
- ✅ Import 6 Hedgehog Grafana dashboards (ConfigMap auto-discovery)
- ✅ Deploy ArgoCD in EMKC with multi-cluster access to VLAB
- ✅ Deploy Gitea in EMKC with hedgehog-config repository
- ✅ Solve EMKC-VLAB networking (SSH tunnel + Docker bridge gateway)
- ✅ Document ideal environment setup for student VM appliance replication
- ✅ Validate all tools work in integrated environment (verify-environment.sh)
- ✅ Test complete GitOps workflow (Gitea → ArgoCD → VLAB → Grafana)

**Phase 2b: Module Design (✅ COMPLETE - Oct 16, 2025)**
- ✅ Module 1.1: Welcome to Fabric Operations (APPROVED - v1.0)
- ✅ Module 1.2: How Hedgehog Works (APPROVED - v2.1 GitOps)
- ✅ Module 1.3: Mastering the Three Interfaces (APPROVED - v1.1 Event-Based)
- ✅ Module 1.4: Course 1 Recap & Forward Map (APPROVED - v1.0) ⭐ **Course 1 Complete**
- ✅ Module 2.1: Define VPC Network (APPROVED - v1.0)
- ✅ Module 2.2: Attach Servers to VPC (APPROVED - v1.0)
- ✅ Module 2.3: Connectivity Validation (APPROVED - v1.0)
- ✅ Module 2.4: Decommission & Cleanup (APPROVED - v1.0) ⭐⭐ **Course 2 Complete**
- ✅ Module 3.1: Fabric Telemetry Overview (APPROVED - v1.0)
- ✅ Module 3.2: Dashboard Interpretation (APPROVED - v1.0)
- ✅ Module 3.3: Events & Status Monitoring (APPROVED - v1.0)
- ✅ Module 3.4: Pre-Support Diagnostic Checklist (APPROVED - v1.0) ⭐⭐⭐ **Course 3 Complete**
- ✅ Module 4.1: Diagnosing Fabric Issues (APPROVED - v1.0)
- ✅ Module 4.2: Rollback & Recovery (APPROVED - v1.0)
- ✅ Module 4.3: Coordinating with Support (APPROVED - v1.0)
- ✅ Module 4.4: Post-Incident Review (APPROVED - v1.0) ⭐⭐⭐⭐ **Course 4 Complete**
- 🏆 **ALL 16 CORE MODULES DESIGNED** (100% milestone achieved)
- ✅ Module Dependency Graph complete (MODULE_DEPENDENCY_GRAPH.md)
- ✅ Capstone Assessment design complete (CAPSTONE_ASSESSMENT_DESIGN.md)

### Phase 3: Content Development (Iterative)
**Goal:** Create high-quality, technically accurate modules

**4 Courses × 4 Modules = 16 core modules + optional background modules**

#### Course 1: Foundations & Interfaces
- Module 1.1: Welcome & Mindset
- Module 1.2: Architecture & Control Model
- Module 1.3: Interfaces: CLI / API / UI
- Module 1.4: Recap & Forward Map

#### Course 2: Provisioning & Connectivity
- Module 2.1: Define VPC Network
- Module 2.2: Attach Servers to VPC
- Module 2.3: Connectivity Validation
- Module 2.4: Decommission & Cleanup

#### Course 3: Observability & Fabric Health
- Module 3.1: Fabric Telemetry Overview
- Module 3.2: Dashboard Interpretation
- Module 3.3: Events & Status Monitoring
- Module 3.4: Pre-Support Diagnostic Checklist

#### Course 4: Troubleshooting, Recovery & Escalation
- Module 4.1: Diagnosing Fabric Issues
- Module 4.2: Rollback & Recovery
- Module 4.3: Coordinating with Support
- Module 4.4: Post-Incident Review

#### Optional Background Modules
- Kubernetes CRD Fundamentals
- Network Primitives (routing, forwarding, segmentation)
- IP Addressing & Subnetting Primer
- YAML Best Practices
- GitOps Workflow Basics
- Observability 101 (Logs, Metrics, Traces)
- Network Troubleshooting Methodology

### Phase 4: Quality Assurance
**Goal:** Ensure technical accuracy and pedagogical excellence

- Technical review of all CRD examples
- Lab testing in vlab environment
- Peer review against learning philosophy
- Accessibility check
- Timing validation (15min per module)

### Phase 5: Publication & Launch
**Goal:** Deploy to production learning platform

- Migrate polished content to `hh-learn` repo
- Sync to HubDB
- Create pathway and course JSON files
- Set up progress tracking
- Create certification assessment

## Consultant Document Mapping

### Key Insights from Consultant Docs

**Strong Points:**
- Excellent pedagogical structure (4 courses × 4 modules)
- Clear learning progression
- Emphasis on hands-on labs
- Capstone assessment design
- Modular/composable approach

**Gaps to Address:**
- Placeholder names (e.g., "TenantNetwork", "HostAttachment") need mapping to real CRDs
- GitOps/ArgoCD mentioned but not present in default vlab
- Some assumed capabilities need verification (e.g., fault injection)
- Needs grounding in actual Hedgehog architecture

### Technical Mapping: Consultant → Hedgehog Reality

| Consultant Placeholder | Real Hedgehog Concept | Notes |
|------------------------|----------------------|-------|
| "TenantNetwork" | `VPC` | Virtual Private Cloud CRD |
| "HostAttachment" | `VPCAttachment` + underlying `Server`/`Connection` | Binding servers to VPCs |
| Wiring/Topology | Wiring Diagram API: `Switch`, `Server`, `Connection` | Physical topology definition |
| External connectivity | `External`, `ExternalAttachment`, `ExternalPeering` | Border leaf concepts |
| Network segmentation | `VLANNamespace`, `IPv4Namespace` | Resource allocation |
| GitOps | Manual kubectl + YAML (for now) | May need to adjust narrative |
| Diagnostics | kubectl commands + fabric status | Need to document available commands |

## Vlab Environment Capabilities

**Confirmed Available:**
- ✅ Kubernetes API access (kubectl)
- ✅ Default spine-leaf topology (2 spines, 5 leaves, 10 servers)
- ✅ Hedgehog CRDs installed (vpc, wiring, agent APIs)
- ✅ hhfab utility for wiring diagram management

**To Verify:**
- Prometheus/Grafana dashboards
- Fabric Proxy (metrics collection)
- Event streaming
- Fault injection capabilities
- Available diagnostic commands

**Default Topology:**
- 2× Spine switches (spine-01, spine-02)
- 5× Leaf switches:
  - leaf-01, leaf-02 (MCLAG pair - "mclag-1")
  - leaf-03, leaf-04 (ESLAG pair - "eslag-1")
  - leaf-05 (standalone/orphan leaf)
- 10× Servers (server-01 through server-10)
- Connection types: MCLAG, ESLAG, bundled, unbundled, fabric

## Quality Gates

Each module must pass these gates before moving to next phase:

1. **Technical Accuracy** - All CRDs, commands, and workflows verified in vlab
2. **Philosophy Alignment** - Embodies learning philosophy principles
3. **Hands-On Validation** - Lab exercises tested and timed
4. **Accessibility** - Understandable to both cloud-native and networking audiences
5. **Completeness** - All required sections present per authoring guide

## Project Management Approach

### Issue-Based Tracking
- All work tracked in GitHub issues
- Issue templates for modules, courses, research tasks
- Labels: `phase-1-research`, `phase-2-design`, `phase-3-content`, `course-1`, `course-2`, etc.
- Milestones for each phase

### Pair Development Model
- **Course Lead (Claude):** Process adherence, quality assurance, architecture, planning
- **Dev Agent:** Research, content drafting, lab testing, technical validation
- All agent dispatches via GitHub issue comments
- Preserve course lead context by delegating legwork

### Documentation Standards
- Living documents: This PROJECT_PLAN.md, technical mapping docs
- Everything else: GitHub issues
- Keep repos clean and organized
- Development: `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/`
- Publication: `/home/ubuntu/afewell-hh/hh-learn/content/`

## Success Criteria

### Technical Success
- All lab exercises work in vlab
- All CRD examples are valid and tested
- Timing targets met (15min modules, 4hr total)
- Zero technical inaccuracies

### Pedagogical Success
- Learning philosophy embodied throughout
- Clear progression from confidence to competence
- Accessible to target audiences
- Hands-on labs enhance understanding

### Project Success
- Clean, organized repositories
- Comprehensive issue tracking
- High-quality, reusable content
- Ready for open source release

## Current Status

**Phase:** Phase 3 - Content Development 🚀

### Phase 1: COMPLETE ✅ (Oct 15, 2025)

**Delivered:**
- ✅ **3,600+ lines of research documentation**
  - VLAB_CAPABILITIES.md (688 lines)
  - OBSERVABILITY.md (789 lines)
  - CRD_REFERENCE.md (1,010 lines)
  - WORKFLOWS.md (1,076 lines)
- ✅ **Critical findings identified**
  - Observability is opt-in (external stack required)
  - No GitOps/ArgoCD (kubectl-native approach)
  - Event-based reconciliation (minimal CRD status fields)
  - Agent CRD is source of truth for switch state
- ✅ **Course 3 redesigned** (COURSE_3_REDESIGN.md)
- ✅ **Curriculum adjustments planned** (Issue #4)
- ✅ **Issues #1 and #2 closed**

**Quality:** Exceeded expectations ⭐⭐⭐⭐⭐

### Phase 2: ✅ COMPLETE (Oct 16, 2025)

**Goal:** Transform research into detailed module designs

**Approach:** Best effort, quality over speed

**CRITICAL UPDATE (Oct 15):** Module design must be based on "ideal" Hedgehog environment with Prometheus/Grafana/ArgoCD/Gitea. Pausing module design to set up ideal environment first.

**Architecture Pivot (Oct 15):** Discovered that Hedgehog control node is immutable infrastructure (Flatcar Linux appliance). All student tools now deployed in External Management K3s Cluster (EMKC) - a separate k3d cluster on Ubuntu host. See ADR-001-emkc-separation.md for full architectural decision record.

**Phase 2a: Ideal Environment Setup (✅ COMPLETE - Oct 15, 2025)**
Student VM will replicate local environment setup. Environment fully operational:

1. ✅ **kubectl fabric CLI** - Installed and documented (500+ line reference)
2. ✅ **EMKC Cluster** - Created k3d cluster "observability" on Ubuntu host
3. ✅ **Prometheus/Grafana/Loki** - Deployed in EMKC via Helm with Remote Write enabled
4. ✅ **Hedgehog Telemetry** - Configured Alloy agents pushing via fabric-proxy to EMKC
5. ✅ **Grafana Dashboards** - All 6 Hedgehog dashboards imported via ConfigMap
6. ✅ **ArgoCD + Gitea** - Deployed in EMKC with complete GitOps workflow
7. ✅ **Networking Solution** - SSH tunnel + Docker bridge gateway (scriptable)
8. ✅ **Documentation** - Complete setup guide, validation script, troubleshooting
9. ✅ **End-to-End Validation** - GitOps workflow tested and working

**Phase 2b: Module Design (PAUSED - Resume After Environment Setup)**
- Course 1: Foundations & Interfaces (Modules 1.1-1.4)
- Course 2: Provisioning & Connectivity (Modules 2.1-2.4)
- Course 3: Grafana-Based Observability (Modules 3.1-3.4) - **NOT kubectl-native**
- Course 4: Troubleshooting & Recovery (Modules 4.1-4.4)
- Capstone: Integrated assessment design

**Current Focus:**
- ✅ Module 1.1 design APPROVED (read-only exploration - v1.0)
- ✅ Module 1.2 design APPROVED (GitOps workflow - v2.1)
- ✅ Module 1.3 design APPROVED (Three Interfaces deep dive - v1.1 Event-Based)
- ✅ Module 1.4 design APPROVED (Course 1 Recap & Forward Map - v1.0)
- ✅ Phase 2a COMPLETE (ideal environment fully operational)
- ✅ **COURSE 1 DESIGN COMPLETE** - All 4 modules approved ⭐
- ✅ Module 2.1 design COMPLETE (Define VPC Network - v1.0)
- ✅ Module 2.2 design COMPLETE (Attach Servers to VPC - v1.0)
- ✅ Module 2.3 design COMPLETE (Connectivity Validation - v1.0)
- ✅ Module 2.4 design COMPLETE (Decommission & Cleanup - v1.0)
- ✅ **COURSE 2 DESIGN COMPLETE** - All 4 modules designed ⭐⭐
- ✅ Module 3.1 design APPROVED (Fabric Telemetry Overview - v1.0)
- ✅ Module 3.2 design APPROVED (Dashboard Interpretation - v1.0)
- ✅ Module 3.3 design APPROVED (Events & Status Monitoring - v1.0)
- ✅ Module 3.4 design APPROVED (Pre-Support Diagnostic Checklist - v1.0)
- ✅ **COURSE 3 DESIGN COMPLETE** - All 4 modules approved ⭐⭐⭐
- ✅ Module 4.1 design APPROVED (Diagnosing Fabric Issues - v1.0)
- ✅ Module 4.2 design APPROVED (Rollback & Recovery - v1.0)
- ✅ Module 4.3 design APPROVED (Coordinating with Support - v1.0)
- ✅ Module 4.4 design APPROVED (Post-Incident Review - v1.0)
- ✅ **COURSE 4 DESIGN COMPLETE** - All 4 modules approved ⭐⭐⭐⭐
- 🏆 **PHASE 2B COMPLETE** - All 16 core modules designed (100% milestone)
- ✅ Module Dependency Graph created (MODULE_DEPENDENCY_GRAPH.md)
- ✅ Capstone Assessment designed (CAPSTONE_ASSESSMENT_DESIGN.md)
- 🏆 **PHASE 2 COMPLETE** - All deliverables finished (Oct 16, 2025)

**Deliverables Completed:**
- ✅ Ideal environment setup documentation (EMKC with Prometheus/Grafana/ArgoCD/Gitea)
- ✅ Module design specifications (1.1-4.4) - All 16 core modules approved
- ✅ Lab exercise specifications using Grafana + kubectl fabric CLI
- ✅ Assessment questions and rubrics
- ✅ Module dependency graph (MODULE_DEPENDENCY_GRAPH.md)
- ✅ Capstone assessment design (CAPSTONE_ASSESSMENT_DESIGN.md)
- 📋 Timing validation (deferred to Phase 4: Quality Assurance)

**Next Actions:**
1. ✅ Phase 2a complete - ideal environment operational
2. ✅ Module 1.2 redesigned with GitOps workflow (v2.1 APPROVED)
3. ✅ Module 1.3 designed and APPROVED (v1.1 Event-Based)
4. ✅ Module 1.3 validation by dev agent - Course lead architectural decision (event-based reconciliation)
5. ✅ Module 1.4 designed and APPROVED (v1.0)
6. ✅ **COURSE 1 COMPLETE** - All 4 modules approved
7. ✅ **COURSE 2 COMPLETE** - All 4 modules designed (Modules 2.1-2.4)
8. ✅ **COURSE 3 COMPLETE** - All 4 modules designed (Modules 3.1-3.4)
9. ✅ **COURSE 4 COMPLETE** - All 4 modules designed (Modules 4.1-4.4)
10. 🏆 **PHASE 2B COMPLETE** - All 16 core modules designed (100% milestone achieved)

### Phase 3: 🚀 IN PROGRESS (Oct 16, 2025)

**Goal:** Transform 16 approved module designs into polished, production-ready learning content

**Approach:** Iterative, module-by-module content development following critical path

**Progress:** 4 of 16 modules complete (25%)

🎉 **COURSE 1 COMPLETE: Foundations & Interfaces (4/4 modules, 3,019 lines, 96.5% avg quality)**

**Completed Modules:**
- ✅ Module 1.1: Welcome to Fabric Operations (PUBLISHED - 525 lines, 7/7 quality gates, 100%)
- ✅ Module 1.2: How Hedgehog Works (PUBLISHED - 724 lines, 6.5/7 quality gates, 93%)
- ✅ Module 1.3: Mastering the Three Interfaces (PUBLISHED - 1,186 lines, 6.5/7 quality gates, 93%)
- ✅ Module 1.4: Course Recap & Forward Map (PUBLISHED - 584 lines, 7/7 quality gates, 100%)

**Course 1 Metrics:**
- **Total Content:** 3,019 lines
- **Average Quality:** 96.5%
- **Perfect Scores:** 2/4 modules (1.1, 1.4)
- **Total Duration:** ~50 minutes
- **Status:** 🎓 COMPLETE - Ready for Course 2

**Documentation:**
- ✅ Phase 3 Plan created (PHASE_3_PLAN.md) - Complete workflow, standards, templates
- ✅ Issue #13 completed - Module 1.1 approved and committed
- ✅ Issue #14 completed - Module 1.2 approved and committed
- ✅ Issue #15 completed - Module 1.3 approved and committed
- ✅ Issue #16 completed - Module 1.4 approved and committed (Course 1 finale)

**Active Issues:**
- Issue #4: Curriculum Adjustments Tracking (ongoing)
- 📋 Next: Issue #17 - Module 2.1 (Begin Course 2: Provisioning & Connectivity)

**Blockers:** None

**Risks:**
- Timing targets (15min/module) may need adjustment - WILL VALIDATE
- Context management for long project - USING GITHUB EXTENSIVELY
- Scope creep - STRICT ADHERENCE TO LEARNING PHILOSOPHY

---

**Last Updated:** 2025-10-16
**Project Lead:** Claude (Course Owner)
**Dev Agent:** Phase 1 complete (exceptional work), ready for Phase 2 validation tasks
**Repository:** https://github.com/afewell-hh/learn-content-scratchpad
**Status:** ✅ Phase 1 Complete | ✅ Phase 2 Complete | 🚀 Phase 3 In Progress (4/16 modules, 25%) | 🎓 Course 1 Complete
