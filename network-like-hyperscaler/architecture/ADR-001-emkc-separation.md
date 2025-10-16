# ADR-001: External Management K3s Cluster (EMKC) Separation

**Status:** Accepted
**Date:** 2025-10-15
**Decision Makers:** Course Lead (Art), Dev Agent (Claude)
**Related Issues:** #6 - Phase 2a Environment Setup

---

## Context

During Phase 2a environment setup, we discovered critical architectural constraints that required a fundamental redesign of the student learning environment:

1. **Hedgehog Control Node is Immutable**: The control node is a Flatcar Linux appliance that customers are not allowed to modify. It runs only Hedgehog Fabric services.

2. **Network Isolation by Default**: VLAB uses isolated usernet networking. Control node has no internet access and cannot reach host network by default.

3. **Initial Failed Approach**: Attempted to deploy observability/GitOps tools (Prometheus, Grafana, Loki, ArgoCD, Gitea) inside control node's k3s cluster via Helm, which failed due to:
   - No internet connectivity (ImagePullBackOff errors)
   - Violates immutable infrastructure principle
   - Not representative of real customer deployments

## Decision

We will deploy all student-facing management tools in a **separate single-node K3s cluster running as a container on the Ubuntu host**, called the **External Management K3s Cluster (EMKC)**.

### Architecture

```
Ubuntu Host (192.168.88.204)
├── EMKC (k3d cluster "observability")
│   ├── Prometheus (port 9090)
│   ├── Grafana (port 3000)
│   ├── Loki (port 3100)
│   ├── ArgoCD (pending)
│   └── Gitea (pending)
└── VLAB (nested VMs via QEMU/KVM)
    ├── Control Node (immutable)
    │   ├── Hedgehog Fabric k3s
    │   └── fabric-proxy (forwards telemetry)
    ├── Switches (7x)
    │   └── Alloy agents → fabric-proxy → EMKC
    └── Servers (10x)
```

### Network Connectivity

- VLAB started with `hhfab vlab up --controls-restricted=false`
- This flag enables control node access to host network
- Switches push telemetry via fabric-proxy to EMKC services on host IP

### EMKC Implementation

```bash
# Create EMKC with port mappings
k3d cluster create observability \
  --port 3000:30080@server:0 \    # Grafana
  --port 9090:30090@server:0 \    # Prometheus
  --port 3100:30100@server:0 \    # Loki
  --api-port 6550 \
  --servers 1 --agents 0

# Deploy via Helm (internet access available on host)
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace observability \
  --set prometheus.prometheusSpec.enableRemoteWriteReceiver=true \
  --set grafana.service.type=NodePort \
  --set prometheus.service.type=NodePort

helm install loki grafana/loki-stack \
  --namespace observability
```

## Consequences

### Positive

1. **Clear Separation of Concerns**
   - Hedgehog infrastructure (immutable) vs. student tools (mutable)
   - Mirrors real-world deployment patterns
   - Control node remains untouched

2. **Simplified Management**
   - EMKC can be easily reset/recreated
   - No risk of corrupting Hedgehog Fabric
   - Standard Helm deployments work (internet available)

3. **Realistic Learning Environment**
   - Students learn proper observability integration patterns
   - External monitoring stack is production-realistic
   - GitOps workflows demonstrate real operational patterns

4. **Easy Replication**
   - k3d cluster creation is scriptable
   - Helm charts ensure consistent deployment
   - Student VM appliance can include pre-built EMKC

5. **Network Access Pattern**
   - `--controls-restricted=false` is documented VLAB flag
   - Students learn about network boundaries
   - Telemetry flow teaches proxy/forwarding concepts

### Negative

1. **Additional Complexity**
   - Two K3s clusters to manage (Hedgehog + EMKC)
   - Network configuration required (--controls-restricted flag)
   - Students need to understand cluster contexts

2. **Resource Overhead**
   - EMKC requires additional CPU/RAM
   - Estimated: ~2 vCPUs, 4GB RAM for all EMKC services

3. **Documentation Burden**
   - Must document EMKC setup clearly
   - kubectl context switching needs explanation
   - Network connectivity troubleshooting guide required

### Mitigations

- **Context Confusion**: Create clear aliases and documentation for switching between clusters
- **Resource Overhead**: EMKC is lightweight (single node, no agents)
- **Setup Complexity**: Provide automated setup scripts
- **Troubleshooting**: Document connectivity testing procedures

## Alternatives Considered

### Alternative 1: Deploy Inside Control Node (Rejected)
**Rationale**: Violates immutable infrastructure principle, no internet access, not representative of customer deployments

### Alternative 2: Docker Compose on Host (Rejected)
**Rationale**: ArgoCD requires Kubernetes, mixing deployment methods confusing for students learning K8s-native patterns

### Alternative 3: Cloud-Hosted Services (Rejected)
**Rationale**: Requires internet, complexity of account management, not self-contained for student VMs

### Alternative 4: SSH Port Forwarding (Rejected)
**Rationale**: Overly complex networking, fragile, doesn't teach proper integration patterns

## Implementation

### Phase 1: EMKC Deployment (Complete)
- ✅ k3d cluster created
- ✅ Prometheus/Grafana/Loki deployed
- ✅ Services verified accessible

### Phase 2: Telemetry Integration (In Progress)
- Configure Hedgehog Fabricator with EMKC endpoints
- Verify metrics/logs flow from switches
- Import Grafana dashboards

### Phase 3: GitOps Layer (Pending)
- Deploy ArgoCD to EMKC
- Deploy Gitea to EMKC
- Create example GitOps workflow

### Phase 4: Documentation (Pending)
- IDEAL_ENVIRONMENT_SETUP.md
- STUDENT_VM_REPLICATION_GUIDE.md
- Troubleshooting guide

## References

- **Issue #6**: Phase 2a Environment Setup
- **Context Brief**: `phase-2a-context-brief.md`
- **VLAB Documentation**: `/home/ubuntu/afewell-hh/docs/docs/vlab/`
- **Hedgehog Telemetry Docs**: `/home/ubuntu/afewell-hh/docs/docs/install-upgrade/config.md:134`

## Notes

- EMKC terminology adopted for clarity in documentation and discussions
- `--controls-restricted=false` is intentional and documented pattern
- This architecture is specific to learning environments, not production
- Future: Consider bundling EMKC as pre-built Docker image for student VMs

---

**Approved By:** Course Lead
**Review Date:** 2025-10-15
**Next Review:** After Phase 2a completion
