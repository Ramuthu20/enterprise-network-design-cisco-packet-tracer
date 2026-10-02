# Enterprise Network Design & Implementation

A Cisco Packet Tracer project: a segmented enterprise network using router-on-a-stick inter-VLAN routing, DHCP, a guest-isolation ACL, NAT/PAT, an LACP EtherChannel between two switches, and Layer 2 hardening, with a simulated ISP providing Internet connectivity.

**Skills demonstrated:** VLAN segmentation · 802.1Q trunking · router-on-a-stick · DHCP · extended ACLs · NAT overload · LACP EtherChannel · port security · PortFast / BPDU Guard · native VLAN hardening · verification and troubleshooting

---

## Network Topology

![Network Topology](network-topology.png)

**Design summary:** Two access switches (**SW1**, **SW2**) are joined by a 2-link LACP EtherChannel trunk. SW1 uplinks to the edge router **R1** over a single trunk. R1 provides inter-VLAN routing using subinterfaces (router-on-a-stick), DHCP for all VLANs, and NAT overload toward the **ISP** router over the `203.0.113.0/30` WAN link. The ISP simulates the public Internet, with a loopback at `8.8.8.8` standing in for public DNS.

---

## Addressing & VLAN Plan

### VLANs

| VLAN | Purpose                 | Subnet             | Gateway (R1)    |
| ---- | ----------------------- | ------------------ | --------------- |
| 10   | Management & Servers    | 192.168.10.0/24    | 192.168.10.1    |
| 20   | Staff / IT              | 192.168.20.0/24    | 192.168.20.1    |
| 30   | Guest Wi-Fi             | 192.168.30.0/24    | 192.168.30.1    |
| 99   | Native VLAN (untagged)  | 192.168.99.0/24    | 192.168.99.1    |

### WAN Link

| Device | Interface | Address            | Role                          |
| ------ | --------- | ------------------ | ----------------------------- |
| R1     | Gi0/1     | 203.0.113.1/30     | WAN uplink, NAT outside       |
| ISP    | Gi0/0     | 203.0.113.2/30     | Link to R1                    |
| ISP    | Loopback0 | 8.8.8.8/32         | Simulated public DNS server   |

### DHCP Pools (configured on R1)

| Pool        | Network          | Gateway       | DNS          | Domain           | Excluded         |
| ----------- | ---------------- | ------------- | ------------ | ---------------- | ---------------- |
| POOL_MGMT   | 192.168.10.0/24  | 192.168.10.1  | 192.168.10.5 | enterprise.local | .1 – .10         |
| POOL_STAFF  | 192.168.20.0/24  | 192.168.20.1  | 192.168.10.5 | enterprise.local | .1 – .10         |
| POOL_GUEST  | 192.168.30.0/24  | 192.168.30.1  | 8.8.8.8      | guest.local      | .1 – .10         |

Staff and management clients use the internal DNS server at `192.168.10.5`; guests use public DNS only.

---

## Device & Port Map

| Device | Port(s)      | Role                                   | Mode / VLAN            |
| ------ | ------------ | -------------------------------------- | ---------------------- |
| R1     | Gi0/0        | Trunk to SW1 (subinterfaces .10 .20 .30 .99) | 802.1Q, native 99 |
| R1     | Gi0/1        | WAN to ISP                             | NAT outside            |
| SW1    | Fa0/1–2      | EtherChannel to SW2 (Port-channel1)    | LACP active, trunk     |
| SW1    | Gi0/1        | Trunk uplink to R1                     | Trunk, native 99       |
| SW1    | Fa0/5        | Corporate server                       | Access VLAN 10         |
| SW1    | Fa0/10       | Management PC                          | Access VLAN 10         |
| SW2    | Fa0/1–2      | EtherChannel to SW1 (Port-channel1)    | LACP active, trunk     |
| SW2    | Fa0/20       | Staff PC                               | Access VLAN 20         |
| SW2    | Fa0/24       | Guest PC                               | Access VLAN 30         |
| Both   | All other ports | Unused, shut down                   | Parked in VLAN 99      |

---

## Configuration Highlights

### Trunking & Native VLAN
- All trunks (Port-channel1 on both switches and SW1 Gi0/1 to R1) allow only VLANs **10, 20, 30, 99**.
- **Native VLAN 99** replaces the default VLAN 1 on every trunk, and `switchport nonegotiate` disables DTP.
- On R1, subinterface `Gi0/0.99` uses `encapsulation dot1Q 99 native`.

### Inter-VLAN Routing (Router-on-a-Stick)
R1's `Gi0/0` carries one subinterface per VLAN, each acting as the default gateway for its subnet.

### Guest Isolation ACL (`GUEST_ISOLATION_IN`)
Applied **inbound** on `Gi0/0.30` (Guest gateway):

| Order | Rule | Purpose |
| ----- | ---- | ------- |
| 1–2   | Permit DHCP (UDP 68 → 67) to broadcast and to 192.168.30.1 | Guests can obtain an address |
| 3–4   | Permit DNS (UDP/TCP 53) from the guest subnet to any | Name resolution |
| 5     | Permit ICMP from guests to 192.168.30.1 only | Gateway reachability test |
| 6     | **Deny** 192.168.30.0/24 → 192.168.10.0/24 | Guests cannot reach Management/Servers |
| 7     | **Deny** 192.168.30.0/24 → 192.168.20.0/24 | Guests cannot reach Staff |
| 8     | Permit 192.168.30.0/24 → any | Guests can reach the Internet |

### NAT/PAT
- Inside interfaces: `Gi0/0.10`, `Gi0/0.20`, `Gi0/0.30`. Outside interface: `Gi0/1`.
- `ip nat inside source list 1 interface GigabitEthernet0/1 overload` translates the `192.168.0.0/16` space to R1's WAN address.
- Default route: `ip route 0.0.0.0 0.0.0.0 203.0.113.2` (toward the ISP).
- The ISP needs no route back to the private subnets because all traffic arrives sourced from `203.0.113.1`.

### EtherChannel (LACP)
`Fa0/1–2` on SW1 and SW2 are bundled into **Port-channel1** with `channel-group 1 mode active` (LACP) on both ends, giving link redundancy and aggregated bandwidth between the switches.

### Layer 2 Security
| Feature | Where | Detail |
| ------- | ----- | ------ |
| Port security | All active access ports | Sticky MAC learning, violation mode **restrict** (max 2 on SW2 Fa0/20, default max elsewhere) |
| PortFast + BPDU Guard | All active access ports | Fast link-up; blocks rogue switches |
| Unused ports | All other ports on both switches | Administratively shut down and moved to VLAN 99 |
| Native VLAN 99 + nonegotiate | All trunks | Avoids default-VLAN and DTP abuse |
| Hardening | All devices | `service password-encryption`, login banner, `no ip domain-lookup`, VTY/console login |
| STP mode | Both switches | PVST |

---

## Verification & Testing

### Connectivity Tests

| #  | Test                                       | Expected                              | Result  |
| -- | ------------------------------------------ | ------------------------------------- | ------- |
| 1  | Management PC receives DHCP lease          | 192.168.10.x, DNS 192.168.10.5        | Pass    |
| 2  | Staff PC receives DHCP lease               | 192.168.20.x, DNS 192.168.10.5        | Pass    |
| 3  | Guest PC receives DHCP lease               | 192.168.30.x, DNS 8.8.8.8             | Pass    |
| 4  | Staff → Management server                  | Success (inter-VLAN routing)          | Pass    |
| 5  | Guest → Management / Staff                 | **Blocked** by ACL                    | Pass    |
| 6  | Guest → 8.8.8.8 (ISP)                      | Success via NAT                       | Pass    |
| 7  | Staff → 8.8.8.8 (ISP)                      | Success via NAT                       | Pass    |
| 8  | Staff → Guest PC                           | Blocked: ACL is stateless, so Guest replies are denied | Pass |
| 9  | Disconnect one EtherChannel link, then ping | Connectivity maintained              | Pass    |
| 10 | Connect an unknown device to a secured port | Frames dropped, violation counter increments | Pass |

### Verification Commands & Evidence

| Command | Confirms |
| ------- | -------- |
| `show vlan brief` | VLANs and port assignments |
| `show interfaces trunk` | Allowed VLANs 10,20,30,99 and native VLAN 99 |
| `show etherchannel summary` | Port-channel1 in use, ports flagged `P` (LACP) |
| `show ip dhcp binding` | Leases issued per pool |
| `show access-lists` | `GUEST_ISOLATION_IN` rules with hit counts |
| `show ip nat translations` | PAT entries |
| `show port-security interface fa0/20` | Sticky MAC and violation count |
| `show ip route` | Connected subnets and default route |

---

## Known Limitations & Possible Improvements

- **Stateless ACL:** `GUEST_ISOLATION_IN` is not stateful, so it blocks Guest replies to Staff as well as Guest-initiated traffic. A reflexive or named ACL with `established` would be more flexible.
- **VLAN 99 reachable by guests:** the final permit line allows Guest → `192.168.99.0/24`. An explicit deny for VLAN 99 would close this.
- **Staff VLAN has no ACL:** Staff can reach all VLANs by design, but a least-privilege policy could restrict this.
- **STP mode:** PVST works but Rapid PVST+ would converge faster.
- **Unused ports in VLAN 99:** best practice is a dedicated, unrouted "blackhole" VLAN rather than the native VLAN.

---

## Troubleshooting Log

| Problem | Symptom | Cause | Fix |
| ------- | ------- | ----- | --- |
| DNS misconfiguration | URLs / hostnames would not resolve | DNS settings were configured incorrectly | Corrected the DNS configuration and re-tested name resolution from the Staff and Guest PCs |
| ISP router port misconfiguration | WAN / Internet side not working as expected | Interface settings on the ISP router ports were misconfigured | Reconfigured ISP `Gi0/0` as `203.0.113.2/30`, enabled it, and verified with `show ip interface brief` and a ping from R1 |

---

## What I Learned
- How router-on-a-stick maps each VLAN to a router subinterface, and why the native VLAN must match on both ends of a trunk.
- ACL order and direction matter: first match wins, and an inbound ACL on the guest subinterface is stateless, so it also affects return traffic.
- Name resolution depends on correct DNS settings at every layer (DHCP pool, DNS server, and routing/ACL permits), so "ping works but URLs don't" points to DNS.
- Always verify each interface on every device (`show ip interface brief`); a single misconfigured ISP port can look like a failure elsewhere in the network.

---

## Project Files

| File | Description |
| ---- | ----------- |
| `enterprise-network.pkt` | Packet Tracer project (open with Cisco Packet Tracer) |
| `network-topology.png` | Topology diagram |
| `R1-config.txt` | Edge router: subinterfaces, DHCP pools, guest ACL, NAT/PAT, default route |
| `SW1-config.txt` | Switch 1: VLAN trunks, LACP EtherChannel, port security, uplink to R1 |
| `SW2-config.txt` | Switch 2: VLAN trunks, LACP EtherChannel, port security |
| `ISP-config.txt` | ISP router: WAN link `203.0.113.2/30` and `8.8.8.8` loopback |

*Note: this is an educational lab project. The device passwords in the published configs are lab-only credentials and are not used anywhere else. In a production network I would use unique credentials, SSH instead of Telnet, and centralized AAA (TACACS+/RADIUS).*

## How to Use
1. Install Cisco Packet Tracer.
2. Open `enterprise-network.pkt`.
3. Run the commands in the verification table on each device to reproduce the results.
