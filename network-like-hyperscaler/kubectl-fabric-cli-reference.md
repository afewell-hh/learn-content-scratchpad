# kubectl fabric CLI Reference

**Version:** v0.87.4
**Location on Control Node:** `/opt/bin/kubectl-fabric`
**Installation on Host:** Copied to `/usr/local/bin/kubectl-fabric`

## Overview

The `kubectl fabric` CLI is a kubectl plugin that provides a simplified interface for managing Hedgehog Fabric resources. It wraps kubectl and the Kubernetes client to make common Fabric operations more convenient. The CLI focuses on simplifying object creation and manipulation, while get/list/update operations are typically done using standard `kubectl` commands.

## Installation & Access

### On Control Node
kubectl-fabric is automatically installed at `/opt/bin/kubectl-fabric` on Hedgehog control nodes.

### On Ubuntu Host
To make kubectl-fabric accessible from the host (for student environments):

```bash
# Copy binary from control node (via SSH on port 22000)
sudo scp -P 22000 -i <vlab-sshkey> core@localhost:/opt/bin/kubectl-fabric /usr/local/bin/kubectl-fabric
sudo chmod +x /usr/local/bin/kubectl-fabric

# Set KUBECONFIG to vlab kubeconfig
export KUBECONFIG=/path/to/hhfab-default-topo/vlab/kubeconfig

# Test
kubectl fabric --version
```

## Global Options

```
--verbose, -v  verbose output (includes debug) (default: true)
--help, -h     show help
--version, -V  print the version
```

---

## Command Categories

### 1. VPC Commands
### 2. Switch Commands
### 3. Connection Commands
### 4. SwitchGroup Commands
### 5. External Commands
### 6. Wiring Commands
### 7. Inspect Commands

---

## 1. VPC Commands

**Command:** `kubectl fabric vpc`

Manage Virtual Private Cloud (VPC) resources.

### Subcommands

#### `vpc create`
Create a new VPC with subnet, VLAN, and optional DHCP.

**Usage:**
```bash
kubectl fabric vpc create [options]
```

**Options:**
| Flag | Description | Required | Default |
|------|-------------|----------|---------|
| `--name`, `-n` | VPC name | Yes | - |
| `--subnet` | Subnet CIDR (e.g., 10.0.1.0/24) | Yes | - |
| `--vlan` | VLAN ID | Yes | - |
| `--dhcp` | Enable DHCP | No | false |
| `--dhcp-range-start`, `--dhcp-start` | DHCP range start IP | No | - |
| `--dhcp-range-end`, `--dhcp-end` | DHCP range end IP | No | - |
| `--dhcp-lease-time`, `--dhcp-lease` | DHCP lease time in seconds | No | - |
| `--vpc-mode`, `--mode` | VPC mode (default: l2vni, or l3vni) | No | l2vni |
| `--dhcp-disable-default-route`, `--dhcp-no-default` | Disable default route in DHCP | No | false |
| `--dhcp-advertised-routes` | Custom routes (prefix-gateway format) | No | - |
| `--print`, `-p` | Print object YAML instead of applying | No | false |

**Examples:**
```bash
# Simple VPC without DHCP
kubectl fabric vpc create --name vpc-1 --subnet 10.0.1.0/24 --vlan 1001

# VPC with DHCP enabled
kubectl fabric vpc create --name vpc-2 --subnet 10.0.2.0/24 --vlan 1002 --dhcp --dhcp-start 10.0.2.10

# VPC with custom DHCP range and lease time
kubectl fabric vpc create \
  --name vpc-3 \
  --subnet 10.0.3.0/24 \
  --vlan 1003 \
  --dhcp \
  --dhcp-start 10.0.3.50 \
  --dhcp-end 10.0.3.200 \
  --dhcp-lease 7200
```

#### `vpc attach`
Attach a connection to a VPC, enabling servers on that connection to access the VPC.

**Usage:**
```bash
kubectl fabric vpc attach [options]
```

**Options:**
| Flag | Description | Required | Default |
|------|-------------|----------|---------|
| `--name`, `-n` | VPCAttachment name | No | Auto-generated |
| `--vpc-subnet`, `--subnet` | VPC/subnet (format: vpc-name/subnet-name) | Yes | - |
| `--connection`, `--conn` | Connection name | Yes | - |
| `--nativeVLAN`, `--vlan` | Use untagged traffic (true) vs tagged (false) | No | false |
| `--print`, `-p` | Print object YAML instead of applying | No | false |

**Examples:**
```bash
# Attach VPC to MCLAG connection
kubectl fabric vpc attach \
  --vpc-subnet vpc-1/default \
  --connection server-01--mclag--leaf-01--leaf-02

# Attach with native VLAN (untagged traffic)
kubectl fabric vpc attach \
  --vpc-subnet vpc-2/default \
  --connection server-03--unbundled--leaf-01 \
  --nativeVLAN
```

#### `vpc peer`
Enable peering between two VPCs, allowing traffic to flow between them.

**Usage:**
```bash
kubectl fabric vpc peer --vpc <vpc-1> --vpc <vpc-2>
```

**Examples:**
```bash
# Peer two VPCs
kubectl fabric vpc peer --vpc vpc-1 --vpc vpc-2
```

#### `vpc wipe`
Delete all VPCs, their peerings (including external), and attachments.

**Usage:**
```bash
kubectl fabric vpc wipe
```

**Warning:** This is a destructive operation. Use with caution.

---

## 2. Switch Commands

**Command:** `kubectl fabric switch`

Manage switches in the fabric.

### Subcommands

#### `switch ip`
Get switch management IP address.

**Usage:**
```bash
kubectl fabric switch ip <switch-name>
```

#### `switch ssh`
SSH into a switch (only from control nodes, using management network).

**Usage:**
```bash
kubectl fabric switch ssh <switch-name>
```

#### `switch serial`
Run serial console for a switch (if specified in switch annotations).

**Usage:**
```bash
kubectl fabric switch serial <switch-name>
```

#### `switch reboot`
Gracefully reboot a switch (only if switch is healthy and sending heartbeats).

**Usage:**
```bash
kubectl fabric switch reboot <switch-name>
```

#### `switch power-reset`
Power reset a switch (UNSAFE - skips graceful shutdown, only if switch is healthy).

**Usage:**
```bash
kubectl fabric switch power-reset <switch-name>
```

**Warning:** This is an unsafe operation that skips graceful shutdown.

#### `switch reinstall`
Reinstall a switch by rebooting into ONIE (only if switch is healthy).

**Usage:**
```bash
kubectl fabric switch reinstall <switch-name>
```

#### `switch roce`
Set RoCE mode on a switch (automatically reboots switch).

**Usage:**
```bash
kubectl fabric switch roce <switch-name> <mode>
```

#### `switch ecmp-roce-qpn`
Set ECMP RoCE QPN hashing.

**Usage:**
```bash
kubectl fabric switch ecmp-roce-qpn <switch-name> <value>
```

---

## 3. Connection Commands

**Command:** `kubectl fabric connection`

Manage connections between servers and switches.

### Subcommands

#### `connection get`
Get connection information.

**Usage:**
```bash
kubectl fabric connection get [options]
```

---

## 4. SwitchGroup Commands

**Command:** `kubectl fabric switchgroup`

Manage switch groups.

### Subcommands

#### `switchgroup create`
Create a switch group.

**Usage:**
```bash
kubectl fabric switchgroup create [options]
```

---

## 5. External Commands

**Command:** `kubectl fabric external`

Manage external network connections.

### Subcommands

#### `external create`
Create an external network resource.

**Usage:**
```bash
kubectl fabric external create [options]
```

**Options:**
| Flag | Description | Required |
|------|-------------|----------|
| `--name`, `-n` | External resource name | Yes |
| `--ipv4-namespace`, `--ipns` | IPv4 namespace | No |
| `--inbound-community`, `--in` | Inbound BGP community | No |
| `--outbound-community`, `--out` | Outbound BGP community | No |
| `--print`, `-p` | Print object YAML | No |

**Examples:**
```bash
# Create external with IPv4 namespace
kubectl fabric external create --name ext-1 --ipns public-ips

# Create with BGP communities
kubectl fabric external create \
  --name ext-2 \
  --ipns dmz \
  --inbound-community 65000:100 \
  --outbound-community 65000:200
```

#### `external peer`
Enable peering between an external network and a VPC.

**Usage:**
```bash
kubectl fabric external peer --external <external-name> --vpc <vpc-name>
```

---

## 6. Wiring Commands

**Command:** `kubectl fabric wiring`

General wiring diagram helpers.

### Subcommands

#### `wiring export`
Export the wiring diagram (includes switches, connections, VPCs, externals, etc.).

**Usage:**
```bash
kubectl fabric wiring export [options]
```

**Examples:**
```bash
# Export wiring diagram
kubectl fabric wiring export > current-wiring.yaml
```

---

## 7. Inspect Commands

**Command:** `kubectl fabric inspect`

Inspect Fabric API objects and primitives. Provides human-readable text representations of fabric state.

### Subcommands

#### `inspect fabric`
Inspect overall fabric (control nodes, switches overview, status, serials, etc.).

**Usage:**
```bash
kubectl fabric inspect fabric [options]
```

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `--output`, `-o` | Output format (text, json, yaml) | text |

**Examples:**
```bash
# Text output (human-readable)
kubectl fabric inspect fabric

# JSON output
kubectl fabric inspect fabric -o json

# YAML output
kubectl fabric inspect fabric -o yaml
```

#### `inspect switch`
Inspect a switch (status, used ports, counters, etc.).

**Usage:**
```bash
kubectl fabric inspect switch [options]
```

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `--name`, `-n` | Switch name | Required |
| `--output`, `-o` | Output format (text, json, yaml) | text |
| `--details`, `-d` | Include detailed information (firmware versions) | false |
| `--ports`, `-p` | Include ports and breakouts information | false |
| `--transceivers`, `-t` | Include transceivers information | false |
| `--counters`, `-c` | Include counters | false |
| `--lasers`, `-l` | Include laser details | false |

**Examples:**
```bash
# Basic switch inspection
kubectl fabric inspect switch --name leaf-01

# Detailed inspection with ports and transceivers
kubectl fabric inspect switch -n leaf-01 --details --ports --transceivers

# Full inspection with counters and lasers
kubectl fabric inspect switch -n spine-01 -d -p -t -c -l
```

#### `inspect port`
Inspect a switch port (connection, counters, VPC and External attachments, etc.).

**Usage:**
```bash
kubectl fabric inspect port [options]
```

#### `inspect server`
Inspect a server (connection, VPC attachments, etc.).

**Usage:**
```bash
kubectl fabric inspect server [options]
```

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `--name`, `-n` | Server name | Required |
| `--output`, `-o` | Output format (text, json, yaml) | text |

**Examples:**
```bash
# Inspect server
kubectl fabric inspect server --name server-01

# JSON output
kubectl fabric inspect server -n server-02 -o json
```

#### `inspect connection`
Inspect a connection (VPC and External attachments, Loopback Workaround usage, etc.).

**Usage:**
```bash
kubectl fabric inspect connection [options]
```

#### `inspect vpc`
Inspect a VPC/VPCSubnet (where it's attached, what's reachable from it).

**Usage:**
```bash
kubectl fabric inspect vpc [options]
```

**Examples:**
```bash
# Inspect VPC
kubectl fabric inspect vpc --name vpc-1
```

#### `inspect bgp`
Inspect BGP neighbors.

**Usage:**
```bash
kubectl fabric inspect bgp [options]
```

#### `inspect lldp`
Inspect LLDP neighbors.

**Usage:**
```bash
kubectl fabric inspect lldp [options]
```

#### `inspect ip`
Inspect an IP address (IPv4Namespace, VPCSubnet, DHCPLease, or External/StaticExternal usage).

**Usage:**
```bash
kubectl fabric inspect ip <ip-address>
```

**Examples:**
```bash
# Inspect IP allocation
kubectl fabric inspect ip 10.0.1.50
```

#### `inspect mac`
Inspect a MAC address (switch ports, DHCP leases).

**Usage:**
```bash
kubectl fabric inspect mac <mac-address>
```

**Examples:**
```bash
# Inspect MAC address
kubectl fabric inspect mac 00:50:56:ab:cd:ef
```

#### `inspect access`
Inspect access between a pair of IPs, server names, or VPCSubnets. Everything except external IPs will be translated to VPCSubnets.

**Usage:**
```bash
kubectl fabric inspect access <source> <destination>
```

**Examples:**
```bash
# Inspect access between servers
kubectl fabric inspect access server-01 server-02

# Inspect access between IPs
kubectl fabric inspect access 10.0.1.10 10.0.2.20

# Inspect access between VPC subnets
kubectl fabric inspect access vpc-1/default vpc-2/default
```

---

## Common Workflows

### Workflow 1: Create VPC and Attach Servers

```bash
# 1. Create VPC with DHCP
kubectl fabric vpc create \
  --name production \
  --subnet 10.100.0.0/24 \
  --vlan 2000 \
  --dhcp \
  --dhcp-start 10.100.0.50

# 2. Attach servers to VPC
kubectl fabric vpc attach \
  --vpc-subnet production/default \
  --connection server-01--mclag--leaf-01--leaf-02

kubectl fabric vpc attach \
  --vpc-subnet production/default \
  --connection server-02--mclag--leaf-01--leaf-02

# 3. Inspect VPC to verify attachments
kubectl fabric inspect vpc --name production

# 4. Inspect server connectivity
kubectl fabric inspect server --name server-01
```

### Workflow 2: Multi-Tier Application with VPC Peering

```bash
# Create web tier VPC
kubectl fabric vpc create --name web-tier --subnet 10.10.0.0/24 --vlan 2010 --dhcp

# Create app tier VPC
kubectl fabric vpc create --name app-tier --subnet 10.20.0.0/24 --vlan 2020 --dhcp

# Create database tier VPC
kubectl fabric vpc create --name db-tier --subnet 10.30.0.0/24 --vlan 2030 --dhcp

# Attach servers to tiers
kubectl fabric vpc attach --vpc-subnet web-tier/default --connection server-01--mclag--leaf-01--leaf-02
kubectl fabric vpc attach --vpc-subnet app-tier/default --connection server-02--mclag--leaf-01--leaf-02
kubectl fabric vpc attach --vpc-subnet db-tier/default --connection server-03--unbundled--leaf-01

# Peer tiers (web can talk to app, app can talk to db)
kubectl fabric vpc peer --vpc web-tier --vpc app-tier
kubectl fabric vpc peer --vpc app-tier --vpc db-tier

# Inspect access
kubectl fabric inspect access server-01 server-02  # web to app (should work)
kubectl fabric inspect access server-02 server-03  # app to db (should work)
kubectl fabric inspect access server-01 server-03  # web to db (should NOT work - no peering)
```

### Workflow 3: Fabric Health Inspection

```bash
# Overall fabric status
kubectl fabric inspect fabric

# Inspect all switches
kubectl fabric inspect switch -n leaf-01 --details --ports
kubectl fabric inspect switch -n leaf-02 --details --ports
kubectl fabric inspect switch -n spine-01 --details
kubectl fabric inspect switch -n spine-02 --details

# Check BGP neighbors
kubectl fabric inspect bgp

# Check LLDP neighbors
kubectl fabric inspect lldp

# Inspect specific connection
kubectl fabric inspect connection --name server-01--mclag--leaf-01--leaf-02
```

### Workflow 4: Export Wiring for Documentation

```bash
# Export current wiring diagram
kubectl fabric wiring export > production-wiring-$(date +%Y%m%d).yaml
```

---

## Integration with Standard kubectl

kubectl-fabric is designed to work alongside standard kubectl commands:

```bash
# Create VPC with kubectl-fabric
kubectl fabric vpc create --name vpc-1 --subnet 10.0.1.0/24 --vlan 1001 --dhcp

# View VPC with kubectl
kubectl get vpc -A
kubectl describe vpc vpc-1 -n default

# View VPC attachments
kubectl get vpcattachment -A

# View agents (switch state)
kubectl get agent -n fab
kubectl describe agent leaf-01 -n fab

# View events
kubectl get events -n fab --sort-by='.lastTimestamp'

# View switch CRDs
kubectl get switch -n fab
```

---

## Tips & Best Practices

### 1. Use `--print` for Dry-Run
Before creating resources, use `--print` to preview the YAML:

```bash
kubectl fabric vpc create --name test --subnet 10.0.0.0/24 --vlan 100 --print
```

### 2. Set KUBECONFIG Persistently
Add to `~/.bashrc` or `~/.zshrc`:

```bash
export KUBECONFIG=/home/ubuntu/afewell-hh/hhfab-default-topo/vlab/kubeconfig
```

### 3. Use Aliases for Common Commands
```bash
alias kf='kubectl fabric'
alias kfi='kubectl fabric inspect'
alias kfv='kubectl fabric vpc'
```

### 4. Combine kubectl fabric with kubectl
```bash
# Create VPC with kubectl fabric, then watch reconciliation with kubectl
kubectl fabric vpc create --name vpc-1 --subnet 10.0.1.0/24 --vlan 1001
kubectl get events -n default --watch
```

### 5. Use Inspect Commands for Troubleshooting
The `inspect` commands provide human-readable output that's perfect for understanding fabric state:

```bash
# Quick fabric overview
kubectl fabric inspect fabric

# Deep dive on specific switch
kubectl fabric inspect switch -n leaf-01 -d -p -t -c
```

---

## Known Limitations

1. **kubectl fabric requires kubeconfig access** - Must have valid kubeconfig pointing to the Hedgehog control plane
2. **Some operations require switch health** - Commands like `reboot`, `power-reset`, `reinstall` only work if the switch is healthy and sending heartbeats
3. **Not all CRD fields exposed** - kubectl-fabric simplifies common operations; for advanced configurations, use `kubectl apply -f` with full CRD YAML
4. **Namespace assumptions** - Most resources are created in the `default` namespace unless otherwise specified

---

## Version Compatibility

This reference is based on **kubectl-fabric v0.87.4** (Fabricator v0.41.3).

Always check version compatibility:
```bash
kubectl fabric --version
kubectl get fabricator -n fab -o yaml | grep version
```

---

## Additional Resources

- **Hedgehog Documentation:** https://docs.hedgehog.cloud
- **Fabric CRD Reference:** `/home/ubuntu/afewell-hh/docs/docs/reference/`
- **kubectl Plugin Development:** https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/

---

**Document Status:** Complete
**Last Updated:** 2025-10-15
**Phase 2a:** Environment Setup Research
