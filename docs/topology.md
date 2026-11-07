# Topology

[← Back to README](../README.md)

## Physical topology

```mermaid
flowchart LR
    HUB["Bell Giga Hub"] -- "Ethernet" --- ASAo["ASA 5505<br/>outside (Vlan2)"]
    ASAi["ASA 5505<br/>inside (Vlan1)"] -- "Ethernet" --- SW["Cisco 3750G-24TS"]
    SW -- "Access · VLAN 1" --- PC["Desktop PC"]
    SW -- "Gi1/0/24 · 802.1Q trunk" --- R320["Dell R320<br/>nic0 → vmbr0"]
    ASAo -.- ASAi
```

## Logical topology

```mermaid
flowchart TD
    INET([Internet])
    OUT["Outside · 192.168.2.0/24<br/>Giga Hub .1 · ASA .29"]
    V1["VLAN 1 · Management · 192.168.1.0/24<br/>ASA .1 · 3750G .2 · Proxmox .10 · PC"]
    V50["VLAN 50 · SECURITY-LAB · 192.168.50.0/24<br/>SVI .1 · Kali .10 · Metasploitable .20"]

    INET --- OUT
    OUT -- "ASA PAT" --- V1
    V1 -- "3750G inter-VLAN routing" --- V50
```

## Traffic paths

| Flow | Path |
| ---- | ---- |
| PC → Proxmox | PC → 3750G (VLAN 1) → Proxmox `192.168.1.10` |
| Kali → Metasploitable | Same subnet, switched inside VLAN 50 on `vmbr0` |
| Kali → Internet | Kali → SVI `192.168.50.1` → 3750G default route → ASA `192.168.1.1` → PAT to `192.168.2.29` → Giga Hub |
| Metasploitable → Internet | Blocked by design: no default gateway |

## Initial topology

The original diagram, drawn before VLAN 50 existed, while the switch was still at factory defaults:

![Initial physical topology](../screenshots/initial-physical-topology.png)
