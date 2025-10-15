# Course 3 Redesign: Kubernetes-Native Observability

**Status:** Draft Design
**Last Updated:** 2025-10-15
**Context:** Issue #4 - Curriculum adjustments based on vlab reality

## Problem Statement

**Original Consultant Design:**
- Course 3: "Observability & Fabric Health"
- Assumed Grafana dashboards readily available
- Modules focused on Prometheus metrics, dashboard interpretation, alert rules

**Vlab Reality:**
- Observability is opt-in, requires external LGTM stack
- Default vlab has no Grafana/Prometheus
- Alloy agents exist but disabled by default

**Impact:** Course 3 requires complete redesign to work with default vlab setup

## Design Philosophy

**Turn Constraint into Strength:**
1. **Kubernetes-Native First**: Master kubectl and CRD status before adding dashboards
2. **Progressive Enhancement**: Basic → Advanced, not all-at-once
3. **Transferable Skills**: kubectl proficiency applies beyond Hedgehog
4. **Real-World Aligned**: Many users start without full observability stack

**Learning Philosophy Alignment:**
- ✅ "Focus on What Matters Most" - Core fabric health before fancy dashboards
- ✅ "Confidence Before Comprehensiveness" - Master basics first
- ✅ "Abstraction as Empowerment" - Understand CRDs directly
- ✅ "Modularity & Composability" - Observability becomes optional module

## Redesigned Course Structure

### Course 3: Kubernetes-Native Fabric Monitoring

**Goal:** Enable learners to monitor fabric health, detect issues, and collect diagnostics using kubectl and Kubernetes-native tools.

**Duration:** ~60 minutes (4 modules × 15 min)

---

### Module 3.1: Fabric Status & Health Checks

**Learning Objectives:**
- Interpret CRD status fields to assess fabric health
- Use kubectl to check switch and agent readiness
- Understand reconciliation states (Ready, Pending, Error)
- Identify healthy vs. unhealthy fabric states

**Core Content:**

**Understanding CRD Status:**
```bash
# VPC status
kubectl get vpc my-vpc -o yaml | grep -A 10 status:

# Key fields:
# - status.state: Ready | Pending | Error
# - status.vni: Auto-allocated VXLAN VNI
# - status.subnets[].state: Active | Pending | Error
```

**Switch Health Checks:**
```bash
# List all switches
kubectl get switches

# Expected healthy output:
NAME       PROFILE   ROLE          AGE
leaf-01    vs        server-leaf   2h
leaf-02    vs        server-leaf   2h
# ... all switches present

# Detailed switch status
kubectl describe switch leaf-01

# Check agent status
kubectl get agents
# Expect: All agents in Ready state
```

**Connection Status:**
```bash
# List connections
kubectl get connections

# Check specific connection
kubectl describe connection server-01--mclag--leaf-01--leaf-02

# Status indicates: Link up, MCLAG synchronized
```

**Lab Exercise:**
1. Check status of all VPCs in the fabric
2. Identify which switches are Ready vs. Pending
3. Inspect VPCAttachment status for a server
4. Interpret status fields to determine health

**Assessment:**
- Given status output, identify if fabric is healthy
- Match status values to their meanings
- Scenario: "Status shows `Pending` - what does this mean?"

---

### Module 3.2: Kubernetes Events & Change Tracking

**Learning Objectives:**
- Monitor Kubernetes events for fabric changes
- Correlate events to CRD operations
- Filter and search events effectively
- Understand event types and severity

**Core Content:**

**Event Fundamentals:**
```bash
# View all recent events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Filter by object type
kubectl get events --all-namespaces --field-selector involvedObject.kind=VPC

# Filter by specific object
kubectl get events --field-selector involvedObject.name=my-vpc

# Watch events in real-time
kubectl get events --all-namespaces --watch
```

**Event Types:**
- **Normal**: Successful operations (VPC created, reconciled)
- **Warning**: Issues requiring attention (validation failed, dependencies missing)

**Correlation Example:**
```bash
# Create VPC
kubectl apply -f vpc.yaml

# Immediately view events
kubectl get events --field-selector involvedObject.name=my-vpc

# Expected event sequence:
# 1. Created: VPC object created
# 2. Reconciling: Controller processing
# 3. Allocated VNI: VNI assigned
# 4. Ready: Reconciliation complete
```

**Event-Based Troubleshooting:**
```bash
# Find errors in last hour
kubectl get events --all-namespaces \
  --field-selector type=Warning \
  --sort-by='.lastTimestamp' | tail -20

# Common event messages:
# - "VPC subnet VLAN conflict" → VLAN already in use
# - "Connection not found" → Referenced connection doesn't exist
# - "Validation failed" → Check spec for errors
```

**Lab Exercise:**
1. Create a VPC and watch events
2. Trigger an error (e.g., invalid CIDR) and identify via events
3. Filter events for last 30 minutes
4. Correlate events across multiple CRD types

**Assessment:**
- Interpret event messages to identify root cause
- Write kubectl commands to filter specific events
- Scenario: "Find all Warning events for VPCs in last hour"

---

### Module 3.3: Logs & Controller Diagnostics

**Learning Objectives:**
- Access fabric controller logs
- Access switch agent logs
- Interpret log messages for troubleshooting
- Understand log levels and filtering

**Core Content:**

**Controller Logs:**
```bash
# Fabric controller runs in 'fab' namespace
kubectl get pods -n fab

# Expected pods:
# - fabric-controller-manager-xxx
# - fabric-proxy-xxx
# - fabric-kube-rbac-proxy-xxx

# View controller logs
kubectl logs -n fab deployment/fabric-controller-manager -f

# Filter for specific CRD reconciliation
kubectl logs -n fab deployment/fabric-controller-manager | grep "VPC/my-vpc"

# Look for errors
kubectl logs -n fab deployment/fabric-controller-manager | grep -i error
```

**Agent Logs (Advanced):**
```bash
# Agents run on switches, not in Kubernetes
# Access via switch console or SSH (if configured)

# From controller, view agent status
kubectl get agents

# Agent CRD contains status.lastHeartbeat, status.conditions
kubectl describe agent leaf-01
```

**Log Interpretation:**
```
# Example controller log entry:
I1015 10:15:30.123456 1 vpc_controller.go:142]
  "Reconciling VPC" vpc="my-vpc" namespace="default"

# Components:
# - I = Info level
# - Timestamp: 10:15:30
# - File:line: vpc_controller.go:142
# - Message: Reconciling VPC
# - Context: vpc name, namespace
```

**Common Log Patterns:**
- **Reconciliation loops**: VPC reconciled every few minutes (normal)
- **Validation errors**: "Invalid CIDR" → spec problem
- **Dependency errors**: "Connection not found" → reference issue

**Lab Exercise:**
1. Tail controller logs while creating a VPC
2. Find log entries related to VNI allocation
3. Trigger an error and identify the log message
4. Determine controller version from logs

**Assessment:**
- Match log levels to meanings (Info, Warning, Error)
- Identify which pod to check for specific issues
- Scenario: "VPC won't reconcile - which logs to check?"

---

### Module 3.4: Pre-Support Diagnostic Collection

**Learning Objectives:**
- Collect comprehensive diagnostic data before escalating
- Create CRD dumps for support tickets
- Package logs, events, and status effectively
- Know what information support needs

**Core Content:**

**Diagnostic Checklist:**
```bash
# 1. Capture CRD states
kubectl get vpcs,vpcattachments,vpcpeerings -A -o yaml > crds-vpc.yaml
kubectl get switches,servers,connections -A -o yaml > crds-wiring.yaml
kubectl get agents -A -o yaml > crds-agents.yaml

# 2. Capture events (last hour)
kubectl get events --all-namespaces \
  --sort-by='.lastTimestamp' > events.log

# 3. Capture controller logs (last 1000 lines)
kubectl logs -n fab deployment/fabric-controller-manager \
  --tail=1000 > controller.log

# 4. Capture switch/agent status
kubectl describe switches > switches-status.txt
kubectl describe agents > agents-status.txt

# 5. Create timestamp marker
date > diagnostic-timestamp.txt
```

**Diagnostic Package Script:**
```bash
#!/bin/bash
# diagnostic-collect.sh

TIMESTAMP=$(date +%Y%m%d-%H%M%S)
OUTDIR="hedgehog-diagnostics-${TIMESTAMP}"
mkdir -p ${OUTDIR}

echo "Collecting diagnostics to ${OUTDIR}/"

# CRDs
kubectl get vpcs,vpcattachments,vpcpeerings -A -o yaml > ${OUTDIR}/crds-vpc.yaml
kubectl get switches,servers,connections -A -o yaml > ${OUTDIR}/crds-wiring.yaml
kubectl get agents -A -o yaml > ${OUTDIR}/crds-agents.yaml

# Events
kubectl get events --all-namespaces --sort-by='.lastTimestamp' > ${OUTDIR}/events.log

# Logs
kubectl logs -n fab deployment/fabric-controller-manager --tail=2000 > ${OUTDIR}/controller.log

# Status
kubectl describe switches > ${OUTDIR}/switches-status.txt
kubectl describe agents > ${OUTDIR}/agents-status.txt

# Version info
kubectl version > ${OUTDIR}/kubectl-version.txt
kubectl get pods -n fab -o yaml > ${OUTDIR}/fab-namespace.yaml

# Compress
tar czf ${OUTDIR}.tar.gz ${OUTDIR}
echo "Diagnostics collected: ${OUTDIR}.tar.gz"
```

**Support Ticket Best Practices:**
1. **Concise Problem Statement**: "VPC my-vpc stuck in Pending state for 30 minutes"
2. **Steps to Reproduce**: Exact commands run
3. **Expected Behavior**: "VPC should transition to Ready"
4. **Actual Behavior**: "Status.state remains Pending"
5. **Diagnostic Package**: Attach tar.gz
6. **Attempted Troubleshooting**: "Checked events, found warning about VLAN conflict"

**Lab Exercise:**
1. Run diagnostic collection script
2. Review captured files to ensure completeness
3. Write a mock support ticket with diagnostics
4. Practice describing a problem concisely

**Assessment:**
- List 5 essential diagnostic artifacts
- Write effective support ticket description
- Scenario: "VPC peering fails - what to collect?"

---

## Optional Advanced Module: Grafana Observability Stack

**Prerequisite:** Complete Course 3 core modules

**Module:** Setting Up External Observability (60 minutes)

**Learning Objectives:**
- Configure Alloy agents for metric collection
- Set up Grafana Cloud integration (or self-hosted LGTM)
- Import Hedgehog dashboards
- Interpret dashboard metrics

**Content Outline:**
1. Grafana Cloud account setup
2. Configure `defaultAlloyConfig` in Fabricator
3. Rebuild vlab with observability enabled
4. Import six Hedgehog dashboards
5. Dashboard walkthrough and interpretation
6. Alert rule configuration
7. Correlation: metrics + events + logs

**Lab:** Configure observability for vlab, explore dashboards

**Note:** This module is OPTIONAL - not required for HCFO certification. Advanced users can enhance their monitoring, but core curriculum focuses on kubectl-native approach.

---

## Implementation Strategy

### Phase 2 (Design):
- [ ] Detailed module outlines (above is framework)
- [ ] Specific lab exercises with validation steps
- [ ] Assessment questions and scenarios
- [ ] Module dependency mapping

### Phase 3 (Content):
- [ ] Write full module content (following authoring guide)
- [ ] Create kubectl cheat sheets
- [ ] Develop troubleshooting scenarios
- [ ] Test all labs in vlab
- [ ] Peer review against learning philosophy

### Success Metrics:
- Learners can check fabric health without external tools
- Learners confidently interpret CRD status
- Learners collect comprehensive diagnostics
- Learners understand when/how to escalate
- 100% compatible with default vlab (no external dependencies)

---

## Pedagogical Advantages

**Why This Approach is Better:**

1. **Simpler Cognitive Load**
   - One tool (kubectl) vs. multiple (kubectl + Grafana + Prometheus)
   - Fewer abstractions to learn initially
   - Direct interaction with source of truth (Kubernetes API)

2. **Deeper Understanding**
   - See actual CRD status, not just dashboard visualization
   - Understand reconciliation at API level
   - Learn Kubernetes event model (transferable skill)

3. **Progressive Enhancement**
   - Master kubectl → then add dashboards (optional)
   - Confidence before complexity
   - Natural learning progression

4. **Real-World Readiness**
   - Many users won't have full observability stack immediately
   - kubectl skills work everywhere Kubernetes runs
   - Can troubleshoot even without fancy tooling

5. **Hedgehog Philosophy Aligned**
   - "Focus on What Matters Most" - core health checks first
   - "Confidence Before Comprehensiveness" - basics before advanced
   - "Teach the Why" - understand CRDs, not just dashboards

---

## Migration from Consultant Design

**Original Course 3:**
- 3.1: Fabric Telemetry Overview (Prometheus)
- 3.2: Dashboard Interpretation (Grafana)
- 3.3: Alerts & Event Handling
- 3.4: Pre-Support Diagnostic Checklist ✅ (keeping)

**New Course 3:**
- 3.1: Fabric Status & Health Checks (kubectl)
- 3.2: Kubernetes Events & Change Tracking (kubectl)
- 3.3: Logs & Controller Diagnostics (kubectl)
- 3.4: Pre-Support Diagnostic Collection ✅ (adapted)

**Salvaged Content:**
- Module 3.4 core concept retained
- Observability concepts moved to optional advanced module
- Dashboard content preserved for those who want it

---

**Status:** Ready for Phase 2 detailed design
**Next Steps:**
1. Wait for CRD examples from dev agent
2. Create detailed lab exercises
3. Write assessment questions
4. Validate against vlab capabilities
