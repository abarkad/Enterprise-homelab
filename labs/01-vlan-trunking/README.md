# Lab 01 – VLANs and 802.1Q trunking

[← Back to labs](../README.md) · **Status:** 🔲 Planned

## Objective

Add a new **USERS** VLAN (VLAN 20), route it on the 3750G, carry it to Proxmox over the existing trunk, and prove that hosts in VLAN 20 are isolated at Layer 2 but routed at Layer 3.

## Topology changes

| Item | Value |
| ---- | ----- |
| New VLAN | 20 – USERS – `192.168.20.0/24` |
| SVI | `Vlan20` – `192.168.20.1/24` |
| Test host A | Proxmox VM (Linux, network device tagged **20**) – `192.168.20.10` |
| Test host B (optional) | Laptop on access port `Gi1/0/10` (VLAN 20) |
| Trunk | `Gi1/0/24` allowed VLANs: 1, 20, 50 |

## Configuration

### 1. Create the VLAN and SVI

```text
conf t
vlan 20
 name USERS
exit
interface Vlan20
 description USERS
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
```

### 2. Add VLAN 20 to the trunk

```text
interface GigabitEthernet1/0/24
 switchport trunk allowed vlan add 20
```

> Use `add`. Typing `switchport trunk allowed vlan 20` would **replace** the list and cut VLAN 1 (Proxmox management) and VLAN 50.

### 3. Access port for a physical host (optional)

```text
interface GigabitEthernet1/0/10
 description USERS-TEST
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
end
write memory
```

### 4. Proxmox VM

Create a small Linux VM (or clone an existing one), set its network device to `vmbr0` with **VLAN tag 20**, and configure:

```text
IP      192.168.20.10/24
Gateway 192.168.20.1
```

## Verification

| Check | Command | Expected | Evidence |
| ----- | ------- | -------- | -------- |
| VLAN exists | `show vlan brief` | VLAN 20 USERS active | `screenshots/` |
| Trunk carries VLAN 20 | `show interfaces trunk` | `1,20,50` allowed and forwarding on Gi1/0/24 | |
| SVI up | `show ip interface brief \| include Vlan` | Vlan20 up/up | |
| Route present | `show ip route connected` | `192.168.20.0/24` connected | |
| MAC learned | `show mac address-table vlan 20` | VM MAC on Gi1/0/24 | |
| Gateway reachable | VM: `ping 192.168.20.1` | Replies | |
| Inter-VLAN routing | VM: `ping 192.168.50.10` | Replies (routed by 3750G) | |
| Path | VM: `traceroute 192.168.50.10` | First hop `192.168.20.1` | |

> **Internet from VLAN 20 is out of scope here.** The ASA doesn't know `192.168.20.0/24` yet; that's handled in [Lab 02](../02-nat-port-forwarding/README.md).

## Break / fix

**Fault:** remove VLAN 20 from the trunk.

```text
interface Gi1/0/24
 switchport trunk allowed vlan remove 20
```

Document: the symptom on the VM, what `show interfaces trunk` and `show mac address-table vlan 20` show, why the SVI state changes (or doesn't if a physical access port in VLAN 20 is still up), and the fix. Use the [break/fix template](../../troubleshooting/TEMPLATE.md).

## What I learned

_To fill in after completing the lab._
