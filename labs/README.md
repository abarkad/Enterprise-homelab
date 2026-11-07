# Labs

[← Back to README](../README.md)

Small, focused labs built on top of the main homelab. Each one covers a single topic end to end: objective, design, configuration, verification and a break/fix exercise.

| # | Lab | Devices | Topics | Status |
| - | --- | ------- | ------ | ------ |
| 01 | [VLANs and 802.1Q trunking](01-vlan-trunking/README.md) | 3750G, Proxmox | VLAN creation, SVIs, access ports, trunk allowed lists, VLAN-tagged VMs | 🔲 Planned |
| 02 | [NAT, PAT and port forwarding](02-nat-port-forwarding/README.md) | ASA 5505 | Object NAT, dynamic PAT, static PAT, outside ACLs, `show xlate`, `packet-tracer` | 🔲 Planned |
| 03 | [DHCP server and relay](03-dhcp-server-relay/README.md) | 3750G, Linux VM | DHCP pools, exclusions, DORA, `ip helper-address` | 🔲 Planned |
| 04 | [ACLs and port security](04-acl-port-security/README.md) | 3750G | Extended ACLs, VTY access-class, port security, err-disable recovery | 🔲 Planned |
| 05 | [IPv6 dual stack](05-ipv6-dual-stack/README.md) | 3750G, VMs | SDM templates, IPv6 SVIs, SLAAC, IPv6 inter-VLAN routing | 🔲 Planned |

Status legend: 🔲 Planned · 🟡 In progress · ✅ Completed (with evidence)

## Addressing used across the labs

| VLAN | Name | Network | Gateway (3750G SVI) | Introduced in |
| ---- | ---- | ------- | ------------------- | ------------- |
| 1 | default (management) | `192.168.1.0/24` | ASA `192.168.1.1` | Base lab |
| 20 | USERS | `192.168.20.0/24` | `192.168.20.1` | Lab 01 |
| 50 | SECURITY-LAB | `192.168.50.0/24` | `192.168.50.1` | Base lab |

## Before every lab

```text
! Save a known-good configuration you can roll back to
copy running-config startup-config
copy running-config flash:pre-labXX.cfg
```

ASA:

```text
write memory
copy running-config disk0:/pre-labXX.cfg
```

**Rollback:** `configure replace flash:pre-labXX.cfg` (IOS) or reload without saving.

## Lab write-up structure

Every lab follows the same structure so a reader can skim them quickly:

1. **Objective:** what the lab proves
2. **Topology:** what changes compared with the base lab
3. **Configuration:** the commands, step by step
4. **Verification:** commands plus the evidence (screenshots or output)
5. **Break / fix:** one fault introduced on purpose, then diagnosed
6. **What I learned**
