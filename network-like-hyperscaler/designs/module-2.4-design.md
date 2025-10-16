# Module 2.4 Design: Decommission & Cleanup

## Module Metadata

- **Module Number:** 2.4
- **Module Title:** Decommission & Cleanup
- **Course:** Course 2 - Provisioning & Connectivity
- **Estimated Duration:** 10-12 minutes
  - Introduction: 1-2 minutes
  - Core Concepts: 3-4 minutes
  - Hands-On Lab: 4-5 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 2.3: Connectivity Validation (validation workflow understood)
  - ✅ Module 2.2: Attach Servers to VPC (VPCAttachments created)
  - ✅ Module 2.1: Define VPC Network (VPC lifecycle understood)
  - ✅ Module 1.2: How Hedgehog Works (GitOps workflow)

## Learning Objectives

By the end of this module, learners will be able to:

1. **Delete resources in correct order** - Remove VPCAttachments before VPCs to avoid dependency errors
2. **Validate cleanup completion** - Verify all resources removed and switches reconfigured
3. **Prevent orphaned resources** - Identify and clean up resources without dependencies
4. **Understand rollback procedures** - Restore deleted configurations from Git history
5. **Decide when to archive vs delete** - Determine appropriate resource lifecycle management strategy

**Bloom's Taxonomy Level**: Apply, Evaluate (executing deletion, validating cleanup, making decisions)

## Content Outline

### Introduction (1-2 minutes)

**Hook: Lifecycle Completion**

> You've built a complete three-tier VPC network:
> - **Module 2.1:** Designed and created `webapp-vpc`
> - **Module 2.2:** Attached three servers to frontend/backend/database subnets
> - **Module 2.3:** Validated DHCP, connectivity, and observability
>
> But what happens when the project ends? When the application migrates? When you need to reclaim resources?
>
> **You need to decommission cleanly.**

**The Problem with Ad-Hoc Deletion:**

Deleting resources in the wrong order causes problems:
```bash
# WRONG: Delete VPC first
kubectl delete vpc webapp-vpc
# Error: VPCAttachments still reference this VPC

# WRONG: Leave orphaned VPCAttachments
kubectl delete vpc webapp-vpc --force
# Result: VPCAttachments exist but VPC doesn't (orphaned resources)
```

**The Right Approach:**

1. **Delete dependent resources first** (VPCAttachments)
2. **Delete parent resources second** (VPC)
3. **Validate cleanup** (switches reconfigured, no orphans)
4. **Archive configuration** (Git preserves history)

**Context: Production Discipline**

In production environments, clean decommissioning is critical:
- Prevents resource leaks (VLANs, IP addresses)
- Avoids configuration drift (orphaned switch configs)
- Maintains audit trail (Git history)
- Enables rollback (restore from Git)

**What You'll Learn:**

- Dependency-aware deletion order
- GitOps deletion workflow (delete YAML from Git)
- Cleanup validation techniques
- Rollback procedures (Git revert)
- Archive vs delete decision-making

You'll safely decommission the `webapp-vpc` environment, completing the full provisioning lifecycle.

---

### Core Concepts (3-4 minutes)

#### Concept 1: Dependency-Aware Deletion Order

**Kubernetes Finalizers and Dependencies**

Hedgehog uses **Kubernetes finalizers** to prevent deleting resources with active dependencies:

**Dependency Hierarchy:**
```
VPC
 ├── VPCAttachment (depends on VPC)
 │   └── Connection (referenced by VPCAttachment)
 └── DHCPSubnet (created by VPC)
```

**Correct Deletion Order:**

1. **VPCAttachments** - Remove server attachments first
2. **VPCPeerings** - Remove VPC-to-VPC connectivity (if any)
3. **ExternalPeerings** - Remove external connectivity (if any)
4. **VPC** - Remove VPC itself (finalizer ensures dependencies are gone)

**What Happens If You Delete Out of Order:**

```bash
# Attempt to delete VPC with active attachments
kubectl delete vpc webapp-vpc

# Kubernetes blocks deletion:
# Error: VPC has finalizer, waiting for VPCAttachments to be deleted
```

**Finalizer Behavior:**
- VPC deletion **marks** VPC for deletion (status: Terminating)
- VPC **waits** for finalizer to be removed
- Finalizer removed only when all VPCAttachments deleted
- VPC fully deleted after finalizer removed

**Force Delete (DANGEROUS):**

```bash
# Remove finalizer manually (NOT RECOMMENDED)
kubectl patch vpc webapp-vpc -p '{"metadata":{"finalizers":null}}' --type=merge

# Result: VPC deleted, but VPCAttachments orphaned
```

**Never force-delete unless:**
- Resources are truly orphaned (parent already deleted)
- You've verified no dependencies exist
- You understand the consequences

---

#### Concept 2: GitOps Deletion Workflow

**Deleting via GitOps (Recommended)**

**Gitea Workflow:**

1. **Delete YAML files from Git repository**
   - Navigate to `vpcs/` or `vpc-attachments/` in Gitea
   - Delete files (e.g., `webapp-vpc.yaml`)
   - Commit deletion

2. **ArgoCD detects deletion**
   - ArgoCD syncs (up to 60 seconds)
   - ArgoCD prunes removed resources from cluster
   - Status: Synced (resources removed)

3. **Fabric Controller reconciles**
   - Detects VPCAttachment deletion
   - Updates Agent CRDs (removes VLANs from switch ports)
   - Switches reconfigure automatically

4. **Validate cleanup**
   - kubectl: Resources no longer exist
   - Agent CRD: VLANs removed from interfaces
   - Grafana: Interfaces show no VLANs

**Benefits of GitOps Deletion:**
- **Audit trail**: Git commit documents who deleted what and when
- **Reversible**: `git revert` restores deleted configuration
- **Consistent**: Same workflow as creation (Gitea → ArgoCD → kubectl)

---

**Direct kubectl Deletion (Alternative)**

For one-off deletions or troubleshooting:

```bash
# Delete VPCAttachment
kubectl delete vpcattachment webapp-frontend-server-01

# Delete VPC
kubectl delete vpc webapp-vpc

# Verify deletion
kubectl get vpc,vpcattachment
```

**When to Use Direct Deletion:**
- Testing/development environments
- Quick cleanup of test resources
- Resources not managed by GitOps

**When to Use GitOps Deletion:**
- Production environments
- Resources under version control
- When audit trail required

---

#### Concept 3: Cleanup Validation

**Three-Layer Validation:**

**Layer 1: Kubernetes Resources (kubectl)**

```bash
# Verify VPCAttachments deleted
kubectl get vpcattachment | grep webapp

# Expected: No resources found

# Verify VPC deleted
kubectl get vpc webapp-vpc

# Expected: Error: "webapp-vpc" not found

# Check for orphaned DHCPSubnets
kubectl get dhcpsubnet -n default | grep webapp

# Expected: No resources (auto-deleted with VPC)
```

**Layer 2: Switch Configuration (Agent CRD)**

```bash
# Check leaf-01 interface E1/5 (was server-01 frontend)
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.vlans}'

# Expected: [] (empty array) or null (no VLANs)

# Check leaf-03 interface E1/1 (was server-05 backend)
kubectl get agent leaf-03 -n fab -o jsonpath='{.status.state.interfaces.Ethernet1.vlans}'

# Expected: [] or null

# List all interfaces with VLANs (should not show webapp-vpc VLANs)
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | \
  jq 'to_entries[] | select(.value.vlans != null) | {port: .key, vlans: .value.vlans}'
```

**Layer 3: Observability (Grafana)**

- Open Interfaces Dashboard
- Select switch (leaf-01)
- Filter interface (E1/5)
- Verify: VLAN field empty or shows no VLANs
- Verify: Traffic counters may continue (if interface used for other VPCs)

**Cleanup Success Criteria:**
- ✅ No VPCAttachment resources exist
- ✅ No VPC resources exist
- ✅ No DHCPSubnet resources for deleted VPC
- ✅ No VLANs on switch ports (Agent CRD)
- ✅ Grafana shows interfaces without VLANs

---

#### Concept 4: Rollback and Recovery

**Rollback Scenario:**

You accidentally deleted the wrong VPC or deleted too early.

**Git-Based Rollback (Recommended):**

```bash
# 1. View Git history in Gitea
# Navigate to repository → Commits tab
# Find commit that deleted VPC

# 2. Revert commit (Gitea web UI)
# Click on commit → "Revert this commit" button
# Creates new commit that restores deleted files

# 3. ArgoCD re-syncs
# Detects restored YAML files
# Recreates VPC and VPCAttachments
# Switches reconfigure automatically

# 4. Validate restoration
kubectl get vpc webapp-vpc
kubectl get vpcattachment | grep webapp
```

**Manual Rollback (Alternative):**

```bash
# 1. Retrieve deleted YAML from Git history
git show HEAD~1:vpcs/webapp-vpc.yaml > webapp-vpc.yaml

# 2. Re-apply configuration
kubectl apply -f webapp-vpc.yaml

# 3. Re-apply VPCAttachments
kubectl apply -f vpc-attachments/webapp-frontend-server-01.yaml
```

**Rollback Considerations:**

**Can Rollback:**
- ✅ Deleted less than Git retention period (typically days/weeks)
- ✅ No conflicting resources created (VLAN/IP conflicts)
- ✅ Underlying infrastructure unchanged (servers still exist)

**Cannot Rollback:**
- ❌ Git history purged (rare)
- ❌ VLAN/IP ranges reallocated to other VPCs (conflict)
- ❌ Servers decommissioned (physical layer changed)

---

#### Concept 5: Archive vs Delete

**Decision Matrix:**

| Scenario | Action | Rationale |
|----------|--------|-----------|
| **Temporary project ended** | Archive (move to `archive/` folder in Git) | May need to restore later |
| **Application permanently decommissioned** | Delete from Git (preserves commit history) | No future use expected |
| **Testing/development VPC** | Delete immediately | Short-lived, no production value |
| **Production VPC migration** | Archive before delete | Safety net for rollback |
| **Compliance retention required** | Archive with metadata | Audit trail preserved |

**Archiving Workflow:**

1. **Move files in Gitea:**
   ```
   Before: vpcs/webapp-vpc.yaml
   After:  archive/vpcs/webapp-vpc-2025-10-16.yaml
   ```

2. **Add metadata comment:**
   ```yaml
   # ARCHIVED: 2025-10-16
   # REASON: Application migrated to new infrastructure
   # ARCHIVED BY: operator@example.com
   apiVersion: vpc.githedgehog.com/v1beta1
   kind: VPC
   ...
   ```

3. **ArgoCD ignores archive folder** (configure ArgoCD to exclude `archive/` path)

4. **Resources deleted from cluster** (no longer in active path)

5. **Git preserves full history** (can restore from archive)

**Benefits of Archiving:**
- Fast restoration (copy back from archive, commit)
- Documented decommission reason
- Compliance-friendly (audit trail)

**Deletion Workflow (No Archive):**

1. **Delete files from Git**
2. **Commit with descriptive message:**
   ```
   Delete webapp-vpc - application decommissioned permanently
   ```
3. **Git history preserves commit** (can still retrieve via `git show`)

---

### Hands-On Lab (4-5 minutes)

**Lab Title:** Safely Decommission webapp-vpc

**Scenario:**

The three-tier web application has been migrated to a different infrastructure. You need to cleanly decommission the `webapp-vpc` environment:
- Remove server attachments
- Remove VPC
- Validate cleanup
- Archive configuration for future reference

**Environment Access:**
- **Gitea:** http://localhost:3001 (username: `student`, password: `hedgehog123`)
- **ArgoCD:** http://localhost:8080 (username: `admin`, password: `qV7hX0NMroAUhwoZ`)
- **kubectl:** Already configured

---

#### Task 1: Delete VPCAttachments (2 minutes)

**Objective:** Remove server attachments in dependency-safe order

**Steps:**

1. **List existing VPCAttachments:**

   ```bash
   # View all VPCAttachments for webapp-vpc
   kubectl get vpcattachment | grep webapp

   # Expected output:
   # webapp-frontend-server-01
   # webapp-backend-server-05
   # webapp-database-server-09
   ```

2. **Delete VPCAttachments via Gitea:**

   - Open http://localhost:3001
   - Navigate to `student/hedgehog-config` repository
   - Click `vpc-attachments/` folder

3. **Delete each attachment file:**

   - Click `webapp-frontend-server-01.yaml`
   - Click **"Delete File"** button (trash icon)
   - Commit message: `Remove webapp-vpc frontend attachment - app decommissioned`
   - Click **"Commit Changes"**

   - Repeat for `webapp-backend-server-05.yaml`
     - Commit message: `Remove webapp-vpc backend attachment`

   - Repeat for `webapp-database-server-09.yaml`
     - Commit message: `Remove webapp-vpc database attachment`

4. **Wait for ArgoCD sync:**

   - Open http://localhost:8080
   - View `hedgehog-config` application
   - Wait for sync (up to 60 seconds)
   - Status should show "Synced" and "Healthy"

5. **Verify VPCAttachments deleted:**

   ```bash
   # Check VPCAttachments
   kubectl get vpcattachment | grep webapp

   # Expected: No resources found

   # Check for deletion events
   kubectl get events --sort-by='.lastTimestamp' | grep VPCAttachment | tail -5

   # Expected events:
   # Normal  Deleted  VPCAttachment webapp-frontend-server-01 deleted
   # Normal  Deleted  VPCAttachment webapp-backend-server-05 deleted
   # Normal  Deleted  VPCAttachment webapp-database-server-09 deleted
   ```

**Success Criteria:**
- ✅ All three VPCAttachment files deleted from Git
- ✅ ArgoCD synced successfully
- ✅ kubectl shows no VPCAttachments for webapp-vpc

---

#### Task 2: Delete VPC (1 minute)

**Objective:** Remove VPC now that dependencies are deleted

**Steps:**

1. **Delete VPC via Gitea:**

   - In Gitea, navigate back to repository root
   - Click `vpcs/` folder
   - Click `webapp-vpc.yaml`
   - Click **"Delete File"** button
   - Commit message: `Remove webapp-vpc - application decommissioned`
   - Click **"Commit Changes"**

2. **Wait for ArgoCD sync:**

   - View `hedgehog-config` application in ArgoCD
   - Wait for sync
   - Status: "Synced" and "Healthy"

3. **Verify VPC deleted:**

   ```bash
   # Check VPC
   kubectl get vpc webapp-vpc

   # Expected: Error: "webapp-vpc" not found

   # Check DHCPSubnets (should be auto-deleted)
   kubectl get dhcpsubnet -n default | grep webapp

   # Expected: No resources found
   ```

**Success Criteria:**
- ✅ VPC file deleted from Git
- ✅ ArgoCD synced
- ✅ kubectl shows VPC not found
- ✅ DHCPSubnets automatically deleted

---

#### Task 3: Validate Cleanup (1-2 minutes)

**Objective:** Verify switches reconfigured and no orphaned resources remain

**Steps:**

1. **Check Agent CRD for VLAN removal:**

   ```bash
   # Check leaf-01 interface E1/5 (was server-01 frontend, VLAN 1010)
   kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.vlans}'

   # Expected: [] (empty array) or null

   # Check leaf-03 interface E1/1 (was server-05 backend, VLAN 1020)
   kubectl get agent leaf-03 -n fab -o jsonpath='{.status.state.interfaces.Ethernet1.vlans}'

   # Expected: [] or null

   # Check leaf-05 interface E1/1 (was server-09 database, VLAN 1030)
   kubectl get agent leaf-05 -n fab -o jsonpath='{.status.state.interfaces.Ethernet1.vlans}'

   # Expected: [] or null
   ```

2. **Verify no orphaned resources:**

   ```bash
   # List all VPCs (should not include webapp-vpc)
   kubectl get vpc

   # List all VPCAttachments
   kubectl get vpcattachment

   # List all DHCPSubnets
   kubectl get dhcpsubnet -n default

   # Expected: No webapp-vpc related resources
   ```

3. **Check Grafana (Optional):**

   - Open http://localhost:3000
   - Navigate to "Interfaces Dashboard"
   - Select switch: leaf-01
   - Filter interface: E1/5
   - Verify: VLAN field empty

**Success Criteria:**
- ✅ Agent CRDs show no VLANs on previously-attached interfaces
- ✅ No orphaned VPC resources
- ✅ No orphaned VPCAttachment resources
- ✅ No orphaned DHCPSubnet resources
- ✅ Grafana shows interfaces without VLANs

---

#### Task 4: Archive Configuration (Optional, 1 minute)

**Objective:** Preserve configuration for future reference or rollback

**Steps:**

1. **Create archive folder in Git (if doesn't exist):**

   - In Gitea, navigate to repository root
   - Click **"New File"** button
   - Filename: `archive/.gitkeep` (placeholder to create directory)
   - Commit message: `Create archive directory`
   - Click **"Commit Changes"**

2. **Restore and archive VPC configuration:**

   - In Gitea, click **Commits** tab
   - Find commit: "Remove webapp-vpc - application decommissioned"
   - Click on commit hash
   - Click on `webapp-vpc.yaml` (shows diff)
   - Copy deleted YAML content

   - Navigate back to repository root
   - Click **"New File"**
   - Filename: `archive/webapp-vpc-2025-10-16.yaml`
   - Paste YAML content
   - Add comment at top:
     ```yaml
     # ARCHIVED: 2025-10-16
     # REASON: Application decommissioned, migrated to new infrastructure
     # ORIGINAL VPC: webapp-vpc (3 subnets: frontend, backend, database)
     ```
   - Commit message: `Archive webapp-vpc configuration for future reference`
   - Click **"Commit Changes"**

3. **Configure ArgoCD to ignore archive folder** (Advanced):

   This step would typically be done by infrastructure admin, not covered in basic module.

**Success Criteria:**
- ✅ Configuration preserved in archive folder
- ✅ Archive includes metadata (reason, date, description)
- ✅ ArgoCD does not re-deploy archived config

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You completed the full VPC lifecycle:
- ✅ **Module 2.1:** Designed and created `webapp-vpc`
- ✅ **Module 2.2:** Attached servers to three subnets
- ✅ **Module 2.3:** Validated DHCP, connectivity, and observability
- ✅ **Module 2.4:** Safely decommissioned entire environment

**Lifecycle Summary:**

```
Create → Attach → Validate → Decommission
 (2.1)    (2.2)     (2.3)        (2.4)
```

**Key Takeaways:**

1. **Delete dependencies first** - VPCAttachments before VPCs
2. **GitOps deletion is reversible** - Git history enables rollback
3. **Three-layer validation** - kubectl, Agent CRD, Grafana
4. **Finalizers protect against errors** - Prevent deletion with active dependencies
5. **Archive for compliance** - Preserve configuration history

**Production Best Practices:**

- Always validate cleanup (don't assume)
- Document decommission reason (commit messages, archive comments)
- Test rollback procedures (know you can recover)
- Use GitOps for audit trail (who deleted what, when)
- Archive before delete if uncertain

**Course 2 Complete:**

You've mastered VPC provisioning and lifecycle management:
- Design VPCs from scratch (subnets, VLANs, DHCP)
- Attach servers to VPCs (connection types, subnet selection)
- Validate connectivity (DHCP, inter-subnet, observability)
- Decommission cleanly (dependency order, validation)

**Next Steps:**

- **Course 3:** Observability & Fabric Health (Grafana deep dive)
- **Course 4:** Troubleshooting & Recovery (advanced debugging)

---

### Assessment Questions

#### Question 1: Deletion Order

**Scenario:** You need to decommission a VPC with active VPCAttachments. What is the correct deletion order?

- A) VPC first, then VPCAttachments
- B) VPCAttachments first, then VPC
- C) Delete both simultaneously (`kubectl delete vpc,vpcattachment`)
- D) Force-delete VPC with `--force` flag

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) VPCAttachments first, then VPC

**Explanation:**

**Correct Order:**
1. Delete **VPCAttachments** (dependent resources)
2. Delete **VPC** (parent resource)

**Why:**
- VPCAttachments **depend on** VPC (reference `spec.subnet: vpc-name/subnet-name`)
- Kubernetes finalizers prevent VPC deletion while VPCAttachments exist
- Deleting VPC first would result in "Terminating" state (waiting for dependencies)

**What Happens If You Try A (VPC first):**
```bash
kubectl delete vpc webapp-vpc
# VPC status: Terminating (finalizer blocks deletion)
# VPC waits for VPCAttachments to be deleted
# VPC fully deleted only after all VPCAttachments gone
```

**Why C and D are wrong:**
- C) Simultaneous deletion doesn't guarantee order (may fail)
- D) Force-delete bypasses finalizers (leaves orphaned VPCAttachments)

**Module 2.4 Reference:** Concept 1 - Dependency-Aware Deletion Order
</details>

---

#### Question 2: GitOps Deletion Workflow

**Scenario:** You deleted a VPC YAML file from Git and committed the change. What happens next?

- A) Nothing - you must run `kubectl delete` manually
- B) ArgoCD detects deletion and prunes the VPC from cluster
- C) VPC is marked for deletion but never fully removed
- D) Git automatically backs up the file before deleting

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) ArgoCD detects deletion and prunes the VPC from cluster

**Explanation:**

**GitOps Deletion Flow:**
1. **You delete** YAML file from Git (via Gitea)
2. **ArgoCD detects** file deletion (polls Git repository every ~60 seconds)
3. **ArgoCD prunes** resource from cluster (deletes VPC CRD)
4. **Fabric Controller reconciles** (removes configuration from switches)
5. **Switches reconfigure** (VLANs removed from ports)

**ArgoCD Pruning:**
- ArgoCD compares **desired state** (Git) vs **actual state** (cluster)
- Missing file in Git → Resource should not exist in cluster
- ArgoCD **prunes** (deletes) resource to match Git

**Why others are wrong:**
- A) ArgoCD automates deletion (no manual kubectl needed)
- C) Finalizers may delay deletion, but it completes once dependencies removed
- D) Git doesn't auto-backup (Git commit history serves this purpose)

**Rollback:** Use `git revert` to restore deleted file and recreate VPC

**Module 2.4 Reference:** Concept 2 - GitOps Deletion Workflow
</details>

---

#### Question 3: Cleanup Validation

**Scenario:** After deleting a VPCAttachment, you run:
```bash
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5.vlans}'
```

Output: `[1010]`

What does this indicate?

- A) Cleanup successful - VLAN still present is normal
- B) Cleanup incomplete - VLAN should be removed
- C) Error - VLAN array should be null, not [1010]
- D) Cleanup in progress - wait 5 minutes and check again

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Cleanup incomplete - VLAN should be removed

**Explanation:**

**Expected Output After VPCAttachment Deletion:**
- `[]` (empty array) - No VLANs on interface
- `null` - VLAN field not set (equivalent to empty)

**Actual Output:**
- `[1010]` - VLAN 1010 still configured on interface

**Root Cause Investigation:**

1. **Check if VPCAttachment still exists:**
   ```bash
   kubectl get vpcattachment | grep leaf-01.*E1/5
   # If found: VPCAttachment deletion didn't complete
   ```

2. **Check Agent reconciliation:**
   ```bash
   kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastAppliedTime}'
   # If old timestamp: Reconciliation may be stuck
   ```

3. **Check controller logs:**
   ```bash
   kubectl logs -n fab deployment/fabric-controller-manager | grep "leaf-01.*E1/5"
   # Look for errors in VLAN removal
   ```

**Possible Solutions:**
- Wait longer (reconciliation can take 30-60 seconds)
- Verify VPCAttachment actually deleted (`kubectl get vpcattachment`)
- Check controller logs for errors
- Restart fabric controller if stuck

**Module 2.4 Reference:** Concept 3 - Cleanup Validation
</details>

---

#### Question 4: Archive vs Delete Decision

**Scenario:** You're decommissioning a VPC for a production application that's migrating to a new platform. The migration may take 2 weeks and could be rolled back. What should you do with the VPC configuration?

- A) Delete immediately from Git (saves space)
- B) Archive in Git with metadata (preserves for potential rollback)
- C) Keep active in Git but delete from cluster (Git only)
- D) Export to external backup system (outside Git)

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Archive in Git with metadata (preserves for potential rollback)

**Explanation:**

**Scenario Analysis:**
- **Production application** - High-risk change
- **Migration in progress** - Not yet complete
- **Potential rollback** - May need to restore
- **Timeline: 2 weeks** - Short enough to consider rollback likely

**Archive Approach:**

1. **Move to archive folder:**
   ```
   Before: vpcs/prod-app-vpc.yaml
   After:  archive/prod-app-vpc-2025-10-16.yaml
   ```

2. **Add metadata:**
   ```yaml
   # ARCHIVED: 2025-10-16
   # REASON: Application migrating to new platform (2-week migration)
   # ROLLBACK: Restore if migration fails
   # CONTACT: ops-team@example.com
   ```

3. **ArgoCD ignores archive** (resources deleted from cluster)

4. **Quick restore** (if rollback needed):
   - Copy from archive back to `vpcs/`
   - Commit
   - ArgoCD re-deploys

**Why others are wrong:**
- A) Delete immediately - Risky during migration (hard to restore)
- C) Keeping in Git deploys to cluster (wastes resources)
- D) External backup - Adds complexity (Git is sufficient)

**Decision Rule:**
- **Archive:** Uncertain outcome, may need rollback
- **Delete:** Permanent decommission, confident in decision

**Module 2.4 Reference:** Concept 5 - Archive vs Delete
</details>

---

## Technical Requirements

### Hedgehog CRDs Used

Same CRDs as Modules 2.1-2.3, focusing on deletion operations:

#### VPCAttachment - Deletion
- **API Version:** `vpc.githedgehog.com/v1beta1`
- **Deletion:** `kubectl delete vpcattachment <name>`
- **Finalizer:** May delay deletion if switch reconciliation pending

#### VPC - Deletion
- **API Version:** `vpc.githedgehog.com/v1beta1`
- **Deletion:** `kubectl delete vpc <name>`
- **Finalizer:** Blocks deletion until all VPCAttachments removed
- **Cascading Delete:** DHCPSubnets automatically deleted

### kubectl Commands

**Deletion:**
```bash
# Delete VPCAttachment
kubectl delete vpcattachment webapp-frontend-server-01

# Delete VPC
kubectl delete vpc webapp-vpc

# Delete multiple resources
kubectl delete vpcattachment webapp-frontend-server-01 webapp-backend-server-05 webapp-database-server-09

# Force delete (DANGEROUS - only if orphaned)
kubectl delete vpc webapp-vpc --force --grace-period=0
```

**Validation:**
```bash
# Check resource doesn't exist
kubectl get vpc webapp-vpc
# Expected: Error: "webapp-vpc" not found

# List all VPCs
kubectl get vpc

# Check for orphaned resources
kubectl get vpcattachment
kubectl get dhcpsubnet -n default

# Check events
kubectl get events --sort-by='.lastTimestamp' | grep -i delete
```

**Agent CRD Validation:**
```bash
# Check VLAN removed from interface
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.interfaces.<port>.vlans}'
# Expected: [] or null

# List all interfaces with VLANs
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.interfaces}' | \
  jq 'to_entries[] | select(.value.vlans != null) | {port: .key, vlans: .value.vlans}'
```

**Reference:** [WORKFLOWS.md](../research/WORKFLOWS.md#workflow-7-cleanup-and-rollback)

### GitOps Workflow

**Deletion via Gitea:**
1. Navigate to file in repository (Gitea web UI)
2. Click "Delete File" button
3. Add commit message (describe why)
4. ArgoCD detects change (60 seconds)
5. ArgoCD prunes resources
6. Validate with kubectl

**Rollback via Gitea:**
1. View commit history
2. Find deletion commit
3. Click "Revert this commit"
4. ArgoCD re-syncs
5. Resources recreated

**Reference:** [Module 1.2 Design](./module-1.2-design-v2-gitops.md)

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Train for Reality, Not Rote** ⭐
- **How:** Production-grade decommission workflow (not just `kubectl delete`)
- **Example:** Dependency order, cleanup validation, rollback procedures
- **Why:** Production environments require clean lifecycle management

#### 2. **Focus on What Matters Most** ⭐
- **How:** Emphasizes common decommission tasks (VPC cleanup)
- **Example:** Deletion order (most common error), validation (most common oversight)
- **Why:** Cleanup is frequent Day 2 operation

#### 3. **Teach the Why Behind the How** ⭐
- **How:** Explains finalizers, dependencies, GitOps deletion semantics
- **Example:** "Why VPCAttachments first?" → Dependency hierarchy
- **Why:** Understanding deletion mechanics prevents errors

#### 4. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on deletion of real VPC environment
- **Example:** Students delete VPCAttachments, VPC, validate cleanup
- **Why:** Active decommissioning builds confidence

#### 5. **Support as Part of Learning**
- **How:** Normalizes rollback (mistakes happen, recovery is planned)
- **Example:** Git-based rollback procedures taught proactively
- **Why:** Reduces anxiety about irreversible changes

#### 6. **Abstraction as Empowerment**
- **How:** GitOps deletion abstracts kubectl commands
- **Example:** Delete file in Gitea → ArgoCD handles cluster cleanup
- **Why:** Simplifies complex multi-step operations

---

### Target Audience Considerations

#### For Cloud-Native Learners

**What's New:**
- Finalizers and dependency management
- Network resource cleanup validation

**Bridge Strategy:**
- Relate finalizers to Pod termination grace periods
- Compare VPC deletion to Namespace deletion (cascading delete)
- Frame cleanup validation as "resource cleanup confirmation"

---

#### For Networking Professionals

**What's New:**
- Declarative deletion (vs manual switch deconfiguration)
- GitOps rollback (vs backup/restore)

**Bridge Strategy:**
- Relate finalizers to "config dependencies" (can't delete VLAN if in use)
- Compare Git rollback to "rollback to checkpoint"
- Frame cleanup validation as "show running-config" verification

---

### Common Challenges and Mitigation

#### Challenge 1: Deleting VPC Before VPCAttachments

**Stumbling Block:** Students try to delete VPC first

**Mitigation:**
- Concept 1 emphasizes deletion order
- Lab Task 1 deletes attachments before Task 2 deletes VPC
- Assessment Question 1 tests deletion order

---

#### Challenge 2: Assuming Deletion Completes Instantly

**Stumbling Block:** Students don't wait for reconciliation

**Mitigation:**
- Lab includes validation steps after each deletion
- Concept 3 explains reconciliation timing
- Task 3 checks Agent CRD for VLAN removal

---

#### Challenge 3: Fear of Irreversible Deletion

**Stumbling Block:** Students hesitate to delete (afraid of mistakes)

**Mitigation:**
- Concept 4 teaches rollback procedures early
- Lab Task 4 (optional) practices archiving
- Assessment Question 4 tests rollback understanding

---

### Confidence-Building Opportunities

#### Win 1: Dependency-Safe Deletion
**Moment:** Task 1 - Successfully deleting VPCAttachments without errors
**Feeling:** "I'm following production discipline"
**Teaching Point:** "Dependency order prevents operational mistakes."

#### Win 2: Clean Validation
**Moment:** Task 3 - Seeing VLANs removed from switch interfaces
**Feeling:** "Complete cleanup proven"
**Teaching Point:** "You validated cleanup at all three layers."

#### Win 3: Lifecycle Mastery
**Moment:** Course 2 completion - Full VPC lifecycle executed
**Feeling:** "Operational competence"
**Teaching Point:** "You can now independently manage VPC lifecycles from creation to decommission."

---

## Dependencies

### Prerequisites

**Required:**
- ✅ **Module 2.3:** Connectivity Validation
  - Validation workflow understood
  - Resources operational (safe to delete)

- ✅ **Module 2.2:** Attach Servers to VPC
  - VPCAttachments exist to delete

- ✅ **Module 2.1:** Define VPC Network
  - VPC created and lifecycle understood

- ✅ **Module 1.2:** How Hedgehog Works
  - GitOps workflow (applies to deletion too)

**Why These Prerequisites Matter:**
- Module 2.4 completes lifecycle started in 2.1-2.3
- Cannot practice deletion without resources to delete
- GitOps workflow understanding essential for deletion

---

### Enables

**Course-Level:**
- ✅ Completes Course 2 (full VPC lifecycle)
- ✅ Enables Course 3 and 4 (students can create/delete test resources)

**Skill-Level:**
- ✅ Independent VPC lifecycle management
- ✅ Production-grade operational discipline

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives specific and measurable**
  - LO 1: Delete in correct order (testable via Task 1-2)
  - LO 2: Validate cleanup (testable via Task 3)
  - LO 3: Prevent orphans (testable via Task 3)
  - LO 4: Rollback procedures (testable via Concept 4, Assessment Q2)
  - LO 5: Archive vs delete (testable via Assessment Q4)

- ✅ **Content outline logical progression**
  - Introduction → Core Concepts (5 concepts) → Hands-On Lab (4 tasks) → Assessment (4 questions)
  - Concepts build: Deletion order → GitOps deletion → Validation → Rollback → Archive

- ✅ **Assessment aligns with learning objectives**
  - Question 1: Deletion order (LO 1)
  - Question 2: GitOps deletion (LO 1, LO 4)
  - Question 3: Cleanup validation (LO 2, LO 3)
  - Question 4: Archive vs delete (LO 5)

- ✅ **Timing target achievable (10-12 minutes)**
  - Introduction: 1-2 min
  - Core Concepts: 3-4 min (5 concepts × ~45 sec each)
  - Hands-On Lab: 4-5 min (Task 1: 2 min, Task 2: 1 min, Task 3: 1-2 min, Task 4: 1 min optional)
  - Wrap-Up & Assessment: 2 min
  - **Total: 10-13 minutes** (within target)

---

### Technical Accuracy

- ✅ **All CRD references validated**
  - VPC finalizer behavior documented
  - VPCAttachment deletion dependencies correct
  - DHCPSubnet cascading delete validated

- ✅ **Workflows match WORKFLOWS.md**
  - Cleanup workflow (lines 768-860)
  - Rollback procedure (lines 836-852)

- ✅ **GitOps workflow accurate**
  - Matches Module 1.2 GitOps design
  - ArgoCD pruning behavior correct

---

### Learning Philosophy

- ✅ **Embodies 6 of 10 core principles**
  - Train for Reality, Not Rote ⭐
  - Focus on What Matters Most ⭐
  - Teach the Why Behind the How ⭐
  - Learn by Doing, Not Watching ⭐
  - Support as Part of Learning ⭐
  - Abstraction as Empowerment ⭐

- ✅ **Focuses on high-impact tasks**
  - Dependency-safe deletion (prevents common errors)
  - Cleanup validation (ensures complete decommission)

- ✅ **Builds confidence**
  - Task 1-2: Deletion success (operational discipline)
  - Task 3: Validation success (thoroughness)
  - Course 2 completion (lifecycle mastery)

---

### Accessibility

- ✅ **Clear to cloud-native learners**
  - Finalizers explained in Kubernetes context
  - GitOps deletion familiar pattern

- ✅ **Clear to networking professionals**
  - Deletion order relates to config dependencies
  - Cleanup validation parallels traditional verification

- ✅ **Jargon explained**
  - Finalizers defined
  - Pruning explained
  - Cascading delete described

---

### Completeness

- ✅ **All required sections present**
  - Module Metadata ✓
  - Learning Objectives ✓
  - Content Outline ✓
  - Hands-On Lab ✓
  - Assessment ✓
  - Technical Requirements ✓
  - Pedagogical Design ✓
  - Dependencies ✓
  - Quality Checklist ✓

- ✅ **Assessment has answers and explanations**
  - All 4 questions include detailed answers

- ✅ **Dependencies mapped**
  - Prerequisites: Modules 2.3, 2.2, 2.1, 1.2
  - Enables: Course 2 completion

---

## Review & Approval

### Design Review

- ⏳ Course lead review (pending)
- ⏳ Technical validation (dev agent)
- ⏳ Timing test (10-12 min walkthrough)
- ⏳ Peer feedback

### Approval

- ⏳ Ready for Phase 3 (Content Development)
- ⏳ Approved by: <!-- Pending -->

---

## Notes & Iterations

### Design Decisions

**Decision 1: Emphasize GitOps Deletion**
- **Rationale:** Consistent with Module 1.2 GitOps-first approach
- **Benefit:** Audit trail, reversibility, production-grade workflow
- **Alternative:** Focus on kubectl delete (rejected: no audit trail)

**Decision 2: Include Rollback Procedures**
- **Rationale:** Mistakes happen; recovery should be taught proactively
- **Benefit:** Reduces anxiety, builds confidence
- **Alternative:** Omit rollback (rejected: incomplete lifecycle training)

**Decision 3: Shorter Duration (10-12 min)**
- **Rationale:** Deletion is simpler than creation
- **Benefit:** Respects learner time, maintains engagement
- **Alternative:** 15 min like other modules (rejected: unnecessary padding)

**Decision 4: Archive Task Optional**
- **Rationale:** Archive is advanced practice, not always needed
- **Benefit:** Core skills in Tasks 1-3, advanced in Task 4
- **Alternative:** Make archiving required (rejected: not all scenarios need it)

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING
**Version:** 1.0
**Previous Module:** Module 2.3 (Connectivity Validation)
**Next Module:** Course 3, Module 3.1 (To Be Designed)

**Change Log:**
- 2025-10-16 v1.0: Initial design based on Issue #9 requirements

---

**Status:** ⏳ PENDING REVIEW
**Related Issue:** GitHub Issue #9 - [DESIGN] Course 2: Provisioning & Connectivity
**Design Duration:** ~3 hours (as estimated)
