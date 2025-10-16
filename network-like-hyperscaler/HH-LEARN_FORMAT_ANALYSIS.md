# HH-Learn Format Analysis & Remediation Plan
## Network Like Hyperscaler - Course 1 Foundations

**Document Version:** 1.0
**Date:** 2025-10-16
**Status:** Analysis Complete - Ready for Remediation

---

## Executive Summary

This document analyzes the format differences between the current Course 1 modules in the Network Like Hyperscaler pathway and the required hh-learn platform format. It provides a detailed remediation plan to convert all 4 modules to the proper platform structure.

**Current State:**
- 4 modules in custom format at: `/home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler/course-1-foundations/`
- Modules use custom Markdown structure with inline metadata
- Files named: `module-1.X-name.md`

**Required State:**
- Individual module directories at: `/home/ubuntu/afewell-hh/hh-learn/content/modules/<slug>/README.md`
- YAML front matter with required fields
- Course JSON file at: `/home/ubuntu/afewell-hh/hh-learn/content/courses/network-like-hyperscaler-foundations.json`
- Pathway JSON file at: `/home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler.json`

**Estimated Effort:** 4-6 hours (manual conversion recommended due to content quality and structure variations)

---

## Table of Contents

1. [Platform Structure Overview](#platform-structure-overview)
2. [Current Module Analysis](#current-module-analysis)
3. [Format Differences](#format-differences)
4. [Required Platform Structure](#required-platform-structure)
5. [Detailed Remediation Plan](#detailed-remediation-plan)
6. [Conversion Checklist](#conversion-checklist)
7. [Testing & Validation](#testing--validation)

---

## Platform Structure Overview

### How HH-Learn Organizes Content

The hh-learn platform uses a three-tier hierarchy:

```
Pathways (Top Level)
  └─ Courses (Middle Level)
      └─ Modules (Individual Units)
```

**Pathways:**
- JSON files in `content/pathways/`
- Define high-level learning journeys
- Reference courses (not modules directly)
- Example: `getting-started.json`

**Courses:**
- JSON files in `content/courses/`
- Group related modules into cohesive learning units
- Reference module slugs
- Example: `course-authoring-101.json`

**Modules:**
- Individual directories in `content/modules/<slug>/`
- Each contains `README.md` with front matter
- Optional `meta.json` for additional metadata
- Example: `content/modules/authoring-basics/README.md`

### Why This Structure Matters

1. **Flexibility:** Modules can be reused across multiple courses
2. **Discovery:** Platform can query by tags, difficulty, etc.
3. **SEO:** Front matter provides metadata for search engines
4. **Sync:** Content syncs to HubDB for dynamic rendering
5. **GitOps:** Clear separation between content and organization

---

## Current Module Analysis

### Module Inventory

| Current File | Title | Duration | Status |
|--------------|-------|----------|--------|
| `module-1.1-welcome.md` | Welcome to Fabric Operations | 15 min | Well-structured |
| `module-1.2-how-hedgehog-works.md` | How Hedgehog Works: The Control Model | 15 min | Well-structured |
| `module-1.3-mastering-interfaces.md` | Mastering the Three Interfaces | 15 min | Well-structured |
| `module-1.4-course-recap.md` | Course 1 Recap & Forward Map | 8-10 min | Well-structured |

### Current Format Characteristics

**Location:**
```
/home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler/course-1-foundations/
```

**File Structure:**
```markdown
# Module 1.X: Title

**Course:** Course 1 - Foundations & Interfaces
**Duration:** 15 minutes
**Prerequisites:** [List]

---

## Introduction
[Content...]

## Core Concepts
[Content...]

## Hands-On Lab
[Content...]

## Wrap-Up
[Content...]

## Assessment
[Quiz questions...]

## Reference
[Links and references...]
```

**Key Observations:**

1. **No YAML Front Matter:** Metadata is inline in Markdown
2. **Good Content Structure:** Sections are well-organized
3. **Comprehensive:** Includes scenarios, labs, assessments
4. **Proper Markdown:** Code blocks, tables, lists are correct
5. **Internal Links:** Some references to other pathway files
6. **Assessment Integrated:** Quiz questions are in Markdown

---

## Format Differences

### 1. Front Matter Requirements

#### Current Format (Inline Metadata)
```markdown
# Module 1.1: Welcome to Fabric Operations

**Course:** Course 1 - Foundations & Interfaces
**Duration:** 15 minutes
**Prerequisites:** Basic command-line familiarity, general understanding of networks
```

#### Required Format (YAML Front Matter)
```yaml
---
title: "Welcome to Fabric Operations"
slug: "welcome-to-fabric-operations"
difficulty: "beginner"
estimated_minutes: 15
version: "v1.0"
validated_on: "2025-10-16"
tags:
  - hedgehog
  - fabric-operations
  - foundations
  - getting-started
description: "Learn the Fabric Operator role, explore the Hedgehog vlab environment, and understand the three-tier resource hierarchy using kubectl."
order: 10
---
```

**Missing Fields:**
- `slug` - URL-friendly identifier (REQUIRED)
- `difficulty` - beginner/intermediate/advanced (REQUIRED)
- `estimated_minutes` - Numeric value (REQUIRED)
- `tags` - Array for filtering/discovery (REQUIRED)
- `description` - 120-160 chars for SEO (REQUIRED)
- `order` - Sort weight (RECOMMENDED)
- `version` - Content version tracking (RECOMMENDED)
- `validated_on` - Last validation date (RECOMMENDED)

### 2. Directory Structure

#### Current
```
content/pathways/network-like-hyperscaler/course-1-foundations/
├── module-1.1-welcome.md
├── module-1.2-how-hedgehog-works.md
├── module-1.3-mastering-interfaces.md
└── module-1.4-course-recap.md
```

#### Required
```
content/modules/
├── welcome-to-fabric-operations/
│   └── README.md
├── how-hedgehog-works/
│   └── README.md
├── mastering-three-interfaces/
│   └── README.md
└── course-1-recap/
    └── README.md

content/courses/
└── network-like-hyperscaler-foundations.json

content/pathways/
└── network-like-hyperscaler.json
```

**Key Changes:**
- Modules move to `content/modules/<slug>/README.md`
- Each module gets its own directory
- Course becomes a JSON file
- Pathway becomes a JSON file (may not exist yet)

### 3. Content Structure Alignment

#### Current Structure
```markdown
## Introduction
### A Day in the Life
### What You'll Learn
### Learning Objectives
### Setting Expectations

## Core Concepts
### Concept 1: ...
### Concept 2: ...

## Hands-On Lab
### Lab Title: ...
### Task 1: ...
### Task 2: ...

## Wrap-Up
### Key Takeaways
### Preview of Module 1.X

## Assessment
### Quiz Questions

## Reference
### Hedgehog CRDs Used
### kubectl Commands Reference
```

#### Required Structure (from template)
```markdown
# Module Title

Introduction paragraph

## Learning Objectives
- Objective 1
- Objective 2

## Prerequisites
- Prereq 1
- Prereq 2

## Scenario: <Scenario Title>
Context...

### Step 1: <Step Name>
Instructions...

### Step 2: <Step Name>
Instructions...

## Concepts & Deep Dive
Explanations...

## Troubleshooting
- Symptom → Cause → Fix

## Resources
- Links...
```

**Alignment Assessment:**
- ✅ Current structure is VERY CLOSE to required
- ✅ Main sections align well
- ⚠️ Some reordering needed (Prerequisites before Scenario)
- ⚠️ "Hands-On Lab" should be "Scenario"
- ⚠️ "Core Concepts" should be "Concepts & Deep Dive"
- ✅ Assessment can remain (not in template but acceptable)

### 4. Metadata Location

| Metadata | Current Location | Required Location |
|----------|------------------|-------------------|
| Title | H1 in Markdown | Both front matter `title` AND H1 |
| Slug | N/A | Front matter `slug` (required) |
| Duration | Inline bold text | Front matter `estimated_minutes` |
| Prerequisites | Inline bold text | H2 section after Learning Objectives |
| Difficulty | Implicit | Front matter `difficulty` |
| Tags | N/A | Front matter `tags` array |
| Description | N/A | Front matter `description` |

### 5. Assessment Handling

**Current:** Quiz questions are embedded in module Markdown

**Options for Required Format:**
1. **Keep in Markdown** (Recommended): Acceptable per platform - assessment sections are valid
2. **Separate file:** Could move to `assessment.md` but not required
3. **Interactive:** Future enhancement - integrate with quiz system

**Recommendation:** Keep assessments in Markdown for now. The platform supports rich content in modules.

### 6. Internal References

**Current Issues:**
- Links to pathway-specific files: `../../../network-like-hyperscaler/hedgehogLearningPhilosophy.md`
- Links to research files: `../../../network-like-hyperscaler/research/CRD_REFERENCE.md`

**Required Changes:**
- Research files should be in a shared location or referenced by public URL
- Cross-module references should use slugs: `intro-to-kubernetes` (system auto-links)
- External docs should use full URLs

---

## Required Platform Structure

### Module Front Matter Template

```yaml
---
title: "<Display title for module>"
slug: "<lowercase-hyphen-slug>"
difficulty: "beginner"  # beginner | intermediate | advanced
estimated_minutes: 15
version: "v1.0"
validated_on: "2025-10-16"
tags:
  - hedgehog
  - fabric-operations
  - networking
  - getting-started
description: "120-160 character description for SEO and module cards. What learners will achieve."
order: 10  # Lower numbers appear first

# Optional but recommended
pathway_slug: "network-like-hyperscaler"
pathway_name: "Network Like Hyperscaler"

# Optional: Agent-ready maintenance (future use)
products:
  - name: hedgehog-fabric
    repo: hedgehog-cloud/fabric
    min_version: 1.0.0
scenarios:
  - id: core-task
    title: "Scenario title"
    persona: "Fabric Operator"
    environment: "Hedgehog vlab"
    steps: []
ai_hints:
  environment: "vlab"
  retries: 1
---
```

### Course JSON Template

```json
{
  "slug": "network-like-hyperscaler-foundations",
  "title": "Course 1: Foundations & Interfaces",
  "summary_markdown": "<p>Learn to operate Hedgehog Fabric with confidence. This course teaches the operator role, GitOps workflow, and operational interfaces.</p>",
  "modules": [
    "welcome-to-fabric-operations",
    "how-hedgehog-works",
    "mastering-three-interfaces",
    "course-1-recap"
  ],
  "badge_image_url": "",
  "content_blocks": [
    {
      "id": "intro",
      "type": "text",
      "title": "Welcome to Foundations & Interfaces",
      "body_markdown": "Course introduction text here."
    },
    {
      "id": "prereqs",
      "type": "callout",
      "title": "Prerequisites",
      "body_markdown": "**Required:**\n- Basic command-line familiarity\n- General networking understanding"
    },
    {
      "id": "module1",
      "type": "module_ref",
      "module_slug": "welcome-to-fabric-operations"
    },
    {
      "id": "module2",
      "type": "module_ref",
      "module_slug": "how-hedgehog-works"
    },
    {
      "id": "module3",
      "type": "module_ref",
      "module_slug": "mastering-three-interfaces"
    },
    {
      "id": "module4",
      "type": "module_ref",
      "module_slug": "course-1-recap"
    },
    {
      "id": "next-steps",
      "type": "text",
      "title": "What's Next?",
      "body_markdown": "Continue to Course 2: Provisioning Operations to apply these skills hands-on."
    }
  ],
  "display_order": 1,
  "tags": "hedgehog,fabric-operations,foundations",
  "estimated_minutes": 53
}
```

### Pathway JSON Template

```json
{
  "slug": "network-like-hyperscaler",
  "title": "Network Like Hyperscaler",
  "summary_markdown": "Learn to operate Hedgehog Fabric like a hyperscaler: declarative infrastructure, GitOps workflows, and Kubernetes-native management. This pathway covers foundations, provisioning, and advanced operations.",
  "courses": [
    "network-like-hyperscaler-foundations",
    "network-like-hyperscaler-provisioning",
    "network-like-hyperscaler-observability",
    "network-like-hyperscaler-advanced"
  ],
  "display_order": 10,
  "tags": "hedgehog,fabric,operations,advanced"
}
```

---

## Detailed Remediation Plan

### Phase 1: Preparation (30 minutes)

#### Step 1.1: Create Directory Structure
```bash
# Create module directories
mkdir -p /home/ubuntu/afewell-hh/hh-learn/content/modules/welcome-to-fabric-operations
mkdir -p /home/ubuntu/afewell-hh/hh-learn/content/modules/how-hedgehog-works
mkdir -p /home/ubuntu/afewell-hh/hh-learn/content/modules/mastering-three-interfaces
mkdir -p /home/ubuntu/afewell-hh/hh-learn/content/modules/course-1-recap

# Verify structure
ls -la /home/ubuntu/afewell-hh/hh-learn/content/modules/
```

#### Step 1.2: Define Module Slugs and Metadata

| Module | Slug | Difficulty | Est. Minutes | Order |
|--------|------|------------|--------------|-------|
| Module 1.1 | `welcome-to-fabric-operations` | beginner | 15 | 10 |
| Module 1.2 | `how-hedgehog-works` | beginner | 15 | 20 |
| Module 1.3 | `mastering-three-interfaces` | beginner | 15 | 30 |
| Module 1.4 | `course-1-recap` | beginner | 10 | 40 |

**Tag Strategy:**
- Common tags: `hedgehog`, `fabric-operations`, `foundations`
- Module-specific tags:
  - 1.1: `getting-started`, `kubectl`, `operator-role`
  - 1.2: `gitops`, `reconciliation`, `controllers`
  - 1.3: `interfaces`, `troubleshooting`, `grafana`, `gitea`
  - 1.4: `recap`, `course-summary`

### Phase 2: Module Conversion (2-3 hours)

For EACH module, follow this process:

#### Module Conversion Checklist

**A. Create Front Matter**
1. ✅ Add YAML delimiter `---`
2. ✅ Add `title` (from current H1, remove "Module X.X:" prefix)
3. ✅ Add `slug` (lowercase-hyphen version)
4. ✅ Add `difficulty: "beginner"`
5. ✅ Add `estimated_minutes` (from current inline)
6. ✅ Add `version: "v1.0"`
7. ✅ Add `validated_on: "2025-10-16"`
8. ✅ Add `tags` array (see tag strategy)
9. ✅ Add `description` (write 120-160 chars summarizing module)
10. ✅ Add `order` (see table above)
11. ✅ Add `pathway_slug: "network-like-hyperscaler"`
12. ✅ Add `pathway_name: "Network Like Hyperscaler"`
13. ✅ Close YAML with `---`

**B. Update Content Structure**
1. ✅ Keep H1 title (remove "Module X.X:" prefix)
2. ✅ Remove inline metadata (Course, Duration, Prerequisites lines)
3. ✅ Keep "Introduction" section as-is
4. ✅ Rename "Core Concepts" → "Concepts & Deep Dive"
5. ✅ Rename "Hands-On Lab" → "Scenario: <Title>"
6. ✅ Move Prerequisites to H2 section after Learning Objectives (if inline)
7. ✅ Keep "Wrap-Up" section
8. ✅ Keep "Assessment" section (if present)
9. ✅ Rename "Reference" → "Resources" (or keep as-is)

**C. Fix Internal Links**
1. ✅ Update pathway-specific links
2. ✅ Update research file links
3. ✅ Convert module references to slugs
4. ✅ Ensure all external links have full URLs

**D. Validate Content**
1. ✅ All code blocks have language hints (```bash, ```yaml, etc.)
2. ✅ No broken Markdown formatting
3. ✅ Description is 120-160 characters
4. ✅ Front matter YAML is valid

**E. Save File**
1. ✅ Save as `/content/modules/<slug>/README.md`
2. ✅ Verify file exists and is readable

### Phase 3: Course JSON Creation (30 minutes)

#### Step 3.1: Create Course JSON

**File:** `/home/ubuntu/afewell-hh/hh-learn/content/courses/network-like-hyperscaler-foundations.json`

```json
{
  "slug": "network-like-hyperscaler-foundations",
  "title": "Course 1: Foundations & Interfaces",
  "summary_markdown": "<p>Master the foundational skills for operating Hedgehog Fabric. This course introduces the Fabric Operator role, explains how Hedgehog's control model works, and teaches you to use kubectl, Gitea, and Grafana fluently.</p>\n<p><strong>What You'll Learn:</strong></p>\n<ul>\n<li>Navigate fabric topology using kubectl</li>\n<li>Understand GitOps reconciliation and the control loop</li>\n<li>Choose the right interface (kubectl, Gitea, Grafana) for any task</li>\n<li>Apply systematic troubleshooting methodology</li>\n</ul>\n<p>By the end of this course, you'll operate a Hedgehog Fabric with confidence.</p>",
  "modules": [
    "welcome-to-fabric-operations",
    "how-hedgehog-works",
    "mastering-three-interfaces",
    "course-1-recap"
  ],
  "badge_image_url": "",
  "content_blocks": [
    {
      "id": "intro",
      "type": "text",
      "title": "Welcome to Foundations & Interfaces",
      "body_markdown": "This course teaches you to **operate** Hedgehog Fabric with confidence. You'll learn the operator role, understand how the control model works, and master the three operational interfaces.\n\nYou're not learning to design fabrics from scratch—you're learning to manage an existing fabric effectively and troubleshoot with confidence."
    },
    {
      "id": "prereqs",
      "type": "callout",
      "title": "Prerequisites",
      "body_markdown": "**Required:**\n- Basic command-line familiarity (navigating directories, running commands)\n- General understanding of networking concepts (VLANs, subnets, switches)\n\n**Helpful but not required:**\n- Kubernetes basics (helpful but not mandatory)\n- Git and version control experience\n- Experience with monitoring tools like Grafana"
    },
    {
      "id": "module1",
      "type": "module_ref",
      "module_slug": "welcome-to-fabric-operations"
    },
    {
      "id": "module2",
      "type": "module_ref",
      "module_slug": "how-hedgehog-works"
    },
    {
      "id": "module3",
      "type": "module_ref",
      "module_slug": "mastering-three-interfaces"
    },
    {
      "id": "module4",
      "type": "module_ref",
      "module_slug": "course-1-recap"
    },
    {
      "id": "next-steps",
      "type": "text",
      "title": "What's Next?",
      "body_markdown": "After completing Course 1, continue to **Course 2: Provisioning Operations** where you'll:\n\n- Design and create VPCs from scratch\n- Attach servers to VPCs using VPCAttachment CRDs\n- Validate end-to-end connectivity\n- Safely decommission resources\n\nCourse 2 applies everything you learned here to real provisioning operations."
    }
  ],
  "display_order": 1,
  "tags": "hedgehog,fabric-operations,foundations,kubectl,gitops",
  "estimated_minutes": 55
}
```

### Phase 4: Pathway JSON Creation (15 minutes)

#### Step 4.1: Check if Pathway Exists

```bash
ls -la /home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler.json
```

If it doesn't exist, create it:

**File:** `/home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler.json`

```json
{
  "slug": "network-like-hyperscaler",
  "title": "Network Like Hyperscaler",
  "summary_markdown": "Learn to operate Hedgehog Fabric using the same principles that hyperscalers use: declarative infrastructure, GitOps workflows, and Kubernetes-native management.\n\nThis pathway covers everything from foundational concepts to advanced operations, preparing you to manage production fabric deployments with confidence.",
  "courses": [
    "network-like-hyperscaler-foundations"
  ],
  "display_order": 10,
  "tags": "hedgehog,fabric,operations,gitops,kubernetes"
}
```

**Note:** Additional courses (Course 2, 3, 4) can be added to the `courses` array as they're converted.

### Phase 5: Testing & Validation (30-60 minutes)

#### Step 5.1: Validate YAML Front Matter

```bash
# For each module, validate front matter
head -30 /home/ubuntu/afewell-hh/hh-learn/content/modules/welcome-to-fabric-operations/README.md

# Check for common issues:
# - Missing closing ---
# - Invalid YAML syntax
# - Unquoted strings with special chars
# - Missing required fields
```

#### Step 5.2: Validate JSON Files

```bash
# Validate course JSON
cat /home/ubuntu/afewell-hh/hh-learn/content/courses/network-like-hyperscaler-foundations.json | jq .

# Validate pathway JSON
cat /home/ubuntu/afewell-hh/hh-learn/content/pathways/network-like-hyperscaler.json | jq .
```

#### Step 5.3: Check Module References

```bash
# Verify all modules referenced in course JSON exist
cd /home/ubuntu/afewell-hh/hh-learn/content/modules
ls -d welcome-to-fabric-operations how-hedgehog-works mastering-three-interfaces course-1-recap
```

#### Step 5.4: Test Sync (if possible)

```bash
# Run content sync to HubDB
npm run sync:content

# Check for errors in output
# Expected: 4 modules synced successfully
```

#### Step 5.5: Visual Inspection

1. Open each module README.md
2. Verify front matter is complete
3. Check that content renders correctly in Markdown preview
4. Verify all code blocks have language hints
5. Check description length (120-160 chars)

---

## Conversion Checklist

### Module 1.1: Welcome to Fabric Operations

**File Conversion:**
- [ ] Create directory: `content/modules/welcome-to-fabric-operations/`
- [ ] Add front matter with all required fields
- [ ] Update title (remove "Module 1.1:" prefix)
- [ ] Remove inline metadata
- [ ] Rename sections as needed
- [ ] Fix internal links
- [ ] Validate YAML
- [ ] Save as `README.md`

**Front Matter Fields:**
- [ ] title: "Welcome to Fabric Operations"
- [ ] slug: "welcome-to-fabric-operations"
- [ ] difficulty: "beginner"
- [ ] estimated_minutes: 15
- [ ] version: "v1.0"
- [ ] validated_on: "2025-10-16"
- [ ] tags: [hedgehog, fabric-operations, foundations, getting-started, kubectl, operator-role]
- [ ] description: (120-160 chars)
- [ ] order: 10
- [ ] pathway_slug: "network-like-hyperscaler"
- [ ] pathway_name: "Network Like Hyperscaler"

**Content Updates:**
- [ ] H1: "Welcome to Fabric Operations" (no module number)
- [ ] Remove: `**Course:**`, `**Duration:**`, `**Prerequisites:**` lines
- [ ] Keep: All sections (Introduction, Core Concepts, Hands-On Lab, etc.)
- [ ] Rename: "Core Concepts" → "Concepts & Deep Dive"
- [ ] Rename: "Hands-On Lab" → "Scenario: Explore Your Fabric Environment"
- [ ] Fix: Internal links to pathway files
- [ ] Validate: All code blocks have language hints

### Module 1.2: How Hedgehog Works

**File Conversion:**
- [ ] Create directory: `content/modules/how-hedgehog-works/`
- [ ] Add front matter with all required fields
- [ ] Update title
- [ ] Remove inline metadata
- [ ] Rename sections as needed
- [ ] Fix internal links
- [ ] Validate YAML
- [ ] Save as `README.md`

**Front Matter Fields:**
- [ ] title: "How Hedgehog Works: The Control Model"
- [ ] slug: "how-hedgehog-works"
- [ ] difficulty: "beginner"
- [ ] estimated_minutes: 15
- [ ] version: "v1.0"
- [ ] validated_on: "2025-10-16"
- [ ] tags: [hedgehog, fabric-operations, gitops, reconciliation, controllers, kubernetes]
- [ ] description: (120-160 chars)
- [ ] order: 20
- [ ] pathway_slug: "network-like-hyperscaler"
- [ ] pathway_name: "Network Like Hyperscaler"

**Content Updates:**
- [ ] H1: "How Hedgehog Works: The Control Model"
- [ ] Remove inline metadata
- [ ] Rename: "Core Concepts" → "Concepts & Deep Dive"
- [ ] Rename: "Hands-On Lab" → "Scenario: Create a VPC and Observe Reconciliation"
- [ ] Fix internal links

### Module 1.3: Mastering the Three Interfaces

**File Conversion:**
- [ ] Create directory: `content/modules/mastering-three-interfaces/`
- [ ] Add front matter with all required fields
- [ ] Update title
- [ ] Remove inline metadata
- [ ] Rename sections as needed
- [ ] Fix internal links
- [ ] Validate YAML
- [ ] Save as `README.md`

**Front Matter Fields:**
- [ ] title: "Mastering the Three Interfaces"
- [ ] slug: "mastering-three-interfaces"
- [ ] difficulty: "beginner"
- [ ] estimated_minutes: 15
- [ ] version: "v1.0"
- [ ] validated_on: "2025-10-16"
- [ ] tags: [hedgehog, fabric-operations, kubectl, gitea, grafana, troubleshooting, interfaces]
- [ ] description: (120-160 chars)
- [ ] order: 30
- [ ] pathway_slug: "network-like-hyperscaler"
- [ ] pathway_name: "Network Like Hyperscaler"

**Content Updates:**
- [ ] H1: "Mastering the Three Interfaces"
- [ ] Remove inline metadata
- [ ] Keep: Multi-part structure (Part 1, 2, 3)
- [ ] Rename scenario sections appropriately
- [ ] Fix internal links

### Module 1.4: Course 1 Recap & Forward Map

**File Conversion:**
- [ ] Create directory: `content/modules/course-1-recap/`
- [ ] Add front matter with all required fields
- [ ] Update title
- [ ] Remove inline metadata
- [ ] Rename sections as needed
- [ ] Fix internal links
- [ ] Validate YAML
- [ ] Save as `README.md`

**Front Matter Fields:**
- [ ] title: "Course 1 Recap & Forward Map"
- [ ] slug: "course-1-recap"
- [ ] difficulty: "beginner"
- [ ] estimated_minutes: 10
- [ ] version: "v1.0"
- [ ] validated_on: "2025-10-16"
- [ ] tags: [hedgehog, fabric-operations, recap, course-summary, review]
- [ ] description: (120-160 chars)
- [ ] order: 40
- [ ] pathway_slug: "network-like-hyperscaler"
- [ ] pathway_name: "Network Like Hyperscaler"

**Content Updates:**
- [ ] H1: "Course 1 Recap & Forward Map"
- [ ] Remove inline metadata
- [ ] Keep: All recap sections
- [ ] Update: Forward references to Course 2 (ensure they'll work as module slugs)
- [ ] Fix internal links

### Course JSON

- [ ] Create: `content/courses/network-like-hyperscaler-foundations.json`
- [ ] Add all required fields
- [ ] Reference all 4 module slugs in order
- [ ] Add content_blocks for intro, prereqs, module refs, next steps
- [ ] Validate JSON syntax
- [ ] Calculate total estimated_minutes (sum of all modules)

### Pathway JSON

- [ ] Check if exists: `content/pathways/network-like-hyperscaler.json`
- [ ] Create if missing
- [ ] Reference course slug: "network-like-hyperscaler-foundations"
- [ ] Validate JSON syntax

### Final Validation

- [ ] All 4 module directories exist with README.md
- [ ] All front matter YAML is valid
- [ ] All descriptions are 120-160 characters
- [ ] All module slugs match directory names
- [ ] Course JSON references correct module slugs
- [ ] Pathway JSON references correct course slug
- [ ] All code blocks have language hints
- [ ] All internal links updated
- [ ] Run `npm run sync:content` successfully
- [ ] Verify modules appear in platform (if possible)

---

## Testing & Validation

### Validation Script

Create a validation script to check all requirements:

```bash
#!/bin/bash
# validate-conversion.sh

echo "Validating Network Like Hyperscaler Course 1 Conversion"
echo "======================================================="

MODULES=(
  "welcome-to-fabric-operations"
  "how-hedgehog-works"
  "mastering-three-interfaces"
  "course-1-recap"
)

FAILED=0

echo ""
echo "Checking module directories..."
for module in "${MODULES[@]}"; do
  if [ -d "content/modules/$module" ]; then
    echo "✓ Directory exists: $module"
  else
    echo "✗ Missing directory: $module"
    FAILED=1
  fi

  if [ -f "content/modules/$module/README.md" ]; then
    echo "  ✓ README.md exists"
  else
    echo "  ✗ README.md missing"
    FAILED=1
  fi
done

echo ""
echo "Validating front matter..."
for module in "${MODULES[@]}"; do
  FILE="content/modules/$module/README.md"
  if [ ! -f "$FILE" ]; then
    continue
  fi

  echo "Checking $module:"

  # Check for required fields
  REQUIRED_FIELDS=("title" "slug" "difficulty" "estimated_minutes" "tags" "description")
  for field in "${REQUIRED_FIELDS[@]}"; do
    if grep -q "^$field:" "$FILE"; then
      echo "  ✓ Has $field"
    else
      echo "  ✗ Missing $field"
      FAILED=1
    fi
  done

  # Check description length
  DESC=$(grep "^description:" "$FILE" | sed 's/description: "\(.*\)"/\1/')
  DESC_LEN=${#DESC}
  if [ $DESC_LEN -ge 120 ] && [ $DESC_LEN -le 160 ]; then
    echo "  ✓ Description length OK ($DESC_LEN chars)"
  else
    echo "  ⚠ Description length: $DESC_LEN chars (should be 120-160)"
  fi
done

echo ""
echo "Checking JSON files..."
if [ -f "content/courses/network-like-hyperscaler-foundations.json" ]; then
  echo "✓ Course JSON exists"
  if jq . "content/courses/network-like-hyperscaler-foundations.json" > /dev/null 2>&1; then
    echo "  ✓ Valid JSON"
  else
    echo "  ✗ Invalid JSON"
    FAILED=1
  fi
else
  echo "✗ Course JSON missing"
  FAILED=1
fi

if [ -f "content/pathways/network-like-hyperscaler.json" ]; then
  echo "✓ Pathway JSON exists"
  if jq . "content/pathways/network-like-hyperscaler.json" > /dev/null 2>&1; then
    echo "  ✓ Valid JSON"
  else
    echo "  ✗ Invalid JSON"
    FAILED=1
  fi
else
  echo "⚠ Pathway JSON missing (may already exist)"
fi

echo ""
if [ $FAILED -eq 0 ]; then
  echo "✅ All validations passed!"
  exit 0
else
  echo "❌ Some validations failed. Please review."
  exit 1
fi
```

### Manual Testing Checklist

After conversion:

1. **Visual Inspection:**
   - [ ] Open each README.md in a Markdown viewer
   - [ ] Verify front matter renders correctly
   - [ ] Check that content is readable
   - [ ] Confirm code blocks have syntax highlighting

2. **Link Testing:**
   - [ ] Click through all internal module references
   - [ ] Verify external links work
   - [ ] Check that pathway references resolve

3. **Platform Integration:**
   - [ ] Run `npm run sync:content`
   - [ ] Check sync output for errors
   - [ ] Verify modules appear in HubDB (if accessible)
   - [ ] Test module rendering on platform (if accessible)

4. **Cross-Reference Testing:**
   - [ ] Course JSON references all modules
   - [ ] Module slugs match directory names
   - [ ] Pathway JSON references course
   - [ ] Tags are consistent across related modules

---

## Recommendations

### Approach: Manual vs Automated

**Recommended: Manual Conversion (4-6 hours)**

**Why manual?**
1. Content quality is high - minimal risk of breaking things
2. Allows thoughtful description writing (120-160 chars is crucial for SEO)
3. Opportunity to improve structure/flow during conversion
4. Can fix any content issues discovered during review
5. Better tag selection based on content understanding

**When to automate:**
- If converting 20+ modules (automation ROI improves)
- If modules are highly standardized
- After successfully doing 1-2 manual conversions to establish patterns

### Suggested Work Order

**Session 1 (2 hours):**
1. Set up directory structure
2. Convert Module 1.1 completely (use as reference)
3. Create course JSON
4. Create/update pathway JSON
5. Test Module 1.1 end-to-end

**Session 2 (2 hours):**
1. Convert Module 1.2
2. Convert Module 1.3
3. Test both modules
4. Update course JSON if needed

**Session 3 (1-2 hours):**
1. Convert Module 1.4
2. Final validation of all modules
3. Run sync and test platform integration
4. Document any issues/learnings

### Description Writing Tips

Each module needs a 120-160 character description. Formula:

**"Learn [skill/concept]. [Key outcome/benefit]. [What makes it unique/valuable]."**

Examples:

**Module 1.1:**
> "Learn the Fabric Operator role, explore the Hedgehog vlab environment, and understand the three-tier resource hierarchy using kubectl." (147 chars)

**Module 1.2:**
> "Understand how CRD reconciliation works, observe the control loop in action, and learn what happens when you create a VPC." (124 chars)

**Module 1.3:**
> "Master when to use kubectl, Gitea, and Grafana. Learn systematic troubleshooting by correlating data across all three interfaces." (131 chars)

**Module 1.4:**
> "Consolidate your Course 1 knowledge with self-check questions and preview Course 2 provisioning operations." (109 chars - too short, expand)

Better version for 1.4:
> "Review Course 1 foundations with knowledge checks, understand what you've accomplished, and prepare for hands-on provisioning in Course 2." (140 chars)

### Tag Strategy Rationale

**Common tags across all modules:**
- `hedgehog` - Platform name (helps with search)
- `fabric-operations` - High-level category
- `foundations` - Course-level tag

**Module-specific tags (3-4 additional):**
- Choose based on key topics covered
- Include tool names (`kubectl`, `gitea`, `grafana`)
- Include skill categories (`troubleshooting`, `monitoring`)
- Include concepts (`gitops`, `reconciliation`)

**Total tags per module: 6-8** (good for discovery without over-tagging)

---

## Appendix: Example Conversions

### Example: Module 1.1 Front Matter

```yaml
---
title: "Welcome to Fabric Operations"
slug: "welcome-to-fabric-operations"
difficulty: "beginner"
estimated_minutes: 15
version: "v1.0"
validated_on: "2025-10-16"
tags:
  - hedgehog
  - fabric-operations
  - foundations
  - getting-started
  - kubectl
  - operator-role
description: "Learn the Fabric Operator role, explore the Hedgehog vlab environment, and understand the three-tier resource hierarchy using kubectl."
order: 10
pathway_slug: "network-like-hyperscaler"
pathway_name: "Network Like Hyperscaler"
---

# Welcome to Fabric Operations

Welcome to module authoring! You're a Fabric Operator at a growing company...
```

### Example: Course JSON Snippet

```json
{
  "slug": "network-like-hyperscaler-foundations",
  "title": "Course 1: Foundations & Interfaces",
  "modules": [
    "welcome-to-fabric-operations",
    "how-hedgehog-works",
    "mastering-three-interfaces",
    "course-1-recap"
  ],
  "content_blocks": [
    {
      "id": "module1",
      "type": "module_ref",
      "module_slug": "welcome-to-fabric-operations"
    }
  ],
  "estimated_minutes": 55
}
```

---

## Summary

This analysis provides a comprehensive roadmap for converting Course 1 modules to the hh-learn platform format. The conversion is straightforward because:

1. **Content is high quality** - Well-structured, comprehensive, pedagogically sound
2. **Structure is close** - Current structure aligns well with required template
3. **Changes are mostly additive** - Adding front matter, not rewriting content
4. **Clear requirements** - Platform expectations are well-documented

**Key Actions:**
- Add YAML front matter to all modules
- Move modules to individual directories
- Create course JSON file
- Create/update pathway JSON file
- Fix internal links
- Validate and test

**Estimated Effort:** 4-6 hours of focused work, best done in 3 sessions.

**Recommended Approach:** Manual conversion for quality and thoughtful description writing.

**Next Steps:**
1. Review this analysis
2. Begin with Module 1.1 conversion as reference
3. Use conversion checklist for each module
4. Test incrementally
5. Document any issues for future conversions

---

**Document End**
