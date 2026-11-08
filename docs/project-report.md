# Secure Multi-Site Enterprise Network: Project Report

**Author:** Buddhika Sandaru (M. A. B. S. L. Marasinghe)  
**Programme:** Information Technology, Institute of Technology, University of Moratuwa  
**Tool:** Cisco Packet Tracer  
**Date:** January 2026 

---

## 1. Introduction

This project designs and builds a secure network for a small company with one head office and two branch offices. It was built in Cisco Packet Tracer to practise the skills from the CCNA course: switching, routing, network services and basic security. The goal was a design that is easy to explain, easy to test and close to what a real small enterprise would use.

## 2. Requirements

- Three sites connected together and to a simulated internet
- Head office users separated into Staff, Servers, Guest and Management groups
- Guests may use the internet but not internal networks
- Remote management only with SSH
- Redundant link between the head office switches
- Protection of access ports from unknown devices and fake DHCP servers

## 3. Design

The head office uses one router (R-HQ) and two switches (SW-HQ1, SW-HQ2). Each branch has one router and one switch. R-HQ connects to the branches with serial links and to an ISP router that stands for the internet.

Main design choices:

- **VLANs and router-on-a-stick:** four VLANs (10 Staff, 20 Servers, 30 Guest, 99 Management) share one trunk to R-HQ, which holds one subinterface per VLAN.
- **EtherChannel:** two cables between the head office switches work as one logical link (LACP Port-Channel 1).
- **OSPF area 0:** all three routers exchange routes; R-HQ also advertises a default route (`default-information originate`).
- **DHCP and NAT:** every site hands out addresses, and R-HQ translates private addresses to its public interface address (NAT Overload/PAT).
- **Security:** an extended ACL isolates guests, SSH replaces Telnet, and the switches use port security, DHCP snooping and shutdown of unused ports.

Figure 1 shows the topology (see `diagrams/topology-overview.png`). Full addressing and VLAN tables are in `docs/design.md`.

## 4. Implementation

The network was built in stages and tested after each stage:

1. Placed devices, installed the HWIC serial modules in the three routers that need it, and connected 13 cables.
2. Configured VLANs, the EtherChannel and the trunk on the head office switches.
3. Configured IP addresses, subinterfaces and serial links, then enabled OSPF on all routers.
4. Configured DHCP pools and NAT overload on R-HQ.
5. Applied the guest ACL, SSH-only access, port security, DHCP snooping and unused-port shutdown.

The device configurations are preserved in the `configs/` folder.

## 5. Testing and results

Nine comprehensive tests were conducted as defined in `docs/test-plan.md`.

| Test | Result |
|---|---|
| Head office to both branches (T1, T2) | Pass |
| Internet access through NAT (T3) | Pass |
| Guest cannot reach internal networks (T4) | Pass |
| Guest can reach the internet (T5) | Pass |
| Telnet refused, SSH works (T6, T7) | Pass |
| Port security reacts to a second device (T8) | Pass |
| Connectivity survives one EtherChannel cable removed (T9) | Pass |

All nine test cases completed with 100% success. End-to-end ping verification confirmed full routability across HQ and both branch locations via OSPF. The failover test validated that dropping one physical member of the EtherChannel trunk caused zero dropped pings between PC-Staff and PC-Server. The guest isolation ACL successfully blocked internal LAN access while permitting web/internet traffic translated through NAT.

## 6. Problems and fixes

During the implementation and testing phases, several realistic network issues were diagnosed and resolved:

1. **Serial interface naming mismatch:** When cabling R-HQ, `Serial0/0/0` was not available because the HWIC serial card was inserted into slot 3. As a result, the interfaces are designated `Serial0/3/0` and `Serial0/3/1`. The configuration commands and OSPF network statements were adjusted to reference slot 3.
2. **DHCP dropped by DHCP Snooping:** PC-Guest initially failed to acquire an IP address from R-HQ and assigned itself a 169.254.x.x link-local address. Troubleshooting with `show ip dhcp snooping` showed that DHCP snooping was dropping server responses because trunk uplinks (Port-Channel 1 and Gi0/1) defaulted to untrusted. Applying `ip dhcp snooping trust` on all uplinks towards R-HQ resolved the issue immediately.
3. **OSPF Neighbor Adjacency blocked by passive interface:** The OSPF neighbor between R-HQ and R-B1 remained down after configuration. Because `passive-interface default` had been enabled under OSPF process 1, hello packets were blocked on the serial link. Adding `no passive-interface s0/3/0` and `no passive-interface s0/3/1` allowed OSPF hellos to exchange and neighbor adjacencies reached FULL state.

## 7. Conclusion and future work

The network meets all project requirements. Working through each stage showed how VLANs, routing, services and security fit together, and how important step-by-step testing is.

Possible future improvements:

- Automate configuration backups with Python and Netmiko
- Add monitoring with SNMP and Syslog (Zabbix or LibreNMS)
- Add gateway redundancy (HSRP), a site-to-site IPsec VPN and IPv6 dual-stack
