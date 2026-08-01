# CCIE DC v3.1 - 20-Week Intensive Study Plan

## Prerequisite Knowledge

- CCNP DC or equivalent (3+ years DC experience)
- Comfortable with NX-OS CLI, basic routing/switching
- Basic understanding of VXLAN, ACI concepts
- Python scripting basics (variables, loops, functions)
- Familiarity with Linux CLI

## Daily Schedule

| Time Block | Activity | Duration |
|------------|----------|----------|
| Morning (6:00-8:00) | Theory reading / video courses | 2 hours |
| Midday (12:00-13:00) | Flashcards / review previous day | 1 hour |
| Evening (18:00-22:00) | Hands-on labs | 4 hours |
| Weekend (8:00-16:00) | Extended labs / mock scenarios | 8 hours |

**Theory vs Lab Ratio: 30% theory / 70% lab**

- Weekdays: 3 hours theory + 4 hours lab = 7 hours/day
- Weekends: 1 hour theory + 7 hours lab = 8 hours/day
- Total: ~45 hours/week

> **CCIE Exam Tip:** If you can only do 4 hours/day, make ALL 4 hours lab.
> Theory without lab is worthless for the CCIE. You must build muscle memory.

---

## Phase 1: Network Fundamentals (Weeks 1-6)

### Week 1: L2 Protocols - STP and VLANs

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | 802.1Q trunking, native VLAN, VLAN design | Build 4-switch topology, trunk config |
| Tue | PVST+ BPDU format, port roles, port states | STP root election, manipulate priority |
| Wed | Rapid-PVST+ convergence, proposal/agreement | Rapid-PVST+ failover timing |
| Thu | MST regions, IST, CIST, MSTI | MST config, verify region match |
| Fri | STP in DC: why it blocks, suboptimal paths | Break STP: loop, root change, TCN |
| Sat | Review + flashcards | Full STP lab from scratch (timed) |
| Sun | Review | STP troubleshooting scenarios (3) |

**Milestone**: Configure and troubleshoot PVST+, Rapid-PVST+, MST from memory.

**Resources**:
- Cisco Press: "Cisco NX-OS and Cisco Data Center Switching" (Ch. 3-5)
- CBT Nuggets: NX-OS L2 course
- File: `01-network/l2-protocols.md` (STP sections)

### Week 2: vPC Deep Dive

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | vPC architecture: peer-link, peer-keepalive | Basic vPC config (2x N9K) |
| Tue | vPC operational rules, orphan ports | Orphan port traffic flow analysis |
| Wed | vPC + STP interaction, consistency checks | Type 1 and Type 2 mismatch labs |
| Thu | vPC failure scenarios, dual-active | Kill peer-link, observe behavior |
| Fri | vPC + VXLAN: anycast gateway | vPC with anycast SVI |
| Sat | Review | Full vPC build from scratch (timed: 45 min) |
| Sun | Review | vPC troubleshooting (3 scenarios) |

**Milestone**: Build vPC, break it, fix it. Explain every failure mode.

**Resources**:
- Cisco Press: "Cisco NX-OS and Cisco Data Center Switching" (Ch. 6)
- INE: CCIE DC vPC module
- File: `01-network/l2-protocols.md` (vPC sections)

### Week 3: LACP, Port Channels, FEX

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | LACP modes, system/port priority, LACP rate | LACP port-channel config |
| Tue | Static vs LACP, hash algorithms | Compare hash algorithms, verify |
| Wed | FEX architecture, pinning vs port-channel | FEX config (if hardware available) |
| Thu | FabricPath/TRILL concepts | Review FabricPath theory (no lab) |
| Fri | L2 review: STP + vPC + LACP combined | Combined L2 lab (all features) |
| Sat | Review | Timed L2 comprehensive lab |
| Sun | Review | L2 troubleshooting marathon |

**Milestone**: All L2 topics configured from memory in under 60 minutes.

### Week 4: OSPF for DC

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | OSPF areas, LSA types 1-7 | Multi-area OSPF config |
| Tue | NSSA, totally stubby, area types | NSSA + stub area labs |
| Wed | OSPFv3 (IPv6) | OSPFv3 config on Nexus |
| Thu | OSPF underlay for VXLAN (/31 links) | OSPF underlay on spine-leaf |
| Fri | OSPF in VRF, graceful restart, BFD | VRF OSPF + BFD config |
| Sat | Review | Full OSPF underlay build (timed) |
| Sun | Review | OSPF troubleshooting scenarios |

**Milestone**: OSPF underlay for VXLAN configured from memory.

**Resources**:
- GitHub: [vikiev/OSPF](https://github.com/vikiev/OSPF)
- Cisco Press: "OSPF Network Design Solutions"
- File: `01-network/l3-protocols.md` (OSPF sections)

### Week 5: BGP for DC

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | eBGP/iBGP, route reflectors | RR config on spine |
| Tue | BGP path selection (13 steps) | Manipulate path selection |
| Wed | BGP communities (standard, extended, large) | Community-based policy |
| Thu | BGP EVPN (AF l2vpn evpn) | BGP EVPN neighbor config |
| Fri | BGP underlay (unnumbered, listen-range) | Unnumbered BGP underlay |
| Sat | Review | Full BGP EVPN underlay build (timed) |
| Sun | Review | BGP troubleshooting scenarios |

**Milestone**: BGP EVPN underlay with unnumbered interfaces from memory.

**Resources**:
- GitHub: [vikiev/BGP](https://github.com/vikiev/BGP)
- Cisco Press: "BGP Design and Implementation"
- File: `01-network/l3-protocols.md` (BGP sections)

### Week 6: VXLAN/EVPN + Multicast

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | VXLAN encap, VNI, VTEP, MTU | Basic VXLAN config |
| Tue | EVPN RT1-5, RD/RT | EVPN route verification |
| Wed | Symmetric vs Asymmetric IRB | Symmetric IRB config |
| Thu | L3VNI, inter-VXLAN routing | L3VNI + VRF config |
| Fri | Multicast: IGMP, PIM SM, RP | PIM config on underlay |
| Sat | Review | Full VXLAN fabric (4 leaf, 2 spine) |
| Sun | Review | VXLAN + multicast troubleshooting |

**Milestone**: Complete VXLAN EVPN fabric with L3VNI from memory in 90 minutes.

**Resources**:
- GitHub: [vikiev/Vxlan](https://github.com/vikiev/Vxlan)
- GitHub: [vikiev/Multicast](https://github.com/vikiev/Multicast)
- File: `01-network/vxlan-evpn.md`
- File: `01-network/multicast.md`

### Phase 1 Milestone Check (End of Week 6)

You MUST be able to do the following from memory, in under 2 hours:
- [ ] Build a 4-leaf/2-spine topology
- [ ] Configure OSPF or BGP underlay
- [ ] Configure VXLAN with BGP EVPN
- [ ] Configure vPC on leaf pairs
- [ ] Configure VLANs, VNIs, SVIs, L3VNI
- [ ] Configure PIM multicast on underlay
- [ ] Verify end-to-end VXLAN connectivity
- [ ] Troubleshoot 3 common VXLAN issues

> **Lab Exam Warning:** If you cannot build a VXLAN fabric from memory in 90 minutes
> by end of Week 6, you are BEHIND. Add 1-2 extra weeks before moving to Phase 2.

---

## Phase 2: QoS, Security, Segment Routing (Weeks 7-9)

### Week 7: QoS for Data Center

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | DC QoS model, lossless Ethernet, PFC | PFC config on Nexus 9000 |
| Tue | CoS, DSCP, trust boundaries | QoS classification/marking |
| Wed | Network QoS, jumbo frames, pause | Network QoS policy |
| Thu | ECN/WRED, random-detect | WRED config |
| Fri | QoS for FCoE (no-drop), RoCEv2 | FCoE no-drop class |
| Sat | Review | 8-class QoS model full config |
| Sun | Review | QoS troubleshooting |

**Milestone**: 8-class QoS policy + PFC + FCoE no-drop from memory.

**Resources**:
- Cisco Press: "End-to-End QoS Network Design" (DC chapters)
- File: `01-network/qos-security.md` (QoS sections)

### Week 8: Security for Data Center

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | CoPP on Nexus 9000, default policy | CoPP custom config |
| Tue | DHCP snooping, DAI, IP Source Guard | DAI + DHCP snooping lab |
| Wed | PACL, VACL, RACL | VACL config and testing |
| Thu | 802.1X, MAB, EAP types | 802.1X config (if possible) |
| Fri | RBAC, AAA (TACACS+/RADIUS) | RBAC role creation |
| Sat | Review | Security comprehensive lab |
| Sun | Review | Security troubleshooting |

**Milestone**: CoPP + DAI + DHCP snooping + RBAC from memory.

**Resources**:
- Cisco Press: "Cisco Data Center Security"
- File: `01-network/qos-security.md` (Security sections)

### Week 9: Segment Routing + Phase 2 Review

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | SR concepts, SIDs, segment lists | SR concepts review |
| Tue | SR-MPLS: Prefix-SID, Adj-SID, Node-SID | OSPF SR config on Nexus |
| Wed | SR-TE: explicit paths, constraints | SR-TE policy config |
| Thu | SRv6 concepts, SRH, uSID | SRv6 theory review |
| Fri | SR vs LDP vs RSVP-TE comparison | Phase 2 comprehensive review |
| Sat | Review | Phase 2 timed lab (QoS + Security + SR) |
| Sun | Review | Phase 2 troubleshooting |

**Milestone**: SR concepts explained, basic OSPF SR configured.

**Resources**:
- Cisco Press: "Segment Routing for Dummies" (free Cisco whitepaper)
- File: `01-network/segment-routing.md`

### Phase 2 Milestone Check (End of Week 9)

- [ ] Configure 8-class QoS model with PFC
- [ ] Configure FCoE no-drop class
- [ ] Configure CoPP with custom rate limits
- [ ] Configure DAI + DHCP snooping
- [ ] Create custom RBAC role
- [ ] Explain SR-MPLS, SR-TE, SRv6 concepts
- [ ] Configure basic OSPF SR

---

## Phase 3: ACI (Weeks 10-12)

### Week 10: ACI Fabric and Policy Model

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | ACI architecture, APIC, spine-leaf | APIC GUI exploration |
| Tue | Tenants, VRF, BD, subnet config | Tenant + VRF + BD creation |
| Wed | AP, EPG, static/dynamic binding | EPG config with static ports |
| Thu | Contracts, filters, subjects | Contract between EPGs |
| Fri | BD settings: unicast routing, ARP, L2 unknown | BD tuning |
| Sat | Review | Full ACI tenant build (timed) |
| Sun | Review | ACI policy troubleshooting |

**Milestone**: Complete ACI tenant with 2 EPGs + contract from memory.

**Resources**:
- GitHub: [vikiev/aci-ccie-dc](https://github.com/vikiev/aci-ccie-dc)
- File: `05-aci/aci-complete.md`

### Week 11: ACI L3Out, VMM, Service Graph

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | L3Out: routed domain, external EPG | L3Out with BGP config |
| Tue | L3Out: OSPF, static routes | L3Out with OSPF |
| Wed | VMM integration: VMware VDS | VMM domain + EPG mapping |
| Thu | Service Graph: L4-L7, PBR | Service graph with firewall |
| Fri | Multi-Pod, multi-site concepts | Multi-site theory review |
| Sat | Review | ACI L3Out + VMM combined lab |
| Sun | Review | ACI troubleshooting scenarios |

**Milestone**: ACI L3Out (BGP) + VMM domain from memory.

### Week 12: ACI Advanced + Review

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | ACI multi-site (MSO), stretched BD | Multi-site theory |
| Tue | ACI REST API, object model | API calls with Postman |
| Wed | ACI + Ansible modules | Ansible ACI playbook |
| Thu | ACI troubleshooting methodology | ACI fault resolution |
| Fri | Phase 3 comprehensive review | Full ACI scenario lab |
| Sat | Review | Timed ACI build (90 min) |
| Sun | Review | ACI troubleshooting marathon |

**Milestone**: ACI fabric fully configured including L3Out, VMM, contracts in 90 min.

### Phase 3 Milestone Check (End of Week 12)

- [ ] Build ACI tenant with VRF, BD, AP, EPG
- [ ] Configure contracts with filters
- [ ] Configure L3Out with BGP and OSPF
- [ ] Configure VMM domain (VMware)
- [ ] Explain multi-site architecture
- [ ] Make ACI REST API calls
- [ ] Troubleshoot ACI connectivity issues

> **Lab Exam Warning:** ACI is 15% of the exam but candidates often spend too much time
> on it. Practice the GUI workflow until it's fast. Know where every button is.

---

## Phase 4: Compute (Weeks 13-14)

### Week 13: UCS

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | UCS architecture: FI, IOM, chassis | UCS Manager exploration |
| Tue | Service profiles, templates | Service profile creation |
| Wed | UCS B-Series vs C-Series vs X-Series | Compare architectures |
| Thu | Firmware policies, host firmware packages | Firmware management |
| Fri | UCS networking: vNIC templates, VLANs | vNIC template config |
| Sat | Review | UCS comprehensive lab |
| Sun | Review | UCS troubleshooting |

**Milestone**: UCS service profile from template, bound to blade.

**Resources**:
- File: `02-compute/ucs.md`
- Cisco Press: "Cisco UCS Handbook"

### Week 14: VMware, Containers, Intersight

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | VMware vSphere: vCenter, ESXi, vDS | vDS config (nested ESXi) |
| Tue | VM networking, port groups, SR-IOV | VM network config |
| Wed | Containers: Docker, K8s basics | Docker + K8s quick lab |
| Thu | ACI-K8s integration (CNI) | Theory review |
| Fri | Cisco Intersight: policies, profiles | Intersight exploration |
| Sat | Review | Phase 4 comprehensive review |
| Sun | Review | Phase 4 troubleshooting |

**Milestone**: Explain UCS, VMware, K8s, Intersight. Configure basic UCS + VMware.

### Phase 4 Milestone Check (End of Week 14)

- [ ] Create UCS service profile from template
- [ ] Explain UCS B/C/X-Series differences
- [ ] Configure VMware vDS
- [ ] Explain Kubernetes basics and ACI integration
- [ ] Navigate Cisco Intersight

---

## Phase 5: Storage (Weeks 15-16)

### Week 15: Fibre Channel and FCoE

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | FC fundamentals: framing, FC-ID, logins | FC concepts review |
| Tue | MDS 9000: VSAN, zoning | VSAN + zoning config |
| Wed | FCoE: FIP, CNA, lossless Ethernet | FCoE config on Nexus |
| Thu | Zoning: hard/soft, smart zoning, TCAM | Zoning scenarios |
| Fri | Port channels on MDS, FSPF | MDS port-channel config |
| Sat | Review | FC + FCoE comprehensive lab |
| Sun | Review | Storage troubleshooting |

**Milestone**: MDS VSAN + zoning + FCoE on Nexus from memory.

**Resources**:
- File: `03-storage/fibre-channel.md`
- Cisco Press: "Cisco MDS 9000 Series Configuration Guide"

### Week 16: NVMe-oF, iSCSI, Storage Review

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | NVMe-oF: NVMe/FC, NVMe/RoCE, NVMe/TCP | NVMe concepts review |
| Tue | iSCSI: IQN, CHAP, MDS config | iSCSI config on MDS |
| Wed | Storage networking design | Design exercise |
| Thu | Storage + Network integration | Combined storage/network lab |
| Fri | Phase 5 comprehensive review | Full storage lab (timed) |
| Sat | Review | Storage troubleshooting marathon |
| Sun | Review | Phase 5 assessment |

**Milestone**: Configure VSAN, zoning, FCoE, iSCSI. Explain NVMe-oF.

### Phase 5 Milestone Check (End of Week 16)

- [ ] Configure MDS VSAN and zoning
- [ ] Configure FCoE on Nexus 9000
- [ ] Configure iSCSI on MDS
- [ ] Explain NVMe-oF (FC, RoCE, TCP)
- [ ] Troubleshoot FC connectivity

> **Lab Exam Warning:** Storage is 15% of the exam. Many network-focused candidates
> neglect MDS/FC. Do NOT skip this. Zoning and VSAN config WILL appear.

---

## Phase 6: Automation (Weeks 17-18)

### Week 17: Python and NX-OS APIs

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | Python for NX-OS: NX-API, RESTCONF | Python + NX-API scripts |
| Tue | NETCONF/YANG on Nexus | NETCONF with ncclient |
| Wed | gRPC/gNMI telemetry | gRPC dial-in config |
| Thu | Python: requests, json, xmltodict | API automation scripts |
| Fri | NX-OS model-driven programmability | YANG model exploration |
| Sat | Review | Python automation lab (timed) |
| Sun | Review | API troubleshooting |

**Milestone**: Python script to configure VXLAN via REST API.

**Resources**:
- File: `04-automation/python-nxos.md`
- Cisco DevNet: "Network Programmability" learning path

### Week 18: Ansible, Terraform, NDFC

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | Ansible: NX-OS modules, inventory | Ansible NX-OS playbook |
| Tue | Ansible: ACI modules | Ansible ACI playbook |
| Wed | Terraform: ACI provider | Terraform ACI config |
| Thu | Terraform: NX-OS provider, state | Terraform NX-OS config |
| Fri | NDFC REST API, fabric management | NDFC API exploration |
| Sat | Review | Automation comprehensive lab |
| Sun | Review | Phase 6 assessment |

**Milestone**: Ansible playbook for VXLAN fabric + Terraform for ACI tenant.

### Phase 6 Milestone Check (End of Week 18)

- [ ] Write Python script using NX-OS REST API
- [ ] Use NETCONF to configure Nexus
- [ ] Write Ansible playbook for NX-OS
- [ ] Write Ansible playbook for ACI
- [ ] Write Terraform config for ACI
- [ ] Explain gRPC telemetry

---

## Phase 7: Assurance + Mock Labs (Weeks 19-20)

### Week 19: Assurance + First Mock Labs

| Day | Theory (2h) | Lab (4h) |
|-----|-------------|----------|
| Mon | Telemetry: streaming, gRPC, sensors | Telemetry config on Nexus |
| Tue | NDFC: discovery, compliance, reporting | NDFC exploration |
| Wed | Monitoring: SNMP, syslog, ERSPAN | Monitoring config |
| Thu | Troubleshooting methodology | Structured troubleshooting |
| Fri | **MOCK LAB 1** (full 8 hours) | Full exam simulation |
| Sat | **MOCK LAB 1** (continued) | Score + review |
| Sun | Review mock lab weaknesses | Targeted remediation |

**Milestone**: Complete first full 8-hour mock lab. Score yourself.

**Resources**:
- File: `06-assurance/telemetry-monitoring.md`
- File: `06-assurance/ndfc-fabric-management.md`

### Week 20: Final Mock Labs + Exam Prep

| Day | Activity | Duration |
|-----|----------|----------|
| Mon | **MOCK LAB 2** (full 8 hours) | 8 hours |
| Tue | Review Mock Lab 2, fix weak areas | 6 hours |
| Wed | **MOCK LAB 3** (full 8 hours) | 8 hours |
| Thu | Review Mock Lab 3, final weak areas | 6 hours |
| Fri | Light review: flashcards, show commands | 4 hours |
| Sat | Rest / light review only | 2 hours |
| Sun | **EXAM DAY** (or final prep if exam is Mon) | - |

**Milestone**: 3 full mock labs completed. All domains at passing level.

### Phase 7 Milestone Check (End of Week 20)

- [ ] Complete 3 full 8-hour mock labs
- [ ] Score 80%+ on each mock lab
- [ ] All show commands are muscle memory
- [ ] Can build VXLAN fabric in <90 minutes
- [ ] Can configure ACI tenant in <60 minutes
- [ ] Can troubleshoot any domain systematically
- [ ] Time management is solid (finish all 3 modules)

---

## Red Zone Topics (Common Failure Points)

These are the topics that cause the most exam failures. Master them.

### Red Zone 1: VXLAN Troubleshooting

**Why candidates fail**: They can configure VXLAN but can't troubleshoot it.

**You must know**:
- `show nve peers` - VTEP adjacency
- `show nve vni` - VNI status
- `show bgp l2vpn evpn route-type 2` - MAC/IP routes
- `show bgp l2vpn evpn route-type 3` - IMET routes
- Common breaks: NVE source-interface wrong, BGP EVPN AF missing, VNI mismatch
- MTU issues: underlay MTU must be overlay + 50 bytes
- Asymmetric routing blackhole: missing L3VNI or wrong VRF

**Practice**: Break your VXLAN lab in 5 different ways. Fix each one. Time yourself.

### Red Zone 2: ACI Contracts

**Why candidates fail**: Contract misconfiguration is the #1 ACI exam issue.

**You must know**:
- Contract scope: VRF, tenant, global, application-profile
- Filter vs subject vs contract hierarchy
- Provider/consumer EPG assignment
- Taboo contracts (deny all)
- vzAny (apply to all EPGs in VRF)
- Implicit deny: no contract = no traffic

**Practice**: Build 5 different contract scenarios. Break them. Fix them.

### Red Zone 3: FCoE

**Why candidates fail**: Network engineers don't know storage.

**You must know**:
- FCoE requires lossless Ethernet (PFC on FCoE VLAN)
- FIP: FCoE Initialization Protocol (discovery, login)
- VN2VN vs FIP snooping
- FCoE VLAN must be dedicated, no other traffic
- CNA: Converged Network Adapter
- FCoE on Nexus: `feature fcoe`, VLAN config, VSAN mapping

**Practice**: Configure FCoE end-to-end. Verify with `show fcoe`. Break PFC. Fix it.

### Red Zone 4: EVPN Route Types

**Why candidates fail**: They memorize config but don't understand the routes.

**You must know**:
- RT-2 (MAC/IP): which VTEP has which MAC + IP
- RT-3 (IMET): BUM flood list (who to replicate to)
- RT-5 (IP Prefix): L3 routes across VNIs
- How to read: `show bgp l2vpn evpn route-type 2`
- RD format: ASN:VNI or IP:VNI
- RT format: ASN:VNI (import/export)

**Practice**: Given a broken EVPN scenario, identify which route type is missing.

### Red Zone 5: vPC Failure Scenarios

**Why candidates fail**: They configure vPC but don't understand failure modes.

**You must know**:
- Peer-link down + keepalive up: secondary suspends all vPC ports
- Peer-link down + keepalive down: dual-active (split-brain)
- Peer-keepalive down + peer-link up: vPC still works (warning only)
- Orphan ports: traffic only on the switch where they connect
- vPC consistency check failure: port suspended

**Practice**: Kill peer-link. Kill keepalive. Kill both. Observe. Fix.

---

## Recommended Resources by Phase

### Phase 1 (Network)
| Resource | Type | Focus |
|----------|------|-------|
| Cisco Press: NX-OS and DC Switching | Book | L2, vPC, VXLAN |
| INE CCIE DC v3.1 | Video + Lab | All network topics |
| CBT Nuggets: Nexus 9000 | Video | NX-OS features |
| GitHub: vikiev/Vxlan | Lab | VXLAN hands-on |
| GitHub: vikiev/Multicast | Lab | Multicast hands-on |
| GitHub: vikiev/OSPF | Lab | OSPF hands-on |
| GitHub: vikiev/BGP | Lab | BGP hands-on |

### Phase 2 (QoS/Security/SR)
| Resource | Type | Focus |
|----------|------|-------|
| Cisco Press: End-to-End QoS | Book | QoS design |
| Cisco QoS configuration guide | Doc | NX-OS QoS config |
| Cisco CoPP configuration guide | Doc | CoPP on Nexus |
| Cisco SR whitepapers | Doc | SR concepts |

### Phase 3 (ACI)
| Resource | Type | Focus |
|----------|------|-------|
| GitHub: vikiev/aci-ccie-dc | Lab | ACI hands-on |
| Cisco ACI configuration guide | Doc | ACI reference |
| Virtronics ACI course | Video | ACI deep dive |
| Cisco ACI APIC REST API guide | Doc | API automation |

### Phase 4 (Compute)
| Resource | Type | Focus |
|----------|------|-------|
| Cisco UCS configuration guide | Doc | UCS reference |
| VMware vSphere documentation | Doc | VMware reference |
| Cisco Intersight documentation | Doc | Intersight |

### Phase 5 (Storage)
| Resource | Type | Focus |
|----------|------|-------|
| Cisco MDS 9000 configuration guide | Doc | MDS reference |
| Cisco FCoE configuration guide | Doc | FCoE on Nexus |
| Cisco NVMe-oF whitepaper | Doc | NVMe concepts |

### Phase 6 (Automation)
| Resource | Type | Focus |
|----------|------|-------|
| Cisco DevNet learning paths | Web | API fundamentals |
| Ansible NX-OS module docs | Doc | Ansible modules |
| Terraform ACI provider docs | Doc | Terraform ACI |
| Python for Network Engineers | Book | Python networking |

### Phase 7 (Assurance + Mocks)
| Resource | Type | Focus |
|----------|------|-------|
| INE CCIE DC mock labs | Lab | Exam simulation |
| Virtronics mock labs | Lab | Exam simulation |
| Cisco NDFC documentation | Doc | NDFC reference |

---

## Mock Lab Schedule

| Mock | Date | Focus | Duration |
|------|------|-------|----------|
| Mock 1 | Week 19, Fri-Sat | Full exam simulation | 8 hours |
| Mock 2 | Week 20, Mon | Full exam simulation | 8 hours |
| Mock 3 | Week 20, Wed | Full exam simulation | 8 hours |
| Mini-mock 1 | Week 12, Sat | Network + ACI only | 4 hours |
| Mini-mock 2 | Week 16, Sat | Storage + Compute | 4 hours |
| Mini-mock 3 | Week 18, Sat | Automation + Assurance | 4 hours |

### Mock Lab Rules

1. **No notes, no internet, no books** - exam conditions only
2. **Time yourself strictly** - 8 hours, hard stop
3. **Type all commands** - no copy-paste
4. **Score yourself honestly** - use the scoring rubric
5. **Review every mistake** - write down what you got wrong
6. **Repeat weak areas** - rebuild the broken parts from memory

### Scoring Rubric (Self-Assessment)

| Score | Meaning | Action |
|-------|---------|--------|
| 90%+ | Ready to pass | Maintain, light review |
| 80-89% | Likely pass | Fix weak areas, 1 more mock |
| 70-79% | Borderline | 2 more weeks of targeted study |
| 60-69% | Not ready | 4 more weeks, focus on Red Zone topics |
| <60% | Not ready | Re-evaluate study plan, consider delay |

> **CCIE Exam Tip:** The passing score is approximately 80%. If you're scoring 85%+
> on mock labs consistently, you're ready. If you're at 75%, you need more time.

---

## Rest Days and Recovery

### Built-In Rest Schedule

| Week | Rest Day | Activity |
|------|----------|----------|
| Every week | Sunday evening | NO study. Rest. Sleep. |
| Week 4 | Saturday afternoon | Half-day off (after morning lab) |
| Week 8 | Saturday afternoon | Half-day off (after morning lab) |
| Week 12 | Full Sunday | Complete rest day |
| Week 16 | Full Sunday | Complete rest day |
| Week 19 | Thursday | Light review only (2 hours max) |

> **CCIE Exam Tip:** Burnout is real. 45 hours/week for 20 weeks is sustainable ONLY
> with rest. If you skip rest days, your retention drops and your lab speed suffers.
> The exam tests SPEED under pressure - a tired brain is slow. Rest is not optional.

---

## "If You're Behind" Catch-Up Strategy

### Diagnostic: Are You Behind?

You are BEHIND if:
- You cannot complete the Phase milestone checklist by the end of the phase
- Your mock lab scores are below 70% at Week 19
- You are spending >50% of lab time reading notes instead of typing from memory
- You cannot build a VXLAN fabric in under 90 minutes by end of Week 6

### Catch-Up Plan A: 2 Weeks Behind

| Action | Time Impact |
|--------|-------------|
| Cut theory reading by 50% (lab only) | Save 10 hrs/week |
| Skip FabricPath, SRv6 deep dives (conceptual only) | Save 4 hrs |
| Combine Phase 4+5 weekends into single storage/compute block | Save 8 hrs |
| Add 1 extra mock lab at Week 20 | +8 hrs |
| Focus Red Zone topics during freed time | Targeted |

### Catch-Up Plan B: 4+ Weeks Behind

| Action | Time Impact |
|--------|-------------|
| Extend plan to 24 weeks (add 4 weeks) | +180 hrs |
| Weeks 21-22: Red Zone topic deep dive | VXLAN, ACI, FCoE |
| Weeks 23-24: Full mock labs (4 mocks) | +32 hrs |
| Consider delaying exam by 1 month | Mental reset |
| Hire a CCIE DC tutor for 2 sessions/week | Targeted help |

### Priority Order When Cutting Content

If you MUST cut topics, cut in this order (least to most exam-critical):

1. FabricPath/TRILL (conceptual only, never configured)
2. SRv6 (conceptual only)
3. SR P2MP (conceptual only)
4. GLBP (rarely tested, HSRP/VRRP more common)
5. 802.1X full config (MAB fallback is enough)
6. Terraform (Ansible is more commonly tested)
7. Intersight (conceptual is sufficient)

NEVER cut:
- VXLAN/EVPN (25% of network domain)
- vPC (most tested L2 topic)
- BGP EVPN (overlay control plane)
- PFC/QoS (storage depends on it)
- ACI contracts (15% of exam)
- MDS zoning (15% of exam)

> **Lab Exam Warning:** If you are 4+ weeks behind at Week 16, seriously consider
> delaying your exam. The CCIE DC pass rate is ~25% for first attempts. Going in
> underprepared wastes $1600+ and 6 months of your life. An extra month of study
> doubles your pass probability.

---

## Key Takeaway

> **The CCIE DC lab exam is a SPEED and ACCURACY test, not a knowledge test.**
> You already know enough if you're reading this. The question is: can you DO it
> in 8 hours, under pressure, with no notes? That requires 400+ hours of lab
> practice. This plan gives you that. Follow it. Trust it. Execute.

---

*Study plan version: 1.0 | CCIE DC v3.1 | 20-week intensive track*
*Total study hours: ~900 hours (45 hrs/week x 20 weeks)*
