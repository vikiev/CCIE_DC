# CCIE Data Center v3.1 - Complete Lab Exam Mastery

> **This curriculum is built for ONE purpose: passing the CCIE DC v3.1 lab exam (8-hour practical).**
> Every topic, every lab, every tip is oriented toward the 38000+ CCIE DC standard.

---

## CCIE DC v3.1 Blueprint Breakdown

The CCIE Data Center v3.1 lab exam tests six domains. You must be competent in ALL six.
There is no "optional" domain. Weakness in any area can cost you the exam.

### Domain Weights

| # | Domain | Weight | Key Topics |
|---|--------|--------|------------|
| 1 | **Network** | **30%** | L2 protocols (vPC, STP, LACP), L3 protocols (OSPF, BGP, IS-IS), VXLAN/EVPN, Multicast, QoS, Security, Segment Routing |
| 2 | **Compute** | **15%** | UCS B/C/X-Series, VMware vSphere, Containers/Kubernetes, Cisco Intersight |
| 3 | **Storage Networking** | **15%** | Fibre Channel, FCoE, NVMe-oF, iSCSI, MDS 9000, VSAN, Zoning |
| 4 | **Automation & Programmability** | **15%** | Python (NX-OS), Ansible, Terraform, NX-OS REST/NETCONF/gRPC, ACI API, NDFC |
| 5 | **ACI** | **15%** | Fabric setup, Tenants/AP/EPG/BD, Contracts, L3Out, VMM integration, Multi-site, Service Graph |
| 6 | **Assurance & Operations** | **10%** | Telemetry, NDFC/DCNM, Monitoring, Troubleshooting methodology |

### Domain 1: Network (30%) - Detailed Breakdown

This is the LARGEST domain and the foundation of the entire exam.

| Sub-topic | Weight within Domain | Details |
|-----------|---------------------|---------|
| L2 Protocols | 20% | vPC, STP (PVST+/Rapid-PVST+/MST), LACP, Port Channels, FEX, FabricPath concepts |
| L3 Protocols | 20% | OSPF (v2/v3), BGP (EVPN, RR), IS-IS, FHRP (HSRP/VRRP), PBR, VRF-Lite |
| VXLAN/EVPN | 25% | VXLAN encap, EVPN RT1-5, Symmetric/Asymmetric IRB, L3VNI, Multi-site |
| Multicast | 10% | IGMP, PIM (SM/SSM/ASM), RP mechanisms, VXLAN BUM, EVPN BUM |
| QoS | 10% | PFC, ECN/WRED, CoS/DSCP, 8-class model, FCoE no-drop, RoCEv2 |
| Security | 10% | CoPP, DAI, DHCP snooping, 802.1X, RBAC, AAA, VACL/PACL/RACL |
| Segment Routing | 5% | SR-MPLS, SR-TE, SRv6 concepts, IGP extensions |

### Domain 2: Compute (15%) - Detailed Breakdown

| Sub-topic | Details |
|-----------|---------|
| UCS B-Series | Blade chassis, IOM, service profiles, templates, firmware policies |
| UCS C-Series | Rack servers, CIMC, integration with UCS Manager |
| UCS X-Series | X9508 chassis, X-Fabric, X-Series compute nodes |
| VMware vSphere | vCenter, ESXi, vDS, VM networking, storage policies |
| Containers/K8s | Docker, Kubernetes, CNI, ACI-K8s integration |
| Cisco Intersight | Cloud management, policies, profiles, firmware |

### Domain 3: Storage Networking (15%) - Detailed Breakdown

| Sub-topic | Details |
|-----------|---------|
| Fibre Channel | FC framing, FC-ID, FLOGI/PLOGI/PRLI, N_Port/F_Port/E_Port |
| FCoE | VN2VN, FIP, CNA, lossless Ethernet (PFC), FCoE VLAN |
| NVMe-oF | NVMe over Fabrics, NVMe/FC, NVMe/RoCE, NVMe/TCP |
| iSCSI | iSCSI naming (IQN/EUI), CHAP, MDS iSCSI config |
| MDS 9000 | NX-OS for MDS, VSAN, zoning (hard/soft), port channels |
| VSAN | VSAN isolation, IVR, trunking, FSPF |
| Zoning | Zone sets, zones, members (pWWN, FC-ID), smart zoning, TCAM |

### Domain 4: Automation & Programmability (15%) - Detailed Breakdown

| Sub-topic | Details |
|-----------|---------|
| Python for NX-OS | NX-API, RESTCONF, model-driven programmability |
| Ansible | NX-OS modules, ACI modules, playbooks, inventory |
| Terraform | ACI provider, NX-OS provider, state management |
| NX-OS APIs | REST API, NETCONF/YANG, gRPC/gNMI, NX-API CLI |
| ACI API | REST API, APIC object model, MO hierarchy, AAA |
| NDFC | REST API, fabric discovery, policy management |

### Domain 5: ACI (15%) - Detailed Breakdown

| Sub-topic | Details |
|-----------|---------|
| Fabric Setup | APIC cluster, leaf/spine discovery, fabric policies |
| Policy Model | Tenants, VRF, BD, AP, EPG, contracts, filters |
| L3Out | Routed domains, external EPG, BGP/OSPF L3Out |
| VMM Integration | VMware VMM, VDS, EPG-to-port-group mapping |
| Multi-site | MSO, stretched tenants, inter-site L3Out |
| Service Graph | L4-L7 device integration, PBR, one-arm/two-arm |

### Domain 6: Assurance & Operations (10%) - Detailed Breakdown

| Sub-topic | Details |
|-----------|---------|
| Telemetry | Streaming telemetry, gRPC dial-in/out, sensors, subscriptions |
| NDFC/DCNM | Fabric management, discovery, compliance, reporting |
| Monitoring | SNMP, syslog, NetFlow/IPFIX, ERSPAN, health scores |
| Troubleshooting | Methodology, show commands, packet captures, log analysis |

---

## Repository Map

This curriculum integrates with existing GitHub repositories. Each repo contains
detailed lessons, labs, and configurations.

| Blueprint Topic | Repository / File | Status |
|----------------|-------------------|--------|
| Multicast (IGMP, PIM, VXLAN BUM) | [github.com/vikiev/Multicast](https://github.com/vikiev/Multicast) (9 chapters) | Complete |
| OSPF for DC | [github.com/vikiev/OSPF](https://github.com/vikiev/OSPF) | Complete |
| BGP for DC | [github.com/vikiev/BGP](https://github.com/vikiev/BGP) | Complete |
| VXLAN Lessons | [github.com/vikiev/Vxlan](https://github.com/vikiev/Vxlan) | Complete |
| ACI for CCIE DC | [github.com/vikiev/aci-ccie-dc](https://github.com/vikiev/aci-ccie-dc) | Complete |
| L2 Protocols (vPC, STP, LACP) | `01-network/l2-protocols.md` | This curriculum |
| L3 Protocols (OSPF, BGP, IS-IS) | `01-network/l3-protocols.md` | This curriculum |
| VXLAN/EVPN Deep Dive | `01-network/vxlan-evpn.md` | This curriculum |
| QoS and Security | `01-network/qos-security.md` | This curriculum |
| Segment Routing | `01-network/segment-routing.md` | This curriculum |
| UCS, VMware, Containers | `02-compute/ucs.md`, `02-compute/vmware-containers.md` | Exists |
| Fibre Channel, NVMe, iSCSI | `03-storage/fibre-channel.md`, `03-storage/nvme-iscsi.md` | Exists |
| Python, Ansible, Terraform | `04-automation/python-nxos.md`, `04-automation/ansible-terraform.md` | Exists |
| ACI Complete | `05-aci/aci-complete.md` | Exists |
| Telemetry, NDFC | `06-assurance/telemetry-monitoring.md`, `06-assurance/ndfc-fabric-management.md` | Exists |

---

## Gaps - What Still Needs to Be Created

The following topics are covered in this curriculum but may need additional lab files:

| Gap | Priority | Notes |
|-----|----------|-------|
| FEX configuration labs | Medium | Covered in l2-protocols.md but limited hands-on |
| FabricPath/TRILL deep dive | Low | Conceptual only - rarely tested in v3.1 |
| NVMe-oF hands-on labs | High | Needs MDS 9000 + NVMe-capable storage |
| SRv6 configuration | Medium | Nexus 9000 SRv6 support is evolving |
| ACI Multi-site full lab | High | Requires 2+ APIC clusters + MSO |
| Kubernetes CNI with ACI | Medium | Requires K8s cluster + ACI integration |
| NDFC REST API labs | Medium | Requires NDFC instance |
| Full 8-hour mock labs | Critical | Must build 3-4 complete mock scenarios |

---

## How to Use This Curriculum

### Step 1: Assess Your Baseline
- Take a practice diagnostic (any CCIE DC practice lab)
- Identify your weakest 2 domains
- Spend extra time on those domains in Phase 1

### Step 2: Follow the Study Plan
- Open `study-plan.md` and follow the 20-week schedule
- Each week has specific readings, labs, and milestones
- Do NOT skip labs. The exam is 100% hands-on.

### Step 3: Use the Repos
- Clone all 5 GitHub repos locally
- Work through each repo's labs in order
- Cross-reference with the curriculum files in `01-network/`

### Step 4: Build Speed
- The exam is 8 hours. You need to configure FAST.
- Practice typing configs from memory (no copy-paste in exam)
- Time yourself: each module should be completable in the allocated time

### Step 5: Troubleshoot Relentlessly
- 25% of the exam is troubleshooting
- Break your own labs and fix them
- Practice with show commands until they are muscle memory

### Step 6: Mock Labs
- Weeks 19-20: do full 8-hour mock labs
- Simulate exam conditions: no notes, no internet, timed
- Score yourself honestly

---

## Lab Environment Requirements

### Option A: Cisco CML (Cisco Modeling Labs)

| Component | Requirement |
|-----------|-------------|
| CML Version | 2.6+ |
| NX-OSv 9000 images | 10.3(x) or later (need 4+ images) |
| IOSv / IOSvL2 | For L2/L3 testing |
| RAM | 64 GB minimum (128 GB recommended) |
| CPU | 8+ cores |
| Storage | 200 GB SSD |

CML supports: NX-OSv 9000 (VXLAN, vPC, BGP EVPN), IOSv, IOSvL2
CML does NOT support: MDS 9000, UCS, ACI APIC, FEX

### Option B: EVE-NG

| Component | Requirement |
|-----------|-------------|
| EVE-NG Pro | 5.0+ (Community edition is limited) |
| NX-OSv 9000 | 10.3(x) images |
| vIOS / vIOSL2 | For routing/switching |
| RAM | 64 GB minimum |
| CPU | 8+ cores (16 recommended) |
| Storage | 500 GB SSD |

EVE-NG additionally supports: VMware ESXi (nested), some MDS emulation

### Option C: Physical Gear (Ideal but Expensive)

| Device | SKU | Qty | Purpose |
|--------|-----|-----|---------|
| Nexus 93180YC-FX | N9K-C93180YC-FX | 4 | Leaf switches (VXLAN, vPC) |
| Nexus 9364C | N9K-C9364C | 2 | Spine switches |
| Nexus 9336C-FX2 | N9K-C9336C-FX2 | 2 | Border leaf / multi-site |
| Nexus 3548 | N3K-C3548P-10G | 2 | Access / FEX parent |
| FEX 2400 | N2248TP-E-1GE | 2 | FEX labs |
| MDS 9148T | DS-C9148T-K9 | 2 | Storage / FC / FCoE |
| MDS 9706 | DS-C9706 | 1 | Director-class storage |
| UCS 6454 FI | UCS-FI-6454 | 2 | UCS fabric interconnects |
| UCS 5108 Chassis | UCS-B-5108-AC2 | 1 | Blade chassis |
| UCS B200 M6 | UCSB-B200-M6 | 2 | Blade servers |
| UCS C240 M6 | UCSC-C240-M6SX | 2 | Rack servers |
| APIC-L3 | APIC-L3 | 3 | ACI APIC controllers |
| Nexus 93180YC-FX (ACI) | N9K-C93180YC-FX | 4 | ACI leaf/spine |

### Software / Tools

| Tool | Purpose |
|------|---------|
| Wireshark | Packet analysis (VXLAN, EVPN, PIM) |
| Postman / curl | REST API testing |
| VS Code + Python | Automation labs |
| Ansible 2.14+ | Automation labs |
| Terraform 1.5+ | IaC labs |
| Git | Version control for configs |
| SecureCRT / PuTTY | Terminal access |
| Draw.io / Visio | Topology diagrams |

---

## Exam Day Strategy

### The 8-Hour Lab Breakdown

The CCIE DC lab exam is 8 hours, divided into 3 modules:

```
+------------------------------------------------------------------+
|                    8-HOUR LAB EXAM TIMELINE                       |
+------------------------------------------------------------------+
|                                                                    |
|  Hour 0:00 - 0:15   Read ALL modules, plan your approach          |
|  Hour 0:15 - 3:15   MODULE 1: DEPLOY (3 hours)                    |
|  Hour 3:15 - 3:30   Break / Review / Save configs                 |
|  Hour 3:30 - 6:30   MODULE 2: CONFIGURE (3 hours)                 |
|  Hour 6:30 - 6:40   Break / Review / Save configs                 |
|  Hour 6:40 - 8:00   MODULE 3: TROUBLESHOOT (1 hr 20 min)         |
|                                                                    |
+------------------------------------------------------------------+
```

### Module 1: Deploy (3 hours, ~35% of points)

- Build the infrastructure from scratch
- Typical tasks: fabric setup, VXLAN underlay, OSPF/BGP, vPC, VLANs, SVIs
- **Strategy**: Start with underlay routing, then overlay, then services
- **Pacing**: You should be 80% done at the 2:30 mark
- **Common tasks**:
  - Configure OSPF or BGP underlay on spine-leaf
  - Set up VXLAN with BGP EVPN
  - Configure vPC between leaf pairs
  - Create VLANs, VNIs, SVIs, L3VNIs
  - Set up FHRP (anycast gateway)

### Module 2: Configure (3 hours, ~40% of points)

- Add services and features on top of the deployed infrastructure
- Typical tasks: multicast, QoS, security, ACI, storage, automation
- **Strategy**: Do ACI and automation tasks first (they are time-boxed)
- **Pacing**: Allocate time per task based on point value
- **Common tasks**:
  - PIM multicast configuration
  - QoS policies (PFC, 8-class model)
  - CoPP, DAI, DHCP snooping
  - ACI tenant/AP/EPG/BD/contracts
  - Python/Ansible automation scripts
  - MDS zoning, VSAN configuration

### Module 3: Troubleshoot (1 hr 20 min, ~25% of points)

- Fix broken configurations across all domains
- **Strategy**: Read ALL trouble tickets first, then start with easiest
- **Methodology**:
  1. Read the ticket carefully (what is broken, what is expected)
  2. Check the obvious first (interface up/down, wrong VLAN, missing config)
  3. Use show commands systematically
  4. Verify your fix before moving on
  5. Do NOT break other things while fixing
- **Common breaks**:
  - Wrong VNI mapping
  - Missing BGP neighbor under EVPN AF
  - vPC consistency mismatch
  - Wrong PIM mode on interface
  - ACL blocking traffic
  - Wrong VRF assignment

### Exam Day Tips

1. **Save configs constantly**: `copy run start` after every major change
2. **Read the ENTIRE task before starting**: Don't assume, read every word
3. **Use the topology diagram**: Keep it open, refer to it constantly
4. **Don't get stuck**: If a task takes >20 min, skip it and come back
5. **Verify everything**: Don't assume it works, use show commands
6. **Manage your time**: Wear a watch, check time every 30 minutes
7. **Stay calm**: Everyone feels overwhelmed. Breathe. Execute.
8. **No copy-paste**: You must type all commands. Practice typing speed.
9. **Know your show commands**: `show nve`, `show bgp l2vpn evpn`, `show vpc`
10. **Double-check IP addresses**: One wrong octet = zero points for that task

---

## Table of Contents

### This Curriculum

| File | Topic | Lines |
|------|-------|-------|
| `README.md` | This file - curriculum overview | - |
| `study-plan.md` | 20-week study plan | - |
| `01-network/l2-protocols.md` | vPC, STP, LACP, FEX, FabricPath | 1000+ |
| `01-network/l3-protocols.md` | OSPF, BGP, IS-IS, FHRP, PBR, VRF | 1000+ |
| `01-network/vxlan-evpn.md` | VXLAN, EVPN, IRB, L3VNI, Multi-site | 1200+ |
| `01-network/multicast.md` | Multicast overview + repo links | 400+ |
| `01-network/qos-security.md` | QoS (PFC, ECN) + Security (CoPP, DAI) | 1000+ |
| `01-network/segment-routing.md` | SR-MPLS, SR-TE, SRv6 | 600+ |

### Existing Files (Do Not Overwrite)

| File | Topic |
|------|-------|
| `02-compute/ucs.md` | UCS B/C/X-Series |
| `02-compute/vmware-containers.md` | VMware vSphere, Containers, K8s |
| `03-storage/fibre-channel.md` | Fibre Channel, FCoE, MDS |
| `03-storage/nvme-iscsi.md` | NVMe-oF, iSCSI |
| `04-automation/python-nxos.md` | Python for NX-OS |
| `04-automation/ansible-terraform.md` | Ansible, Terraform |
| `05-aci/aci-complete.md` | ACI complete guide |
| `06-assurance/telemetry-monitoring.md` | Telemetry, monitoring |
| `06-assurance/ndfc-fabric-management.md` | NDFC/DCNM |

### GitHub Repositories

| Repo | Chapters | Topics |
|------|----------|--------|
| [Multicast](https://github.com/vikiev/Multicast) | 9 | IGMP, PIM, RP, VXLAN BUM, EVPN BUM, troubleshooting |
| [OSPF](https://github.com/vikiev/OSPF) | Multiple | OSPF for DC, areas, LSAs, underlay |
| [BGP](https://github.com/vikiev/BGP) | Multiple | BGP for DC, EVPN, RR, path selection |
| [Vxlan](https://github.com/vikiev/Vxlan) | Multiple | VXLAN fundamentals, config, troubleshooting |
| [ACI](https://github.com/vikiev/aci-ccie-dc) | Multiple | ACI policy model, L3Out, VMM, multi-site |

---

## Key Takeaway

> **CCIE Exam Tip:** The CCIE DC lab exam is not about knowing everything - it's about
> configuring it FAST and TROUBLESHOOTING it accurately. Speed comes from repetition.
> Accuracy comes from understanding. Build both.

> **Lab Exam Warning:** The #1 reason candidates fail is TIME MANAGEMENT, not lack of
> knowledge. Practice full 8-hour mocks under exam conditions. If you can't finish
> in 8 hours at home, you won't finish in the exam.

---

*Curriculum version: 1.1 | CCIE DC v3.1 | Last updated: 2026*
*Built for CCIE DC #38000+ candidates*
*Review: CCIE DC examiner-grade quality check applied*
