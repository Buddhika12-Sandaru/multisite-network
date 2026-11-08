# Troubleshooting log

This log documents issues encountered while building and verifying the multi-site network lab in Cisco Packet Tracer, along with the diagnostic steps and solutions applied.

## Log

| # | Symptom | How I investigated | Cause | Fix | Lesson |
|---|---|---|---|---|---|
| 1 | Serial0/0/0 and Serial0/0/1 were missing in the port list of R-HQ (only Serial0/3/0 and Serial0/3/1 were offered) | Looked at the port list shown while cabling | The serial module sits in slot 3, so the ports are numbered 0/3/x instead of 0/0/x | Used Serial0/3/0 and Serial0/3/1 for the cables and changed the interface names in the IP and OSPF commands to match | Port names follow the slot number, so check the real port list before typing commands |
| 2 | PC-Guest (VLAN 30) did not get an IP address from DHCP and defaulted to APIPA (169.254.x.x) | Checked `show ip dhcp binding` on R-HQ and `show ip dhcp snooping` on SW-HQ1 / SW-HQ2 | DHCP snooping was active on VLAN 30, but the trunk interfaces connecting towards the DHCP server were untrusted | Added `ip dhcp snooping trust` to Port-Channel 1 and Gi0/1 on the switches towards R-HQ | When DHCP snooping is enabled, switch ports default to untrusted; explicitly configure `trust` on uplinks facing valid DHCP servers |
| 3 | OSPF neighbor adjacency did not form between R-HQ and R-B1 over the serial WAN link | Ran `show ip ospf neighbor` on R-HQ (output was empty) and verified interface status | `passive-interface default` was enabled under OSPF, blocking Hello packets across all interfaces | Configured `no passive-interface s0/3/0` and `no passive-interface s0/3/1` under `router ospf 1` on R-HQ | When using `passive-interface default` for LAN security, explicitly enable routing hellos on WAN links |

## Common pitfalls in this topology

- No serial ports on the router (module not installed, or installed in a different slot, so port names differ)
- DCE end of the serial cable is on the wrong router, so there is no clock rate configured
- Native VLAN or allowed VLANs differ between the two ends of a trunk
- Router-on-a-stick subinterface missing `encapsulation dot1Q`
- OSPF neighbour missing because a serial port is still passive
- NAT inside/outside not set on every interface
- DHCP snooping blocks replies because the uplink is not trusted
- SSH refused because the domain name or RSA key was missing
