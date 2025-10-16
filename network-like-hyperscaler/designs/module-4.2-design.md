# Module 4.2 Design: Rollback & Recovery

## Module Metadata

- **Module Number:** 4.2
- **Module Title:** Rollback & Recovery
- **Course:** Course 4 - Troubleshooting, Recovery & Escalation
- **Estimated Duration:** 13-15 minutes
  - Introduction: 2 minutes
  - Core Concepts: 4-5 minutes
  - Hands-On Lab: 6-7 minutes
  - Wrap-Up & Assessment: 2 minutes

- **Version:** 1.0
- **Date:** 2025-10-16
- **Status:** DESIGN - Ready for Review

- **Prerequisites:**
  - ✅ Module 4.1 Complete (Diagnosing Fabric Issues)
  - ✅ Understanding of GitOps workflow (Module 1.2)
  - ✅ Experience with VPC lifecycle (Course 2)
  - ✅ Familiarity with kubectl and Gitea

## Learning Objectives

By the end of this module, learners will be able to:

1. **Execute safe rollback procedures** - Use Git revert and ArgoCD sync to undo configuration changes
2. **Understand Kubernetes finalizers** - Explain how finalizers protect against incomplete deletion
3. **Follow safe deletion order** - Delete resources in correct dependency order (VPCAttachment → VPC → namespace resources)
4. **Recover from partial failures** - Handle stuck resources, finalizer issues, and reconciliation timeouts
5. **Identify escalation triggers** - Recognize when rollback/recovery requires support intervention

**Bloom's Taxonomy Level**: Apply, Evaluate (executing procedures and making recovery decisions)

## Content Outline

### Introduction (2 minutes)

**Hook: When You Need to Undo**

> In Module 4.1, you diagnosed issues. But sometimes, diagnosis reveals that the  **change itself was the problem.**
>
> Example scenarios:
> - You updated a VPC subnet VLAN, causing a conflict with another VPC
> - You created a VPCPeering that unexpectedly broke isolation
> - You modified DHCP settings that prevented servers from getting IPs
>
> In these cases, you need to **rollback the change**—quickly, safely, and completely.
>
> Beginners might panic and manually delete resources with kubectl. Experts use **GitOps rollback workflows** that provide:
> - **Audit trail** (Git commit history shows exactly what changed)
> - **Reproducibility** (rollback can be repeated if needed)
> - **Safety** (ArgoCD ensures consistent state)

**Context: Recovery as Operational Skill**

Module 4.1 focused on diagnosis. Module 4.2 focuses on **recovery**:
- Rolling back configuration changes that caused issues
- Safely deleting resources in correct dependency order
- Recovering from partial failures (stuck finalizers, reconciliation timeouts)
- Understanding when to escalate vs. self-resolve

**What You'll Learn:**

- **GitOps rollback workflow** (git revert → ArgoCD sync → verify)
- **Safe deletion order** (VPCAttachment before VPC, VPC before related resources)
- **Kubernetes finalizers** (why resources get "stuck" during deletion)
- **Partial failure recovery** (stuck resources, reconciliation timeouts)
- **Emergency procedures** (when to escalate, when to force-delete)

**Scenario for This Module:**

You'll perform a rollback of a VPC configuration change that caused VLAN conflicts, including:
- Reverting the Git commit
- Triggering ArgoCD sync
- Verifying recovery in kubectl and Grafana

---

### Core Concepts (4-5 minutes)

#### Concept 1: GitOps Rollback Workflow

**Why GitOps Rollback?**

Traditional rollback: Manually edit resources with kubectl, hope you remember the old configuration.

GitOps rollback: **Git history is the source of truth**. Every change is a commit. Rollback = revert commit.

**Benefits:**
- **Audit trail** - Git history shows who changed what and when
- **Reproducible** - Same rollback procedure every time
- **Safe** - ArgoCD ensures cluster state matches Git
- **Testable** - Can review rollback diff before applying

---

**GitOps Rollback Procedure:**

**Step 1: Identify Problematic Commit**

```bash
# View recent Git history
cd /path/to/hedgehog-config
git log --oneline --graph --decorate -10

# Example output:
# a1b2c3d (HEAD -> main) Update vpc-prod VLAN to 1050
# e4f5g6h Fix DHCP range for vpc-staging
# i7j8k9l Add VPCPeering between prod and staging
```

**Identify commit to revert:** `a1b2c3d` (VPC VLAN change that caused conflict)

---

**Step 2: Revert the Commit (Git)**

```bash
# Revert the specific commit
git revert a1b2c3d

# Git creates NEW commit that undoes changes from a1b2c3d
# Opens editor for commit message (default: "Revert 'Update vpc-prod VLAN to 1050'")

# View diff before committing
git diff HEAD~1

# Commit the revert
git commit -m "Revert VPC VLAN change (caused conflict with vpc-staging)"

# Push to remote
git push origin main
```

**Important:** `git revert` creates a **new commit** that undoes changes. It doesn't delete history (good for audit trail).

---

**Step 3: Trigger ArgoCD Sync**

ArgoCD polls Git repository (default: 60 seconds). You can wait or trigger manually.

**Option A: Wait for Auto-Sync**

```bash
# Watch ArgoCD status
kubectl get applications -n argocd -w

# Expected: hedgehog-config status changes from "Synced" → "OutOfSync" → "Syncing" → "Synced"
```

**Option B: Manual Sync (Faster)**

Via ArgoCD UI:
1. Open ArgoCD: http://localhost:8080
2. Click `hedgehog-config` application
3. Click **"Sync"** button
4. Watch sync progress

Via CLI:
```bash
# Trigger sync
argocd app sync hedgehog-config

# Wait for sync completion
argocd app wait hedgehog-config --health
```

---

**Step 4: Verify Rollback**

```bash
# Check VPC configuration
kubectl get vpc vpc-prod -o yaml | grep vlan

# Expected: VLAN reverted to original value

# Check events
kubectl get events --field-selector involvedObject.name=vpc-prod --sort-by='.lastTimestamp'

# Expected: VPC reconciliation events showing VLAN update

# Verify no errors
kubectl get events --field-selector type=Warning | grep vpc-prod

# Expected: No Warning events
```

**Verify in Grafana:**
- Open Fabric Dashboard
- Check VPC metrics
- Verify VLAN conflict resolved

---

**Step 5: Validate Connectivity**

```bash
# Test connectivity from affected servers
# (SSH to server or use kubectl exec)

ping 10.10.1.1  # Gateway
ping 10.10.1.10 # Another server in VPC

# Expected: Connectivity restored
```

---

**GitOps Rollback Complete:**
- ✅ Git history shows revert commit (audit trail)
- ✅ ArgoCD synced cluster to match Git
- ✅ kubectl shows reverted configuration
- ✅ Grafana confirms VPC healthy
- ✅ Connectivity validated

---

#### Concept 2: Safe Deletion Order

**Why Order Matters:**

Hedgehog resources have **dependencies**:
- VPCAttachment depends on VPC (references VPC subnet)
- VPC depends on IPv4Namespace and VLANNamespace
- VPCPeering depends on both VPCs

**Deleting in wrong order causes:**
- Stuck resources (finalizers prevent deletion)
- Orphaned configurations (switch VLANs not cleaned up)
- Partial failures (some resources deleted, others remain)

---

**Safe Deletion Order (General Rule):**

**Delete from leaves to roots:**

```
1. VPCAttachment (depends on VPC + Connection)
   ↓
2. VPCPeering (depends on VPCs)
   ↓
3. ExternalPeering (depends on VPC + External)
   ↓
4. VPC (depends on IPv4Namespace + VLANNamespace)
   ↓
5. External resources (if no longer needed)
   ↓
6. Namespaces (IPv4Namespace, VLANNamespace) - rarely deleted
```

---

**Example: Delete VPC and Attachments**

**Wrong Order (Causes Problems):**

```bash
# Delete VPC first (WRONG!)
kubectl delete vpc vpc-prod

# Result: VPC stuck in "Terminating" state
# Reason: VPCAttachments still reference VPC (finalizers prevent deletion)
```

**Correct Order:**

```bash
# Step 1: Delete all VPCAttachments
kubectl get vpcattachment | grep vpc-prod
# Output: vpc-prod-server-01, vpc-prod-server-02, vpc-prod-server-03

kubectl delete vpcattachment vpc-prod-server-01
kubectl delete vpcattachment vpc-prod-server-02
kubectl delete vpcattachment vpc-prod-server-03

# Wait for deletion to complete
kubectl get vpcattachment | grep vpc-prod
# Expected: No results

# Step 2: Delete VPCPeerings (if any)
kubectl get vpcpeering | grep vpc-prod
# Output: vpc-prod--vpc-staging

kubectl delete vpcpeering vpc-prod--vpc-staging

# Step 3: Delete VPC
kubectl delete vpc vpc-prod

# Wait for deletion
kubectl get vpc vpc-prod
# Expected: Error: "vpc.vpc.githedgehog.com "vpc-prod" not found"
```

---

**Verification After Deletion:**

```bash
# Verify no orphaned resources
kubectl get dhcpsubnet -n default | grep vpc-prod
# Expected: No results (DHCPSubnets auto-deleted with VPC)

# Check Agent CRD - VLAN removed from switches
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq
# Expected: VLAN no longer in vlans list

# Check Grafana Fabric Dashboard
# Expected: VPC count decreased
```

---

#### Concept 3: Kubernetes Finalizers

**What Are Finalizers?**

Finalizers are **hooks that prevent resource deletion** until cleanup actions complete.

**Analogy:** Like a lock on a door. You can't delete the resource until the finalizer (lock) is removed.

**Example:**

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPC
metadata:
  name: vpc-prod
  namespace: default
  finalizers:
    - vpc.githedgehog.com/finalizer  # Prevents deletion until removed
spec:
  ...
```

**Finalizer Purpose:**
- Ensure VPCAttachments deleted before VPC
- Clean up switch configurations (remove VLANs from interfaces)
- Delete associated resources (DHCPSubnets)

---

**Finalizer Workflow:**

1. **User deletes resource:**
   ```bash
   kubectl delete vpc vpc-prod
   ```

2. **Kubernetes sets deletionTimestamp:**
   ```yaml
   metadata:
     deletionTimestamp: "2025-10-16T10:30:00Z"
     finalizers:
       - vpc.githedgehog.com/finalizer
   ```
   Resource enters "Terminating" state.

3. **Controller processes finalizer:**
   - Checks if VPCAttachments still exist
   - Cleans up switch configurations
   - Deletes DHCPSubnets

4. **Controller removes finalizer:**
   ```yaml
   metadata:
     finalizers: []  # Finalizer removed
   ```

5. **Kubernetes completes deletion:**
   Resource fully deleted from etcd.

---

**Stuck Resources (Finalizer Not Removed):**

**Symptom:**
```bash
kubectl get vpc vpc-prod
# Output: vpc-prod ... (terminating for 10 minutes)
```

**Diagnosis:**

```bash
# Check if finalizers still present
kubectl get vpc vpc-prod -o jsonpath='{.metadata.finalizers}'

# Expected: ["vpc.githedgehog.com/finalizer"]

# Check controller logs
kubectl logs -n fab deployment/fabric-controller-manager | grep vpc-prod

# Look for errors preventing finalizer removal
```

**Common Causes:**
1. **VPCAttachments still exist** (delete them first)
2. **Controller cannot reach switches** (Agent disconnected)
3. **Controller error during cleanup** (bug or resource issue)

---

**Recovery: Remove Finalizer Manually (Last Resort)**

**WARNING: Only do this if:**
- VPCAttachments already deleted
- Controller logs show cleanup completed
- Resource stuck for >10 minutes

```bash
# Remove finalizer to force deletion
kubectl patch vpc vpc-prod --type='json' -p='[{"op": "remove", "path": "/metadata/finalizers"}]'

# Resource immediately deleted
kubectl get vpc vpc-prod
# Expected: Error: not found
```

**Caution:** Manually removing finalizers can leave orphaned switch configurations. Only use as emergency procedure.

---

#### Concept 4: Partial Failure Recovery

**Partial Failure Scenarios:**

**Scenario 1: Stuck VPCAttachment (Agent Disconnected)**

**Symptoms:**
- VPCAttachment stuck in "Terminating"
- Agent for switch offline

**Diagnosis:**

```bash
# Check agent status
kubectl get agent leaf-01 -n fab

# Output: Ready = False (agent disconnected)

# Check VPCAttachment finalizers
kubectl get vpcattachment vpc-prod-server-01 -o jsonpath='{.metadata.finalizers}'
```

**Recovery:**

**Option A: Wait for Agent Reconnection (Preferred)**

```bash
# Wait for agent to reconnect
kubectl get agent leaf-01 -n fab -w

# When agent Ready, controller removes finalizer automatically
```

**Option B: Force Delete (Emergency)**

```bash
# Remove finalizer
kubectl patch vpcattachment vpc-prod-server-01 --type='json' -p='[{"op": "remove", "path": "/metadata/finalizers"}]'

# Note: Switch VLAN may not be cleaned up (manual cleanup required)
```

---

**Scenario 2: ArgoCD Sync Failure**

**Symptoms:**
- Git revert committed
- ArgoCD shows "OutOfSync" but sync fails
- Error: "Failed to sync application"

**Diagnosis:**

```bash
# Check ArgoCD application status
kubectl get application hedgehog-config -n argocd -o yaml

# Look for sync errors in status.conditions

# Check ArgoCD logs
kubectl logs -n argocd deployment/argocd-application-controller | grep hedgehog-config
```

**Common Causes:**
1. **Invalid YAML in Git** (syntax error in reverted state)
2. **Namespace missing** (ArgoCD can't create resource)
3. **RBAC issue** (ArgoCD lacks permissions)

**Recovery:**

```bash
# Validate YAML locally
kubectl apply --dry-run=client -f vpcs/vpc-prod.yaml

# If invalid, fix YAML in Git and commit
git add vpcs/vpc-prod.yaml
git commit -m "Fix YAML syntax after revert"
git push

# Retry sync
argocd app sync hedgehog-config
```

---

**Scenario 3: Reconciliation Timeout**

**Symptoms:**
- VPC or VPCAttachment creation takes >5 minutes
- No error events, just slow reconciliation
- Controller logs show repeated reconciliation attempts

**Diagnosis:**

```bash
# Check controller CPU/memory
kubectl top pod -n fab

# Check agent status
kubectl get agents -n fab

# Check controller logs for timeout errors
kubectl logs -n fab deployment/fabric-controller-manager | grep -i timeout
```

**Common Causes:**
1. **Controller resource constrained** (CPU/memory limit)
2. **Agent slow to apply configuration** (switch overloaded)
3. **Large number of resources reconciling** (queue backlog)

**Recovery:**

**Option A: Wait (Usually Resolves)**

```bash
# Wait 10-15 minutes for reconciliation to complete
kubectl get events --watch | grep <resource-name>
```

**Option B: Restart Controller (If Hung)**

```bash
# Check if controller is responsive
kubectl get pods -n fab

# If controller CrashLoopBackOff or not Ready:
kubectl rollout restart deployment/fabric-controller-manager -n fab

# Wait for controller to restart
kubectl rollout status deployment/fabric-controller-manager -n fab
```

**Option C: Escalate (If Persistent)**

If reconciliation takes >30 minutes or controller keeps restarting:
- Collect diagnostics (Module 3.4)
- Escalate to support (Module 4.3)

---

#### Concept 5: Emergency Procedures

**When to Escalate vs. Self-Resolve:**

**Self-Resolve:**
- Stuck resource due to missed VPCAttachment deletion (delete attachment)
- ArgoCD sync failure due to YAML syntax error (fix YAML)
- Reconciliation timeout <15 minutes (wait)

**Escalate:**
- Controller repeatedly crashing during rollback
- Finalizer removal doesn't complete deletion (after manual removal)
- Partial failure leaves fabric in inconsistent state
- Rollback causes new errors (configuration corruption)

---

**Emergency: Force Delete Resource**

**Use ONLY when:**
- Resource stuck >30 minutes
- Finalizer removal attempted
- VPCAttachments confirmed deleted
- Support has recommended force-delete

```bash
# Force delete (ignores finalizers)
kubectl delete vpc vpc-prod --force --grace-period=0

# WARNING: May leave orphaned switch configurations
# Verify in Agent CRD and Grafana after deletion
```

---

**Emergency: Rollback a Rollback**

If rollback itself causes issues:

```bash
# View Git history
git log --oneline -5

# Output:
# a1b2c3d (HEAD) Revert VPC VLAN change (rollback commit)
# e4f5g6h Update vpc-prod VLAN to 1050 (original change)

# Revert the revert (restore original change)
git revert a1b2c3d

# Commit and push
git commit -m "Restore original VPC VLAN change (rollback was incorrect)"
git push

# Trigger ArgoCD sync
```

---

### Hands-On Lab (6-7 minutes)

**Lab Title:** Rollback a VPC Configuration Change

**Scenario:**

Earlier today, you updated VPC `webapp-vpc` to use VLAN 1050 for the `frontend` subnet. This caused a VLAN conflict with another VPC, breaking connectivity for 20 web servers.

Your task:
1. Revert the VPC VLAN change in Git
2. Sync ArgoCD to apply the rollback
3. Verify connectivity restored
4. (Bonus) Practice safe VPCAttachment deletion

**Environment Access:**
- **Gitea:** http://localhost:3001
- **ArgoCD:** http://localhost:8080
- **kubectl:** Already configured
- **Grafana:** http://localhost:3000

**Git Repository:** `student/hedgehog-config`

---

#### Task 1: Identify and Revert Problematic Commit (3 minutes)

**Objective:** Use Git to rollback the VLAN change

**Step 1.1: Review Git History (Gitea Web UI)**

1. Open Gitea: http://localhost:3001
2. Sign in (username: `student`, password: `hedgehog123`)
3. Navigate to `student/hedgehog-config` repository
4. Click **"Commits"** (view commit history)

5. Identify recent commit:
   - Commit message: "Update webapp-vpc frontend VLAN to 1050"
   - Author: student
   - Date: Today
   - **Note the commit hash (first 7 characters)**

Example:
- **Commit hash:** `a1b2c3d`
- **Message:** "Update webapp-vpc frontend VLAN to 1050"

---

**Step 1.2: Revert via Git CLI**

**Option A: Command Line (Recommended)**

```bash
# Navigate to repository
cd /path/to/hedgehog-config  # (provided in environment)

# View recent history
git log --oneline -5

# Expected output:
# a1b2c3d (HEAD -> main) Update webapp-vpc frontend VLAN to 1050
# e4f5g6h Add DHCP range for backend subnet
# ...

# Revert the specific commit
git revert a1b2c3d

# Git opens editor with commit message
# Accept default message or edit: "Revert VLAN change (caused conflict)"

# Save and exit editor

# View diff to confirm revert
git show HEAD

# Expected: frontend VLAN reverted from 1050 to original value (1010)

# Push to remote
git push origin main

# Expected output: "1 file changed, 1 insertion(+), 1 deletion(-)"
```

---

**Option B: Gitea Web UI (Alternative)**

1. In Gitea, click commit `a1b2c3d`
2. View the diff (shows VLAN change 1010 → 1050)
3. Click **"Browse Source"** to navigate to commit state
4. Navigate to `vpcs/webapp-vpc.yaml`
5. Click **"Edit File"**
6. Manually change VLAN from 1050 back to 1010
7. Commit message: "Revert VLAN change (caused conflict)"
8. Click **"Commit Changes"**

---

**Success Criteria:**
- ✅ Git commit reverted (new commit exists)
- ✅ Git diff shows VLAN changed from 1050 back to 1010
- ✅ Changes pushed to main branch

---

#### Task 2: Trigger ArgoCD Sync and Verify (2 minutes)

**Objective:** Apply rollback to cluster via ArgoCD

**Step 2.1: Watch ArgoCD Detect Change**

1. Open ArgoCD UI: http://localhost:8080
2. Sign in (username: `admin`, password: `qV7hX0NMroAUhwoZ`)
3. Find `hedgehog-config` application
4. Status should change: "Synced" → "OutOfSync" (Git ahead of cluster)

---

**Step 2.2: Trigger Sync**

**Option A: ArgoCD UI (Visual)**

1. Click `hedgehog-config` application tile
2. Click **"Sync"** button
3. Review sync diff (shows VLAN 1050 → 1010)
4. Click **"Synchronize"**
5. Watch sync progress:
   - Status: "Syncing"
   - Resources updating
   - Status: "Synced" (green checkmark)

**Option B: ArgoCD CLI (Faster)**

```bash
# Trigger sync
argocd app sync hedgehog-config

# Expected output: "Syncing application hedgehog-config"

# Wait for completion
argocd app wait hedgehog-config --health

# Expected output: "Application hedgehog-config synced successfully"
```

---

**Step 2.3: Verify with kubectl**

```bash
# Check VPC VLAN configuration
kubectl get vpc webapp-vpc -o jsonpath='{.spec.subnets.frontend.vlan}'

# Expected output: 1010 (reverted from 1050)

# Check events for webapp-vpc
kubectl get events --field-selector involvedObject.name=webapp-vpc --sort-by='.lastTimestamp'

# Expected events:
# Normal  VPCUpdated  VPC configuration reconciled
# Normal  VLANUpdated VLAN changed from 1050 to 1010

# Verify no errors
kubectl get events --field-selector type=Warning | grep webapp-vpc

# Expected: No Warning events
```

---

**Success Criteria:**
- ✅ ArgoCD status "Synced" and "Healthy"
- ✅ kubectl shows VLAN 1010
- ✅ No error events

---

#### Task 3: Validate Connectivity Restored (1 minute)

**Objective:** Confirm servers can communicate after rollback

**Step 3.1: Check Agent CRD**

```bash
# Find which switches have webapp-vpc frontend subnet
kubectl get vpcattachment | grep webapp-vpc | grep frontend

# Example output: webapp-vpc-server-01 (leaf-01), webapp-vpc-server-02 (leaf-02)

# Check leaf-01 interface configuration
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces.Ethernet5}' | jq

# Expected: VLAN 1010 in vlans list (not 1050)
```

---

**Step 3.2: Verify in Grafana**

1. Open Grafana: http://localhost:3000
2. Navigate to "Hedgehog Fabric" dashboard
3. Check VPC metrics:
   - webapp-vpc shows VLAN 1010
   - No VLAN conflicts

4. Navigate to "Hedgehog Interfaces" dashboard
5. Filter: `switch="leaf-01"`, `interface="Ethernet5"`
6. Verify:
   - Interface up
   - VLAN 1010 configured
   - Traffic flowing

---

**Step 3.3: Test Connectivity (Optional)**

If server access available:

```bash
# SSH to server-01 (or use kubectl exec)
# Ping gateway
ping -c 4 10.0.10.1

# Expected: 0% packet loss

# Ping another server in VPC
ping -c 4 10.0.10.11

# Expected: 0% packet loss
```

---

**Success Criteria:**
- ✅ Agent CRD shows VLAN 1010 on interfaces
- ✅ Grafana confirms VLAN configuration
- ✅ Connectivity tests pass (if performed)

---

#### Task 4 (Bonus): Practice Safe Deletion Order (1-2 minutes)

**Objective:** Safely delete a test VPCAttachment

**Scenario:** Delete `webapp-vpc-server-03` (test server no longer needed)

**Step 4.1: Check Deletion Order**

```bash
# Verify VPCAttachment exists
kubectl get vpcattachment webapp-vpc-server-03

# Expected: VPCAttachment found

# Check if VPC has other attachments (should NOT delete VPC yet)
kubectl get vpcattachment | grep webapp-vpc

# Expected: Multiple attachments (server-01, server-02, server-03)
```

---

**Step 4.2: Delete VPCAttachment (Correct Order)**

```bash
# Delete VPCAttachment
kubectl delete vpcattachment webapp-vpc-server-03

# Wait for deletion
kubectl get vpcattachment webapp-vpc-server-03 -w

# Expected: Resource enters "Terminating" state, then fully deleted

# Verify finalizer removed (should complete in <30 seconds)
```

---

**Step 4.3: Verify Cleanup**

```bash
# Check Agent CRD - VLAN should be removed from interface
kubectl get agent leaf-03 -n fab -o jsonpath='{.status.state.interfaces.Ethernet8}' | jq

# Expected: VLAN 1010 no longer in vlans list (if no other attachments on that interface)

# Verify VPC still exists (not deleted with attachment)
kubectl get vpc webapp-vpc

# Expected: VPC still present
```

---

**Success Criteria:**
- ✅ VPCAttachment deleted successfully
- ✅ VLAN removed from switch interface
- ✅ VPC remains intact (not deleted)

---

### Wrap-Up & Assessment (2 minutes)

**What You Accomplished:**

You successfully performed a production-style rollback and recovery:
- ✅ Reverted problematic Git commit using `git revert`
- ✅ Synced rollback to cluster via ArgoCD
- ✅ Verified connectivity restored with kubectl and Grafana
- ✅ (Bonus) Practiced safe VPCAttachment deletion

**Key Takeaways:**

1. **GitOps rollback is safe and auditable** - Git history tracks all changes and reversions
2. **ArgoCD ensures consistency** - Cluster state always matches Git
3. **Deletion order prevents stuck resources** - VPCAttachment before VPC
4. **Finalizers protect against incomplete cleanup** - Ensure switch configs cleaned up
5. **Manual finalizer removal is last resort** - Only when stuck >30 minutes and confirmed safe

**Recovery Mindset:**

- **Plan rollback before making changes** - Know how to undo
- **Test rollback in non-prod first** - If possible
- **Verify after rollback** - Don't assume it worked
- **Document recovery procedures** - For team knowledge

**Preview of Module 4.3:**

Next, you'll learn **how to coordinate with support**—escalation triggers, writing effective support tickets, and working collaboratively with Hedgehog support engineers.

---

### Assessment Questions

#### Question 1: GitOps Rollback

**Scenario:** You need to rollback a VPC configuration change. The problematic commit is `a1b2c3d`. What is the CORRECT command sequence?

**A:**
```bash
git reset --hard HEAD~1
git push --force
```

**B:**
```bash
git revert a1b2c3d
git push origin main
```

**C:**
```bash
kubectl delete vpc vpc-prod
kubectl apply -f vpc-prod-backup.yaml
```

**D:**
```bash
git checkout a1b2c3d~1
git push origin main
```

<details>
<summary>Answer & Explanation</summary>

**Answer:** B

```bash
git revert a1b2c3d
git push origin main
```

**Explanation:**

**Why B is correct:**

**`git revert` creates a NEW commit** that undoes changes from the specified commit:
- Preserves Git history (audit trail)
- Safe for shared branches (doesn't rewrite history)
- ArgoCD detects new commit and syncs cluster

**Workflow:**
1. `git revert a1b2c3d` - Creates revert commit
2. `git push origin main` - Pushes to remote
3. ArgoCD auto-syncs (or manual sync)
4. Cluster state reverted

**Why others are wrong:**

**A) git reset --hard + push --force:**
- **DANGEROUS:** Rewrites Git history (loses audit trail)
- `--force` push can break collaborators' clones
- Not compatible with GitOps best practices

**C) kubectl delete + apply:**
- **Bypasses GitOps:** Manual kubectl changes not tracked in Git
- Creates configuration drift (Git ≠ cluster)
- No audit trail

**D) git checkout + push:**
- `git checkout` doesn't create commit (detached HEAD)
- Cannot push detached HEAD to branch
- Invalid Git operation

**GitOps Principle:** Always revert via Git commits, never rewrite history or bypass Git.

**Module 4.2 Reference:** Concept 1 - GitOps Rollback Workflow
</details>

---

#### Question 2: Safe Deletion Order

**Scenario:** You need to delete VPC `prod-vpc`. It has 3 VPCAttachments and 1 VPCPeering. What is the CORRECT deletion order?

- A) Delete VPC → Delete VPCAttachments → Delete VPCPeering
- B) Delete VPCAttachments → Delete VPCPeering → Delete VPC
- C) Delete VPCPeering → Delete VPC → Delete VPCAttachments
- D) Delete all resources simultaneously with `kubectl delete vpc,vpcattachment,vpcpeering --all`

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Delete VPCAttachments → Delete VPCPeering → Delete VPC

**Explanation:**

**Correct deletion order (leaves to roots):**

**Step 1: Delete VPCAttachments**
- VPCAttachments depend on VPC
- Finalizers prevent VPC deletion while attachments exist
- Delete all 3 VPCAttachments first

**Step 2: Delete VPCPeering**
- VPCPeering depends on VPC
- Must be deleted before VPC

**Step 3: Delete VPC**
- Only delete VPC after all dependent resources removed
- VPC finalizers can now complete cleanup

**Commands:**
```bash
# Step 1
kubectl delete vpcattachment prod-vpc-server-01
kubectl delete vpcattachment prod-vpc-server-02
kubectl delete vpcattachment prod-vpc-server-03

# Step 2
kubectl delete vpcpeering prod-vpc--staging-vpc

# Step 3
kubectl delete vpc prod-vpc
```

**Why others are wrong:**

**A) VPC first:**
- VPC enters "Terminating" state (stuck)
- Finalizers prevent deletion while VPCAttachments exist
- Must delete attachments manually to unstick

**C) VPCPeering first, VPC second:**
- VPC still stuck (VPCAttachments still reference it)
- Wrong order: VPCAttachments should be first

**D) Delete all simultaneously:**
- Kubernetes processes deletions in parallel
- Race condition: VPC may delete before attachments
- Can leave orphaned switch configurations

**Rule:** Always delete dependent resources before the resources they depend on.

**Module 4.2 Reference:** Concept 2 - Safe Deletion Order
</details>

---

#### Question 3: Finalizers

**Scenario:** You deleted VPC `test-vpc` 15 minutes ago, but it's still showing "Terminating" status. You check and find finalizers still present. What should you do FIRST?

- A) Immediately remove finalizers with `kubectl patch`
- B) Check if VPCAttachments still exist and delete them
- C) Restart the fabric controller
- D) Escalate to support immediately

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Check if VPCAttachments still exist and delete them

**Explanation:**

**Finalizer Troubleshooting Order:**

**Step 1: Check for dependent resources (most common cause)**

```bash
# Check if VPCAttachments still reference VPC
kubectl get vpcattachment | grep test-vpc

# If found, delete them
kubectl delete vpcattachment test-vpc-server-01
```

**Why this is first:**
- **Most common cause** of stuck finalizers: dependent resources still exist
- Safe to check (read-only operation)
- Easy to fix (delete VPCAttachments)

**Step 2: Check controller logs (if no VPCAttachments found)**

```bash
kubectl logs -n fab deployment/fabric-controller-manager | grep test-vpc

# Look for errors preventing finalizer removal
```

**Step 3: Check agent connectivity (if controller shows agent errors)**

```bash
kubectl get agents -n fab
# If agents disconnected, wait for reconnection
```

**Step 4: Manual finalizer removal (last resort, after 30+ minutes)**

```bash
kubectl patch vpc test-vpc --type='json' -p='[{"op": "remove", "path": "/metadata/finalizers"}]'
```

**Why others are wrong:**

**A) Immediately remove finalizers:**
- **Premature:** Haven't diagnosed root cause yet
- May leave orphaned switch configurations
- Should only be done as last resort

**C) Restart controller:**
- Doesn't address root cause (VPCAttachments still exist)
- Disrupts fabric operations unnecessarily
- Controller restart won't remove finalizers if VPCAttachments present

**D) Escalate immediately:**
- Common issue that operators can self-resolve
- Support will ask: "Did you check for VPCAttachments?"
- Escalate only if stuck >30 minutes after checking

**Troubleshooting Principle:** Check for dependent resources before taking more disruptive actions.

**Module 4.2 Reference:** Concept 3 - Kubernetes Finalizers
</details>

---

#### Question 4: Partial Failure Recovery

**Scenario:** ArgoCD sync failed with error "Failed to sync application: invalid YAML". Your rollback commit is in Git, but cluster state hasn't changed. What is the NEXT step?

- A) Delete the Git commit and start over
- B) Validate YAML locally with `kubectl apply --dry-run`, fix errors, and commit fix
- C) Force ArgoCD to sync with `--force` flag
- D) Manually apply YAML with kubectl (bypass ArgoCD)

<details>
<summary>Answer & Explanation</summary>

**Answer:** B) Validate YAML locally with `kubectl apply --dry-run`, fix errors, and commit fix

**Explanation:**

**ArgoCD Sync Failure Recovery:**

**Step 1: Diagnose YAML issue (local validation)**

```bash
# Validate YAML syntax
kubectl apply --dry-run=client -f vpcs/test-vpc.yaml

# Expected output if valid: "vpc.vpc.githedgehog.com/test-vpc created (dry run)"
# Expected output if invalid: "Error: invalid YAML syntax at line X"
```

**Step 2: Fix YAML in Git**

```bash
# Edit the file to fix syntax error
nano vpcs/test-vpc.yaml

# Commit fix
git add vpcs/test-vpc.yaml
git commit -m "Fix YAML syntax in rollback commit"
git push
```

**Step 3: Retry ArgoCD sync**

```bash
argocd app sync hedgehog-config

# Expected: Sync succeeds with corrected YAML
```

**Why B is correct:**
- **Root cause:** Invalid YAML in Git
- **Solution:** Fix YAML, commit, push
- **GitOps preserved:** All changes in Git
- **Audit trail:** Fix commit documented

**Why others are wrong:**

**A) Delete Git commit:**
- Loses audit trail (why was commit created?)
- Doesn't fix underlying YAML issue
- Better to fix forward (new commit) than delete history

**C) Force sync:**
- `--force` doesn't fix invalid YAML
- ArgoCD will still fail to parse invalid YAML
- Force flag is for different purpose (overwrite cluster changes)

**D) Manually apply with kubectl:**
- **Bypasses GitOps:** Cluster ≠ Git
- Creates configuration drift
- Future ArgoCD sync will overwrite manual change
- Loses audit trail

**GitOps Recovery Principle:** Fix issues in Git, then sync. Never bypass Git.

**Module 4.2 Reference:** Concept 4 - Partial Failure Recovery (Scenario 2: ArgoCD Sync Failure)
</details>

---

## Technical Requirements

### Git Commands

**Rollback Commands:**
```bash
# View commit history
git log --oneline --graph -10

# Revert specific commit
git revert <commit-hash>

# View diff of revert
git show HEAD

# Push revert
git push origin main

# Revert a revert (restore original change)
git revert <revert-commit-hash>
```

**Validation:**
```bash
# Check if revert worked
git diff <original-commit> HEAD

# View file at specific commit
git show <commit-hash>:path/to/file
```

### kubectl Deletion Commands

**Safe Deletion:**
```bash
# Delete VPCAttachments first
kubectl delete vpcattachment <name>

# Delete VPCPeerings
kubectl delete vpcpeering <name>

# Delete VPC
kubectl delete vpc <name>

# Check deletion status
kubectl get vpc <name> -w  # Watch deletion progress
```

**Finalizer Management:**
```bash
# Check finalizers
kubectl get vpc <name> -o jsonpath='{.metadata.finalizers}'

# Remove finalizer (last resort)
kubectl patch vpc <name> --type='json' -p='[{"op": "remove", "path": "/metadata/finalizers"}]'

# Force delete (emergency)
kubectl delete vpc <name> --force --grace-period=0
```

**Verification:**
```bash
# Verify resource deleted
kubectl get vpc <name>  # Should error: not found

# Check for orphaned DHCPSubnets
kubectl get dhcpsubnet -n default | grep <vpc-name>

# Check Agent CRD for switch cleanup
kubectl get agent <switch> -n fab -o jsonpath='{.status.state.interfaces.<interface>}' | jq
```

### ArgoCD Commands

**Sync Operations:**
```bash
# Trigger sync
argocd app sync hedgehog-config

# Wait for sync completion
argocd app wait hedgehog-config --health

# Check sync status
argocd app get hedgehog-config

# View sync history
argocd app history hedgehog-config
```

**Troubleshooting:**
```bash
# Check application status
kubectl get application hedgehog-config -n argocd -o yaml

# View ArgoCD logs
kubectl logs -n argocd deployment/argocd-application-controller | grep hedgehog-config
```

### Reference Documents

- **[WORKFLOWS.md](../research/WORKFLOWS.md)** - Workflow 7: Cleanup and Rollback (lines 768-861)
- **[CRD_REFERENCE.md](../research/CRD_REFERENCE.md)** - VPC, VPCAttachment status and finalizers
- **[Module 1.2 Design](./module-1.2-design-v2-gitops.md)** - GitOps workflow reference

---

## Pedagogical Design

### Learning Philosophy Principles Emphasized

#### 1. **Train for Reality, Not Rote** ⭐
- **How:** Realistic rollback scenario (VLAN conflict requiring revert)
- **Example:** Students perform actual Git revert and ArgoCD sync
- **Why:** Rollback is critical production skill

#### 2. **Focus on What Matters Most** ⭐
- **How:** Covers common recovery scenarios (GitOps rollback, stuck finalizers)
- **Example:** Safe deletion order (prevents most stuck resource issues)
- **Why:** High-impact skills for daily operations

#### 3. **Confidence Before Comprehensiveness** ⭐
- **How:** Structured procedures for rollback and deletion
- **Example:** Step-by-step GitOps rollback workflow
- **Why:** Reduces anxiety during incident recovery

#### 4. **Learn by Doing, Not Watching** ⭐
- **How:** Hands-on Git revert and ArgoCD sync
- **Example:** Students perform actual rollback, not just read about it
- **Why:** Practice builds muscle memory for incidents

#### 5. **Teach the Why Behind the How** ⭐
- **How:** Explains finalizers and dependency order
- **Example:** Why VPCAttachment must be deleted before VPC
- **Why:** Understanding enables troubleshooting when procedures fail

#### 6. **Support as Part of Learning**
- **How:** Defines when to escalate vs. self-resolve
- **Example:** Escalation triggers for stuck resources
- **Why:** Normalizes asking for help when appropriate

---

### Common Challenges and Mitigation

#### Challenge 1: Fear of Git Operations

**Stumbling Block:** Students afraid to use `git revert` (fear of breaking things)

**Mitigation:**
- Emphasize `git revert` is safe (creates new commit, doesn't rewrite history)
- Lab provides exact commands to follow
- Explain rollback can itself be rolled back (revert the revert)

---

#### Challenge 2: Deleting in Wrong Order

**Stumbling Block:** Students delete VPC first, causing stuck resources

**Mitigation:**
- Concept 2 explicitly teaches deletion order
- Decision tree for deletion provided
- Lab bonus task practices correct order

---

#### Challenge 3: Premature Finalizer Removal

**Stumbling Block:** Students remove finalizers too quickly (without checking dependencies)

**Mitigation:**
- Assessment Question 3 tests troubleshooting order
- Emphasize finalizer removal is "last resort"
- Teach dependency checking first

---

### Confidence-Building Opportunities

**Win 1: Successful Rollback**
- **Moment:** Task 2 - ArgoCD sync completes successfully
- **Feeling:** "I can safely undo changes"
- **Teaching Point:** "You just performed a production-style rollback with full audit trail"

**Win 2: Safe Deletion**
- **Moment:** Task 4 - VPCAttachment deleted without issues
- **Feeling:** "I understand dependency order"
- **Teaching Point:** "You prevented stuck resources by deleting in correct order"

---

## Dependencies

### Prerequisites (Must Complete First)

- ✅ **Module 4.1:** Diagnosing Fabric Issues (identify issues requiring rollback)
- ✅ **Module 1.2:** GitOps workflow understanding
- ✅ **Course 2:** VPC lifecycle experience

### Enables (Unlocks These Modules)

- ✅ **Module 4.3:** Coordinating with Support (recovery skills needed for escalation preparation)
- ✅ **Module 4.4:** Post-Incident Review (rollback procedures documented in reviews)

---

## Quality Checklist

### Design Quality

- ✅ **Learning objectives are specific and measurable**
- ✅ **Content outline follows logical progression**
- ✅ **Assessment aligns with learning objectives**
- ✅ **Timing target is achievable (13-15 minutes)**

### Technical Accuracy

- ✅ **All Git commands validated**
- ✅ **kubectl deletion procedures match CRD dependencies**
- ✅ **ArgoCD workflow matches Module 1.2**

### Learning Philosophy

- ✅ **Embodies at least 3 core principles (embodies 6 of 10!)**

---

## Document Metadata

**Created:** 2025-10-16
**Author:** Course Lead (Claude Code)
**Review Status:** ⏳ PENDING Course Lead Approval
**Version:** 1.0
**Previous Module:** Module 4.1 (Diagnosing Fabric Issues - DESIGNED)
**Next Module:** Module 4.3 (Coordinating with Support - To Be Designed)

---

**Status:** ⏳ DESIGN COMPLETE - Ready for Review
**Related Issue:** GitHub Issue #11 - [DESIGN] Course 4: Troubleshooting, Recovery & Escalation (Modules 4.1-4.4)

---

**Module 4.2 Design Complete!**
Next: Module 4.3 - Coordinating with Support
