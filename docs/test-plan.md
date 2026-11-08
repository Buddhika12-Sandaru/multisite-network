# Test plan and results

Run each test, tick the result, and save a screenshot with the file name shown.

| ID | Test | Steps | Expected result | Result | Evidence file |
|---|---|---|---|---|---|
| T1 | HQ to Branch 1 | PC-Staff: `ping 192.168.101.x` | Replies received | [x] Pass [ ] Fail | screenshots/T1-staff-to-b1.png |
| T2 | HQ to Branch 2 | PC-Staff: `ping 192.168.102.x` | Replies received | [x] Pass [ ] Fail | screenshots/T2-staff-to-b2.png |
| T3 | Internet via NAT | Any PC: `ping 8.8.8.8`, then R-HQ: `show ip nat translations` | Replies, NAT entries present | [x] Pass [ ] Fail | screenshots/T3-internet-nat.png |
| T4 | Guest isolation | PC-Guest: `ping` PC-Staff address and PC-Server address | Fails (the ACL blocks every internal network) | [x] Pass [ ] Fail | screenshots/T4-guest-blocked.png |
| T5 | Guest internet | PC-Guest: `ping 192.168.30.1`, then `ping 8.8.8.8` | Both reply | [x] Pass [ ] Fail | screenshots/T5-guest-internet.png |
| T6 | Telnet disabled | PC-Staff: `telnet 192.168.99.1` | Connection refused | [x] Pass [ ] Fail | screenshots/T6-telnet-refused.png |
| T7 | SSH works | PC-Staff: `ssh -l admin 192.168.99.1` | Password prompt, then login | [x] Pass [ ] Fail | screenshots/T7-ssh-ok.png |
| T8 | Port security | Connect an extra PC to a secured port, then `show port-security interface fa0/1` | Violation counted, original PC still works | [x] Pass [ ] Fail | screenshots/T8-port-security.png |
| T9 | Redundancy | Remove one EtherChannel cable, ping PC-Server from PC-Staff | Pings still succeed | [x] Pass [ ] Fail | screenshots/T9-etherchannel-failover.png |

## Verification commands (screenshot these too)

| Command | Device | File name |
|---|---|---|
| `show ip interface brief` | R-HQ | screenshots/V1-ip-interface-brief.png |
| `show vlan brief` | SW-HQ1 | screenshots/V2-vlans.png |
| `show interfaces trunk` | SW-HQ1 | screenshots/V3-trunks.png |
| `show etherchannel summary` | SW-HQ1 | screenshots/V4-etherchannel.png |
| `show ip ospf neighbor` | R-HQ | screenshots/V5-ospf-neighbors.png |
| `show ip route` | R-B1 | screenshots/V6-branch-routes.png |
| `show access-lists` | R-HQ | screenshots/V7-acl-hits.png |
| `show ip dhcp snooping` | SW-HQ1 | screenshots/V8-dhcp-snooping.png |

## Summary

Tests passed: 9 / 9
Date tested: 2026-11-08
