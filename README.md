Enterprise Network Design & Implementation

A Cisco Packet Tracer project focused on designing and implementing a segmented enterprise network with VLANs, inter-VLAN routing, DHCP, ACLs, NAT/PAT, EtherChannel, and Layer 2 security.

Project Overview

This project demonstrates the design and implementation of a structured enterprise network that provides network segmentation, controlled communication between departments, automated IP addressing, Layer 2 security, and external network connectivity.

The network was designed to support separate Staff and Guest networks while providing appropriate routing, traffic control, redundancy, and Internet access.

Network Topology

![Network Topology](topology/network-topology.png)
Technologies & Networking Concepts

* Cisco Packet Tracer
* VLANs
* 802.1Q Trunking
* Native VLAN
* Inter-VLAN Routing
* DHCP
* Access Control Lists (ACLs)
* NAT/PAT (NAT Overload)
* EtherChannel
* LACP
* Spanning Tree Protocol (STP)
* PortFast
* BPDU Guard
* Port Security
* Default Route
* WAN Connectivity

Network Segmentation

The network is divided into separate VLANs to provide logical segmentation between different groups of users.

| VLAN    | Purpose     |
| ------- | ----------- |
| VLAN 10 | Staff       |
| VLAN 20 | Guest       |
| VLAN 99 | Native VLAN |

VLAN segmentation creates separate Layer 2 broadcast domains, while inter-VLAN routing provides Layer 3 communication where permitted.

Key Implementations

VLANs & 802.1Q Trunking

VLANs were configured to logically separate network traffic. 802.1Q trunking was implemented between network devices to transport traffic from multiple VLANs over shared links.

Native VLAN

A dedicated native VLAN was configured for trunk links to handle untagged traffic.

Inter-VLAN Routing

Layer 3 routing was configured to enable communication between different VLANs where required.

DHCP

DHCP was configured to automatically provide network addressing information to clients, including:

* IP address
* Subnet mask
* Default gateway
* DNS server

Access Control Lists

ACLs were implemented to control traffic between network segments according to defined access requirements.

NAT/PAT

NAT overload (PAT) was configured to allow multiple private-network devices to access external networks using a shared public IP address.

EtherChannel & LACP

Multiple physical links were aggregated into logical EtherChannel connections using LACP, providing link redundancy and increased aggregate bandwidth.

Layer 2 Security

The network includes several Layer 2 protection mechanisms:

* Port Security
* PortFast
* BPDU Guard
* Native VLAN configuration

These features help protect access ports and improve network stability.

Routing & WAN Connectivity

A default route and WAN connectivity were configured to provide a path from the internal network toward external networks.

Verification & Testing

The network configuration was verified using Cisco IOS commands and connectivity tests, including:

* VLAN verification
* Trunk verification
* EtherChannel/LACP verification
* DHCP verification
* ACL verification
* NAT/PAT verification
* Inter-VLAN connectivity testing
* End-to-end connectivity testing

Project Files

| File                     | Description                     |
| ------------------------ | ------------------------------- |
| `enterprise-network.pkt` | Cisco Packet Tracer project     |
| `network-topology.png`   | Network topology diagram        |
| `router-config.txt`      | Router configuration            |
| `switch1-config.txt`     | Switch configuration            |
| `switch2-config.txt`     | Additional switch configuration |

Tools

Cisco Packet Tracer
Used to design, configure, test, and troubleshoot the network topology.

Project Objective

The objective of this project was to gain practical experience in enterprise network design and Cisco networking, including network segmentation, routing, traffic control, Layer 2 security, redundancy, IP address management, and external network connectivity.
