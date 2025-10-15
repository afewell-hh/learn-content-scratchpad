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
- Hedgehog vlab (default topology)
- Nested k3s cluster for tooling
- Prometheus + Grafana (with Hedgehog dashboards pre-configured)
- ArgoCD + Gitea (for GitOps workflows)
- kubectl fabric CLI plugin installed and accessible

**Phase 2a: Environment Setup (IN PROGRESS)**
- ✅ Research kubectl fabric CLI plugin capabilities
- 📋 Set up Prometheus/Grafana stack in local environment
- 📋 Configure Hedgehog telemetry (Alloy → Prometheus/Loki)
- 📋 Import 6 Hedgehog Grafana dashboards
- 📋 Set up ArgoCD + Gitea for GitOps workflows
- 📋 Document ideal environment setup for student VM appliance replication
- 📋 Validate all tools work in integrated environment

**Phase 2b: Module Design (PAUSED - Resume After 2a)**
- Design actual lab exercises for each module
- Map real CRD workflows to learning objectives
- Use documented Hedgehog observability tools (Grafana, kubectl fabric CLI)
- Identify troubleshooting scenarios we can create
- Define prerequisite knowledge for each module
- Create module dependency graph
- Design capstone assessment with specific tasks

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

**Phase:** Phase 2 - Architecture & Design 🎨

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

### Phase 2: IN PROGRESS 🔄

**Goal:** Transform research into detailed module designs

**Approach:** Best effort, quality over speed

**CRITICAL UPDATE (Oct 15):** Module design must be based on "ideal" Hedgehog environment with Prometheus/Grafana/ArgoCD/Gitea. Pausing module design to set up ideal environment first.

**Phase 2a: Ideal Environment Setup (ACTIVE)**
Student VM will replicate local environment setup. Must set up locally first:

1. **kubectl fabric CLI** - Research and install kubectl plugin (super powerful tool)
2. **Prometheus/Grafana** - Set up observability stack with Hedgehog dashboards
3. **Hedgehog Telemetry** - Configure Alloy agents to push metrics/logs
4. **ArgoCD + Gitea** - Set up GitOps workflows (GitHub too complex for student sessions)
5. **Documentation** - Document setup for VM appliance replication

**Phase 2b: Module Design (PAUSED - Resume After Environment Setup)**
- Course 1: Foundations & Interfaces (Modules 1.1-1.4)
- Course 2: Provisioning & Connectivity (Modules 2.1-2.4)
- Course 3: Grafana-Based Observability (Modules 3.1-3.4) - **NOT kubectl-native**
- Course 4: Troubleshooting & Recovery (Modules 4.1-4.4)
- Capstone: Integrated assessment design

**Current Focus:**
- ✅ Module 1.1 design APPROVED (read-only exploration)
- ⚠️ Module 1.2 design PAUSED (needs Grafana/kubectl fabric CLI)
- 🔄 Setting up ideal environment
- 📋 14 modules + capstone to design after environment ready

**Deliverables Planned:**
- Ideal environment setup documentation
- Module design specifications (1.1-4.4)
- Lab exercise specifications using Grafana + kubectl fabric CLI
- Assessment questions and rubrics
- Module dependency graph
- Capstone assessment design
- Timing validation

**Next Actions:**
1. Research kubectl fabric CLI plugin
2. Set up Prometheus/Grafana stack
3. Configure Hedgehog telemetry
4. Import Hedgehog dashboards
5. Set up ArgoCD + Gitea
6. Resume Module 1.2 design with proper tools

**Active Issues:**
- Issue #3: Phase 2 Architecture & Design (updated with plan)
- Issue #4: Curriculum Adjustments Tracking (ongoing)

**Blockers:** None

**Risks:**
- Timing targets (15min/module) may need adjustment - WILL VALIDATE
- Context management for long project - USING GITHUB EXTENSIVELY
- Scope creep - STRICT ADHERENCE TO LEARNING PHILOSOPHY

---

**Last Updated:** 2025-10-15
**Project Lead:** Claude (Course Owner)
**Dev Agent:** Phase 1 complete (exceptional work), ready for Phase 2 validation tasks
**Repository:** https://github.com/afewell-hh/learn-content-scratchpad
**Status:** ✅ Phase 1 Complete | 🔄 Phase 2 Active | Ahead of Schedule
