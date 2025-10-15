# Hedgehog CRD Reference with Examples and Status Fields

**Status:** Complete
**Last Updated:** 2025-10-15
**Purpose:** Comprehensive reference for Hedgehog Custom Resource Definitions (CRDs) with real examples and status field documentation

---

## Table of Contents

- [VPC Resources](#vpc-resources)
  - [VPC](#vpc)
  - [VPCAttachment](#vpcattachment)
  - [VPCPeering](#vpcpeering)
  - [IPv4Namespace](#ipv4namespace)
  - [VLANNamespace](#vlannamespace)
- [Wiring Resources](#wiring-resources)
  - [Switch](#switch)
  - [Server](#server)
  - [Connection](#connection)
  - [SwitchGroup](#switchgroup)
- [External Connectivity](#external-connectivity)
  - [External](#external)
  - [ExternalAttachment](#externalattachment)
  - [ExternalPeering](#externalpeering)
- [Agent and Monitoring](#agent-and-monitoring)
  - [Agent](#agent)
- [Status Field Reference](#status-field-reference)

---

## VPC Resources

### VPC

A Virtual Private Cloud (VPC) provides an isolated private network for resources with support for multiple subnets, user-defined VLANs, and on-demand DHCP.

#### Spec Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPC
metadata:
  name: vpc-1
  namespace: default
spec:
  # IPv4Namespace limits which subnets can be used
  ipv4Namespace: default

  # VLANNamespace limits which VLAN IDs can be used
  vlanNamespace: default

  # VPC mode: "" (default/l2vni), "l3vni", or "l3flat"
  mode: ""

  # Default isolation/restriction behavior for subnets
  defaultIsolated: false
  defaultRestricted: false

  # Subnets configuration
  subnets:
    default:  # Subnet name
      subnet: 10.10.1.0/24
      gateway: 10.10.1.1  # Optional, defaults to .1
      vlan: 1001
      isolated: false  # Isolate from other subnets within VPC
      restricted: false  # Isolate hosts within subnet from each other

      dhcp:
        enable: true
        range:
          start: 10.10.1.10
          end: 10.10.1.99
        options:
          dnsServers:
            - 1.1.1.1
            - 8.8.8.8
          timeServers:
            - 10.10.1.1
          interfaceMTU: 1500
          leaseTimeSeconds: 3600
          disableDefaultRoute: false
          advertisedRoutes:
            - destination: 10.12.10.0/24
              gateway: 10.10.1.2

    backend:  # Additional subnet
      subnet: 10.10.2.0/24
      vlan: 1002
      isolated: true  # Isolated from other subnets
      dhcp:
        relay: 10.99.0.100  # Use third-party DHCP server

  # Permit lists - define which subnets can communicate
  permit:
    - [default, backend]  # Allow default and backend to communicate

  # Static routes within the VPC
  staticRoutes:
    - prefix: 10.100.0.0/24
      nextHops:
        - 10.10.1.254
```

#### Status Example

```yaml
status: {}
```

**Status Fields:** VPC status is minimal - reconciliation state is tracked by Kubernetes conditions and events rather than explicit status fields.

#### Checking VPC Status

```bash
# List all VPCs
kubectl get vpc

# Get VPC details with status
kubectl get vpc vpc-1 -o yaml

# Check VPC events
kubectl get events --field-selector involvedObject.name=vpc-1

# Wait for VPC to be reconciled
kubectl wait --for=condition=Ready vpc/vpc-1 --timeout=5m
```

---

### VPCAttachment

A VPCAttachment binds a VPC subnet to a specific server Connection, making the VPC available on the server port(s).

#### Spec Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCAttachment
metadata:
  name: vpc-1-server-01
  namespace: default
spec:
  # Full VPC subnet name: "vpc-name/subnet-name"
  subnet: vpc-1/default

  # Connection name (from Connection CRD)
  connection: server-01--mclag--leaf-01--leaf-02

  # Optional: use native VLAN (untagged) instead of tagged VLAN
  nativeVLAN: false
```

#### Status Example

```yaml
status: {}
```

**Status Fields:** VPCAttachment status is minimal - check events for reconciliation state.

#### Checking VPCAttachment Status

```bash
# List all VPC attachments
kubectl get vpcattachment

# Get attachment details
kubectl get vpcattachment vpc-1-server-01 -o yaml

# Check which VPCs are attached to a connection
kubectl get vpcattachment --field-selector spec.connection=server-01--mclag--leaf-01--leaf-02

# Check events
kubectl get events --field-selector involvedObject.name=vpc-1-server-01
```

---

### VPCPeering

A VPCPeering enables VPC-to-VPC connectivity. Can be local (on same switches) or remote (via border switches).

#### Local Peering Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCPeering
metadata:
  name: vpc-1--vpc-2
  namespace: default
spec:
  # Permit defines peering policy
  permit:
    - vpc-1: {}  # All subnets
      vpc-2: {}  # All subnets
```

#### Remote Peering Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCPeering
metadata:
  name: vpc-1--vpc-3-remote
  namespace: default
spec:
  permit:
    - vpc-1: {}
      vpc-3: {}
  remote: border  # SwitchGroup name for border/mixed leaves
```

#### Subnet Filtering Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: VPCPeering
metadata:
  name: vpc-1--vpc-2-filtered
  namespace: default
spec:
  permit:
    # Specific subnets of vpc-1 can communicate to specific subnets of vpc-2
    - vpc-1:
        subnets: [default, backend]
      vpc-2:
        subnets: [dmz]
    # Another permit entry
    - vpc-1:
        subnets: [management]
      vpc-2:
        subnets: [management]
```

#### Status Example

```yaml
status: {}
```

#### Checking VPCPeering Status

```bash
# List all VPC peerings
kubectl get vpcpeering

# Get peering details
kubectl get vpcpeering vpc-1--vpc-2 -o yaml

# Check events
kubectl get events --field-selector involvedObject.name=vpc-1--vpc-2
```

---

### IPv4Namespace

An IPv4Namespace defines a set of non-overlapping IPv4 address ranges available for VPC subnets. All VPC subnets within a single IPv4Namespace are non-overlapping.

#### Spec Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: IPv4Namespace
metadata:
  name: default
  namespace: default
spec:
  subnets:
    - 10.0.0.0/16  # VPC subnets must be within these ranges
    - 192.168.0.0/16
```

#### Status Example

```yaml
status: {}
```

---

### VLANNamespace

A VLANNamespace defines a set of VLAN ranges available for VPC subnets. Each switch can belong to one or more disjoint VLANNamespaces.

#### Spec Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: VLANNamespace
metadata:
  name: default
  namespace: default
spec:
  ranges:
    - from: 1000
      to: 2999
```

#### Status Example

```yaml
status: {}
```

---

## Wiring Resources

### Switch

A Switch represents a physical switch in the fabric.

#### Spec Example - Spine Switch

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Switch
metadata:
  name: spine-01
  namespace: fab
spec:
  role: spine
  description: "Spine switch 1"
  profile: vs  # SwitchProfile name

  # Boot/provisioning information
  boot:
    mac: 0c:20:12:ff:05:00  # Management port MAC

  # Switch configuration
  asn: 65100
  ip: 172.30.128.1  # Fabric IP
  vtepIP: 172.30.12.1  # VTEP IP for VXLAN
  protocolIP: 172.30.8.1  # BGP Router ID

  # ECMP configuration
  ecmp:
    roceQPN: false

  # Redundancy: none for spine
  redundancy: {}
```

#### Spec Example - MCLAG Leaf Switch

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Switch
metadata:
  name: leaf-01
  namespace: fab
spec:
  role: server-leaf
  description: "Leaf switch 1 - MCLAG pair"
  profile: vs

  boot:
    mac: 0c:20:12:ff:00:00

  # Switch groups
  groups:
    - mclag-1

  # Redundancy configuration
  redundancy:
    group: mclag-1
    type: mclag

  # VLANNamespaces this switch participates in
  vlanNamespaces:
    - default

  # BGP and IP configuration
  asn: 65101
  ip: 172.30.128.11
  vtepIP: 172.30.12.11
  protocolIP: 172.30.8.11

  ecmp:
    roce QPN: false

  # Optional: port configurations
  portGroupSpeeds: {}
  portSpeeds: {}
  portBreakouts: {}
  enableAllPorts: false
```

#### Spec Example - ESLAG Leaf Switch

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Switch
metadata:
  name: leaf-03
  namespace: fab
spec:
  role: server-leaf
  description: "Leaf switch 3 - ESLAG group"
  profile: vs

  boot:
    mac: 0c:20:12:ff:02:00

  groups:
    - eslag-1

  redundancy:
    group: eslag-1
    type: eslag  # EVPN-based multi-homing

  vlanNamespaces:
    - default

  asn: 65103
  ip: 172.30.128.13
  vtepIP: 172.30.12.13
  protocolIP: 172.30.8.13

  ecmp: {}
  redundancy: {}
```

#### Status Example

```yaml
status: {}
```

**Status Fields:** Switch status is minimal. For detailed switch state, check the corresponding Agent CRD.

#### Checking Switch Status

```bash
# List all switches
kubectl get switches -n fab

# Get switch details
kubectl get switch leaf-01 -n fab -o yaml

# Check switch agent status (detailed switch state)
kubectl get agent leaf-01 -n fab -o yaml

# Check switch events
kubectl get events -n fab --field-selector involvedObject.name=leaf-01
```

---

### Server

A Server represents a physical server or compute node connected to the fabric.

#### Spec Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Server
metadata:
  name: server-01
  namespace: fab
spec:
  description: "Server 01 - MCLAG connected"
  profile: ""  # Optional ServerProfile name
```

#### Status Example

```yaml
status: {}
```

**Status Fields:** Server status is minimal - check Connection resources for connectivity status.

---

### Connection

A Connection represents physical and logical connections between devices in the fabric.

#### MCLAG Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-01--mclag--leaf-01--leaf-02
  namespace: fab
spec:
  mclag:
    links:
      - server:
          port: server-01/enp2s1
        switch:
          port: leaf-01/E1/5
      - server:
          port: server-01/enp2s2
        switch:
          port: leaf-02/E1/5
    mtu: 9000  # Optional MTU configuration
    fallback: false  # Optional LACP fallback
```

#### MCLAG Domain Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: leaf-01--mclag-domain--leaf-02
  namespace: fab
spec:
  mclagDomain:
    peerLinks:  # Carry both control and data traffic
      - switch1:
          port: leaf-01/E1/3
        switch2:
          port: leaf-02/E1/3
      - switch1:
          port: leaf-01/E1/4
        switch2:
          port: leaf-02/E1/4
    sessionLinks:  # Control plane only
      - switch1:
          port: leaf-01/E1/1
        switch2:
          port: leaf-02/E1/1
      - switch1:
          port: leaf-01/E1/2
        switch2:
          port: leaf-02/E1/2
```

#### ESLAG Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-05--eslag--leaf-03--leaf-04
  namespace: fab
spec:
  eslag:
    links:
      - server:
          port: server-05/enp2s1
        switch:
          port: leaf-03/E1/1
      - server:
          port: server-05/enp2s2
        switch:
          port: leaf-04/E1/1
    mtu: 9000
    fallback: false
```

#### Bundled Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-10--bundled--leaf-05
  namespace: fab
spec:
  bundled:
    links:
      - server:
          port: server-10/enp2s1
        switch:
          port: leaf-05/E1/2
      - server:
          port: server-10/enp2s2
        switch:
          port: leaf-05/E1/3
    mtu: 9000
```

#### Unbundled Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: server-09--unbundled--leaf-05
  namespace: fab
spec:
  unbundled:
    link:
      server:
        port: server-09/enp2s1
      switch:
        port: leaf-05/E1/1
    mtu: 9000
```

#### Fabric Connection Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: Connection
metadata:
  name: spine-01--fabric--leaf-01
  namespace: fab
spec:
  fabric:
    links:
      - leaf:
          port: leaf-01/E1/8
          ip: 172.30.128.11/31
        spine:
          port: spine-01/E1/1
          ip: 172.30.128.10/31
      - leaf:
          port: leaf-01/E1/9
          ip: 172.30.128.13/31
        spine:
          port: spine-01/E1/2
          ip: 172.30.128.12/31
```

#### Status Example

```yaml
status: {}
```

**Status Fields:** Connection status is minimal. Check switch interfaces and Agent status for link state.

---

### SwitchGroup

A SwitchGroup is a marker API object to group switches together for redundancy or policy purposes.

#### Spec Example

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: SwitchGroup
metadata:
  name: mclag-1
  namespace: fab
spec: {}
```

```yaml
apiVersion: wiring.githedgehog.com/v1beta1
kind: SwitchGroup
metadata:
  name: eslag-1
  namespace: fab
spec: {}
```

#### Status Example

```yaml
status: {}
```

---

## External Connectivity

### External

An External object represents an external system connected to the Fabric, available to a specific IPv4Namespace.

#### Spec Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: External
metadata:
  name: internet
  namespace: default
spec:
  ipv4Namespace: default

  # BGP communities for route filtering
  inboundCommunity: 65102:5000   # Filter routes FROM external
  outboundCommunity: 50000:50001  # Stamp routes TO external
```

#### Status Example

```yaml
status: {}
```

---

### ExternalAttachment

An ExternalAttachment defines how a specific switch connects to an external system via BGP peering.

#### Spec Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: ExternalAttachment
metadata:
  name: internet-leaf-01
  namespace: default
spec:
  # External object name
  external: internet

  # Connection object name (typically external or staticExternal connection type)
  connection: leaf-01--external--router-01

  # Switch port configuration
  switch:
    vlan: 100  # Optional VLAN for subinterface
    ip: 10.0.0.1/30  # IP on switch port

  # BGP neighbor configuration
  neighbor:
    asn: 65000  # External system's ASN
    ip: 10.0.0.2  # External system's IP (without prefix)
```

#### Status Example

```yaml
status: {}
```

#### Checking ExternalAttachment Status

```bash
# List external attachments
kubectl get externalattachment

# Check BGP neighbor status via Agent
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}'
```

---

### ExternalPeering

An ExternalPeering connects a VPC to an External system with route filtering.

#### Spec Example

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
        - default  # Advertise these VPC subnets to external
        - backend
    external:
      name: internet
      prefixes:
        - prefix: 0.0.0.0/0  # Permit default route from external
```

#### Default Route Example

```yaml
apiVersion: vpc.githedgehog.com/v1beta1
kind: ExternalPeering
metadata:
  name: vpc-1--internet-default
  namespace: default
spec:
  permit:
    vpc:
      name: vpc-1
      subnets: [default]
    external:
      name: internet
      prefixes:
        - prefix: 0.0.0.0/0  # Allow default route
```

#### Status Example

```yaml
status: {}
```

---

## Agent and Monitoring

### Agent

The Agent CRD is an internal API object managed by the controller that contains comprehensive switch state. This is where detailed status information is stored.

#### Status Example (Comprehensive)

```yaml
apiVersion: agent.githedgehog.com/v1beta1
kind: Agent
metadata:
  name: leaf-01
  namespace: fab
status:
  # Agent version and tracking
  version: v0.41.3
  installID: abc123
  runID: xyz789
  bootID: boot456
  lastHeartbeat: "2025-10-15T10:30:00Z"

  # Configuration application tracking
  lastAttemptTime: "2025-10-15T10:29:50Z"
  lastAttemptGen: 15
  lastAppliedTime: "2025-10-15T10:29:55Z"
  lastAppliedGen: 15

  # Detailed switch state
  state:
    nos:
      asicVersion: "broadcom"
      softwareVersion: "4.2.0-Enterprise_Base"
      platformName: "x86_64-dellemc_s5248f_c3538-r0"
      serialNumber: "SN123456"
      uptime: "21:21:27 up 1 day, 23:26"

    # Interface state (example)
    interfaces:
      Ethernet0:
        enabled: true
        admin: up
        oper: up
        mac: "00:11:22:33:44:55"
        speed: "25G"
        counters:
          inb: 123456789   # Input bytes
          outb: 987654321  # Output bytes
          ine: 0           # Input errors
          oute: 0          # Output errors

    # BGP neighbor state (example)
    bgpNeighbors:
      default:  # VRF name
        "172.30.128.10":  # Neighbor IP
          state: established
          enabled: true
          localAS: 65101
          peerAS: 65100
          prefixes:
            ipv4-unicast:
              rec: 10
              sent: 5

    # Platform state (example)
    platform:
      psus:
        PSU1:
          presence: true
          status: true
          inVoltage: 120.0
          outVoltage: 12.0
      fans:
        Fan1:
          presence: true
          status: true
          speed: 8000.0
      temps:
        CPU:
          temp: 45.0
          highThreshold: 85.0
          critHighThreshold: 95.0

    # Critical resources (ASIC tables)
    criticalResources:
      stats:
        ipv4RoutesUsed: 150
        ipv4RoutesAvailable: 32000
        ipv4NexthopsUsed: 200
        ipv4NexthopsAvailable: 16000

  # Kubernetes conditions
  conditions:
    - type: Ready
      status: "True"
      lastTransitionTime: "2025-10-15T10:00:00Z"
      reason: AgentReady
      message: "Agent is ready and switch is configured"
```

#### Checking Agent Status

```bash
# List all agents
kubectl get agents -n fab

# Get detailed agent status
kubectl get agent leaf-01 -n fab -o yaml

# Check agent readiness
kubectl wait --for=condition=Ready agent/leaf-01 -n fab --timeout=5m

# View switch BGP neighbors
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.bgpNeighbors}' | jq

# View switch interfaces
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.interfaces}' | jq

# View platform health
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.platform}' | jq

# Check ASIC resource usage
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.state.criticalResources}' | jq

# View last heartbeat
kubectl get agent leaf-01 -n fab -o jsonpath='{.status.lastHeartbeat}'
```

---

## Status Field Reference

### Understanding Status Fields

Most Hedgehog CRDs have minimal status fields, relying on Kubernetes conditions and events for state tracking. The primary detailed status information is in the Agent CRD.

#### Common Status Patterns

1. **Empty Status** (`status: {}`)
   - Most CRDs: VPC, VPCAttachment, VPCPeering, External, ExternalAttachment, ExternalPeering
   - Switch, Server, Connection, SwitchGroup
   - Status tracked via events and conditions

2. **Agent Status** (Detailed)
   - Comprehensive switch state
   - NOS information
   - Interface states
   - BGP neighbor status
   - Platform health (PSUs, fans, temps)
   - ASIC resource usage
   - Kubernetes conditions

#### Reconciliation States

Resources progress through reconciliation states tracked by:
- **Events:** Normal/Warning events indicate progress or issues
- **Conditions:** Kubernetes conditions (Ready, etc.) when applicable
- **Controller Logs:** Detailed reconciliation information

#### Common Event Types

**Normal Events:**
- `Created`: Resource created
- `Reconciling`: Controller processing
- `ReconcileSuccess`: Successfully reconciled
- `Ready`: Resource is ready

**Warning Events:**
- `ValidationFailed`: Spec validation error
- `DependencyMissing`: Referenced resource not found
- `ReconcileFailed`: Reconciliation error

#### Checking Reconciliation Status

```bash
# General pattern for any resource
kubectl get events --field-selector involvedObject.name=<resource-name>

# Watch events in real-time
kubectl get events --watch

# Filter by type
kubectl get events --field-selector type=Warning

# Check controller logs
kubectl logs -n fab deployment/fabric-controller-manager -f

# Describe resource (includes events)
kubectl describe <resource-type> <resource-name>
```

---

## Summary

This reference provides:
- Complete spec examples for all major Hedgehog CRDs
- Status field documentation and interpretation
- Commands for checking resource status
- Understanding of reconciliation patterns
- Agent CRD as the source of detailed switch state

**Key Takeaways:**
1. Most CRDs have minimal status - use events and Agent CRD for state
2. Agent CRD contains comprehensive switch operational state
3. Use `kubectl get events` to track reconciliation
4. Use `kubectl describe` for quick status overview
5. Check controller logs for detailed reconciliation information

---

**Next Steps:**
- See [WORKFLOWS.md](./WORKFLOWS.md) for kubectl-based operational workflows
- See [OBSERVABILITY.md](./OBSERVABILITY.md) for monitoring and diagnostics
- See [VLAB_CAPABILITIES.md](./VLAB_CAPABILITIES.md) for vlab topology details
