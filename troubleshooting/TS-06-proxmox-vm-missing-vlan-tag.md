# TS-06 – Proxmox VM missing its VLAN tag

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L2 (virtual) · **Devices:** Proxmox, 3750G

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"After rebuilding Kali's network card, it can't reach its gateway or Metasploitable, even though its IP config is unchanged."*

## Fault introduced

```text
# on Proxmox – remove the VLAN tag from Kali's NIC
qm set 101 --net0 virtio=<KALI-MAC>,bridge=vmbr0
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| Kali → `192.168.50.1` | ❌ |
| Kali → Metasploitable | ❌ |
| Metasploitable → VLAN 50 SVI | ✅ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | Kali IP config | `ip a`, `ip route` – unchanged | _fill in_ | _ruled out / confirmed_ |
| 2 | ARP | `ip neigh` – gateway FAILED | _fill in_ | _ruled out / confirmed_ |
| 3 | VM NIC config | `qm config 101 \| grep net0` – no `tag=50` | _fill in_ | _ruled out / confirmed_ |
| 4 | Bridge VLAN membership | `bridge vlan show` on Proxmox | _fill in_ | _ruled out / confirmed_ |
| 5 | Which VLAN the switch sees Kali in | `show mac address-table address <KALI-MAC>` – VLAN 1 | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-06-evidence.png) -->

## Root cause

Without `tag=50`, Kali's traffic leaves the bridge untagged and lands in VLAN 1 (the trunk's native VLAN). It still uses a `192.168.50.x` address, so it's in the wrong broadcast domain: its ARP requests never reach the VLAN 50 gateway or Metasploitable.

## Fix

```text
qm set 101 --net0 virtio=<KALI-MAC>,bridge=vmbr0,tag=50
```

## Verification

| Test | Result |
| --- | --- |
| `qm config 101` shows `tag=50` | ✅ |
| Switch sees Kali's MAC in VLAN 50 | ✅ |
| Kali → gateway and Metasploitable | ✅ |

## Lesson

In virtual networks, the VLAN tag lives on the VM's NIC. When a host has the right IP but can't ARP anything, check which VLAN the switch actually learned its MAC in.
