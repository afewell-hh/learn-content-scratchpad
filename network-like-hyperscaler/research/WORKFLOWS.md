# Hedgehog kubectl-Based VPC Workflows

**Status:** Complete
**Last Updated:** 2025-10-15
**Purpose:** Step-by-step kubectl workflows for common VPC operations

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Workflow 1: Create VPC from Scratch](#workflow-1-create-vpc-from-scratch)
- [Workflow 2: Attach Server to VPC](#workflow-2-attach-server-to-vpc)
- [Workflow 3: Validate Connectivity](#workflow-3-validate-connectivity)
- [Workflow 4: Create VPC Peering](#workflow-4-create-vpc-peering)
- [Workflow 5: Add External Connectivity](#workflow-5-add-external-connectivity)
- [Workflow 6: Update VPC Configuration](#workflow-6-update-vpc-configuration)
- [Workflow 7: Cleanup and Rollback](#workflow-7-cleanup-and-rollback)
- [Troubleshooting Guide](#troubleshooting-guide)

---

## Prerequisites

### Required Access

```bash
# Verify kubectl access
kubectl cluster-info

# Verify you can access the fab namespace
kubectl get pods -n fab

# Check if switches are ready
kubectl get agents -n fab

# Verify wiring resources exist
kubectl get switches,servers,connections -n fab
```

### Understanding the Environment

```bash
# List available IPv4 and VLAN namespaces
kubectl get ipv4namespace
kubectl get vlannamespace

# View switch topology
kubectl get switches -n fab

# View server connections
kubectl get connections -n fab
kubectl get servers -n fab
```

---

## Workflow 1: Create VPC from Scratch

### Overview
This workflow creates a new VPC with a single subnet, DHCP enabled, and validates successful creation.

### Step 1: Check Available Resources

```bash
# Check IPv4Namespace to ensure your subnet is available
kubectl get ipv4namespace default -o yaml

# Expected output shows available ranges:
# spec:
#   subnets:
#   - 10.0.0.0/16

# Check VLANNamespace for available VLANs
kubectl get vlannamespace default -o yaml

# Expected output:
# spec:
#   ranges:
#   - from: 1000
#     to: 2999
```

### Step 2: Create VPC YAML

Create `vpc-1.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPC
metadata:
  name: vpc-1
  namespace: default
spec:
  ipv4Namespace: default
  vlanNamespace: default

  subnets:
    default:
      subnet: 10.10.1.0/24
      gateway: 10.10.1.1
      vlan: 1001
      dhcp:
        enable: true
        range:
          start: 10.10.1.10
          end: 10.10.1.99
        options:
          dnsServers:
            - 1.1.1.1
            - 8.8.8.8
          leaseTimeSeconds: 3600
```

### Step 3: Apply VPC

```bash
# Apply the VPC
kubectl apply -f vpc-1.yaml

# Expected output:
# vpc.vpc.githedgehog.com/vpc-1 created
```

### Step 4: Verify VPC Creation

```bash
# List VPCs
kubectl get vpc

# Expected output:
# NAME    AGE
# vpc-1   10s

# Get detailed VPC information
kubectl get vpc vpc-1 -o yaml

# Check events for the VPC
kubectl get events --field-selector involvedObject.name=vpc-1 --sort-by='.lastTimestamp'

# Expected events:
# Normal  Created        VPC created successfully
# Normal  Reconciling    Reconciling VPC configuration
# Normal  VNIAllocated   VNI 10001 allocated
# Normal  Ready          VPC reconciliation complete
```

### Step 5: Verify DHCP Subnet Creation

```bash
# Check if DHCPSubnet was created (internal resource)
kubectl get dhcpsubnet -n default

# Expected output:
# NAME              AGE
# vpc-1--default    30s

# View DHCP configuration
kubectl get dhcpsubnet vpc-1--default -o yaml
```

### Step 6: Validate VPC on Switches

This will work once switches are fully initialized:

```bash
# Check which switches have the VPC configured
# (Requires VPCAttachment - see Workflow 2)

# View controller logs for VPC reconciliation
kubectl logs -n fab deployment/fabric-controller-manager | grep "vpc-1"
```

### Workflow 1 Complete

**Success Criteria:**
- ✅ VPC created without errors
- ✅ Events show successful reconciliation
- ✅ DHCPSubnet created
- ✅ VNI allocated
- ✅ No Warning events

---

## Workflow 2: Attach Server to VPC

### Overview
Attach a server connection to the VPC, making the VPC available on server ports.

### Step 1: Identify Server Connection

```bash
# List available servers
kubectl get servers -n fab

# Expected output:
# NAME         AGE
# server-01    1h
# server-02    1h
# ...

# List connections for server-01
kubectl get connections -n fab | grep server-01

# Expected output:
# server-01--mclag--leaf-01--leaf-02   1h
```

### Step 2: Create VPCAttachment YAML

Create `vpc-1-server-01.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: vpc-1-server-01
  namespace: default
spec:
  subnet: vpc-1/default
  connection: server-01--mclag--leaf-01--leaf-02
  nativeVLAN: false  # Use tagged VLAN
```

### Step 3: Apply VPCAttachment

```bash
# Apply the attachment
kubectl apply -f vpc-1-server-01.yaml

# Expected output:
# vpcattachment.vpc.githedgehog.com/vpc-1-server-01 created
```

### Step 4: Verify Attachment

```bash
# List VPC attachments
kubectl get vpcattachment

# Expected output:
# NAME               AGE
# vpc-1-server-01    15s

# Get detailed attachment info
kubectl get vpcattachment vpc-1-server-01 -o yaml

# Check events
kubectl get events --field-selector involvedObject.name=vpc-1-server-01 --sort-by='.lastTimestamp'

# Expected events:
# Normal  Created      VPCAttachment created
# Normal  Reconciling  Configuring switches
# Normal  Ready        VPCAttachment active
```

### Step 5: Verify Switch Configuration

```bash
# Check agent status for leaf switches
kubectl get agent leaf-01 -n fab -o yaml | grep -A 20 "interfaces"
kubectl get agent leaf-02 -n fab -o yaml | grep -A 20 "interfaces"

# View controller logs for attachment reconciliation
kubectl logs -n fab deployment/fabric-controller-manager | grep "vpc-1-server-01"
```

### Step 6: Attach Additional Servers (Optional)

```bash
# Create attachment for server-02
cat <<EOF | kubectl apply -f -
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: vpc-1-server-02
  namespace: default
spec:
  subnet: vpc-1/default
  connection: server-02--mclag--leaf-01--leaf-02
  nativeVLAN: false
EOF

# Verify
kubectl get vpcattachment
```

### Workflow 2 Complete

**Success Criteria:**
- ✅ VPCAttachment created successfully
- ✅ No Warning events
- ✅ Switches configured (check via Agent CRD)
- ✅ VLAN configured on switch ports

---

## Workflow 3: Validate Connectivity

### Overview
Validate that servers can communicate within the VPC and obtain DHCP leases.

### Step 1: Check DHCP Leases

```bash
# View DHCP leases
kubectl get dhcpsubnet vpc-1--default -o yaml

# Look for status.allocated field:
# status:
#   allocated:
#     10.10.1.10:
#       expiry: "2025-10-15T11:30:00Z"
#       hostname: "server-01"
#       ip: "10.10.1.10"
```

### Step 2: Verify Server Interface Configuration

```bash
# SSH to server-01
kubectl exec -it server-01 -n fab -- bash
# OR via hhfab:
# hhfab vlab ssh server-01

# Check interface configuration
ip addr show enp2s1

# Expected output:
# enp2s1: <BROADCAST,MULTICAST,UP,LOWER_UP>
#     link/ether 00:11:22:33:44:55
#     inet 10.10.1.10/24 brd 10.10.1.255 scope global dynamic enp2s1

# Check VLAN interface (if tagged)
ip addr show enp2s1.1001

# View routes
ip route

# Expected output:
# default via 10.10.1.1 dev enp2s1.1001
# 10.10.1.0/24 dev enp2s1.1001 proto kernel scope link src 10.10.1.10
```

### Step 3: Test Connectivity Between Servers

```bash
# From server-01, ping server-02
ping -c 4 10.10.1.11

# Expected output:
# PING 10.10.1.11 (10.10.1.11) 56(84) bytes of data.
# 64 bytes from 10.10.1.11: icmp_seq=1 ttl=64 time=0.5 ms
# ...
# 4 packets transmitted, 4 received, 0% packet loss
```

### Step 4: Validate BGP and VXLAN (Advanced)

Once switches are initialized, verify VXLAN tunnels:

```bash
# Check VXLAN interfaces on leaf switches
# (Access via switch SSH when available)

# Check BGP EVPN routes
# show bgp l2vpn evpn

# Check VXLAN tunnel endpoints
# show vxlan tunnel
```

### Step 5: Monitor Events

```bash
# Watch events in real-time
kubectl get events --watch

# Filter for VPC-related events
kubectl get events --all-namespaces | grep vpc-1
```

### Workflow 3 Complete

**Success Criteria:**
- ✅ Servers obtain DHCP leases
- ✅ Servers can ping each other
- ✅ Network interfaces configured correctly
- ✅ VXLAN tunnels established (when switches ready)

---

## Workflow 4: Create VPC Peering

### Overview
Create peering between two VPCs to allow inter-VPC communication.

### Step 1: Create Second VPC

Create `vpc-2.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPC
metadata:
  name: vpc-2
  namespace: default
spec:
  ipv4Namespace: default
  vlanNamespace: default

  subnets:
    default:
      subnet: 10.20.1.0/24
      gateway: 10.20.1.1
      vlan: 1002
      dhcp:
        enable: true
        range:
          start: 10.20.1.10
          end: 10.20.1.99
        options:
          leaseTimeSeconds: 3600
```

```bash
# Apply VPC-2
kubectl apply -f vpc-2.yaml

# Verify
kubectl get vpc
```

### Step 2: Attach vpc-2 to Servers

Create `vpc-2-server-03.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: vpc-2-server-03
  namespace: default
spec:
  subnet: vpc-2/default
  connection: server-03--unbundled--leaf-01
  nativeVLAN: false
```

```bash
# Apply attachment
kubectl apply -f vpc-2-server-03.yaml

# Verify
kubectl get vpcattachment | grep vpc-2
```

### Step 3: Test Isolation (Before Peering)

```bash
# From server-01 (vpc-1), try to ping server-03 (vpc-2)
# This should FAIL - VPCs are isolated by default

ping -c 4 10.20.1.10

# Expected: 100% packet loss or "Destination Host Unreachable"
```

### Step 4: Create VPC Peering

Create `vpc-1--vpc-2-peering.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCPeering
metadata:
  name: vpc-1--vpc-2
  namespace: default
spec:
  permit:
    - vpc-1: {}  # All subnets of vpc-1
      vpc-2: {}  # All subnets of vpc-2
```

```bash
# Apply peering
kubectl apply -f vpc-1--vpc-2-peering.yaml

# Expected output:
# vpcpeering.vpc.githedgehog.com/vpc-1--vpc-2 created
```

### Step 5: Verify Peering

```bash
# List peerings
kubectl get vpcpeering

# Expected output:
# NAME           AGE
# vpc-1--vpc-2   20s

# Check events
kubectl get events --field-selector involvedObject.name=vpc-1--vpc-2 --sort-by='.lastTimestamp'

# Expected events:
# Normal  Created      VPCPeering created
# Normal  Reconciling  Configuring peering routes
# Normal  Ready        VPCPeering active
```

### Step 6: Test Connectivity (After Peering)

```bash
# From server-01 (vpc-1), ping server-03 (vpc-2)
# This should now SUCCEED

ping -c 4 10.20.1.10

# Expected output:
# PING 10.20.1.10 (10.20.1.10) 56(84) bytes of data.
# 64 bytes from 10.20.1.10: icmp_seq=1 ttl=64 time=0.8 ms
# ...
# 4 packets transmitted, 4 received, 0% packet loss
```

### Step 7: Verify Routes

```bash
# On server-01, check routing table
ip route

# Expected to see route to vpc-2 subnet:
# 10.20.1.0/24 via 10.10.1.1 dev enp2s1.1001
```

### Workflow 4 Complete

**Success Criteria:**
- ✅ Two VPCs created
- ✅ VPCPeering created without errors
- ✅ Cross-VPC connectivity verified
- ✅ Routes installed on servers

---

## Workflow 5: Add External Connectivity

### Overview
Connect a VPC to an external system for internet access or external network connectivity.

### Step 1: Verify External Connection Exists

```bash
# List connections to external systems
kubectl get connections -n fab | grep external

# Expected output (example):
# leaf-01--external--router-01   1h
```

### Step 2: Create External Object

Create `external-internet.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: External
metadata:
  name: internet
  namespace: default
spec:
  ipv4Namespace: default
  inboundCommunity: 65102:5000
  outboundCommunity: 50000:50001
```

```bash
# Apply External
kubectl apply -f external-internet.yaml

# Verify
kubectl get external
```

### Step 3: Create ExternalAttachment

Create `external-internet-leaf-01.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: ExternalAttachment
metadata:
  name: internet-leaf-01
  namespace: default
spec:
  external: internet
  connection: leaf-01--external--router-01

  switch:
    vlan: 100
    ip: 10.0.0.1/30

  neighbor:
    asn: 65000
    ip: 10.0.0.2
```

```bash
# Apply ExternalAttachment
kubectl apply -f external-internet-leaf-01.yaml

# Verify
kubectl get externalattachment
```

### Step 4: Create ExternalPeering

Create `vpc-1--internet-peering.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: ExternalPeering
metadata:
  name: vpc-1--internet
  namespace: default
spec:
  permit:
    vpc:
      name: vpc-1
      subnets:
        - default
    external:
      name: internet
      prefixes:
        - prefix: 0.0.0.0/0  # Allow default route
```

```bash
# Apply ExternalPeering
kubectl apply -f vpc-1--internet-peering.yaml

# Verify
kubectl get externalpeering
```

### Step 5: Verify BGP Session

```bash
# Check BGP neighbor status via Agent
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# Look for neighbor 10.0.0.2 with state: established
```

### Step 6: Test External Connectivity

```bash
# From server-01, test internet connectivity
ping -c 4 8.8.8.8

# Check default route
ip route | grep default

# Expected:
# default via 10.10.1.1 dev enp2s1.1001
```

### Workflow 5 Complete

**Success Criteria:**
- ✅ External object created
- ✅ ExternalAttachment configured
- ✅ BGP session established
- ✅ ExternalPeering active
- ✅ Default route received in VPC

---

## Workflow 6: Update VPC Configuration

### Overview
Update an existing VPC to add a subnet or modify DHCP settings.

### Step 1: Add New Subnet to VPC

```bash
# Edit VPC to add subnet
kubectl edit vpc vpc-1

# Or use kubectl patch:
kubectl patch vpc vpc-1 --type='merge' -p '
spec:
  subnets:
    backend:
      subnet: 10.10.2.0/24
      gateway: 10.10.2.1
      vlan: 1003
      isolated: true
      dhcp:
        enable: true
        range:
          start: 10.10.2.10
          end: 10.10.2.99
'
```

### Step 2: Verify Subnet Addition

```bash
# Get updated VPC
kubectl get vpc vpc-1 -o yaml | grep -A 20 "subnets:"

# Check for new DHCPSubnet
kubectl get dhcpsubnet -n default | grep vpc-1
```

### Step 3: Attach Server to New Subnet

Create `vpc-1-backend-server-04.yaml`:

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: vpc-1-backend-server-04
  namespace: default
spec:
  subnet: vpc-1/backend  # New subnet
  connection: server-04--bundled--leaf-02
  nativeVLAN: false
```

```bash
# Apply
kubectl apply -f vpc-1-backend-server-04.yaml

# Verify
kubectl get vpcattachment | grep backend
```

### Step 4: Update DHCP Options

```bash
# Patch VPC to add DNS servers
kubectl patch vpc vpc-1 --type='json' -p='[
  {
    "op": "add",
    "path": "/spec/subnets/default/dhcp/options/dnsServers",
    "value": ["1.1.1.1", "8.8.8.8", "8.8.4.4"]
  }
]'

# Verify
kubectl get vpc vpc-1 -o jsonpath='{.spec.subnets.default.dhcp.options.dnsServers}'
```

### Workflow 6 Complete

**Success Criteria:**
- ✅ VPC updated successfully
- ✅ New subnet created
- ✅ DHCP configuration updated
- ✅ No reconciliation errors

---

## Workflow 7: Cleanup and Rollback

### Overview
Safely remove VPC resources in the correct order to avoid dependency issues.

### Step 1: Remove VPCAttachments First

```bash
# List all attachments for vpc-1
kubectl get vpcattachment | grep vpc-1

# Delete attachments
kubectl delete vpcattachment vpc-1-server-01
kubectl delete vpcattachment vpc-1-server-02
kubectl delete vpcattachment vpc-1-backend-server-04

# Verify deletion
kubectl get vpcattachment
```

### Step 2: Remove VPC Peerings

```bash
# List peerings
kubectl get vpcpeering

# Delete peering
kubectl delete vpcpeering vpc-1--vpc-2

# Verify
kubectl get vpcpeering
```

### Step 3: Remove External Peerings

```bash
# Delete external peering
kubectl delete externalpeering vpc-1--internet

# Verify
kubectl get externalpeering
```

### Step 4: Remove VPC

```bash
# Delete VPC
kubectl delete vpc vpc-1

# Verify deletion
kubectl get vpc
```

### Step 5: Verify Cleanup

```bash
# Check for orphaned DHCPSubnets
kubectl get dhcpsubnet -n default

# Check events for cleanup
kubectl get events --sort-by='.lastTimestamp' | tail -20

# Verify no errors in controller logs
kubectl logs -n fab deployment/fabric-controller-manager --tail=50
```

### Rollback Procedure

If you need to rollback a failed change:

```bash
# Option 1: Reapply previous version from Git
git checkout HEAD~1 vpc-1.yaml
kubectl apply -f vpc-1.yaml

# Option 2: Use kubectl rollout (for deployments)
# Not applicable to CRDs, but keep previous YAML versions

# Option 3: Delete and recreate
kubectl delete vpc vpc-1
kubectl apply -f vpc-1-backup.yaml

# Always verify after rollback
kubectl get events --field-selector involvedObject.name=vpc-1
```

### Workflow 7 Complete

**Success Criteria:**
- ✅ Resources deleted in correct order
- ✅ No orphaned resources
- ✅ No errors in events or logs

---

## Troubleshooting Guide

### VPC Not Creating

**Symptoms:**
- VPC stuck without status
- Warning events

**Debug Steps:**

```bash
# Check events
kubectl get events --field-selector involvedObject.name=vpc-1

# Check controller logs
kubectl logs -n fab deployment/fabric-controller-manager | grep vpc-1

# Common issues:
# - Invalid subnet (outside IPv4Namespace)
# - Invalid VLAN (outside VLANNamespace)
# - Overlapping subnets with existing VPC
```

**Solution:**

```bash
# Verify IPv4Namespace
kubectl get ipv4namespace default -o yaml

# Verify VLANNamespace
kubectl get vlannamespace default -o yaml

# Fix VPC spec and reapply
kubectl apply -f vpc-1-fixed.yaml
```

### VPCAttachment Not Working

**Symptoms:**
- Attachment created but VLAN not on switch
- Warning events

**Debug Steps:**

```bash
# Verify connection exists
kubectl get connection server-01--mclag--leaf-01--leaf-02 -n fab

# Check if switches support the connection type
kubectl get switch leaf-01 -n fab -o yaml

# View agent logs
kubectl logs -n fab agent-leaf-01

# Check controller reconciliation
kubectl logs -n fab deployment/fabric-controller-manager | grep vpc-1-server-01
```

**Solution:**

```bash
# Ensure correct connection name
kubectl get connections -n fab | grep server-01

# Recreate attachment with correct connection
kubectl delete vpcattachment vpc-1-server-01
kubectl apply -f vpc-1-server-01-fixed.yaml
```

### DHCP Not Working

**Symptoms:**
- Servers not getting IP addresses
- DHCPSubnet created but no leases

**Debug Steps:**

```bash
# Check DHCPSubnet status
kubectl get dhcpsubnet vpc-1--default -o yaml

# Check DHCP server pod
kubectl get pods -n default | grep dhcp
kubectl logs -n default <dhcp-pod-name>

# Verify DHCP configuration
kubectl get dhcpsubnet vpc-1--default -o jsonpath='{.spec}'
```

**Solution:**

```bash
# Verify DHCP range is valid
# Ensure server interface is up and tagged with correct VLAN

# Restart DHCP server if needed
kubectl rollout restart deployment/dhcp-server -n default

# Check DHCP relay configuration (if using third-party DHCP)
```

### VPC Peering Not Allowing Traffic

**Symptoms:**
- Peering created but no connectivity
- Pings fail between VPCs

**Debug Steps:**

```bash
# Verify peering configuration
kubectl get vpcpeering vpc-1--vpc-2 -o yaml

# Check events
kubectl get events --field-selector involvedObject.name=vpc-1--vpc-2

# Verify both VPCs in same IPv4Namespace
kubectl get vpc vpc-1 -o jsonpath='{.spec.ipv4Namespace}'
kubectl get vpc vpc-2 -o jsonpath='{.spec.ipv4Namespace}'

# Check isolation settings
kubectl get vpc vpc-1 -o jsonpath='{.spec.subnets.default.isolated}'
```

**Solution:**

```bash
# Ensure VPCs are in same IPv4Namespace
# Verify permit configuration includes desired subnets
# Check that subnets are not marked as isolated (or use permit lists)

# Reapply peering
kubectl delete vpcpeering vpc-1--vpc-2
kubectl apply -f vpc-1--vpc-2-peering.yaml
```

### External BGP Session Not Establishing

**Symptoms:**
- BGP neighbor state not "established"
- No routes received from external

**Debug Steps:**

```bash
# Check Agent status for BGP neighbors
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# View BGP session details
kubectl get externalattachment internet-leaf-01 -o yaml

# Check controller logs
kubectl logs -n fab deployment/fabric-controller-manager | grep internet

# Verify external connection exists
kubectl get connection leaf-01--external--router-01 -n fab
```

**Solution:**

```bash
# Verify neighbor IP and ASN are correct
# Ensure switch IP is reachable from external router
# Check firewall rules if applicable

# Recreate ExternalAttachment
kubectl delete externalattachment internet-leaf-01
kubectl apply -f external-internet-leaf-01.yaml
```

### General Debugging Commands

```bash
# View all VPC resources
kubectl get vpc,vpcattachment,vpcpeering,external,externalattachment,externalpeering -A

# Watch events in real-time
kubectl get events --watch

# Filter Warning events
kubectl get events --field-selector type=Warning

# View controller logs
kubectl logs -n fab deployment/fabric-controller-manager -f

# Check agent readiness
kubectl get agents -n fab

# Describe any resource for quick overview
kubectl describe vpc vpc-1
kubectl describe vpcattachment vpc-1-server-01
```

---

## Summary

This workflows document provides:
- Step-by-step procedures for common VPC operations
- Verification steps for each workflow
- Comprehensive troubleshooting guide
- kubectl commands for all operations

**Key Workflow Principles:**
1. Always verify prerequisites before starting
2. Check events after each operation
3. Delete resources in reverse dependency order
4. Monitor controller logs for reconciliation issues
5. Use describe and events for quick debugging

**Next Steps:**
- See [CRD_REFERENCE.md](./CRD_REFERENCE.md) for detailed CRD documentation
- See [OBSERVABILITY.md](./OBSERVABILITY.md) for monitoring approaches
- See [VLAB_CAPABILITIES.md](./VLAB_CAPABILITIES.md) for topology details
