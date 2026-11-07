# Isolation and hardening

[← Back to README](../README.md)

## Current state

- Lab VMs sit in their own VLAN (50).
- Metasploitable 2 has **no default gateway**.

## Why that is not enough

- **Removing a host's gateway is not a security control.** Once an attacker has root on Metasploitable (as in [Exercise 01](penetration-testing/01-vsftpd-2.3.4/README.md)), they can add a route themselves. Since the ASA performs PAT for `192.168.50.0/24`, the host would then have Internet access.
- **VLAN 50 can reach the management network.** The 3750G routes between VLAN 50 and VLAN 1 without filtering, so a compromised lab host could reach the Proxmox web UI (`192.168.1.10:8006`), the ASA and the switch.

## Planned controls

### 1. ACL on the VLAN 50 SVI (3750G)

```text
ip access-list extended LAB-IN
 remark Metasploitable may not leave VLAN 50
 deny   ip host 192.168.50.20 any
 remark Lab may not reach management
 deny   ip 192.168.50.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip 192.168.50.0 0.0.0.255 any
!
interface Vlan50
 ip access-group LAB-IN in
```

Traffic between Kali and Metasploitable stays inside the subnet (switched, not routed), so the exercises still work.

### 2. Defense in depth on the ASA

Remove `192.168.50.20` from PAT, or add an inside ACL denying it, so the Internet stays blocked even if the switch ACL is changed.

### 3. Dedicated management VLAN

Move management (ASA, switch, Proxmox, PC) off VLAN 1 into a dedicated VLAN (e.g. VLAN 10), and restrict access to the management interfaces to that VLAN.

### 4. Detection

Add logging and a network IDS to detect activity such as the vsftpd backdoor connection on TCP 6200.

## Verification once applied

```text
show access-lists LAB-IN
```

From Metasploitable, after adding a route manually: `ping 8.8.8.8` should fail.
From Kali: `curl -k https://192.168.1.10:8006` should fail, and `ping 192.168.50.20` should succeed.
