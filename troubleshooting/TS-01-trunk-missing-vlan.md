# TS-01 – VLAN missing from the trunk allowed list

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L2 · **Devices:** 3750G, Proxmox

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"After a change on the switch, Kali can no longer reach the Internet or the management network, but it can still reach Metasploitable."*

## Fault introduced

```text
interface GigabitEthernet1/0/24
 switchport trunk allowed vlan remove 50
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| Kali → Metasploitable (`192.168.50.20`) | ✅ still works |
| Kali → gateway `192.168.50.1` | ❌ |
| Kali → Internet | ❌ |
| PC → Proxmox UI | ✅ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | Kali IP/gateway wrong | `ip a`, `ip route` on Kali | _fill in_ | _ruled out / confirmed_ |
| 2 | Gateway not answering ARP | `ip neigh` on Kali | _fill in_ | _ruled out / confirmed_ |
| 3 | SVI down | `show ip interface brief \| include Vlan50` | _fill in_ | _ruled out / confirmed_ |
| 4 | VLAN not carried on trunk | `show interfaces trunk` | _fill in_ | _ruled out / confirmed_ |
| 5 | No MACs learned in VLAN 50 | `show mac address-table vlan 50` | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-01-evidence.png) -->

## Root cause

VLAN 50 was removed from the trunk's allowed list, so tagged VLAN 50 frames from Proxmox are dropped at Gi1/0/24. With no other active port in VLAN 50, the SVI goes **up/down** (autostate) and `192.168.50.0/24` disappears from the routing table. Kali and Metasploitable still talk because they share the same Proxmox bridge: that traffic never reaches the switch.

## Fix

```text
interface GigabitEthernet1/0/24
 switchport trunk allowed vlan add 50
```

## Verification

| Test | Result |
| --- | --- |
| `show interfaces trunk` lists 1,50 (and 20 if Lab 01 is done) | ✅ |
| Vlan50 up/up | ✅ |
| Kali → `8.8.8.8` | ✅ |

## Lesson

Traffic between two VMs on the same host can hide an upstream Layer 2 failure. Always use `allowed vlan add/remove`, never retype the full list, and save `show interfaces trunk` output before and after any trunk change.
