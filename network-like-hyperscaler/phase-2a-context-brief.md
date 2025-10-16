# Phase 2a Context Brief - Environment Setup Pivot

**Date:** 2025-10-15
**Issue:** #6 - Ideal Environment Setup
**Status:** In Progress - Architecture Corrected, VLAB Rebuilding

---

## What Happened

### Original Plan (Incorrect)
Initial approach attempted to deploy Prometheus/Grafana/Loki/ArgoCD/Gitea **INSIDE** the Hedgehog control node's k3s cluster using Helm.

### Critical Blocker Discovered
- Attempted Helm install to control node k3s failed with `ImagePullBackOff`
- Root cause: **Control node has no internet connectivity** (DNS lookup failures for registry.k8s.io)
- VLAB uses isolated usernet networking by default
- Control node is 100% isolated from host network

### Course Lead Intervention - Key Insight
**"The control node is an immutable Flatcar Linux appliance that customers are not allowed to modify."**

This fundamentally changed the architecture understanding:
- ❌ We should NOT deploy additional services inside control node k3s
- ✅ Control node is immutable infrastructure for Hedgehog Fabric only
- ✅ Student tools (observability, GitOps) should run on Ubuntu host

---

## Corrected Architecture

### Infrastructure Separation

```
┌─────────────────────────────────────────────────────────────────┐
│                        Ubuntu Host (Student Access)              │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ k3d Cluster (observability) - Single Node in Docker      │   │
│  │                                                            │   │
│  │  • Prometheus (Remote Write API): localhost:9090         │   │
│  │  • Grafana: localhost:3000                                │   │
│  │  • Loki: localhost:3100                                   │   │
│  │  • ArgoCD: (pending deployment)                           │   │
│  │  • Gitea: (pending deployment)                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  Host IP: 192.168.88.204 (accessible from control node)          │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ Network Access (via --controls-restricted=false)
                              │
┌─────────────────────────────┼───────────────────────────────────┐
│          VLAB (Nested VMs)  │                                    │
│                             │                                    │
│  ┌─────────────────────────┴──────────────────────────────┐    │
│  │ Control Node (Immutable - DO NOT MODIFY)               │    │
│  │                                                          │    │
│  │  • Hedgehog Fabric k3s Cluster                          │    │
│  │  • fabric-proxy (NodePort 31028)                        │    │
│  │  • Alloy agents on switches push to fabric-proxy       │    │
│  │  • fabric-proxy forwards to host Prometheus/Loki       │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Switches (7x) - leaf-01 through leaf-05, spine-01/02    │   │
│  │  • Alloy agents scrape metrics                           │   │
│  │  • Push to fabric-proxy → Host Prometheus/Loki          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Servers (10x) - server-01 through server-10             │   │
│  └──────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────┘
```

### Key Architecture Decisions

1. **Immutable Control Node**
   - Hedgehog control node runs only Hedgehog Fabric services
   - No student modifications allowed
   - Mirrors real-world customer deployments

2. **K3d on Ubuntu Host**
   - Separate single-node k3s cluster running in Docker
   - Hosts all student-facing tools (Grafana, ArgoCD, Gitea, etc.)
   - Easy to manage, replicate, and reset
   - Students interact with tools on familiar localhost URLs

3. **Network Connectivity**
   - VLAB restarted with `--controls-restricted=false` flag
   - Allows control node to reach host network (192.168.88.204)
   - Enables telemetry flow: Switches → fabric-proxy → Host Prometheus/Loki

4. **Port Mappings (k3d → Host)**
   - 3000:30080 → Grafana UI
   - 9090:30090 → Prometheus (with Remote Write API)
   - 3100:30100 → Loki Push API

---

## Current State

### ✅ Completed

1. **kubectl fabric CLI Research & Installation**
   - Binary copied from control node to `/usr/local/bin/kubectl-fabric`
   - Comprehensive reference document created: `kubectl-fabric-cli-reference.md` (500+ lines)
   - All commands, subcommands, and workflows documented

2. **K3d Observability Cluster Deployed**
   - Created cluster: `k3d cluster create observability`
   - Installed via Helm:
     - `kube-prometheus-stack` (Prometheus + Grafana + Alertmanager)
     - `loki-stack` (Loki for logs)
   - All services healthy and verified accessible on host

3. **Telemetry Configuration Prepared**
   - YAML file ready: `/home/ubuntu/afewell-hh/observability-stack/hedgehog-telemetry.yaml`
   - Configured for host endpoints: `http://192.168.88.204:9090` and `:3100`
   - Uses `useControlProxy: true` to route through fabric-proxy

4. **VLAB Rebuilding**
   - Command: `hhfab vlab up --recreate --controls-restricted=false`
   - Status: Running (~6 minutes in, typically takes 10-15 minutes)
   - Will enable control node → host network access

### 📋 Pending (Next Steps)

1. **Test Network Connectivity**
   - SSH to control node (port 22000)
   - Verify can reach `http://192.168.88.204:9090/-/healthy`
   - Verify can reach `http://192.168.88.204:3100/ready`

2. **Apply Telemetry Configuration**
   ```bash
   export KUBECONFIG=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig
   kubectl patch -n fab fabricator/default --type merge \
     --patch-file /home/ubuntu/afewell-hh/observability-stack/hedgehog-telemetry.yaml
   ```

3. **Wait for Switches to Register**
   - Monitor: `kubectl get agents -n fab`
   - Typically takes 10-15 minutes after control node is ready

4. **Verify Telemetry Flow**
   - Check Prometheus targets: http://localhost:9090/targets
   - Query for Hedgehog metrics in Prometheus
   - Check Loki for switch logs

5. **Import Hedgehog Grafana Dashboards**
   - 6 dashboard JSON files located at: `/home/ubuntu/afewell-hh/docs/docs/user-guide/boards/`
   - Import via Grafana UI (localhost:3000) or ConfigMaps
   - Verify dashboards show live data from switches

6. **Deploy ArgoCD to K3d Cluster**
   ```bash
   kubectl create namespace argocd
   helm repo add argo https://argoproj.github.io/argo-helm
   helm install argocd argo/argo-cd --namespace argocd --set server.service.type=NodePort
   ```

7. **Deploy Gitea to K3d Cluster**
   ```bash
   helm repo add gitea-charts https://dl.gitea.io/charts/
   helm install gitea gitea-charts/gitea --namespace gitea --create-namespace
   ```

8. **Create GitOps Workflow Example**
   - Configure Gitea repo with Hedgehog VPC YAML
   - Configure ArgoCD application pointing to Gitea
   - Demonstrate: Git commit → ArgoCD sync → VPC created
   - Observe reconciliation in Grafana dashboards

9. **Documentation**
   - `IDEAL_ENVIRONMENT_SETUP.md` - Complete setup guide
   - `STUDENT_VM_REPLICATION_GUIDE.md` - VM appliance creation guide
   - Include k3d cluster setup, telemetry config, dashboard import

10. **Validation**
    - All success criteria from issue #6 verified
    - Environment ready for Module 1.2 and subsequent module designs

---

## Important Files & Locations

### Documentation
- kubectl fabric CLI reference: `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/kubectl-fabric-cli-reference.md`
- Hedgehog Learning Philosophy: `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/hedgehogLearningPhilosophy.md`
- Phase 1 Research (completed): `/home/ubuntu/afewell-hh/learn_content_scratchpad/network-like-hyperscaler/research/`

### Configuration Files
- Telemetry config: `/home/ubuntu/afewell-hh/observability-stack/hedgehog-telemetry.yaml`
- VLAB kubeconfig: `/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig`
- VLAB SSH key: `/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/sshkey`

### Grafana Dashboards
- Location: `/home/ubuntu/afewell-hh/docs/docs/user-guide/boards/`
- Files:
  1. `grafana_crm.json` - ASIC Critical Resources
  2. `grafana_fabric.json` - BGP/Fabric monitoring
  3. `grafana_interfaces.json` - Interface stats
  4. `grafana_logs.json` - Log aggregation
  5. `grafana_platform.json` - Hardware health (PSU, fans, temp)
  6. `grafana_node_exporter.json` - Linux system metrics

### Binaries
- kubectl-fabric: `/usr/local/bin/kubectl-fabric`
- Helm: `/usr/local/bin/helm`
- k3d: `/usr/local/bin/k3d`

---

## Key Commands

### Access Control Node
```bash
# SSH to control node
ssh -i /home/ubuntu/afewell-hh/hhfab-default-topo/vlab/sshkey \
  -p 22000 core@localhost

# Or use hhfab wrapper
cd /home/ubuntu/afewell-hh/hhfab-default-topo
hhfab vlab ssh -n control-1
```

### Switch Contexts Between Clusters
```bash
# Use Hedgehog VLAB cluster
export KUBECONFIG=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig

# Use k3d observability cluster (default after creation)
export KUBECONFIG=$HOME/.kube/config
# or: kubectl config use-context k3d-observability
```

### Access Services
```bash
# Grafana (admin/prom-operator)
http://localhost:3000

# Prometheus
http://localhost:9090

# Loki
http://localhost:3100
```

### Monitor VLAB Status
```bash
# Check control node readiness
kubectl get nodes --kubeconfig=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig

# Check switches registration
kubectl get agents -n fab --kubeconfig=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig

# Check fabric-proxy service
kubectl get svc -n fab fabric-proxy --kubeconfig=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig
```

---

## Success Criteria (from Issue #6)

- [ ] kubectl fabric CLI accessible from host with full documentation ✅ (COMPLETE)
- [ ] Prometheus receiving metrics from all 7 Hedgehog switches
- [ ] Loki receiving logs from all 7 switches
- [ ] All 6 Grafana dashboards showing live data
- [ ] Gitea running with test repository
- [ ] ArgoCD syncing Hedgehog configs from Gitea
- [ ] GitOps workflow demonstrated (Gitea → ArgoCD → VPC creation)
- [ ] Complete setup documentation written
- [ ] Environment validated and ready for module design

---

## Lessons Learned

1. **Always clarify infrastructure boundaries first**
   - Control node = immutable customer infrastructure
   - Student tooling = separate, mutable environment

2. **Network isolation is default in VLAB**
   - Use `--controls-restricted=false` for host access
   - Understand usernet vs bridged networking

3. **K3d provides perfect middle ground**
   - Kubernetes environment for complex apps (ArgoCD, etc.)
   - No internet required (all images can be pulled locally)
   - Easy to reset and replicate for student VMs

4. **Separation of concerns is crucial**
   - Hedgehog Fabric = control node (immutable)
   - Observability/GitOps = k3d cluster (student tools)
   - Clear boundaries make architecture understandable

---

## References

- Issue #6: `/home/ubuntu/afewell-hh/learn_content_scratchpad` repo
- VLAB Documentation: `/home/ubuntu/afewell-hh/docs/docs/vlab/`
- Hedgehog Telemetry Docs: `/home/ubuntu/afewell-hh/docs/docs/install-upgrade/config.md` (line 134+)
- Grafana Dashboard Docs: `/home/ubuntu/afewell-hh/docs/docs/user-guide/grafana.md`

---

## Next Dev Agent: Start Here

1. **Check VLAB status**: Is control node ready? Are switches registered?
2. **Test connectivity**: Can control node reach host Prometheus/Loki?
3. **Apply telemetry config**: Use prepared YAML file
4. **Verify metrics flow**: Check Prometheus targets and queries
5. **Import dashboards**: Load all 6 Grafana dashboards
6. **Deploy ArgoCD/Gitea**: Add GitOps layer to k3d cluster
7. **Create workflow example**: Demonstrate Git → ArgoCD → VPC
8. **Document everything**: Create setup and replication guides

**Current blocker:** Waiting for VLAB rebuild to complete (started at ~13:29, likely ready by 13:45-14:00)

**Philosophy:** Follow Hedgehog Learning Philosophy - confidence before comprehensiveness, focus on what matters most, teach the why behind the how.

---

**Last Updated:** 2025-10-15 13:50 UTC
**Phase:** 2a - Environment Setup
**Status:** Architecture corrected, awaiting VLAB completion
