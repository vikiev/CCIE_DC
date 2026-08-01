# L2 Protocols for CCIE Data Center

## Prerequisite Knowledge

- Basic switching concepts (MAC address table, frame forwarding)
- CCNA/CCNP-level understanding of VLANs and trunking
- Familiarity with NX-OS CLI (Nexus 9000)
- Understanding of Ethernet frame format

---

## VLANs and Trunking

### VLAN Fundamentals

A VLAN (Virtual LAN) segments a physical switch into multiple logical broadcast domains.
In the data center, VLANs map directly to VNIs for VXLAN overlay networks.

Key VLAN concepts for CCIE DC:
- VLAN ID range: 1-4094 (normal: 1-1005, extended: 1006-4094)
- VLAN 1 is the default VLAN (cannot be deleted)
- Each VLAN is a separate broadcast domain and subnet
- SVI (Switched Virtual Interface) provides L3 gateway for a VLAN

```nxos
vlan 10
  name WEB
vlan 20
  name APP
vlan 30
  name DB
```

### 802.1Q Trunking

802.1Q is the standard trunking protocol. It inserts a 4-byte tag into the Ethernet
frame header between the source MAC and EtherType fields.

802.1Q tag format:
```
+--------+--------+--------+--------+
| TPID   |  PCP   | DEI   |  VID   |
| 0x8100 | 3-bit  | 1-bit | 12-bit |
+--------+--------+--------+--------+
  16 bits  3 bits  1 bit   12 bits
```

- TPID (Tag Protocol Identifier): 0x8100 identifies 802.1Q frame
- PCP (Priority Code Point): 3-bit CoS value (0-7) for QoS
- DEI (Drop Eligible Indicator): 1-bit, formerly CFI
- VID (VLAN Identifier): 12-bit VLAN ID (0-4095)

```nxos
interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30

interface Ethernet1/2
  switchport mode trunk
  switchport trunk allowed vlan 10-100
```

### Native VLAN

The native VLAN carries UNTAGGED traffic on a trunk. By default, this is VLAN 1.

> **CCIE Exam Tip:** Always change the native VLAN from VLAN 1 to an unused VLAN
> (e.g., VLAN 999). Leaving it as VLAN 1 is a security risk and a common exam trap.
> Both sides of a trunk MUST have the same native VLAN or you get a mismatch error.

```nxos
vlan 999
  name NATIVE_UNUSED

interface Ethernet1/1
  switchport mode trunk
  switchport trunk native vlan 999
```

### Allowed VLANs on Trunks

Control which VLANs traverse a trunk. This is critical for security and for
preventing unnecessary broadcast traffic.

```nxos
interface Ethernet1/1
  switchport trunk allowed vlan 10,20,30

interface Ethernet1/2
  switchport trunk allowed vlan add 40

interface Ethernet1/3
  switchport trunk allowed vlan remove 20

interface Ethernet1/4
  switchport trunk allowed vlan none
```

> **Lab Exam Warning:** When adding VLANs to a trunk, use `switchport trunk allowed
> vlan add X`. If you just type `switchport trunk allowed vlan X`, you REPLACE the
> entire list. This is a classic exam mistake that breaks connectivity.

### Verification Commands - VLANs and Trunking

```text
show vlan
show vlan id 10
show interface trunk
show interface Ethernet1/1 switchport
show interface Ethernet1/1 trunk
```

---

## Spanning Tree Protocol (STP)

### Why STP Exists (and Why DCs Hate It)

STP (802.1D) prevents Layer 2 loops in redundant topologies. It does this by
BLOCKING redundant links, which means:

1. **Blocked links carry no traffic** - you pay for redundancy but don't use it
2. **Suboptimal paths** - traffic may take a longer path because the direct link is blocked
3. **Slow convergence** - classic STP takes 30-50 seconds to reconverge
4. **Single active path** - only one path is active between any two points

In the data center, this is unacceptable. DCs need ALL links active (ECMP, vPC, VXLAN).
This is why DC designs use vPC, FabricPath, or VXLAN instead of relying on STP.

However, STP still runs in DCs as a loop-prevention safety net, and you MUST understand
it for the exam.

### BPDU Format

BPDUs (Bridge Protocol Data Units) are the control frames STP uses to communicate.

Configuration BPDU fields:
```
+---------------------------+
| Protocol ID: 0x0000       |  2 bytes
| Protocol Version: 0       |  1 byte
| BPDU Type: 0x00 (config)  |  1 byte
| Flags                     |  1 byte
| Root Bridge ID            |  8 bytes
| Root Path Cost            |  4 bytes
| Bridge ID (sender)        |  8 bytes
| Port ID (sender)          |  2 bytes
| Message Age               |  2 bytes
| Max Age                   |  2 bytes
| Hello Time                |  2 bytes
| Forward Delay             |  2 bytes
+---------------------------+
```

Bridge ID (8 bytes):
```
+-------------------+-------------------+
| Bridge Priority   | MAC Address       |
| 2 bytes           | 6 bytes           |
+-------------------+-------------------+
  Priority: 4-bit priority + 12-bit extended system ID (VLAN ID)
  Default priority: 32768 (0x8000)
```

> **CCIE Exam Tip:** The Bridge ID is the tiebreaker for root election. Lower is better.
> Priority is compared first (in increments of 4096), then MAC address. You manipulate
> root election by changing priority, NOT MAC address.

### PVST+ (Per-VLAN Spanning Tree Plus)

PVST+ runs a separate STP instance for every VLAN. This allows per-VLAN load balancing
by making different switches root for different VLANs.

- Uses Cisco-proprietary BPDUs (multicast to 01-00-0C-CC-CC-CD)
- One instance per VLAN (can be CPU-intensive with many VLANs)
- Default on Cisco switches

```nxos
spanning-tree mode rapid-pvst

spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 8192

interface Ethernet1/1
  spanning-tree vlan 10 port-priority 64
  spanning-tree vlan 10 cost 1000
```

### Rapid-PVST+ (802.1w)

Rapid-PVST+ provides fast convergence (1-3 seconds vs 30-50 seconds for classic STP).

Port roles in Rapid-PVST+:
- **Root Port**: best path to root bridge (one per non-root switch)
- **Designated Port**: forwarding port on each segment (one per segment)
- **Alternate Port**: backup path to root (blocking, fast failover)
- **Backup Port**: backup for a designated port on same switch (blocking)

Port states (reduced from 5 to 3):
- **Discarding**: not learning, not forwarding (combines disabled/blocking/listening)
- **Learning**: learning MACs, not forwarding
- **Forwarding**: learning and forwarding

Convergence mechanisms:
- **Proposal/Agreement (P/A)**: rapid transition on point-to-point links
- **Edge ports**: PortFast equivalent, immediate forwarding
- **BPDU Guard**: shuts down edge port if BPDU received

```nxos
spanning-tree mode rapid-pvst

interface Ethernet1/1
  spanning-tree port type edge
  spanning-tree bpduguard enable

interface Ethernet1/2
  spanning-tree port type network
  spanning-tree guard root
```

### MST (Multiple Spanning Tree, 802.1s)

MST maps multiple VLANs to a single STP instance, reducing CPU overhead.

MST concepts:
- **MST Region**: group of switches with same region name, revision, and VLAN-to-instance map
- **IST (Internal Spanning Tree)**: instance 0, always exists, carries all unmapped VLANs
- **MSTI (MST Instance)**: instances 1-4094, each maps to a set of VLANs
- **CIST (Common and Internal Spanning Tree)**: spans all regions, single root for entire network

```nxos
spanning-tree mode mst

spanning-tree mst configuration
  name DC_REGION
  revision 1
  instance 1 vlan 10-20
  instance 2 vlan 30-40

spanning-tree mst 1 priority 4096
spanning-tree mst 2 priority 8192
```

> **CCIE Exam Tip:** For MST switches to be in the same region, THREE things must match:
> 1. Region name (case-sensitive)
> 2. Revision number
> 3. VLAN-to-instance mapping
> If any of these differ, the switches are in different regions and STP may block links
> unexpectedly. This is a common exam troubleshooting scenario.

### STP Timers

| Timer | Default | Purpose |
|-------|---------|---------|
| Hello | 2 sec | BPDU transmission interval |
| Max Age | 20 sec | Time before BPDU is considered stale |
| Forward Delay | 15 sec | Time in listening + learning states |

> **Lab Exam Warning:** Do NOT change STP timers unless the task explicitly requires it.
> Changing timers can cause instability. If you must change them, change them on the
> root bridge only, and they propagate to all switches.

### STP in the Data Center: Problems

1. **Blocked links = wasted bandwidth**: A 2-spine/4-leaf topology with STP would block
   half the uplinks. vPC and VXLAN solve this.

2. **Suboptimal forwarding**: Traffic from leaf-1 to leaf-4 may go leaf-1 -> spine-1 ->
   spine-2 -> leaf-4 instead of the direct path, because STP blocked the optimal link.

3. **Convergence time**: Even Rapid-PVST+ takes 1-3 seconds. In a DC with thousands of
   VMs, this causes application timeouts.

4. **Scalability**: PVST+ with 1000 VLANs = 1000 STP instances. MST solves this but
   adds configuration complexity.

5. **TCN (Topology Change Notification) storms**: A flapping access port can cause
   TCN floods that clear MAC tables across the entire network.

**DC Solution**: Use vPC for L2 multi-path, VXLAN/EVPN for L2/L3 overlay, and set
STP as a safety net only (edge ports with BPDU Guard).

### Verification Commands - STP

```text
show spanning-tree
show spanning-tree vlan 10
show spanning-tree vlan 10 detail
show spanning-tree root
show spanning-tree bridge
show spanning-tree mst configuration
show spanning-tree mst 1
show spanning-tree interface Ethernet1/1 detail
show spanning-tree inconsistentports
```

### Key Takeaway - STP

> STP is a loop-prevention protocol that blocks redundant links. In the DC, we use
> vPC and VXLAN to make ALL links active. STP still runs as a safety net. Know the
> port roles, states, timers, and how to manipulate root election. MST is preferred
> over PVST+ in large DCs for scalability.

---

## vPC (Virtual Port Channel) - FULL DEEP DIVE

### What is vPC?

vPC (Virtual Port Channel) allows two physical Nexus switches to appear as a SINGLE
logical switch to downstream devices for port-channel purposes. This eliminates STP
blocked ports and provides active-active forwarding.

Without vPC:
```
                    +---------+
                    | Spine-1 |
                    +----+----+
                         |
              +----------+----------+
              |                     |
         +----+----+          +-----+---+
         | Leaf-1  |          | Leaf-2  |
         +----+----+          +-----+---+
              |                     |
              |    STP blocks       |
              |    one of these     |
              |                     |
         +----+---------------------+----+
         |         Server/switch         |
         +-------------------------------+
```

With vPC:
```
                    +---------+
                    | Spine-1 |
                    +----+----+
                         |
              +----------+----------+
              |                     |
         +----+----+          +-----+---+
         | Leaf-1  +--peer---+  Leaf-2  |
         +----+----+  link   +-----+---+
              |                     |
              |    BOTH active      |
              |    (port-channel)   |
              |                     |
         +----+---------------------+----+
         |         Server/switch         |
         +-------------------------------+
```

### vPC Architecture Components

#### Peer-Link

The peer-link connects the two vPC peer switches. It carries:
- Control traffic (vPC hello, CFS messages)
- Data traffic for orphan ports
- BUM traffic for vPC VLANs
- Synchronization traffic

Requirements:
- Must be a port-channel (minimum 2x 10GE, recommended 2x 40GE or 2x 100GE)
- Must be a trunk carrying all vPC VLANs
- Must be on the same line card (if possible) for low latency
- Dedicated link (no other traffic)

#### Peer-Keepalive

The peer-keepalive is a low-bandwidth link (can be a single 1GE link or even go through
a management network) that sends heartbeat messages between vPC peers.

Purpose: Detect if the peer switch is alive when the peer-link fails.
- Uses UDP port 3200
- Default interval: 1 second, timeout: 5 seconds
- Should be on a separate path from the peer-link (out-of-band management preferred)

> **CCIE Exam Tip:** The peer-keepalive does NOT carry data traffic. It only carries
> heartbeat messages. It can be a direct link or go through a management switch.
> Best practice: use the management VRF (mgmt0 interface) for the keepalive.

#### vPC Domain

The vPC domain is the logical grouping of two vPC peers. Identified by a domain ID (1-1000).

```nxos
vpc domain 100
  peer-keepalive destination 10.0.0.2 source 10.0.0.1 vrf management
  peer-switch
  ip arp synchronize
  role priority 1000
```

#### vPC Member Ports

vPC member ports are the ports that form the port-channel to the downstream device.
They are configured as a regular port-channel, then assigned to the vPC domain.

### Full vPC Configuration

#### Switch 1 (Leaf-1) - Primary

```nxos
feature lacp
feature vpc

vlan 10,20,30,999

interface port-channel 100
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  vpc peer-link

interface Ethernet1/49
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

interface Ethernet1/50
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

vpc domain 100
  peer-keepalive destination 192.168.1.2 source 192.168.1.1 vrf management
  peer-switch
  ip arp synchronize
  role priority 1000

interface port-channel 10
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  vpc 10

interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface Ethernet1/2
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active
```

#### Switch 2 (Leaf-2) - Secondary

```nxos
feature lacp
feature vpc

vlan 10,20,30,999

interface port-channel 100
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  vpc peer-link

interface Ethernet1/49
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

interface Ethernet1/50
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

vpc domain 100
  peer-keepalive destination 192.168.1.1 source 192.168.1.2 vrf management
  peer-switch
  ip arp synchronize
  role priority 2000

interface port-channel 10
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  vpc 10

interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface Ethernet1/2
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active
```

### vPC Operational Rules

1. **Only vPC VLANs are forwarded on the peer-link**: If a VLAN is not configured on
   the vPC port-channel, its traffic does NOT cross the peer-link (unless it's an
   orphan port VLAN).

2. **Orphan ports**: A port that is NOT part of a vPC port-channel but carries a vPC
   VLAN. Traffic from orphan ports crosses the peer-link to reach devices on the
   other vPC peer. This is suboptimal - minimize orphan ports.

3. **vPC peer-link must carry all vPC VLANs**: If a VLAN is on the vPC port-channel
   but not on the peer-link, traffic for that VLAN is dropped.

4. **STP root should be the vPC pair**: Use `peer-switch` command so both vPC peers
   appear as a single STP root. This prevents STP from blocking the peer-link.

5. **ARP synchronization**: `ip arp synchronize` ensures both peers have the same ARP
   table after a failover. Critical for anycast gateway.

### vPC and STP Interaction

When `peer-switch` is configured, both vPC peers use the same STP bridge ID. To the
rest of the network, the vPC pair appears as a SINGLE switch.

Without peer-switch:
- Each vPC peer has its own bridge ID
- STP may block the peer-link (sees it as a loop)
- Suboptimal forwarding

With peer-switch:
- Both peers share the lower bridge ID
- STP sees the vPC pair as one switch
- Peer-link is the "backbone" of the logical switch
- All downstream links can be forwarding

> **CCIE Exam Tip:** ALWAYS configure `peer-switch` in vPC. It is required for proper
> STP operation and for vPC + VXLAN anycast gateway to work correctly.

### vPC Consistency Checks

vPC peers must have consistent configurations. Two types of checks:

#### Type 1 (Global) - Must match or vPC does not form

- MTU on peer-link
- VLANs on peer-link and vPC port-channel
- STP mode
- vPC domain ID
- QoS settings (system qos)
- Network QoS (jumbo frames)

If Type 1 check fails: vPC does not come up at all.

#### Type 2 (Interface) - Must match or port is suspended

- Port speed
- Duplex
- Trunk mode
- Allowed VLANs
- Native VLAN
- STP port settings
- Port-channel mode (LACP vs static)

If Type 2 check fails: the specific vPC port is suspended (err-disabled).

```text
show vpc consistency-parameters global
show vpc consistency-parameters interface port-channel 10
```

> **Lab Exam Warning:** vPC consistency mismatch is a VERY common exam troubleshooting
> scenario. If a vPC port is suspended, run `show vpc consistency-parameters` on BOTH
> peers and compare. The mismatch is almost always: allowed VLANs, native VLAN, MTU,
> or speed/duplex.

### vPC Advanced Features

#### Peer-Gateway

By default, when a vPC peer receives a frame destined for the OTHER peer's router MAC,
it drops the frame (because the MAC belongs to the peer, not itself). `peer-gateway`
allows a vPC peer to act as the gateway for the peer's MAC, preventing traffic blackholing
when a host sends traffic to the "wrong" vPC peer's MAC.

```nxos
vpc domain 100
  peer-gateway
```

Without peer-gateway: Host ARPs for Leaf-1's MAC, but frame arrives at Leaf-2. Leaf-2
drops it (MAC mismatch). With peer-gateway: Leaf-2 forwards the frame locally.

> **CCIE Exam Tip:** ALWAYS configure `peer-gateway` in vPC. It is required for proper
> HSRP/anycast-gw operation and prevents intermittent connectivity issues when hosts
> cache the "wrong" gateway MAC after a failover.

#### Auto-Recovery

Auto-recovery allows vPC to recover when BOTH peers reload simultaneously (e.g., power
outage). Without it, both peers wait for the other to come up first - deadlock.

```nxos
vpc domain 100
  auto-recovery
```

Behavior: After both peers reload, the peer with the LOWER role priority becomes primary
and brings up vPC ports. The other peer waits for the peer-link to come up.

> **Lab Exam Warning:** If the exam scenario involves "both switches reloaded and vPC
> is not forming," the fix is `auto-recovery`. Without it, vPC stays down after a
> dual-reload event.

#### Suspend Non-Orphan Ports

When the peer-link fails and the secondary suspends its vPC ports, orphan ports on the
secondary ALSO lose connectivity (their traffic crosses the peer-link). By default,
orphan ports stay up but blackhole traffic. `suspend-non-orphan` is NOT a command -
instead, you should understand that vPC ports are suspended but orphan ports remain up.

The correct behavior:
- vPC member ports on secondary: SUSPENDED (to prevent loop)
- Orphan ports on secondary: REMAIN UP (but traffic may blackhole)
- To handle orphan ports: use `delay restore` and ensure critical hosts use vPC

```nxos
vpc domain 100
  delay restore 150
  delay restore interface-vlan 10
```

### vPC Failure Scenarios

#### Scenario 1: Peer-Link Down, Peer-Keepalive Up

This is the most critical failure. The secondary peer CANNOT determine if the primary
is still forwarding traffic.

Behavior:
1. Secondary peer detects peer-link failure
2. Secondary checks peer-keepalive: primary is ALIVE
3. Secondary assumes primary is still forwarding
4. Secondary SUSPENDS all vPC member ports (to prevent loop)
5. Primary keeps forwarding (it is the operational primary)
6. Orphan ports on secondary lose connectivity

Recovery: Restore peer-link. Secondary re-enables vPC ports after consistency check.

#### Scenario 2: Peer-Link Down, Peer-Keepalive Down

This is the DUAL-ACTIVE (split-brain) scenario. Both peers think the other is dead.

Behavior:
1. Both peers detect peer-link failure
2. Both peers lose peer-keepalive
3. Both peers assume they are the only surviving switch
4. BOTH peers keep forwarding on vPC ports
5. Potential loop / duplicate frames

Mitigation:
- Use dual peer-keepalive paths (direct + via management)
- Configure `delay restore` to slow down recovery
- Use BFD on peer-link for faster detection
- Dual-active detection via LACP fallback (on downstream)

#### Scenario 3: Peer-Keepalive Down, Peer-Link Up

This is a WARNING condition only. vPC continues to operate normally.

Behavior:
1. Peer-keepalive fails
2. vPC peers log a warning
3. vPC continues to forward traffic normally
4. If peer-link ALSO fails, dual-active occurs (no keepalive to detect)

Action: Fix the keepalive link. This is not an emergency but reduces fault tolerance.

#### Scenario 4: One vPC Peer Reloads

Behavior:
1. Reloading peer goes down
2. Surviving peer detects failure via peer-link + keepalive
3. Surviving peer takes over all vPC traffic
4. Downstream port-channel stays up (LACP sees one member)
5. Reloading peer comes back, re-syncs, re-joins vPC

`delay restore 150` delays the rejoining peer from forwarding for 150 seconds,
allowing routing protocols to converge first.

### vPC and VXLAN: Anycast Gateway

In a VXLAN fabric, vPC is used to create an anycast gateway. Both vPC peers share
the SAME virtual IP and virtual MAC for the SVI.

```nxos
interface vlan 10
  no shutdown
  ip address 10.10.10.1/24
  fabric forwarding mode anycast-gw
```

The anycast gateway MAC is derived from the anycast-gw MAC configured globally:

```nxos
fabric forwarding anycast-gw-mac 0000.1111.2222
```

Both vPC peers respond to ARP with the same MAC. The host's default gateway is
always "local" - no HSRP/VRRP needed. This is called the Distributed Anycast Gateway.

Benefits:
- No FHRP election delay
- Optimal forwarding (host always uses local leaf as gateway)
- Seamless VM migration (same gateway MAC everywhere)

### Verification Commands - vPC

```text
show vpc
show vpc brief
show vpc peer-keepalive
show vpc consistency-parameters global
show vpc consistency-parameters interface port-channel 10
show vpc orphan-ports
show vpc statistics
show port-channel summary
show port-channel database
show vpc role
```

Expected output for healthy vPC:
```text
Leaf-1# show vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 100
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off
Delay-restore SVI status          : Timer is off
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id   Port   Status Active vlans
--   ----   ------ -------------------------------------------------
1    Po100  up     10,20,30,999

vPC status
---------------------------------------------------------------------
id    Port   Status Consistency Reason                Active vlans
--    ----   ------ ----------- ------                ---------------
10    Po10   up     success     success               10,20,30
```

### Key Takeaway - vPC

> vPC makes two Nexus switches act as one for port-channel purposes. It eliminates
> STP blocked ports and enables active-active forwarding. Critical components:
> peer-link (data + control), peer-keepalive (heartbeat), vPC domain. Know ALL
> failure scenarios cold - they WILL appear in the troubleshooting module.

---

## LACP (Link Aggregation Control Protocol)

### LACP Fundamentals

LACP (802.3ad / 802.1AX) dynamically negotiates port-channel membership between
two switches. It detects misconfigurations and link failures.

LACP modes:
- **Active**: sends LACP packets, initiates negotiation
- **Passive**: responds to LACP packets, does not initiate
- **On (static)**: no LACP, port-channel always up (no negotiation)

Mode compatibility:
```
Active  + Active  = FORMS
Active  + Passive = FORMS
Passive + Passive = DOES NOT FORM (neither initiates)
On      + On      = FORMS (static, no LACP)
On      + Active  = DOES NOT FORM (mismatch)
```

> **CCIE Exam Tip:** Always use `mode active` on both sides. It's the most robust
> option. `mode passive` + `mode passive` will NEVER form a port-channel - this is
> a classic exam trick question.

### LACP System Priority and Port Priority

LACP elects which side controls the port-channel:
- **System Priority** (default 32768): lower wins. Determines which switch is the
  "decision maker" for the port-channel.
- **Port Priority** (default 32768): lower wins. Determines which ports are active
  when there are more candidate ports than the maximum (8 or 16).

```nxos
lacp system-priority 1000

interface Ethernet1/1
  lacp port-priority 100
  channel-group 10 mode active
```

### LACP Rate

- **Normal** (default): LACP packets every 30 seconds, timeout 90 seconds
- **Fast**: LACP packets every 1 second, timeout 3 seconds

Use fast rate for links where rapid failure detection is critical.

```nxos
interface Ethernet1/1
  lacp rate fast
  channel-group 10 mode active
```

### LACP Configuration

```nxos
feature lacp

interface port-channel 10
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30

interface Ethernet1/1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface Ethernet1/2
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active
```

### Verification Commands - LACP

```text
show lacp counters
show lacp neighbor
show lacp internal
show port-channel summary
show port-channel database
```

---

## Port Channels: Static vs LACP

### Static Port Channel (mode on)

No protocol negotiation. Both sides must be manually configured identically.
If there's a misconfiguration (wrong VLAN, wrong speed), the port-channel may
form but silently drop traffic.

```nxos
interface port-channel 10
  switchport mode trunk

interface Ethernet1/1
  switchport mode trunk
  channel-group 10 mode on

interface Ethernet1/2
  switchport mode trunk
  channel-group 10 mode on
```

### LACP Port Channel (mode active)

LACP negotiates and validates the port-channel. Detects misconfigurations.
Preferred in all production and exam scenarios.

### Hash Algorithms

Port channels distribute traffic across member links using a hash. The hash
determines which link carries which flow.

```nxos
port-channel load-balance src-dst ip-l4port
```

Available hash options on Nexus 9000:
- `src-dst ip`: source + destination IP
- `src-dst ip-l4port`: source + dest IP + L4 port (best for most traffic)
- `src-dst mac`: source + destination MAC
- `src-dst ip-vlan`: source + dest IP + VLAN

> **CCIE Exam Tip:** The default hash on Nexus 9000 is `src-dst ip-l4port`. This is
> usually optimal. Only change it if the task requires it or if you observe polarization
> (all traffic on one link).

### Verification Commands - Port Channels

```text
show port-channel summary
show port-channel database
show port-channel load-balance
show port-channel statistics
show interface port-channel 10
```

---

## FEX (Fabric Extender)

### FEX Architecture

A FEX (Fabric Extender, e.g., N2248TP-E) is a remote line card for a parent Nexus
switch. It has NO local control plane - all forwarding decisions are made by the parent.

```
+-------------------+
|   Parent Nexus    |
|   (N9K, N5K)     |
+--------+----------+
         |  (FEX uplinks: 10GE/40GE)
         |
+--------+----------+
|      FEX          |
|   (N2248, N2400)  |
+--+--+--+--+--+---+
   |  |  |  |  |
   S  S  S  S  S   (Server-facing ports)
```

Key concepts:
- FEX is managed entirely by the parent switch
- FEX ports are configured on the parent as `interface Ethernet1xx/1/y`
- FEX uplinks connect to parent via port-channel
- Two modes: pinning mode and port-channel mode

### Pinning Mode (Default)

Each server port is statically pinned to one uplink. If that uplink fails, the
server port goes down.

- Simple, deterministic
- No load balancing across uplinks for a single flow
- Server loses connectivity if its pinned uplink fails

```nxos
fex 101
  pinning max-links 2
  description "Rack-1-FEX"

interface Ethernet1/49
  fex associate 101
  channel-group 101

interface Ethernet1/50
  fex associate 101
  channel-group 101
```

### Port-Channel Mode

All uplinks form a port-channel. Server traffic is load-balanced across all uplinks.
If one uplink fails, traffic redistributes.

- Better utilization
- Survives single uplink failure
- Requires LACP on parent

```nxos
fex 101
  description "Rack-1-FEX"

interface Ethernet1/49
  fex associate 101
  channel-group 101 mode active

interface Ethernet1/50
  fex associate 101
  channel-group 101 mode active

interface port-channel 101
  fex associate 101
```

### FEX Configuration (Server Ports)

```nxos
interface Ethernet101/1/1
  switchport mode access
  switchport access vlan 10
  spanning-tree port type edge

interface Ethernet101/1/2
  switchport mode trunk
  switchport trunk allowed vlan 10,20
```

### Verification Commands - FEX

```text
show fex
show fex 101
show fex interface Ethernet101/1/1
show fex fex-id 101 detail
show interface Ethernet101/1/1 status
```

> **CCIE Exam Tip:** FEX is rarely a full configuration task in the exam, but you may
> need to verify FEX status or troubleshoot a FEX port. Know `show fex` and how to
> configure a FEX port. FEX does NOT support vPC directly - the parent switch handles vPC.

---

## FabricPath / TRILL (Conceptual)

### What is FabricPath?

FabricPath is Cisco's implementation of TRILL (Transparent Interconnection of Lots of
Links). It replaces STP with IS-IS-based L2 routing, enabling:
- All links active (ECMP at L2)
- No STP blocking
- Loop-free L2 topology
- MAC routing (instead of MAC flooding)

### FabricPath Concepts (Know for Exam)

- **FabricPath IS-IS**: uses IS-IS to build L2 topology (not IP routing)
- **Switch ID**: each FabricPath switch has a unique switch-id (1-4094)
- **FTag**: tree identifier for multicast/BUM traffic
- **Conversational MAC learning**: MACs learned only on conversation, not flooded
- **vPC+**: extension of vPC for FabricPath (two switches appear as one to FP fabric)

### FabricPath vs VXLAN

| Feature | FabricPath | VXLAN |
|---------|-----------|-------|
| Scope | Single DC (L2) | Multi-DC (L2/L3 overlay) |
| Encapsulation | FabricPath header | VXLAN (UDP/IP) |
| Control plane | FP IS-IS | BGP EVPN |
| Scalability | ~500 switches | 16M VNIs |
| Current status | Legacy (being replaced) | Industry standard |

> **CCIE Exam Tip:** FabricPath is LEGACY. It appears on the exam as a conceptual
> question only. You will NOT configure FabricPath in the v3.1 lab. Know what it is,
> how it differs from VXLAN, and that it uses IS-IS. That's it.

---

## Lab 1: Full vPC Configuration (2x Nexus 9000)

### Topology

```
                    +-------------------+
                    |    Spine-1        |
                    |  (not in vPC)     |
                    +---+-----------+---+
                        |           |
                   Eth1/49     Eth1/49
                        |           |
                   +----+----+ +----+----+
                   | Leaf-1  | | Leaf-2  |
                   | (Pri)   | | (Sec)   |
                   +----+----+ +----+----+
                        |           |
                   Eth1/49     Eth1/49
                        |           |
                        +-----+-----+
                              |
                         Peer-Link
                         (Po100)
                              |
                   +----+----+----+----+
                   | Eth1/1      Eth1/1|
                   |    (Po10, vPC 10) |
                   +---------+---------+
                             |
                        +----+----+
                        |  Server |
                        | (LACP)  |
                        +---------+

Keepalive: Leaf-1 mgmt0 (192.168.1.1) <---> Leaf-2 mgmt0 (192.168.1.2)
```

### Leaf-1 Configuration

```nxos
hostname Leaf-1

feature lacp
feature vpc
feature interface-vlan

vlan 10
  name WEB
vlan 20
  name APP
vlan 30
  name DB
vlan 999
  name NATIVE

interface port-channel 100
  description vPC-Peer-Link
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  spanning-tree port type network
  vpc peer-link

interface Ethernet1/49
  description vPC-Peer-Link-Member-1
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

interface Ethernet1/50
  description vPC-Peer-Link-Member-2
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

vpc domain 100
  peer-keepalive destination 192.168.1.2 source 192.168.1.1 vrf management
  peer-switch
  ip arp synchronize
  role priority 1000
  delay restore 150

interface port-channel 10
  description Server-Facing-vPC
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  vpc 10

interface Ethernet1/1
  description Server-Link-1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface Ethernet1/2
  description Server-Link-2
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface vlan 10
  no shutdown
  ip address 10.10.10.2/24
  fabric forwarding mode anycast-gw

interface vlan 20
  no shutdown
  ip address 10.10.20.2/24
  fabric forwarding mode anycast-gw

interface vlan 30
  no shutdown
  ip address 10.10.30.2/24
  fabric forwarding mode anycast-gw

fabric forwarding anycast-gw-mac 0000.1111.2222
```

### Leaf-2 Configuration

```nxos
hostname Leaf-2

feature lacp
feature vpc
feature interface-vlan

vlan 10
  name WEB
vlan 20
  name APP
vlan 30
  name DB
vlan 999
  name NATIVE

interface port-channel 100
  description vPC-Peer-Link
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  spanning-tree port type network
  vpc peer-link

interface Ethernet1/49
  description vPC-Peer-Link-Member-1
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

interface Ethernet1/50
  description vPC-Peer-Link-Member-2
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active

vpc domain 100
  peer-keepalive destination 192.168.1.1 source 192.168.1.2 vrf management
  peer-switch
  ip arp synchronize
  role priority 2000
  delay restore 150

interface port-channel 10
  description Server-Facing-vPC
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  vpc 10

interface Ethernet1/1
  description Server-Link-1
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface Ethernet1/2
  description Server-Link-2
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30
  channel-group 10 mode active

interface vlan 10
  no shutdown
  ip address 10.10.10.3/24
  fabric forwarding mode anycast-gw

interface vlan 20
  no shutdown
  ip address 10.10.20.3/24
  fabric forwarding mode anycast-gw

interface vlan 30
  no shutdown
  ip address 10.10.30.3/24
  fabric forwarding mode anycast-gw

fabric forwarding anycast-gw-mac 0000.1111.2222
```

### Verification

```text
Leaf-1# show vpc brief
vPC domain id                     : 100
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
vPC role                          : primary
Number of vPCs configured         : 1

Leaf-1# show port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        S - Switched    R - Routed
        U - Up (port-channel)
        M - Not in use. Min-links not met

Group  Port-Channel  Type   Protocol  Members
-----  ------------  -----  --------  ----------------------------------------
10     Po10(SU)      Eth    LACP      Eth1/1(P)    Eth1/2(P)
100    Po100(SU)     Eth    LACP      Eth1/49(P)   Eth1/50(P)
```

---

## Lab 2: vPC Troubleshooting (3 Scenarios)

### Scenario 1: vPC Peer-Link Down

**Symptom**: Server on vPC port-channel loses connectivity. One leaf shows vPC ports suspended.

**Diagnosis**:
```text
Leaf-2# show vpc brief
Peer status                       : peer adjacency is down
vPC keep-alive status             : peer is alive
vPC role                          : secondary

Leaf-2# show vpc
vPC Peer-link status
---------------------------------------------------------------------
id   Port   Status Active vlans
--   ----   ------ -------------------------------------------------
1    Po100  down   -

vPC status
---------------------------------------------------------------------
id    Port   Status Consistency Reason                Active vlans
--    ----   ------ ----------- ------                ---------------
10    Po10   down   success     Peer-link is down     -
```

**Root Cause**: Peer-link (Po100) is down. Secondary suspended vPC ports.

**Fix**:
```text
Leaf-2# show interface port-channel 100
Leaf-2# show interface Ethernet1/49
Leaf-2# show interface Ethernet1/50
```

Check physical links, LACP, allowed VLANs on peer-link members.

```nxos
interface Ethernet1/49
  no shutdown
  switchport mode trunk
  switchport trunk allowed vlan 10,20,30,999
  channel-group 100 mode active
```

### Scenario 2: vPC Consistency Mismatch

**Symptom**: vPC port Po10 is suspended on one peer.

**Diagnosis**:
```text
Leaf-1# show vpc consistency-parameters interface port-channel 10
Type 2 : Inconsistent (will cause vPC port to be suspended)

Leaf-1# show vpc consistency-parameters interface port-channel 10
  VLANs allowed on trunk:
    Leaf-1: 10,20,30
    Leaf-2: 10,20       <-- MISMATCH
```

**Root Cause**: Allowed VLANs differ between vPC peers.

**Fix**: Make VLANs consistent on both peers.
```nxos
interface port-channel 10
  switchport trunk allowed vlan 10,20,30
```

### Scenario 3: Dual-Active (Split Brain)

**Symptom**: Both peers are forwarding. Duplicate MACs seen. Intermittent connectivity.

**Diagnosis**:
```text
Leaf-1# show vpc brief
Peer status                       : peer adjacency is down
vPC keep-alive status             : peer is not alive
vPC role                          : primary

Leaf-2# show vpc brief
Peer status                       : peer adjacency is down
vPC keep-alive status             : peer is not alive
vPC role                          : primary
```

Both peers think they are primary. Both are forwarding. Loop exists.

**Root Cause**: Both peer-link AND peer-keepalive are down.

**Fix**: Restore peer-link first, then peer-keepalive. The secondary will suspend
its vPC ports until the peer-link is restored.

```text
Leaf-2# show interface port-channel 100
Leaf-2# show lacp neighbor
```

Restore physical links. Verify vPC reforms:
```text
Leaf-2# show vpc brief
Peer status                       : peer adjacency formed ok
vPC role                          : secondary
```

---

## Lab 3: STP Root Manipulation

### Objective
Make Leaf-1 the root for VLAN 10 and Leaf-2 the root for VLAN 20.

### Configuration

```nxos
Leaf-1(config)# spanning-tree mode rapid-pvst
Leaf-1(config)# spanning-tree vlan 10 priority 4096
Leaf-1(config)# spanning-tree vlan 20 priority 8192

Leaf-2(config)# spanning-tree mode rapid-pvst
Leaf-2(config)# spanning-tree vlan 10 priority 8192
Leaf-2(config)# spanning-tree vlan 20 priority 4096
```

### Verification

```text
Leaf-1# show spanning-tree vlan 10 | include root
Root ID    Priority    4096
           Address     0000.1111.2222
           This bridge is the root

Leaf-1# show spanning-tree vlan 20 | include root
Root ID    Priority    4096
           Address     0000.3333.4444
           Cost        2
           Port        4097 (port-channel100)
```

> **CCIE Exam Tip:** Priority must be in increments of 4096 (0, 4096, 8192, 12288...).
> You cannot set priority to 5000. The command will reject it. Use `show spanning-tree
> root` to verify root for all VLANs at once.

---

## Common Exam Scenarios

### Scenario 1: vPC Build from Scratch (Deploy Module)

**Task**: "Configure vPC domain 100 between Leaf-1 and Leaf-2 with peer-link on Po100 (Eth1/49-50) and vPC 10 to server on Po10 (Eth1/1-2)."

**Time budget**: 15-20 minutes.

**Execution order**:
1. `feature lacp`, `feature vpc`
2. VLANs (all VLANs that will traverse vPC)
3. Peer-link port-channel (Po100) + members (Eth1/49, Eth1/50)
4. `vpc domain 100` + peer-keepalive + peer-switch + ip arp synchronize
5. vPC member port-channel (Po10) + `vpc 10`
6. Member interfaces (Eth1/1, Eth1/2) + `channel-group 10 mode active`
7. Verify: `show vpc brief`

**Zero-credit mistakes**:
- Forgetting `feature vpc` (nothing works)
- Peer-keepalive destination/source swapped
- Missing `peer-switch` (STP blocks peer-link)
- VLAN mismatch between peer-link and vPC port-channel

### Scenario 2: vPC Port Suspended (Troubleshoot Module)

**Task**: "Server on vPC Po10 has no connectivity. One leaf shows Po10 as suspended."

**Diagnosis**:
```text
Leaf-2# show vpc consistency-parameters interface port-channel 10
Type 2 : Inconsistent
  Allowed VLANs: Leaf-1=10,20,30  Leaf-2=10,20   <-- MISMATCH
```

**Fix**: Make VLANs consistent on BOTH peers. Then `show vpc brief` to confirm recovery.

### Scenario 3: STP Root Manipulation with vPC

**Task**: "Make the vPC pair (Leaf-1 + Leaf-2) the STP root for VLANs 10 and 20."

**Key**: With `peer-switch` configured, both peers share the same bridge ID. Set priority on BOTH:
```nxos
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
```

**Common mistake**: Setting priority on only one peer. With peer-switch, both must have the same priority.

---

## Complete Verification Commands Reference

```text
show vlan
show vlan id 10
show interface trunk
show interface Ethernet1/1 switchport
show spanning-tree
show spanning-tree vlan 10
show spanning-tree vlan 10 detail
show spanning-tree root
show spanning-tree bridge
show spanning-tree mst configuration
show spanning-tree inconsistentports
show vpc
show vpc brief
show vpc peer-keepalive
show vpc consistency-parameters global
show vpc consistency-parameters interface port-channel 10
show vpc orphan-ports
show vpc statistics
show vpc role
show lacp counters
show lacp neighbor
show lacp internal
show port-channel summary
show port-channel database
show port-channel load-balance
show fex
show fex 101
show interface port-channel 100
show interface Ethernet1/1
show mac address-table
show mac address-table vlan 10
```

---

## Key Takeaways

1. **VLANs/Trunking**: 802.1Q tags frames with VLAN ID. Always set native VLAN to
   an unused VLAN. Use `add` when modifying allowed VLANs on trunks.

2. **STP**: Prevents L2 loops by blocking redundant links. Know port roles (root,
   designated, alternate, backup), states (discarding, learning, forwarding), and
   timers. Rapid-PVST+ converges in 1-3 seconds. MST scales better than PVST+.

3. **vPC**: Makes two switches appear as one for port-channel. Peer-link carries data,
   peer-keepalive carries heartbeats. Know ALL failure scenarios. Always use
   `peer-switch` and `ip arp synchronize`.

4. **LACP**: Use `mode active` on both sides. Know system priority, port priority,
   and LACP rate (fast = 1 sec hello).

5. **FEX**: Remote line card, no local control plane. Pinning mode vs port-channel mode.

6. **FabricPath**: Legacy L2 routing protocol using IS-IS. Conceptual only for v3.1.

> **Lab Exam Warning:** vPC is the MOST TESTED L2 topic. You will configure it, verify
> it, and troubleshoot it. Know the show commands cold. Know the failure modes cold.
> If you can do vPC in your sleep, you'll pass the L2 portion.

---

*L2 Protocols | CCIE DC v3.1 | Network Domain (30%)*
