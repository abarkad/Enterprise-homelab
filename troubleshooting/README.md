# Troubleshooting scenarios

[← Back to README](../README.md)

Break/fix exercises: I introduce one fault on purpose, then diagnose it like a support ticket, without knowing which device is at fault. Each write-up records the symptoms, the hypotheses I tested in order, the evidence, the root cause and the fix.

For the real problems I hit while building the lab, see [docs/troubleshooting.md](../docs/troubleshooting.md).

## Method

```text
1. Define the problem      what works, what doesn't, since when
2. Gather evidence         ping, traceroute, show commands, captures
3. Bottom-up hypotheses    L1 → L2 → L3 → L4 → L7, one at a time
4. Prove the root cause    before changing anything
5. Fix, then verify        same tests that failed must now pass
6. Document                so the next person fixes it faster
```

## Scenarios

| ID | Scenario | Layer | Devices | Status |
| -- | -------- | ----- | ------- | ------ |
| [TS-01](TS-01-trunk-missing-vlan.md) | VLAN missing from the trunk allowed list | L2 | 3750G, Proxmox | 🔲 |
| [TS-02](TS-02-asa-missing-return-route.md) | ASA missing the return route to VLAN 50 | L3 | ASA 5505, 3750G | 🔲 |
| [TS-03](TS-03-wrong-default-gateway.md) | Wrong default gateway on a host | L3 | Kali VM | 🔲 |
| [TS-04](TS-04-vlan-deleted-svi-down.md) | VLAN deleted – SVI up/down | L2/L3 | 3750G | 🔲 |
| [TS-05](TS-05-asa-nat-missing.md) | NAT rule missing for a subnet | L3/NAT | ASA 5505 | 🔲 |
| [TS-06](TS-06-proxmox-vm-missing-vlan-tag.md) | Proxmox VM missing its VLAN tag | L2 (virtual) | Proxmox, 3750G | 🔲 |

Each lab in [`labs/`](../labs/README.md) also has its own break/fix exercise.

New scenario? Copy [TEMPLATE.md](TEMPLATE.md).
