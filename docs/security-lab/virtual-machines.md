# Security lab virtual machines

[← Back to README](../../README.md)

Both VMs run on Proxmox, attached to `vmbr0` with VLAN tag 50.

## Kali Linux – VM 101

| Item | Value |
| ---- | ----- |
| IP | `192.168.50.10/24` |
| Gateway | `192.168.50.1` |
| VLAN | 50 |
| Role | Attacker / penetration-testing workstation |

Tools used: Nmap, Metasploit Framework, SearchSploit, FTP enumeration scripts, network troubleshooting utilities.

## Metasploitable 2 – VM 102

| Item | Value |
| ---- | ----- |
| IP | `192.168.50.20/24` |
| Gateway | **none** (intentional) |
| VLAN | 50 |
| OS | Ubuntu 8.04, Linux 2.6.24-16-server, i686 |
| Role | Intentionally vulnerable target |

Metasploitable 2 has no default gateway so its vulnerable services cannot initiate connections outside VLAN 50. See [hardening.md](../hardening.md) for why this alone is not enough and how it will be enforced on the network.
