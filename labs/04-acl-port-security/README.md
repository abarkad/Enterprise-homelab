# Lab 04 – ACLs and port security

[← Back to labs](../README.md) · **Status:** 🔲 Planned

## Objective

1. Enforce the VLAN 50 isolation designed in [docs/hardening.md](../../docs/hardening.md) with an extended ACL.
2. Restrict switch management (SSH/Telnet) to VLAN 1 with a VTY `access-class`.
3. Enable port security on an access port, trigger a violation, and recover from **err-disable**.

> ⚠️ Run this from a console cable if possible. A wrong ACL on the VTY lines can lock you out of remote management.

## Part A – VLAN 50 isolation ACL

```text
conf t
ip access-list extended LAB-IN
 remark Metasploitable may not leave VLAN 50
 deny   ip host 192.168.50.20 any log
 remark Lab may not reach management
 deny   ip 192.168.50.0 0.0.0.255 192.168.1.0 0.0.0.255 log
 permit ip 192.168.50.0 0.0.0.255 any
!
interface Vlan50
 ip access-group LAB-IN in
end
```

| Test (from) | Command | Expected | Evidence |
| ----------- | ------- | -------- | -------- |
| Kali | `ping 192.168.50.20` | ✅ Works (same subnet, switched, not filtered) | |
| Kali | `ping 8.8.8.8` | ✅ Works | |
| Kali | `curl -k https://192.168.1.10:8006` | ❌ Blocked | |
| Kali | `ping 192.168.1.1` | ❌ Blocked | |
| Metasploitable (after adding a route to `.1` manually) | `ping 8.8.8.8` | ❌ Blocked | |
| Switch | `show access-lists LAB-IN` | Hit counters on the deny lines | |

## Part B – Restrict management access

```text
conf t
ip access-list standard MGMT-ONLY
 permit 192.168.1.0 0.0.0.255
 deny   any log
!
line vty 0 15
 access-class MGMT-ONLY in
end
```

Test: SSH/Telnet to `192.168.1.2` from the PC (allowed) and to `192.168.50.1` from Kali (refused). Check with `show access-lists MGMT-ONLY`.

## Part C – Port security

Use a **spare access port** (e.g. `Gi1/0/10`), not the PC's management port.

```text
conf t
interface Gi1/0/10
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
end
```

1. Connect device A and generate traffic. Check `show port-security interface Gi1/0/10` (sticky MAC learned).
2. Swap to device B. The port should go **err-disabled**.
3. Diagnose: `show interfaces status err-disabled`, `show port-security`, `show logging | include PSECURE`.
4. Recover: `shutdown` / `no shutdown` on the port (or `errdisable recovery cause psecure-violation`).
5. Optional: repeat with `violation restrict` and compare the behaviour (port stays up, violation counter increments).

## Break / fix

**Fault:** put the ACL on `Vlan50` as `out` instead of `in`. Explain which traffic an outbound ACL on an SVI filters, and why everything routed *into* VLAN 50 (including replies from the Internet to Kali) now hits the implicit `deny any`.

Use the [break/fix template](../../troubleshooting/TEMPLATE.md).

## What I learned

_To fill in after completing the lab._
