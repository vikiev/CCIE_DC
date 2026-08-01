# Assurance and Operations for CCIE DC v3.1

## Prerequisite Knowledge

- NX-OS CLI fundamentals (show commands, configuration)
- Understanding of SNMP (v2c, v3, MIBs, traps)
- Basic understanding of streaming telemetry concepts
- VXLAN EVPN fabric knowledge (NVE, VNI, BGP EVPN)
- Understanding of SPAN/ERSPAN concepts
- Familiarity with syslog and logging
- Basic understanding of REST APIs and gRPC

---

## Model-Driven Telemetry (MDT) on NX-OS

### Telemetry Architecture

```text
Traditional (SNMP polling):
  +----------+    poll every 60s    +----------+
  |   NMS    |<-------------------->|  Switch  |
  | (pull)   |    GET/GETNEXT       | (agent)  |
  +----------+                      +----------+
  Problems: high latency, CPU load, incomplete data, 60s gaps

Model-Driven Telemetry (streaming):
  +----------+    continuous stream  +----------+
  |Collector |<---------------------|  Switch  |
  | (push)   |    gRPC/HTTP2        | (sensor) |
  +----------+                      +----------+
  Benefits: real-time, granular, low overhead, structured data
```

### gRPC Dial-In vs Dial-Out

```text
Dial-In (client initiates):
  +----------+                    +----------+
  | Collector|--- gRPC request -->|  Switch  |
  | (client) |<-- gRPC stream --- | (server) |
  +----------+                    +----------+
  - Collector connects to switch (port 50001)
  - Collector subscribes to sensor paths
  - Switch streams data back
  - Use case: on-demand queries, small scale

Dial-Out (switch initiates):
  +----------+                    +----------+
  | Collector|<-- gRPC stream --- |  Switch  |
  | (server) |                    | (client) |
  +----------+                    +----------+
  - Switch connects to collector
  - Subscription configured on switch
  - Switch pushes data at defined interval
  - Use case: production telemetry, large scale
  - Preferred for CCIE DC exam
```

### Sensor Paths and Subscription Profiles

```text
Sensor Path:
  - DME (Data Management Engine) path to data
  - Format: sys/<class>/<instance>/<property>
  - Examples:
    sys/intf/phys-[eth1/1]/rtSt          (interface state)
    sys/bgp/inst/dom-[default]/peer-[10.1.1.2]/operSt  (BGP peer)
    sys/ospf/inst/dom-[default]/ent-[10.1.1.1]/operSt  (OSPF)
    sys/nvo/items/Nvo-list[nve1]/peer-items             (NVE peers)
    sys/bd/items/BD-list[bd-10100]/fabEncap             (VXLAN BD)

Subscription Profile:
  - Combines: sensor-group + destination-group + sample-interval
  - sensor-group: what data to collect (sensor paths)
  - destination-group: where to send (collector IP, port, protocol)
  - sample-interval: how often (milliseconds)
  - heartbeat-interval: keep-alive (seconds)
```

### On-Change vs Periodic

```text
Periodic (sample-interval):
  - Data sent every N milliseconds regardless of change
  - Example: sample-interval 10000 = every 10 seconds
  - Use case: counters, utilization, trends
  - Generates constant stream (predictable bandwidth)

On-change:
  - Data sent ONLY when value changes
  - Example: interface up/down, BGP peer state change
  - Use case: state changes, events, alarms
  - Generates bursty stream (low bandwidth, immediate)
  - Not all sensor paths support on-change

Configuration:
  Periodic: sample-interval 10000
  On-change: sample-interval 0 (zero = on-change)
```

### NX-OS Telemetry Configuration

```nxos
feature telemetry

telemetry
  destination-group 1
    ip address 192.168.1.200 port 50001 protocol gRPC encoding GPB
    certificate /bootflash/telemetry.crt

  sensor-group 1
    path sys/intf depth 0 query-condition rsp-foreign-subtree=ephemeral
    path sys/bgp/inst/dom-[default]/peer depth 0

  sensor-group 2
    path sys/nvo/items/Nvo-list[nve1] depth 0
    path sys/bd depth 0

  sensor-group 3
    path sys/ospf/inst/dom-[default] depth 0
    path sys/isis/inst depth 0

  subscription 1
    snsr-grp 1 sample-interval 10000
    dst-grp 1

  subscription 2
    snsr-grp 2 sample-interval 5000
    dst-grp 1

  subscription 3
    snsr-grp 3 sample-interval 0
    dst-grp 1
```

### Telemetry for VXLAN

```text
VXLAN-specific sensor paths:
  NVE peers:
    sys/nvo/items/Nvo-list[nve1]/peer-items/Peer-list

  VNI status:
    sys/nvo/items/Nvo-list[nve1]/vni-items/Vni-list[vni-10100]

  BGP EVPN routes:
    sys/bgp/inst/dom-[default]/af-[l2vpn-evpn]/prefix-items

  Bridge Domain:
    sys/bd/items/BD-list[bd-10100]

  Endpoint database:
    sys/epm/items/ep-list

  VXLAN VTEP:
    sys/nvo/items/Nvo-list[nve1]/src-if-items

Configuration for VXLAN telemetry:
```

```nxos
telemetry
  sensor-group 10
    path sys/nvo/items/Nvo-list[nve1]/peer-items depth 0
    path sys/nvo/items/Nvo-list[nve1]/vni-items depth 0
    path sys/bgp/inst/dom-[default]/af-[l2vpn-evpn] depth 0
    path sys/bd depth 0

  subscription 10
    snsr-grp 10 sample-interval 5000
    dst-grp 1
```

### Streaming Telemetry vs SNMP

```text
+-------------------+-------------------+-------------------+
| Feature           | SNMP              | Streaming Telemetry|
+-------------------+-------------------+-------------------+
| Model             | Pull (poll)       | Push (stream)     |
| Granularity       | 60s typical       | 1s or on-change   |
| Data format       | OID/MIB (flat)    | YANG (structured) |
| CPU impact        | High (per poll)   | Low (streaming)   |
| Bandwidth         | Bursty            | Constant          |
| Scale             | Limited           | High              |
| Real-time         | No (60s delay)    | Yes (sub-second)  |
| Historical        | Requires NMS      | Time-series DB    |
| Automation        | Limited           | Full (API-driven) |
+-------------------+-------------------+-------------------+

Why telemetry wins:
  - Sub-second visibility (SNMP: 60s blind spots)
  - Structured data (JSON/protobuf vs flat OIDs)
  - No polling overhead (switch pushes, collector receives)
  - Scales to thousands of devices
  - Enables real-time analytics and AIOps
```

> **CCIE Exam Tip:** Know the telemetry configuration components: destination-group (where), sensor-group (what), subscription (binding + interval). Know that sample-interval 0 = on-change. Know that gRPC dial-out is preferred for production. The exam may ask you to configure or troubleshoot a telemetry subscription.

---

## NDFC (Nexus Dashboard Fabric Controller) / DCNM

### NDFC Architecture

```text
+------------------------------------------+
|         Nexus Dashboard                   |
|  +------------------------------------+  |
|  | NDFC (Fabric Controller)           |  |
|  | - Fabric discovery & management    |  |
|  | - Configuration templates          |  |
|  | - Compliance & drift detection     |  |
|  | - Firmware management              |  |
|  | - REST API                         |  |
|  +------------------------------------+  |
|  +------------------------------------+  |
|  | Orchestrator (Multi-site ACI)      |  |
|  +------------------------------------+  |
|  +------------------------------------+  |
|  | Insights (Analytics)               |  |
|  +------------------------------------+  |
+------------------------------------------+
         |
    +----+----+----+----+
    |    |    |    |    |
  +--+ +--+ +--+ +--+ +--+
  |S1| |S2| |L1| |L2| |L3|
  +--+ +--+ +--+ +--+ +--+
  (Managed NX-OS switches)
```

### Fabric Discovery and Topology

```text
NDFC discovery process:
  1. Add seed switch (management IP, credentials)
  2. NDFC discovers via CDP/LLDP from seed
  3. NDFC SSH to each discovered switch
  4. Collects: hostname, role, interfaces, neighbors
  5. Builds topology map
  6. Assigns roles: spine, leaf, border, super-spine

Topology visualization:
  - Physical topology (cable map)
  - Logical topology (VXLAN, VRF, network)
  - Health score per switch and per fabric
  - Alarm overlay (red/yellow/green)
```

### Configuration Management

```text
NDFC configuration:
  - Templates: predefined configs for VXLAN, L3, etc.
  - Policies: per-switch or per-fabric settings
  - Compliance: compare running vs intended config
  - Drift detection: alert when config changes outside NDFC
  - Remediation: push intended config to fix drift

NDFC for VXLAN:
  - Create fabric (VXLAN EVPN type)
  - Define: ASN, RP address, anycast gateway
  - Add switches (auto-discover or manual)
  - Assign roles (spine, leaf, border)
  - Create VRFs and networks (VLANs + VNIs)
  - Deploy: NDFC generates and pushes full config
```

### NDFC for ACI (Nexus Dashboard)

```text
Nexus Dashboard for ACI:
  - Orchestrator: multi-site ACI management
  - Replaces standalone MSO/NDO
  - Manages: multiple APIC clusters
  - Provides: stretched tenants, inter-site routing
  - Insights: analytics across ACI fabrics
  - Services: app hosting (ISE, Tetration, etc.)

Nexus Dashboard services:
  - Fabric Controller (NDFC): NX-OS fabrics
  - Orchestrator: ACI multi-site
  - Insights: telemetry analytics
  - Services: third-party apps
```

### NDFC Troubleshooting

```text
Fabric health:
  NDFC GUI: Fabric > Overview > Health Score
  - Green: all good
  - Yellow: warnings (non-critical)
  - Red: critical issues

Alarms and events:
  NDFC GUI: Fabric > Alarms
  - Severity: Critical, Major, Minor, Warning
  - Source: switch, interface, protocol
  - Action: acknowledge, clear, investigate

Common NDFC issues:
  - Switch unreachable: check mgmt IP, credentials, SSH
  - Config drift: someone changed config via CLI
  - Compliance failure: template mismatch
  - Discovery failure: CDP/LLDP disabled, wrong credentials
```

---

## NX-OS Monitoring

### SNMP Configuration

```nxos
feature snmp

snmp-server community RO_COMM ro
snmp-server community RW_COMM rw
snmp-server user admin network-admin auth md5 AuthP@ss1 priv aes-128 PrivP@ss1
snmp-server host 192.168.1.200 traps version 2c RO_COMM
snmp-server host 192.168.1.200 use-vrf management

snmp-server enable traps link
snmp-server enable traps bgp
snmp-server enable traps ospf
snmp-server enable traps vtp
snmp-server enable traps entity
snmp-server enable traps feature-control
```

### Syslog Configuration

```nxos
feature logging

logging server 192.168.1.200 severity informational
logging server 192.168.1.201 severity critical
logging source-interface mgmt0
logging timestamp milliseconds
logging monitor severity informational
logging console severity critical
```

### SPAN/ERSPAN

```text
SPAN (Switched Port Analyzer):
  - Local port mirroring (source -> destination on same switch)
  - Source: port, VLAN, or EtherChannel
  - Destination: local port (connected to analyzer)
  - Limitation: destination must be on same switch

ERSPAN (Encapsulated Remote SPAN):
  - Remote port mirroring (across IP network)
  - Source: port/VLAN on source switch
  - Destination: IP address (remote switch or analyzer)
  - Encapsulation: GRE (IP protocol 47)
  - Use case: central capture point, cross-switch monitoring
```

```nxos
monitor session 1
  type erspan-source
  source interface Ethernet1/1 both
  source interface Ethernet1/2 rx
  no shutdown

monitor session 1
  type erspan-source
  destination
    erspan-id 100
    ip address 192.168.1.200
    origin ip address 10.255.1.1
    no shutdown

monitor session 2
  type erspan-destination
  destination interface Ethernet1/48
  source
    erspan-id 100
    ip address 192.168.1.200
    no shutdown
```

### NetFlow/IPFIX

```nxos
feature netflow

flow record CUSTOM_RECORD
  match ipv4 source address
  match ipv4 destination address
  match transport source-port
  match transport destination-port
  match interface input
  collect counter bytes long
  collect counter packets long
  collect timestamp sys-uptime first
  collect timestamp sys-uptime last

flow exporter CUSTOM_EXPORTER
  destination 192.168.1.200
  transport udp 9995
  source-interface mgmt0
  template data timeout 60

flow monitor CUSTOM_MONITOR
  record CUSTOM_RECORD
  exporter CUSTOM_EXPORTER

interface Ethernet1/1
  ip flow monitor CUSTOM_MONITOR input
```

### Embedded Event Manager (EEM)

```text
EEM on NX-OS:
  - Event-driven automation (like IOS EEM)
  - Triggers: syslog, CLI, timer, interface, SNMP
  - Actions: CLI commands, syslog, SNMP trap, reload
  - Use case: auto-remediation, scheduled tasks, alerts
```

```nxos
event manager applet BGP_PEER_DOWN
  event syslog pattern ".*BGP-5-ADJCHANGE.*peer.*Down.*"
  action 1.0 cli command "enable"
  action 2.0 cli command "show ip bgp summary | include Down"
  action 3.0 syslog msg "EEM: BGP peer down detected"
  action 4.0 snmp-trap string "BGP_PEER_DOWN_ALERT"

event manager applet INTERFACE_FLAP
  event syslog pattern ".*ETHPORT-5-IF_DOWN_LINK_FAILURE.*"
  action 1.0 syslog msg "EEM: Interface flap detected"
  action 2.0 snmp-trap string "INTERFACE_FLAP_ALERT"

event manager applet DAILY_BACKUP
  event timer cron cron-entry "0 2 * * *"
  action 1.0 cli command "enable"
  action 2.0 cli command "copy running-config bootflash:backup_$(date).cfg"
```

### NX-OS Health Checks

```text
System health:
  show system resources
  show system uptime
  show environment temperature
  show environment power
  show environment fan
  show hardware
  show module

Interface health:
  show interface brief
  show interface counters errors
  show interface transceiver
  show interface status

Protocol health:
  show ip bgp summary
  show ip ospf neighbors
  show isis neighbors
  show nve peers
  show nve vni

Resource utilization:
  show hardware internal access-list resource tcam region ifacl
  show hardware capacity
  show system internal flash usage
  show process cpu
  show process memory
```

### Performance Monitoring

```text
Interface counters:
  show interface Ethernet1/1 counters
  show interface Ethernet1/1 counters rate
  show interface Ethernet1/1 counters errors

Buffer utilization:
  show hardware internal buffer info
  show platform hardware fed switch active qos queue stats interface Ethernet1/1

TCAM usage:
  show hardware internal access-list resource tcam region ifacl
  show hardware internal access-list resource tcam region vacl
  show hardware internal access-list resource tcam region racl

CPU/Memory:
  show process cpu
  show process memory
  show system resources
```

---

## Troubleshooting Methodology

### CCIE Lab Troubleshooting Approach

```text
Systematic approach (NOT random):
  1. READ the ticket carefully (what is broken, what is expected)
  2. IDENTIFY the layer (L1? L2? L3? Overlay? Policy?)
  3. VERIFY current state (show commands, not guesses)
  4. COMPARE to expected state (what should it be?)
  5. ISOLATE the failure point (where does it break?)
  6. FIX the specific issue (minimal change)
  7. VERIFY the fix (re-test, check counters)
  8. MOVE to next ticket

Time management:
  - 8 hours total, ~6-8 tickets
  - Budget: 45-60 minutes per ticket
  - If stuck > 15 min: move on, come back
  - Easy tickets first (build confidence, bank time)
  - Read ALL tickets before starting (dependencies)
```

### Layer-by-Layer Troubleshooting

```text
Layer 1 (Physical):
  show interface brief (admin/oper status)
  show interface transceiver (light levels, SFP)
  show interface counters errors (CRC, runts, giants)
  show cdp neighbors / show lldp neighbors
  Issues: bad SFP, wrong cable, speed mismatch, admin down

Layer 2 (Data Link):
  show vlan brief (VLAN exists, ports assigned)
  show spanning-tree (STP state, root bridge)
  show port-channel summary (LACP, members)
  show mac address-table (MAC learning)
  Issues: VLAN mismatch, STP blocking, port-channel misconfig

Layer 3 (Network):
  show ip interface brief (IP addresses)
  show ip route (routing table)
  show ip bgp summary (BGP peers)
  show ip ospf neighbors (OSPF adjacency)
  show isis neighbors (IS-IS adjacency)
  Issues: wrong IP, missing route, BGP down, ACL blocking

Overlay (VXLAN):
  show nve peers (VTEP peers)
  show nve vni (VNI status)
  show bgp l2vpn evpn summary (EVPN peers)
  show bgp l2vpn evpn route-type mac-ip (MAC/IP routes)
  show vlan vn-segment (VLAN-to-VNI mapping)
  Issues: NVE peer down, VNI not configured, EVPN route missing

Policy (ACI):
  show endpoint database (endpoint learned?)
  show zoning-rule (contract rules in TCAM)
  show ip arp vrf <vrf> (ARP in VRF)
  show ip route vrf <vrf> (routes in VRF)
  Issues: endpoint not learned, contract missing, VRF misconfig
```

### Key Debug Commands Per Layer

```text
Layer 1:
  debug interface Ethernet1/1
  show interface Ethernet1/1 transceiver detail

Layer 2:
  debug spanning-tree events
  debug lacp events
  debug vlan events

Layer 3:
  debug ip bgp updates
  debug ip ospf events
  debug ip routing
  debug isis adj-packets

Overlay:
  debug nve events
  debug bgp l2vpn evpn updates
  debug vxlan events

Policy:
  debug epm events
  debug zoning-rule events
```

### Common CCIE DC Lab Failure Scenarios

```text
Scenario 1: VXLAN traffic not flowing between leafs
  Check order:
    1. show nve peers (VTEP peers up?)
    2. show bgp l2vpn evpn summary (EVPN sessions up?)
    3. show vlan vn-segment (VNI configured on both leafs?)
    4. show bgp l2vpn evpn route-type mac-ip (MAC routes exchanged?)
    5. show interface nve1 (NVE interface up? source-interface correct?)
  Common fix: missing "send-community extended" under BGP neighbor

Scenario 2: ACI endpoint not reachable
  Check order:
    1. show endpoint database (endpoint learned on leaf?)
    2. show lldp neighbors (physical discovery?)
    3. show vlan (VLAN exists on port?)
    4. show zoning-rule (contract in TCAM?)
    5. show ip arp vrf <vrf> (ARP resolved?)
  Common fix: LLDP disabled on hypervisor, missing contract

Scenario 3: BGP EVPN routes not exchanged
  Check order:
    1. show ip bgp summary (TCP session up?)
    2. show bgp l2vpn evpn summary (AF activated?)
    3. show running-config | section "address-family l2vpn"
    4. show ip route <peer-loopback> (underlay reachability?)
  Common fix: missing "address-family l2vpn evpn" under neighbor

Scenario 4: FCoE not working
  Check order:
    1. show interface vfc (VFC up? bound?)
    2. show fcoe vlan (FCoE VLAN active?)
    3. show priority-flow-control interface (PFC enabled?)
    4. show flogi database (FLOGI successful?)
    5. show zoneset active (zoning correct?)
  Common fix: PFC not enabled end-to-end, MTU mismatch
```

### Time Management During Troubleshooting

```text
Strategy:
  - First 10 min: read ALL tickets, identify dependencies
  - Prioritize: easy/independent tickets first
  - Per ticket: 5 min diagnose, 5 min fix, 5 min verify
  - If stuck: note what you've checked, move on
  - Last 30 min: revisit stuck tickets, final verification
  - Never leave a ticket "partially fixed" (worse than broken)

Common time wasters:
  - Reload without understanding (wastes 5+ min)
  - Random config changes (breaks other things)
  - Not reading ticket carefully (fixes wrong thing)
  - Over-engineering (simple fix needed, complex solution attempted)
```

> **Lab Exam Warning:** The #1 mistake in the troubleshooting section is making changes before understanding the problem. Always: show -> understand -> fix -> verify. Never guess. If you change something and it doesn't help, UNDO it immediately before moving on.

---

## Verification Commands

```text
Telemetry:
  show telemetry subscription
  show telemetry destination-group
  show telemetry sensor-group
  show telemetry subscription detail
  show telemetry internal sensor-path
  show grpc

SNMP:
  show snmp community
  show snmp host
  show snmp user
  show snmp mib

Logging:
  show logging server
  show logging last 50
  show logging logfile

SPAN/ERSPAN:
  show monitor session
  show monitor session 1 detail
  show monitor capture

EEM:
  show event manager policy available
  show event manager policy registered
  show event manager history events

System:
  show system resources
  show environment
  show hardware
  show process cpu
  show process memory
  show hardware internal access-list resource tcam region ifacl
```

---

## Lab 1: gRPC Telemetry Configuration

### Objective
Configure streaming telemetry on NX-OS leaf switches to stream VXLAN and BGP data to a collector.

### Topology

```text
+----------+     +----------+
| Leaf 101 |     | Leaf 102 |
+----+-----+     +-----+----+
     |                  |
     +--------+---------+
              |
        +-----+-----+
        | Collector |
        | 192.168.  |
        | 1.200     |
        | (gRPC)    |
        +-----------+
```

### Configuration (Both Leafs)

```nxos
feature telemetry

telemetry
  destination-group 1
    ip address 192.168.1.200 port 50001 protocol gRPC encoding GPB

  sensor-group 1
    path sys/intf depth 0 query-condition rsp-foreign-subtree=ephemeral

  sensor-group 2
    path sys/bgp/inst/dom-[default]/peer depth 0
    path sys/bgp/inst/dom-[default]/af-[l2vpn-evpn] depth 0

  sensor-group 3
    path sys/nvo/items/Nvo-list[nve1]/peer-items depth 0
    path sys/nvo/items/Nvo-list[nve1]/vni-items depth 0
    path sys/bd depth 0

  sensor-group 4
    path sys/ospf/inst/dom-[default] depth 0
    path sys/isis/inst depth 0

  subscription 1
    snsr-grp 1 sample-interval 10000
    dst-grp 1

  subscription 2
    snsr-grp 2 sample-interval 5000
    dst-grp 1

  subscription 3
    snsr-grp 3 sample-interval 5000
    dst-grp 1

  subscription 4
    snsr-grp 4 sample-interval 0
    dst-grp 1
```

### Verification

```text
leaf101# show telemetry subscription
  Subscription ID: 1
    Sensor Group: 1
    Sample Interval: 10000 ms
    Destination Group: 1 (192.168.1.200:50001, gRPC, GPB)
    State: Active

  Subscription ID: 2
    Sensor Group: 2
    Sample Interval: 5000 ms
    Destination Group: 1
    State: Active

  Subscription ID: 3
    Sensor Group: 3
    Sample Interval: 5000 ms
    Destination Group: 1
    State: Active

  Subscription ID: 4
    Sensor Group: 4
    Sample Interval: 0 (on-change)
    Destination Group: 1
    State: Active

leaf101# show telemetry destination-group
  Destination Group: 1
    IP: 192.168.1.200, Port: 50001
    Protocol: gRPC, Encoding: GPB
    State: Connected
    Bytes Sent: 15234567
    Packets Sent: 4523

leaf101# show telemetry sensor-group
  Sensor Group: 1
    Path: sys/intf, Depth: 0
    State: Active, Last Collection: 2s ago

  Sensor Group: 2
    Path: sys/bgp/inst/dom-[default]/peer, Depth: 0
    Path: sys/bgp/inst/dom-[default]/af-[l2vpn-evpn], Depth: 0
    State: Active, Last Collection: 1s ago

  Sensor Group: 3
    Path: sys/nvo/items/Nvo-list[nve1]/peer-items, Depth: 0
    Path: sys/nvo/items/Nvo-list[nve1]/vni-items, Depth: 0
    Path: sys/bd, Depth: 0
    State: Active, Last Collection: 3s ago
```

---

## Lab 2: SPAN/ERSPAN and EEM Troubleshooting

### Objective
Configure ERSPAN for traffic capture and EEM for automated alerting.

### ERSPAN Configuration

```nxos
monitor session 10
  type erspan-source
  source interface Ethernet1/1 both
  source interface Ethernet1/2 both
  source vlan 100 rx
  no shutdown

monitor session 10
  type erspan-source
  destination
    erspan-id 100
    ip address 192.168.1.200
    origin ip address 10.255.1.1
    ttl 64
    no shutdown

monitor session 20
  type erspan-destination
  destination interface Ethernet1/48
  source
    erspan-id 100
    ip address 192.168.1.200
    no shutdown
```

### EEM Configuration

```nxos
event manager applet NVE_PEER_DOWN
  event syslog pattern ".*NVE-5-NVE_PEER_DOWN.*"
  action 1.0 syslog msg "EEM-ALERT: NVE peer down - checking VXLAN"
  action 2.0 cli command "enable"
  action 3.0 cli command "show nve peers"
  action 4.0 snmp-trap string "NVE_PEER_DOWN"

event manager applet BGP_EVPN_DOWN
  event syslog pattern ".*BGP-5-ADJCHANGE.*l2vpn.*Down.*"
  action 1.0 syslog msg "EEM-ALERT: BGP EVPN peer down"
  action 2.0 snmp-trap string "BGP_EVPN_DOWN"

event manager applet HIGH_CPU
  event syslog pattern ".*SYSMGR-2-SYSTEM_CPU_HIGH.*"
  action 1.0 syslog msg "EEM-ALERT: High CPU detected"
  action 2.0 cli command "enable"
  action 3.0 cli command "show process cpu sort"
  action 4.0 snmp-trap string "HIGH_CPU_ALERT"
```

### Verification

```text
leaf101# show monitor session
  Session 10
    Type: ERSPAN Source
    Status: Active
    Source: Eth1/1 (both), Eth1/2 (both), VLAN 100 (rx)
    Destination: ERSPAN ID 100, IP 192.168.1.200

  Session 20
    Type: ERSPAN Destination
    Status: Active
    Destination: Eth1/48
    Source: ERSPAN ID 100, IP 192.168.1.200

leaf101# show event manager policy registered
  Policy Name: NVE_PEER_DOWN
    Event: syslog pattern "NVE-5-NVE_PEER_DOWN"
    State: Registered
    Run Count: 0

  Policy Name: BGP_EVPN_DOWN
    Event: syslog pattern "BGP-5-ADJCHANGE.*l2vpn.*Down"
    State: Registered
    Run Count: 2

leaf101# show event manager history events
  Event: BGP_EVPN_DOWN
    Time: 2024-01-15 14:23:01
    Actions: syslog, snmp-trap
    Status: Completed
```

> **CCIE Exam Tip:** In the troubleshooting lab, ERSPAN is your friend. If you can't figure out why traffic isn't flowing, set up ERSPAN to capture the actual packets. Look for: (1) Is the packet arriving? (2) Is it encapsulated correctly? (3) Is the destination MAC correct? (4) Is the VLAN tag correct? Packet capture resolves 80% of "mystery" issues.

---

## Common Exam Scenarios

### Scenario 1: Telemetry Subscription Not Streaming

```text
Ticket: "Collector not receiving telemetry data from leaf101"

Diagnosis:
  leaf101# show telemetry subscription
  -> Subscription 1: State: Inactive (PROBLEM)
  -> Reason: "Destination unreachable"

  leaf101# show telemetry destination-group
  -> Destination: 192.168.1.200:50001, State: Disconnected

  leaf101# ping 192.168.1.200 vrf management
  -> 100% packet loss

Root cause: Management route to collector missing

Fix:
  leaf101(config)# ip route 192.168.1.200/32 192.168.1.1 vrf management

Verification:
  leaf101# show telemetry destination-group
  -> State: Connected
  leaf101# show telemetry subscription
  -> State: Active, Bytes Sent: incrementing
```

### Scenario 2: TCAM Exhaustion Causing Contract Failure

```text
Ticket: "New ACI contract not enforcing on leaf102"

Diagnosis:
  leaf102# show zoning-rule | include NEW_CONTRACT
  -> (empty - rule not programmed)

  leaf102# show hardware internal access-list resource tcam region ifacl
  -> Used: 18384, Free: 0 (TCAM FULL)

  leaf102# show zoning-rule | include "0 hits"
  -> 45 rules with zero hits (unused contracts)

Root cause: TCAM exhausted by unused contracts

Fix:
  1. Identify unused contracts (zero hit count)
  2. Remove via APIC: Tenant > Contracts > Delete unused
  3. Wait 60 seconds for TCAM reprogramming
  4. Verify: show hardware internal access-list resource tcam region ifacl
     -> Free: 256 (space available)
  5. Verify: show zoning-rule | include NEW_CONTRACT
     -> Rule present

Key lesson: TCAM is finite. Monitor with:
  show hardware internal access-list resource tcam region ifacl
```

### Scenario 3: ERSPAN Not Capturing Traffic

```text
Ticket: "ERSPAN session configured but no packets on analyzer"

Diagnosis:
  leaf101# show monitor session 10
  -> Type: ERSPAN Source, Status: Active
  -> Destination: ERSPAN ID 100, IP 192.168.1.200

  leaf101# show monitor session 10 counters
  -> Packets mirrored: 0 (PROBLEM)

  leaf101# show monitor session 10 detail
  -> Source: Ethernet1/1 (both)
  -> Eth1/1 status: down (ROOT CAUSE)

Root cause: Source interface is down (no traffic to mirror)

Fix:
  1. Bring up source interface: no shutdown
  2. Or: change source to active interface
  3. Verify traffic flowing on source

Verification:
  leaf101# show monitor session 10 counters
  -> Packets mirrored: 15234 (incrementing)
  Analyzer: tcpdump -i eth0 -> packets received
```

---

## Cross-References

- For NDFC fabric health score and REST API, see `06-assurance/ndfc-fabric-management.md`
- For VXLAN fabric troubleshooting (NVE, BGP EVPN), see `01-network/vxlan-evpn.md`
- For ACI contract troubleshooting (zoning-rule, TCAM), see `05-aci/aci-complete.md`

---

## Key Takeaways

1. **Telemetry > SNMP**: Push-based, sub-second, structured data; know destination-group, sensor-group, subscription
2. **gRPC dial-out**: Switch initiates connection to collector; preferred for production
3. **On-change vs periodic**: sample-interval 0 = on-change; >0 = periodic (milliseconds)
4. **VXLAN telemetry**: NVE peers, VNI status, BGP EVPN routes, BD status
5. **NDFC**: Fabric discovery, config management, compliance, firmware; replaces DCNM
6. **ERSPAN**: Remote mirroring via GRE; source session + destination session
7. **EEM**: Event-driven automation; syslog trigger + CLI/SNMP actions
8. **Troubleshooting**: Systematic layer-by-layer; show first, fix second, verify third
9. **Time management**: Read all tickets, prioritize easy ones, 15-min rule for stuck tickets
10. **TCAM monitoring**: `show hardware internal access-list resource tcam region ifacl` — critical for ACI
