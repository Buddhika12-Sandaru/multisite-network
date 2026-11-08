# Design document

## 1. Requirements

| # | Requirement |
|---|---|
| R1 | Three sites: one head office (HQ) and two branches (B1, B2) |
| R2 | HQ users must be split into Staff, Servers, Guest and Management groups |
| R3 | All sites must reach each other and a simulated internet |
| R4 | Guests may use the internet but must not reach internal networks |
| R5 | Devices must be managed only with encrypted remote access (SSH) |
| R6 | The link between the two HQ switches must tolerate one cable failure |
| R7 | Access ports must be protected against unknown devices and fake DHCP servers |

## 2. Design decisions and reasons

| Decision | Reason |
|---|---|
| VLANs with router-on-a-stick at HQ | Cheap and easy to explain: one trunk cable, one gateway address per VLAN |
| Native VLAN 99 on trunks | Keeps untagged traffic off the user VLANs |
| EtherChannel (LACP) between SW-HQ1 and SW-HQ2 | Doubles bandwidth and survives one cable failure |
| OSPF area 0 | Routes are learned automatically, no static route per branch |
| Default route on R-HQ, advertised with `default-information originate` | Branches learn the way to the internet without their own static route |
| DHCP on the router of each site | No separate server needed; fixed address only for the server |
| NAT overload on R-HQ | One public address shared by all internal networks |
| Extended ACL on the guest subinterface | Blocks all private ranges, then allows everything else (internet) |
| SSH only, Telnet disabled | Telnet sends passwords as clear text |
| Port security, DHCP snooping, unused ports shut | Reduces risk from plugged-in rogue devices |

## 3. Addressing plan

### Router interfaces

| Device | Interface | IP address | Mask |
|---|---|---|---|
| R-HQ | Gi0/0.10 (Staff) | 192.168.10.1 | 255.255.255.0 |
| R-HQ | Gi0/0.20 (Servers) | 192.168.20.1 | 255.255.255.0 |
| R-HQ | Gi0/0.30 (Guest) | 192.168.30.1 | 255.255.255.0 |
| R-HQ | Gi0/0.99 (Management) | 192.168.99.1 | 255.255.255.0 |
| R-HQ | Se0/3/0 (to R-B1) | 10.0.12.1 | 255.255.255.252 |
| R-HQ | Se0/3/1 (to R-B2) | 10.0.13.1 | 255.255.255.252 |
| R-HQ | Gi0/1 (to ISP) | 203.0.113.2 | 255.255.255.252 |
| R-ISP | Gi0/0 | 203.0.113.1 | 255.255.255.252 |
| R-ISP | Loopback0 (simulated internet) | 8.8.8.8 | 255.255.255.255 |
| R-B1 | Gi0/0 (LAN) | 192.168.101.1 | 255.255.255.0 |
| R-B1 | Se0/3/0 (to R-HQ) | 10.0.12.2 | 255.255.255.252 |
| R-B2 | Gi0/0 (LAN) | 192.168.102.1 | 255.255.255.0 |
| R-B2 | Se0/3/0 (to R-HQ) | 10.0.13.2 | 255.255.255.252 |

> In this lab the serial module sits in slot 3, so the serial ports are named Serial0/3/x (not Serial0/0/x).

### Switches and end devices

| Device | Address | Method |
|---|---|---|
| SW-HQ1 (VLAN 99) | 192.168.99.2/24, gateway 192.168.99.1 | static |
| SW-HQ2 (VLAN 99) | 192.168.99.3/24, gateway 192.168.99.1 | static |
| PC-Staff | 192.168.10.x | DHCP |
| PC-Guest | 192.168.30.x | DHCP |
| PC-Server | 192.168.20.10/24, gateway 192.168.20.1, DNS 8.8.8.8 | static |
| PC-B1 | 192.168.101.x | DHCP |
| PC-B2 | 192.168.102.x | DHCP |

## 4. VLAN plan

| VLAN | Name | Subnet | Switch ports |
|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | SW-HQ1 Fa0/1 |
| 20 | SERVERS | 192.168.20.0/24 | SW-HQ2 Fa0/1 |
| 30 | GUEST | 192.168.30.0/24 | SW-HQ2 Fa0/2 |
| 99 | MGMT | 192.168.99.0/24 | trunks only (native) |

## 5. Cable map

| # | From | To | Cable |
|---|---|---|---|
| 1 | R-HQ Gi0/0 | SW-HQ1 Gi0/1 | Copper straight-through |
| 2 | SW-HQ1 Fa0/23 | SW-HQ2 Fa0/23 | Copper cross-over |
| 3 | SW-HQ1 Fa0/24 | SW-HQ2 Fa0/24 | Copper cross-over |
| 4 | R-HQ Gi0/1 | R-ISP Gi0/0 | Copper cross-over |
| 5 | R-HQ Se0/3/0 | R-B1 Se0/3/0 | Serial DCE (DCE end at R-HQ) |
| 6 | R-HQ Se0/3/1 | R-B2 Se0/3/0 | Serial DCE (DCE end at R-HQ) |
| 7 | R-B1 Gi0/0 | SW-B1 Fa0/1 | Copper straight-through |
| 8 | R-B2 Gi0/0 | SW-B2 Fa0/1 | Copper straight-through |
| 9 | SW-B1 Fa0/2 | PC-B1 | Copper straight-through |
| 10 | SW-B2 Fa0/2 | PC-B2 | Copper straight-through |
| 11 | SW-HQ1 Fa0/1 | PC-Staff | Copper straight-through |
| 12 | SW-HQ2 Fa0/1 | PC-Server | Copper straight-through |
| 13 | SW-HQ2 Fa0/2 | PC-Guest | Copper straight-through |

## 6. Security design

- **Guest isolation:** ACL `GUEST-IN` on R-HQ Gi0/0.30 permits the guest gateway, denies all 192.168.0.0/16, permits everything else.
- **Management access:** SSH version 2 with local users, Telnet disabled, console password set, passwords encrypted.
- **Access layer:** sticky-MAC port security (one device per port, violation mode restrict), DHCP snooping on VLANs 10, 20 and 30 with trust only on uplinks, all unused ports administratively shut down.
- **Lab credentials:** these are lab-only values and must never be reused on real equipment.
