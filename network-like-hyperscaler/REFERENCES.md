# Project References & Resources

**Last Updated:** 2025-10-15

This document contains important references and resource locations for the "Network Like a Hyperscaler" learning pathway project.

---

## Vlab Environment

### Default Topology Wiring Diagram
**Location:** `/home/ubuntu/afewell-hh/hhfab-default-topo/include/vlab.generated.yaml`

**Description:** The wiring diagram file for the default vlab topology currently running in the local environment. This file defines:
- All switches (2 spines, 5 leaves)
- All servers (10 servers)
- All connections (MCLAG, ESLAG, bundled, unbundled, fabric)
- IP addressing and ASN assignments
- Redundancy group definitions

**Use Cases:**
- Reference for module design (accurate topology details)
- Validation of lab exercise assumptions
- Troubleshooting vlab configuration
- Understanding physical topology structure

### Vlab Access
- **kubectl config:** `~/.kube/config` (already configured)
- **hhfab directory:** `/home/ubuntu/afewell-hh/hhfab-default-topo/`
- **Hedgehog docs:** `/home/ubuntu/afewell-hh/docs/docs/`

---

## Phase 1 Research Documentation

### Research Deliverables Location
**Directory:** `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/research/`

**Documents:**
1. **VLAB_CAPABILITIES.md** (688 lines)
   - Complete topology mapping
   - Architecture overview
   - CRD hierarchy
   - Educational value analysis

2. **CRD_REFERENCE.md** (1,010 lines)
   - Comprehensive CRD documentation
   - Real YAML examples from vlab
   - Status field reference
   - kubectl command patterns

3. **WORKFLOWS.md** (1,076 lines)
   - 7 step-by-step kubectl workflows
   - VPC operations end-to-end
   - Troubleshooting guide
   - Validation procedures

4. **OBSERVABILITY.md** (789 lines)
   - Telemetry architecture
   - Grafana dashboard reference
   - External stack requirements
   - Monitoring capabilities

**Total:** 3,563 lines of validated technical documentation

---

## Phase 2 Design Documentation

### Module Designs Location
**Directory:** `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/designs/`

**Current Modules:**
1. **module-1.1-design.md** (✅ APPROVED)
   - Welcome to Fabric Operations
   - Read-only exploration lab
   - Confidence-building focus
   - **Validation report:** `module-1.1-validation-report.md`

### Design Templates
**Location:** `.github/ISSUE_TEMPLATE/module-design.md`

**Purpose:** Standardized template for all module design specifications, ensuring consistency and completeness across all 16 modules.

---

## Consultant Planning Documents

**Location:** `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/`

**Documents:**
1. **hedgehogLearningPhilosophy.md**
   - Core learning principles
   - Pedagogical approach
   - Tone and style guidelines
   - **MOST IMPORTANT** - All content must embody this philosophy

2. **pathwayGuide.md**
   - 4 courses × 4 modules structure
   - Learning objectives per module
   - Narrative flow sketches
   - Assessment ideas
   - **Note:** Contains placeholders that have been mapped to real Hedgehog concepts

3. **deliverablePlan.md**
   - Project deliverables outline
   - Content structure
   - Certification design

4. **certificationMessagingDoc.md**
   - HCFO certification messaging
   - Value propositions
   - Target audience

---

## Strategic Documents

### Master Project Plan
**Location:** `network-like-hyperscaler/PROJECT_PLAN.md`

**Contains:**
- Project phases and milestones
- Success criteria
- Quality gates
- Current status
- Learning philosophy principles

### Course 3 Redesign
**Location:** `network-like-hyperscaler/COURSE_3_REDESIGN.md`

**Contains:**
- Kubectl-native monitoring approach
- Redesigned module structure
- Rationale for changes
- Optional Grafana integration path

---

## HHLearn Content Structure

### Production Content Location
**Repository:** `/home/ubuntu/afewell-hh/hh-learn/content/`

**Structure:**
```
content/
├── modules/          # Individual learning modules (README.md with front matter)
├── courses/          # Course JSON files (reference modules)
├── pathways/         # Pathway JSON files (reference courses/modules)
└── [other dirs]      # Supporting content
```

**Sync Scripts:**
- `npm run sync:content` - Sync modules to HubDB
- `npm run sync:pathways` - Sync pathways to HubDB

**Note:** Content is developed in `learn_content_scratchpad` and migrated to `hh-learn` when ready for publication.

---

## GitHub Project Management

### Repository
**URL:** https://github.com/afewell-hh/learn-content-scratchpad

### Active Issues
- **Issue #3:** Phase 2 Architecture & Design (active tracking)
- **Issue #4:** Curriculum Adjustments (ongoing reference)
- **Issue #5:** Dev Agent Phase 2 Dispatch (technical validation)

### Closed Issues
- **Issue #1:** ✅ Vlab Environment Exploration (Phase 1 research)
- **Issue #2:** ✅ Technical Mapping (consultant → reality)

### Issue Templates
- `research-task.md` - Research and discovery work
- `module-development.md` - Module content creation
- `module-design.md` - Module design specifications

---

## Development Workflow

### Scratchpad → Production Migration

**Development Phase:**
1. Design modules in `learn_content_scratchpad/network-like-hyperscaler/designs/`
2. Validate with dev agent
3. Iterate based on feedback
4. Approve for content development

**Content Creation Phase:**
1. Write full module content in `learn_content_scratchpad/network-like-hyperscaler/content/`
2. Follow HHLearn authoring guide format
3. Test labs in vlab
4. Review and refine

**Publication Phase:**
1. Migrate final content to `hh-learn/content/modules/`
2. Create course JSON in `hh-learn/content/courses/`
3. Create pathway JSON in `hh-learn/content/pathways/`
4. Sync to HubDB
5. Verify rendering on hedgehog.cloud/learn

---

## External Resources

### Hedgehog Documentation
- **Local:** `/home/ubuntu/afewell-hh/docs/docs/`
- **Online:** https://docs.hedgehog.githedgehog.com/

### Key Documentation Sections
- `/docs/docs/user-guide/` - VPC, connections, external peering
- `/docs/docs/architecture/` - Fabric architecture, spine-leaf, overlay
- `/docs/docs/vlab/` - Virtual lab setup and usage
- `/docs/docs/reference/` - CLI, API references

---

## Quick Reference

### Most Important Files
1. **Learning Philosophy:** `hedgehogLearningPhilosophy.md` (guides all content)
2. **Project Plan:** `PROJECT_PLAN.md` (master roadmap)
3. **CRD Reference:** `research/CRD_REFERENCE.md` (technical truth)
4. **Workflows:** `research/WORKFLOWS.md` (validated procedures)
5. **Vlab Wiring:** `/home/ubuntu/afewell-hh/hhfab-default-topo/include/vlab.generated.yaml`

### Key Commands
```bash
# Vlab access
kubectl get switches,servers,connections

# hhfab operations
cd /home/ubuntu/afewell-hh/hhfab-default-topo
hhfab validate

# Content sync (when ready)
cd /home/ubuntu/afewell-hh/hh-learn
npm run sync:content
npm run sync:pathways
```

---

**Maintained by:** Course Lead (Claude)
**Purpose:** Central reference for all project resources
**Updates:** As needed when new resources are added
