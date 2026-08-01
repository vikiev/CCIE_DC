# VMware and Containers for CCIE DC v3.1

## Prerequisite Knowledge

- Basic virtualization concepts (hypervisor, VM, vSwitch)
- Ethernet networking (VLANs, VXLAN, trunking)
- Understanding of ACI policy model (Tenant, VRF, BD, EPG, Contract)
- Basic Linux networking (namespaces, bridges, iptables)
- Understanding of container concepts (images, orchestration)
- Familiarity with TCP/IP and DNS

---

## VMware vSphere Fundamentals

### vCenter and ESXi Architecture

```text
+------------------------------------------+
|            vCenter Server                 |
|  (Management, SSO, Inventory, Tasks)     |
+----+----------+----------+----------+----+
     |          |          |          |
+----+---+ +---+----+ +---+----+ +---+----+
| ESXi 1 | | ESXi 2 | | ESXi 3 | | ESXi 4 |
| (Host) | | (Host) | | (Host) | | (Host) |
+--------+ +--------+ +--------+ +--------+
     |          |          |          |
+----+----------+----------+----------+----+
|          Shared Storage (SAN/NAS)        |
+------------------------------------------+
```

#### vCenter Server

- Centralized management of ESXi hosts and VMs
- Provides: SSO, inventory, task scheduling, alarms, permissions
- Required for: DRS, HA, vMotion, VDS, distributed policies
- Deployment: vCenter Server Appliance (VCSA) — Linux-based
- Database: embedded PostgreSQL (VCSA 7.0+)

#### ESXi Host

- Type-1 bare-metal hypervisor
- VMkernel: handles CPU scheduling, memory management, storage, networking
- VMkernel ports (vmk): management, vMotion, storage (iSCSI/NFS), VXLAN
- No general-purpose OS — purpose-built for virtualization

### Clusters, DRS, and HA

#### vSphere Cluster

- Logical grouping of ESXi hosts sharing resources
- Enables: DRS, HA, vMotion, EVC (Enhanced vMotion Compatibility)
- Shared storage requirement for vMotion and HA

#### DRS (Distributed Resource Scheduler)

```text
DRS functionality:
  - Monitors CPU/memory utilization across hosts
  - Recommends or auto-executes vMotion to balance load
  - DRS rules: affinity/anti-affinity (keep VMs together/apart)
  - DRS automation levels:
    - Manual: recommendations only
    - Partially automated: auto initial placement, manual migration
    - Fully automated: auto placement and migration
  - DRS threshold: 1 (conservative) to 5 (aggressive)
```

#### HA (High Availability)

```text
HA functionality:
  - Detects host failure (via heartbeat network)
  - Restarts VMs on surviving hosts
  - Admission control: reserves capacity for failover
  - HA isolation address: IP used to detect network partition
  - VM restart priority: high/medium/low
  - Host isolation response: power off VMs / leave powered on

HA vs DRS:
  - HA: reactive (restart after failure)
  - DRS: proactive (balance before failure)
  - Both require shared storage
  - HA does NOT require vMotion; DRS does
```

#### vMotion

```text
vMotion requirements:
  - Shared storage (VMFS, NFS, vVols)
  - VMkernel port for vMotion traffic (dedicated VLAN)
  - Compatible CPU (EVC masks differences)
  - Same network access (VM port group exists on destination)
  - VM must not have local-only devices (CD-ROM, USB)

vMotion types:
  - Compute vMotion: move VM between hosts (shared storage)
  - Storage vMotion: move VM disks between datastores
  - Enhanced vMotion (vSphere 6+): simultaneous compute + storage
  - Cross-vCenter vMotion: move VM between vCenter instances
```

---

## vSphere Distributed Switch (VDS)

### VDS Architecture

```text
+------------------------------------------+
|            vCenter Server                 |
|  (VDS configuration managed centrally)   |
+----+----------+----------+----------+----+
     |          |          |          |
+----+---+ +---+----+ +---+----+ +---+----+
| ESXi 1 | | ESXi 2 | | ESXi 3 | | ESXi 4 |
| +----+ | | +----+ | | +----+ | | +----+ |
| |VDS | | | |VDS | | | |VDS | | | |VDS | |
| |proxy| | | |proxy| | | |proxy| | | |proxy| |
| +----+ | | +----+ | | +----+ | | +----+ |
+--------+ +--------+ +--------+ +--------+

VDS = single logical switch across all hosts
Each host runs a "proxy" that enforces VDS config
```

### VDS Configuration

```text
VDS key components:
  - Uplinks (vmnic): physical NICs on each host
  - Port groups: logical grouping with common policy (like VLAN)
  - DVPort: individual VM connection point
  - Teaming policy: active/standby, load balancing algorithm
  - Traffic shaping: ingress/egress rate limiting
  - Security: promiscuous mode, MAC changes, forged transmits
  - NetFlow: traffic monitoring
  - Port mirroring (SPAN): traffic capture

VDS vs Standard vSwitch:
  - VDS: centralized config, consistent across hosts, requires vCenter
  - vSS: per-host config, can drift, no vCenter needed
  - VDS required for: NSX, ACI VMM, NIOC, ERSPAN
```

### VDS and ACI Integration

```text
When ACI VMM domain is configured:
  1. ACI APIC creates VDS in vCenter automatically
  2. EPGs map to VDS port groups
  3. ACI pushes: VLAN encapsulation, teaming policy, security policy
  4. VM adapter connects to port group = connects to EPG
  5. ACI leaf learns VM endpoint via LLDP/CDP from VDS

Port group naming: <tenant>|<ap>|<epg>
Example: Prod-Tenant|Web-AP|Web-EPG
```

### Nexus 1000V (Legacy)

```text
Nexus 1000V was a software VDS replacement:
  - VSM (Virtual Supervisor Module): management plane
  - VEM (Virtual Ethernet Module): data plane on each host
  - Provided NX-OS CLI for virtual switch management
  - Supported: port profiles, ERSPAN, QoS, ACLs
  - DEPRECATED: replaced by ACI VMM domain + native VDS
  - Know for exam: historical context, may appear in migration scenarios
```

---

## VM Networking

### Port Groups and VLANs

```text
Standard vSwitch / VDS port group:
  - VLAN ID: 0 (trunk/ESXi managed) or 1-4094 (access)
  - For trunk to VM: VLAN 4095 (guest tagging)
  - Security policy per port group
  - Teaming policy per port group

VM networking flow:
  VM vNIC -> Port Group -> vSwitch/VDS -> vmnic (physical) -> ToR
```

### VXLAN with NSX

```text
NSX-T (now VMware NSX) overlay:
  - Creates VXLAN overlay independent of physical network
  - N-VDS (NSX Virtual Distributed Switch) or VDS (NSX 4.x)
  - TEP (Tunnel Endpoint) on each ESXi host (vmk with VXLAN IP)
  - Segments (logical L2) mapped to VXLAN VNIs
  - Tier-0/Tier-1 gateways for L3 routing
  - Distributed firewall (microsegmentation)

NSX vs ACI:
  - NSX: overlay managed by NSX Manager, control plane local
  - ACI: overlay managed by APIC, control plane distributed (COOP)
  - Both use VXLAN, but different control planes
  - Cannot run NSX overlay and ACI overlay simultaneously on same host
  - ACI VMM domain replaces NSX for network policy in ACI fabrics
```

---

## VMware and ACI VMM Integration

### VMM Domain Configuration

```text
ACI VMM Domain creation:
  1. Create VMM domain (VMware type)
  2. Associate vCenter credentials
  3. Specify DVS name, datacenter, folder
  4. APIC connects to vCenter and creates DVS
  5. Associate EPGs to VMM domain

APIC GUI: Virtual Networking > VMM Domains > VMware > Create
  - Name: VMM-VC01
  - vCenter: vc01.datacenter.local
  - Credentials: admin user
  - DVS Name: ACI-DVS
  - Datacenter: DC01
  - Uplink names: uplink1, uplink2 (must match ESXi vmnic mapping)
```

### EPG to Port Group Mapping

```text
When EPG is associated with VMM domain:
  - APIC creates port group on DVS: <tenant>|<ap>|<epg>
  - Port group gets VLAN from EPG's static binding or dynamic binding
  - VM connected to port group = endpoint in that EPG
  - ACI leaf learns VM MAC via:
    - LLDP/CDP (from VDS uplink)
    - VMware Tools (via APIC <-> vCenter API)
    - COOP database (endpoint learning)

Static path binding for VMM:
  - EPG > Static Ports > VMM domain > DVS > port group
  - Encap: vlan-100 (or vxlan-15001 for VXLAN mode)
```

### DVS Policies Pushed by ACI

```text
ACI pushes to DVS port groups:
  - VLAN ID (from EPG encap)
  - Teaming policy: Route based on IP hash (for LACP) or Explicit failover
  - Security: Promiscuous=Reject, MAC changes=Reject, Forged=Reject
  - MTU: 9000 (for VXLAN)
  - NetFlow: enabled (for ACI visibility)
  - Traffic filtering: contract enforcement (via vSphere distributed firewall)

Verification on ESXi:
  esxcli network vswitch dvs vmware list
  esxcli network vswitch dvs vmware portgroup list
```

### Microsegmentation with VMM

```text
ACI microsegmentation (uSeg) with VMM:
  - uSeg EPG: dynamic membership based on VM attributes
  - Attributes: OS, vNIC MAC, IP, custom tag, vCenter tag
  - Example: all Windows VMs in "DB-uSeg-EPG" regardless of port group
  - Contracts apply to uSeg EPG for micro-level policy
  - Requires: VMware Tools installed, APIC can query vCenter

Configuration:
  APIC GUI: Tenant > AP > EPG > Create uSeg EPG
    - Attribute: OS = Windows
    - Attribute: vCenter Tag = "tier-database"
  Associate uSeg EPG with VMM domain
```

---

## VMware and VXLAN: NSX-T vs ACI

### NSX-T Overlay Architecture

```text
+------------------+
|  NSX Manager     |  (Control plane, policy)
+--------+---------+
         |
+--------+---------+---------+---------+
| ESXi 1 | ESXi 2 | ESXi 3 | ESXi 4 |
| N-VDS  | N-VDS  | N-VDS  | N-VDS  |
| TEP IP | TEP IP | TEP IP | TEP IP |
+--------+---------+---------+---------+
         |
    Physical Underlay (VXLAN transport)
    (Any L3 network - ACI, traditional, etc.)

NSX-T components:
  - Segment: L2 broadcast domain (VXLAN VNI)
  - Tier-1 gateway: distributed router (per-host)
  - Tier-0 gateway: north-south routing (BGP/OSPF to physical)
  - Distributed Firewall: L4-L7 per-VM
  - Load Balancer: distributed or service-based
```

### ACI Overlay Architecture (Comparison)

```text
+------------------+
|  APIC Cluster    |  (Policy, not in data path)
+--------+---------+
         |
+--------+---------+---------+---------+
| Leaf 1 | Leaf 2 | Leaf 3 | Leaf 4 |
| VTEP   | VTEP   | VTEP   | VTEP   |
+--------+---------+---------+---------+
         |
    Spine (IS-IS underlay, COOP database)

ACI components:
  - BD: L2/L3 domain (VXLAN VNID)
  - EPG: policy group (endpoints)
  - Contract: policy between EPGs
  - COOP: endpoint database on spines
  - vzAny: default gateway (anycast)
```

### Key Differences

| Feature | NSX-T | ACI |
|---------|-------|-----|
| Control plane | NSX Manager (centralized) | APIC + COOP (distributed) |
| Data plane | ESXi N-VDS/VDS | ACI Leaf (hardware VTEP) |
| Policy model | Groups + DFW rules | EPG + Contracts |
| Underlay | Any L3 (must provide VXLAN transport) | IS-IS (built-in) |
| Microsegmentation | Distributed Firewall | Contracts + uSeg EPG |
| Routing | Tier-0/1 (distributed) | BD subnet + L3Out |
| Automation | NSX API, Terraform | APIC REST, Ansible |

> **CCIE Exam Tip:** Know that NSX and ACI can coexist but NOT overlay on the same host simultaneously. In hybrid designs, NSX may handle microsegmentation while ACI handles underlay/overlay networking. The exam tests understanding of both architectures.

---

## vSphere Storage

### VMFS (Virtual Machine File System)

```text
VMFS characteristics:
  - Cluster file system (multiple hosts access same datastore)
  - VMFS-6 (vSphere 6.5+): supports 4K sectors, auto-UNMAP
  - Block-level locking (VMkernel handles concurrent access)
  - Presented via FC, FCoE, or iSCSI LUNs
  - Datastore: formatted LUN visible to all cluster hosts

VMFS on FC/FCoE:
  ESXi vmhba (FC HBA or CNA) -> MDS/Nexus -> Storage Array
  - Requires: zoning (ESXi WWPN to storage target)
  - Requires: LUN masking (storage presents LUN to ESXi WWPN)
  - Multipathing: NMP (Native Multipathing Plugin) with PSP
```

### NFS Datastores

```text
NFS on ESXi:
  - NFS v3 or v4.1
  - VMkernel port with NFS traffic enabled
  - No zoning/LUN masking needed (IP-based)
  - Requires: VMkernel port on storage VLAN, route to NFS server
  - Use case: simpler than FC, good for smaller environments

ESXi NFS config:
  esxcli storage nfs add -H 10.1.1.100 -s /export/datastore1 -v NFS-DS1
```

### vVols (Virtual Volumes)

```text
vVols:
  - Storage array manages individual VM disks (not LUNs)
  - VASA provider: storage array exposes capabilities to vCenter
  - Storage Policy Based Management (SPBM): VM gets storage per policy
  - Protocol endpoint: single LUN/path for management traffic
  - Data path: direct from ESXi to storage (bypasses protocol endpoint)
  - Requires: VASA 2.0+ compliant array, vCenter, VMFS not used

vVols vs VMFS:
  - VMFS: LUN-level management, all VMs on LUN share fate
  - vVols: VM-level management, per-VM snapshots/clones on array
```

### iSCSI and FC from ESXi

```text
iSCSI from ESXi:
  - Software iSCSI initiator (vmk port) or hardware (iSCSI HBA)
  - Requires: VMkernel port, iSCSI target IP, CHAP (optional)
  - Multipathing: multiple vmk ports or multiple paths
  - Jumbo frames (MTU 9000) recommended

FC/FCoE from ESXi:
  - FC HBA (QLogic, Emulex) or CNA (VIC, converged)
  - Requires: zoning on MDS, LUN masking on array
  - FCoE: CNA connects to Nexus/MDS FCoE port
  - Multipathing: NMP with Round Robin or Fixed PSP

ESXi storage verification:
  esxcli storage core device list
  esxcli storage core path list
  esxcli storage nmp device list
  esxcli iscsi adapter discovery sendtarget list
```

---

## VMware Troubleshooting in DC Context

### Common Issues

```text
1. VM has no network connectivity:
   - Check port group VLAN matches ACI EPG encap
   - Check VDS uplink teaming (both uplinks active?)
   - Check ACI contract (is traffic permitted?)
   - Check ACI endpoint learning (is VM MAC in COOP?)
   - APIC: Tenant > EPG > Operational > Endpoints

2. vMotion fails:
   - Check VMkernel port for vMotion (IP, VLAN, MTU)
   - Check shared storage accessibility from both hosts
   - Check EVC mode (CPU compatibility)
   - Check network: same L2 domain for vMotion traffic

3. Storage connectivity lost:
   - Check FC: flogi database on MDS, zoning, LUN mapping
   - Check iSCSI: VMkernel port, target reachability, CHAP
   - Check multipathing: esxcli storage core path list
   - Check ACI: if FCoE over ACI, verify VSAN and FCoE VLAN

4. VDS port group missing:
   - Check APIC VMM domain status
   - Check vCenter connectivity from APIC
   - Check EPG association to VMM domain
   - APIC: Virtual Networking > VMM Domains > Faults
```

### Verification Commands

```text
ESXi:
  esxcli network vswitch dvs vmware list
  esxcli network vswitch dvs vmware portgroup list
  esxcli network ip interface list
  esxcli storage core device list
  esxcli storage core path list
  vim-cmd vmsvc/getallvms
  esxtop (interactive performance)

vCenter (PowerCLI):
  Get-VDSwitch
  Get-VDPortgroup
  Get-VM | Get-NetworkAdapter
  Get-Cluster | Get-DrsRecommendation
  Get-VMHost | Get-VMHostHba

ACI (for VMM):
  APIC GUI: Virtual Networking > VMM Domains > Operational
  APIC GUI: Tenant > EPG > Operational > Endpoints
  Leaf CLI: show endpoint database
  Leaf CLI: show lldp neighbors
```

---

## Container Networking Fundamentals

### Container vs VM Networking

```text
VM networking:
  VM -> vNIC -> Port Group -> VDS -> Physical NIC -> Network

Container networking:
  Container -> veth pair -> Container Network (bridge/overlay) -> Host -> Physical NIC -> Network

Key difference:
  - VMs have full virtual NIC with MAC, appear as endpoints
  - Containers share host network stack (Linux namespaces)
  - Container networking is more dynamic (pods created/destroyed rapidly)
  - CNI (Container Network Interface) standardizes plugin model
```

### CNI (Container Network Interface)

```text
CNI specification:
  - Standard API for container network plugins
  - Operations: ADD (attach network), DEL (detach), CHECK (verify)
  - Plugin types:
    - Bridge: local L2 bridge (docker0, cni0)
    - Overlay: VXLAN/Geneve across hosts (Flannel, Calico)
    - Macvlan: container gets direct L2 access
    - SR-IOV: container gets hardware VF
    - Host-local: IPAM (IP address management)

CNI config location: /etc/cni/net.d/
CNI binary location: /opt/cni/bin/
```

### Overlay Networks for Containers

```text
Container overlay (VXLAN-based):
  Host A                    Host B
  +--------+               +--------+
  | Pod 1  |               | Pod 2  |
  | 10.1.1.2|              | 10.1.1.3|
  +---+----+               +----+---+
      | veth                    | veth
  +---+----+               +----+---+
  | cni0   |               | cni0   |
  | bridge |               | bridge |
  +---+----+               +----+---+
      | VXLAN encap             | VXLAN encap
  +---+----+               +----+---+
  | eth0   |               | eth0   |
  | (host) |               | (host) |
  +---+----+               +----+---+
      |                         |
      +---- Physical Network ---+
           (underlay)

VXLAN VNI per container network (similar to ACI BD)
```

---

## Kubernetes Networking Model

### Pod Network

```text
Kubernetes networking requirements:
  1. Every pod gets a unique IP (flat network, no NAT between pods)
  2. Pods can communicate with all other pods without NAT
  3. Nodes can communicate with all pods without NAT
  4. Pod's own IP is what it sees (no translation)

Pod networking:
  +---------------------------+
  | Pod                       |
  | +-------+  +-------+     |
  | |Container| |Container|   |  (shared network namespace)
  | |  A     |  |  B     |   |
  | +---+---+  +---+---+     |
  |     |          |         |
  |     +----+-----+         |
  |          |               |
  |     +----+----+          |
  |     |  eth0   |          |  (pod IP: 10.244.1.5)
  |     +----+----+          |
  +----------|---------------+
             | veth pair
  +----------|---------------+
  |     +----+----+          |
  |     |  cni0   |          |  (host bridge)
  |     +----+----+          |
  |          |               |
  |     +----+----+          |
  |     |  eth0   |          |  (host IP: 192.168.1.10)
  |     +---------+          |
  |     Host / Node          |
  +--------------------------+
```

### Services

```text
Kubernetes Service types:
  - ClusterIP: internal virtual IP (kube-proxy iptables/IPVS)
  - NodePort: expose on all nodes at static port (30000-32767)
  - LoadBalancer: cloud LB or external LB integration
  - ExternalName: DNS CNAME to external service

Service networking:
  Client Pod -> ClusterIP (virtual) -> kube-proxy -> Backend Pod
  (iptables DNAT or IPVS load balancing)

Service CIDR: separate from Pod CIDR
  Example: Pod CIDR 10.244.0.0/16, Service CIDR 10.96.0.0/12
```

### Ingress

```text
Ingress:
  - L7 (HTTP/HTTPS) routing to services
  - Ingress Controller: nginx, traefik, HAProxy, Cisco
  - Ingress resource defines rules (host, path -> service)
  - External traffic -> Ingress Controller -> Service -> Pod

Example:
  api.example.com/v1 -> service-a (port 8080)
  api.example.com/v2 -> service-b (port 9090)
```

---

## Cisco Container Platform and ACI with Kubernetes

### ACI CNI Plugin

```text
ACI CNI (opflex-agent):
  - Integrates K8s pods into ACI policy model
  - Pods become ACI endpoints (learned by leaf)
  - K8s namespaces map to ACI EPGs
  - K8s NetworkPolicy maps to ACI Contracts
  - Pod IPs are in ACI BD subnet

Architecture:
  +------------------+
  |  APIC            |
  +--------+---------+
           |
  +--------+---------+---------+
  | Leaf 1 | Leaf 2 | Leaf 3 |
  +--------+---------+---------+
           |
  +--------+---------+---------+
  | K8s Node 1 | K8s Node 2 |
  | +--------+ | +--------+ |
  | |  Pods  | | |  Pods  | |
  | +--------+ | +--------+ |
  | opflex-agent| opflex-agent|
  +-------------+-------------+

Flow:
  1. Pod created -> CNI plugin called
  2. opflex-agent notifies APIC of new endpoint
  3. APIC assigns EPG (based on namespace/label)
  4. ACI leaf learns pod IP/MAC
  5. Contracts enforce policy between pods
```

### Cisco Container Platform (CCP)

```text
CCP (now part of Intersight Kubernetes Service):
  - Managed K8s platform on UCS/Intersight
  - Provides: cluster lifecycle, networking, storage, monitoring
  - Networking: ACI CNI or Calico
  - Storage: CSI drivers (Portworx, vSphere CSI, NetApp)
  - Load balancing: MetalLB or ACI L4-L7 services
  - Know for exam: CCP integrates UCS compute + ACI network + K8s
```

---

## CNI Plugins: Calico, Flannel, Cilium

### Calico

```text
Calico:
  - Pure L3 networking (no overlay by default)
  - BGP routing between nodes (each node advertises pod CIDR)
  - NetworkPolicy enforcement via iptables/eBPF
  - Can use VXLAN overlay (if BGP not available)
  - Felix agent: per-node, enforces policy
  - BIRD: BGP daemon for route distribution
  - Typha: scales Felix connections to datastore

Calico with ACI:
  - Calico BGP can peer with ACI leaf (L3Out)
  - Or: ACI CNI replaces Calico entirely
  - Hybrid: Calico for pod network, ACI for external connectivity
```

### Flannel

```text
Flannel:
  - Simple overlay network (VXLAN or host-gw)
  - VXLAN mode: encapsulates pod traffic in VXLAN between nodes
  - host-gw mode: L3 routing (requires L2 adjacency between nodes)
  - No built-in NetworkPolicy (often paired with Calico for policy)
  - flanneld agent per node
  - etcd or K8s API for subnet allocation

Flannel VXLAN:
  - VNI: 1 (default)
  - Each node gets /24 subnet from cluster CIDR
  - Pod traffic: Pod -> cni0 -> flannel.1 (VXLAN) -> remote node
```

### Cilium

```text
Cilium:
  - eBPF-based networking and security
  - No iptables (replaces with eBPF programs in kernel)
  - VXLAN or native routing (BGP)
  - L3/L4/L7 NetworkPolicy (HTTP, gRPC, DNS-aware)
  - Hubble: observability (flow visualization)
  - Service mesh capabilities (L7)
  - High performance (eBPF in-kernel processing)

Cilium with ACI:
  - Cilium VXLAN can run over ACI underlay
  - Or: ACI CNI for policy, Cilium for observability
  - Emerging: Cilium Cluster Mesh for multi-cluster
```

### CNI Comparison

| Feature | Calico | Flannel | Cilium | ACI CNI |
|---------|--------|---------|--------|---------|
| Data plane | L3/BGP or VXLAN | VXLAN or host-gw | eBPF + VXLAN/BGP | ACI hardware |
| Policy | NetworkPolicy | None (add Calico) | L7 NetworkPolicy | ACI Contracts |
| Performance | Good | Moderate | Excellent | Excellent (HW) |
| Complexity | Medium | Low | High | Medium |
| ACI integration | BGP peer | Overlay on ACI | Overlay on ACI | Native |

> **CCIE Exam Tip:** For the exam, know how containers connect to ACI EPGs. The key concept: ACI CNI (opflex) makes pods first-class ACI endpoints. Without ACI CNI, containers use overlay (Calico/Flannel) and connect to ACI via L3Out or external EPG.

---

## Containers on UCS/Intersight

### UCS as Container Infrastructure

```text
UCS for K8s:
  - UCS blades/rack servers as K8s worker nodes
  - Service Profiles for consistent node configuration
  - VIC provides vNICs for pod network and storage network
  - SR-IOV for high-performance pod networking
  - Intersight Kubernetes Service (IKS): managed K8s on UCS

Intersight Kubernetes Service:
  - Claim UCS domain in Intersight
  - Create K8s cluster profile (node count, version, network)
  - Intersight provisions: OS, K8s, CNI, CSI
  - Day-2: upgrade, scale, monitor via Intersight
  - Networking: ACI CNI or Calico
```

### Container Networking in VXLAN (ACI Context)

```text
How containers connect to ACI EPGs:

Option 1: ACI CNI (native integration)
  - Pod gets IP from ACI BD subnet
  - Pod MAC learned by ACI leaf
  - Pod is ACI endpoint in EPG
  - Contracts apply between pod EPGs
  - No additional overlay (ACI VXLAN is the overlay)

Option 2: Overlay CNI + ACI L3Out
  - Pod gets IP from CNI-managed CIDR (e.g., 10.244.0.0/16)
  - CNI creates VXLAN overlay between nodes
  - Node connects to ACI via L3Out (BGP/OSPF)
  - ACI learns pod CIDR as external route
  - External EPG represents container network in ACI
  - Contracts between external EPG and internal EPGs

Option 3: Macvlan/SR-IOV (bare-metal containers)
  - Pod gets direct L2 access (own MAC on physical network)
  - ACI leaf learns pod MAC directly
  - Static binding or dynamic learning
  - Highest performance, no overlay overhead
```

---

## Verification Commands

```text
VMware (ESXi):
  esxcli network vswitch dvs vmware list
  esxcli network vswitch dvs vmware portgroup list
  esxcli network ip interface list
  esxcli storage core device list
  esxcli storage core path list
  esxcli iscsi adapter discovery sendtargets list
  esxtop

VMware (vCenter PowerCLI):
  Get-VDSwitch | Get-VDPortgroup
  Get-VM | Get-NetworkAdapter | Select Name, NetworkName, MacAddress
  Get-Cluster | Get-DrsRecommendation
  Get-VMHost | Get-VMHostHba | Select Name, Status

Kubernetes:
  kubectl get pods -o wide
  kubectl get svc
  kubectl get endpoints
  kubectl get networkpolicy
  kubectl describe pod <name>
  kubectl exec -it <pod> -- ip addr
  kubectl exec -it <pod> -- ip route
  calicoctl get nodes
  calicoctl get ippool
  cilium status
  cilium endpoint list

ACI (for VMM/containers):
  APIC GUI: Virtual Networking > VMM Domains
  APIC GUI: Tenant > EPG > Operational > Endpoints
  Leaf: show endpoint database
  Leaf: show lldp neighbors
  Leaf: show ip arp vrf <vrf>
```

---

## Lab 1: VDS Configuration and ACI VMM Domain Verification

### Objective
Verify ACI VMM domain integration with VMware VDS and validate EPG-to-port-group mapping.

### Topology

```text
+----------+     +----------+
| ACI      |     | ACI      |
| Leaf 101 |     | Leaf 102 |
+----+-----+     +-----+----+
     |                  |
     +--------+---------+
              |
        +-----+-----+
        | vCenter   |
        | (VCSA)    |
        +-----+-----+
              |
     +--------+---------+
     |        |         |
+----+--+ +--+----+ +--+----+
| ESXi 1| | ESXi 2| | ESXi 3|
| (VDS) | | (VDS) | | (VDS) |
+-------+ +-------+ +-------+
```

### Step 1: Verify VMM Domain on APIC

```text
APIC GUI:
  Virtual Networking > VMM Domains > VMware > VMM-VC01
  - Status: Connected
  - vCenter: vc01.datacenter.local
  - DVS: ACI-DVS
  - Uplinks: uplink1, uplink2

APIC REST API:
  GET https://apic/api/node/class/vmmDomP.json
  GET https://apic/api/node/class/vmmRsAcc.json
```

### Step 2: Verify DVS and Port Groups

```text
ESXi CLI:
  esxcli network vswitch dvs vmware list
    Name: ACI-DVS
    VDS ID: 50 0a 09 81 82 83 84 85
    Uplinks: vmnic0, vmnic1
    MTU: 9000
    Port groups:
      Prod-Tenant|Web-AP|Web-EPG (VLAN 100)
      Prod-Tenant|DB-AP|DB-EPG (VLAN 200)
      Prod-Tenant|App-AP|App-EPG (VLAN 300)

PowerCLI:
  Get-VDSwitch -Name "ACI-DVS" | Get-VDPortgroup | Format-Table Name, VlanId
```

### Step 3: Verify Endpoint Learning

```text
ACI Leaf 101:
  leaf101# show endpoint database
  Flags: D - Dynamic, S - Static
  VLAN  Domain  MAC Address    IP Address     Interface
  100   Web-EPG 00:50:56:a1:b2:c3 10.1.1.10   Vmm-Domain:ACI-DVS:Web-EPG
  200   DB-EPG  00:50:56:a1:b2:c4 10.1.2.10   Vmm-Domain:ACI-DVS:DB-EPG

  leaf101# show lldp neighbors
  Port      Neighbor Device ID    Neighbor Port
  Eth1/33   esxi01.datacenter.local  vmnic0
  Eth1/34   esxi01.datacenter.local  vmnic1

APIC GUI:
  Tenant > Web-AP > Web-EPG > Operational > Endpoints
  - VM: web-vm-01, IP: 10.1.1.10, MAC: 00:50:56:a1:b2:c3
  - Learned via: VMM domain, LLDP
```

### Step 4: Verify Contract Enforcement

```text
APIC GUI:
  Tenant > Contracts > Standard > Web-to-DB
  - Subject: allow-sql
  - Filter: tcp-1433 (SQL Server)
  - Applied: Web-EPG (provider) -> DB-EPG (consumer)

Test from VM:
  web-vm-01# sqlcmd -S 10.1.2.10 -U sa -P password
  (Should succeed - contract permits TCP 1433)

  web-vm-01# ssh 10.1.2.10
  (Should fail - no contract for TCP 22)

Verify on leaf:
  leaf101# show zoning-rule
  Rule ID  Source     Dest       Filter    Action   Hit Count
  4096     Web-EPG    DB-EPG     tcp-1433  permit   1523
  4097     Web-EPG    DB-EPG     default   deny     42
```

---

## Lab 2: Kubernetes Networking with ACI CNI

### Objective
Deploy a Kubernetes cluster with ACI CNI and verify pod-to-EPG mapping and contract enforcement.

### Topology

```text
+----------+     +----------+
| ACI      |     | ACI      |
| Leaf 101 |     | Leaf 102 |
+----+-----+     +-----+----+
     |                  |
     +--------+---------+
              |
     +--------+---------+
     |        |         |
+----+--+ +--+----+ +--+----+
| K8s   | | K8s   | | K8s   |
| Node 1| | Node 2| | Node 3|
| (UCS) | | (UCS) | | (UCS) |
+-------+ +-------+ +-------+
```

### Step 1: Verify ACI CNI Deployment

```text
kubectl get pods -n kube-system -l app=aci-containers-controller
  NAME                                    READY   STATUS
  aci-containers-controller-5f8d9-x2k4j  1/1     Running

kubectl get pods -n kube-system -l app=opflex-agent
  NAME                    READY   STATUS    NODE
  opflex-agent-node1      1/1     Running   k8s-node-1
  opflex-agent-node2      1/1     Running   k8s-node-2
  opflex-agent-node3      1/1     Running   k8s-node-3
```

### Step 2: Deploy Application and Verify EPG Mapping

```text
kubectl create namespace web-app
kubectl label namespace web-app opflex.apic.io/epg=Web-EPG

kubectl run nginx --image=nginx --namespace=web-app
kubectl get pod nginx -n web-app -o wide
  NAME    READY   STATUS    IP           NODE
  nginx   1/1     Running   10.1.1.50    k8s-node-1

Verify on ACI:
  APIC GUI: Tenant > Prod-Tenant > Web-AP > Web-EPG > Operational > Endpoints
  - Pod: nginx, IP: 10.1.1.50, MAC: 0a:0b:0c:0d:0e:0f
  - Learned via: opflex (K8s node 1)

  leaf101# show endpoint database
  VLAN  Domain   MAC Address       IP Address    Interface
  100   Web-EPG  0a:0b:0c:0d:0e:0f 10.1.1.50    Eth1/33 (k8s-node-1)
```

### Step 3: Verify K8s NetworkPolicy as ACI Contract

```text
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-db
  namespace: web-app
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          opflex.apic.io/epg: DB-EPG
    ports:
    - protocol: TCP
      port: 5432

kubectl apply -f networkpolicy.yaml

Verify on APIC:
  Tenant > Contracts > Standard > web-app-to-db-app
  - Subject: allow-5432
  - Filter: tcp-5432
  - Status: Active

Test:
  kubectl exec -it nginx -n web-app -- curl -s http://db-svc:5432
  (Should succeed)

  kubectl exec -it nginx -n web-app -- curl -s http://db-svc:22
  (Should timeout - no contract)
```

### Step 4: Verify Service (ClusterIP) Connectivity

```text
kubectl expose deployment web --port=80 --target-port=8080 --type=ClusterIP -n web-app
kubectl get svc -n web-app
  NAME   TYPE        CLUSTER-IP    PORT(S)
  web    ClusterIP   10.96.45.12   80/TCP

kubectl exec -it nginx -n web-app -- curl -s http://10.96.45.12
  (Should return web page)

Verify kube-proxy:
  kubectl exec -it nginx -n web-app -- iptables -t nat -L -n | grep 10.96.45.12
  DNAT  tcp  --  0.0.0.0/0  10.96.45.12  to:10.1.1.50:8080
```

> **Lab Exam Warning:** When troubleshooting container networking in the exam, check in this order: (1) Pod IP assigned? (2) CNI agent running? (3) Node-to-node connectivity? (4) ACI endpoint learned? (5) Contract permits traffic? Most failures are at step 4 or 5.

---

## VDS Uplink Policies and LACP

### VDS Teaming and Uplink Policies

```text
VDS uplink teaming policies:
  - Route based on IP hash: load balance across active uplinks
  - Route based on MAC hash: pin VM to uplink by MAC
  - Route based on originating port: pin by DVPort
  - Explicit failover: active/standby (no load balancing)
  - Route based on physical NIC load: dynamic (vSphere 6+)

For ACI VMM domain:
  - ACI pushes: "Route based on IP hash" (for LACP) or "Explicit failover"
  - Must match ACI interface policy group (LACP or static)
  - Mismatch causes traffic loss (common exam issue)
```

### LACP on VDS

```text
LACP on VDS:
  - VDS supports LACP (802.3ad) for uplink aggregation
  - Requires: ACI interface policy group with LACP enabled
  - VDS LACP mode: Active (sends LACP PDUs)
  - ACI side: LACP policy (active/passive) must match

Configuration (ACI side):
  APIC GUI: Fabric > Access Policies > Interface Policies > LACP
    Name: LACP_ACTIVE
    Mode: active
    Min Links: 1
    Max Links: 16

  Interface Policy Group:
    Type: PC (port channel) or VPC
    LACP Policy: LACP_ACTIVE
    CDP: enabled
    LLDP: enabled

Configuration (VDS side - pushed by ACI):
  VDS > Uplink Group > Teaming: Route based on IP hash
  VDS > Uplink Group > LACP: Active, timeout: fast (1s)

Verification:
  ESXi: esxcli network vswitch dvs vmware list
    -> LACP: Enabled, Mode: Active
  ACI Leaf: show lacp neighbors
    -> Neighbor: ESXi host, Port: vmnic0/vmnic1, State: Active
```

> **CCIE Exam Tip:** LACP mismatch between VDS and ACI is a common exam scenario. If ESXi uplinks show "down" or no LACP adjacency: (1) Check ACI LACP policy mode (active vs passive), (2) Check VDS teaming policy matches (IP hash for LACP), (3) Check interface policy group type (PC/VPC, not individual), (4) Verify LLDP enabled (ACI needs LLDP to discover ESXi).

### ACI VMM: Static Binding vs Dynamic Binding

```text
Static Binding (for bare-metal or specific VMs):
  - EPG > Static Ports > select leaf/port + VLAN
  - Endpoint is bound to specific physical port
  - Used for: physical servers, network appliances, routers
  - No VMM domain required
  - MAC learned on specific port

Dynamic Binding (VMM domain):
  - EPG associated with VMM domain (VMware/KVM)
  - APIC creates port group on DVS automatically
  - VM connects to port group = joins EPG dynamically
  - Endpoint learned via LLDP/CDP + VMware Tools
  - VM can vMotion between hosts (EPG follows)

Comparison:
  +-------------------+-------------------+-------------------+
  | Feature           | Static Binding    | Dynamic (VMM)     |
  +-------------------+-------------------+-------------------+
  | Binding target    | Leaf/port/VLAN    | VMM domain/DVS    |
  | Endpoint type     | Physical server   | VM                |
  | Mobility          | Fixed             | vMotion-aware     |
  | VLAN assignment   | Manual            | APIC-managed      |
  | Discovery         | MAC learning      | LLDP + vCenter    |
  | Port group        | N/A               | Auto-created      |
  | Use case          | Bare-metal, L3Out | Virtualized       |
  +-------------------+-------------------+-------------------+

Exam scenario: "VM not in correct EPG"
  - If static binding: check port/VLAN matches
  - If dynamic (VMM): check VMM domain status, LLDP, vCenter
```

---

## Common Exam Scenarios

### Scenario 1: VDS Port Group Missing After EPG Creation

```text
Ticket: "Created EPG but port group not visible in vCenter"

Diagnosis:
  APIC: Virtual Networking > VMM Domains > VMM-VC01
  -> Status: Disconnected (FAULT)

  APIC: VMM Domain > Faults
  -> "Cannot connect to vCenter: authentication failed"

Root cause: vCenter password changed, APIC credentials stale

Fix:
  1. APIC: VMM Domain > vCenter Credentials > Update password
  2. Wait for APIC to reconnect (30-60 seconds)
  3. Verify port groups appear in vCenter

Verification:
  vCenter: Networking > ACI-DVS > Port Groups
  -> Prod-Tenant|Web-AP|Web-EPG (present)
```

### Scenario 2: vMotion Fails Between Hosts

```text
Ticket: "vMotion from ESXi-1 to ESXi-2 fails with network error"

Diagnosis:
  1. Check VMkernel port for vMotion:
     esxcli network ip interface list
     -> vmk1: vMotion enabled, VLAN 900, IP 192.168.90.11

  2. Check destination host vMotion port:
     -> vmk1: vMotion enabled, VLAN 900, IP 192.168.90.12

  3. Check ACI: is VLAN 900 in same BD on both leafs?
     APIC: Tenant > BD > vMotion_BD
     -> Subnet: 192.168.90.0/24 (OK)

  4. Check contract: is vMotion traffic permitted?
     -> No contract between ESXi-EPG and ESXi-EPG (same EPG = OK)

  5. Check MTU:
     esxcli network nic get -n vmnic0
     -> MTU: 1500 (PROBLEM - should be 9000 for VXLAN)

Root cause: MTU mismatch (1500 vs 9000 required for VXLAN)

Fix:
  1. ACI: Interface Policy > MTU: 9000
  2. VDS: MTU: 9000 (pushed by ACI)
  3. Verify: esxcli network vswitch dvs vmware list -> MTU: 9000
```

### Scenario 3: Container Pod Cannot Reach External Network

```text
Ticket: "K8s pods with ACI CNI cannot reach internet"

Diagnosis:
  1. kubectl exec -it pod -- ip route
     -> default via 10.1.1.1 (ACI BD gateway) - OK

  2. kubectl exec -it pod -- ping 10.1.1.1
     -> Success (gateway reachable)

  3. kubectl exec -it pod -- ping 8.8.8.8
     -> Timeout (no external route)

  4. ACI: Tenant > L3Out > EXT_L3OUT
     -> BGP peer: Idle (PROBLEM)

  5. Leaf103: show ip bgp neighbors 10.100.1.2
     -> State: Active (TCP connecting, not established)

  6. Leaf103: show ip interface eth1/49
     -> IP: 10.100.1.1/30, VRF: PROD_VRF - OK

  7. External router: show ip bgp neighbors 10.100.1.1
     -> "AS mismatch: expected 65001, received 65002"

Root cause: BGP AS number mismatch on external router

Fix:
  1. External router: neighbor 10.100.1.1 remote-as 65001
  2. Verify BGP Established
  3. Verify default route received in PROD_VRF
  4. Verify contract between Pod-EPG and External-EPG exists
```

---

## Key Takeaways

1. **VDS is central**: ACI VMM domain creates and manages VDS; EPGs map to port groups automatically
2. **NSX vs ACI**: Both use VXLAN but different control planes; cannot overlay simultaneously on same host
3. **vSphere storage**: VMFS (FC/FCoE/iSCSI), NFS, vVols — know multipathing and zoning requirements
4. **K8s networking model**: Flat pod network, no NAT, CNI plugins provide implementation
5. **ACI CNI (opflex)**: Makes pods first-class ACI endpoints; NetworkPolicy maps to Contracts
6. **CNI plugins**: Calico (BGP/policy), Flannel (simple overlay), Cilium (eBPF/L7) — know trade-offs
7. **Containers on UCS**: Service Profiles for consistent K8s nodes; Intersight Kubernetes Service for managed lifecycle
8. **Troubleshooting**: VMM domain status -> DVS port groups -> endpoint learning -> contract enforcement
9. **LACP on VDS**: Must match ACI LACP policy; teaming = IP hash for LACP; LLDP required for discovery
10. **Static vs Dynamic binding**: Static = port/VLAN (bare-metal); Dynamic = VMM domain (VMs, vMotion-aware)
