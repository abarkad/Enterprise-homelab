# TS-02 – ASA missing the return route to VLAN 50

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L3 · **Devices:** ASA 5505, 3750G

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"Kali can reach Proxmox but not the Internet. Nothing changed on Kali."*

## Fault introduced

```text
no route inside 192.168.50.0 255.255.255.0 192.168.1.2
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| Kali → `192.168.1.2` (switch) | ✅ |
| Kali → Proxmox `192.168.1.10` | ✅ |
| Kali → ASA inside `192.168.1.1` | ❌ |
| Kali → `8.8.8.8` | ❌ |
| PC → Internet | ✅ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | Problem only affects VLAN 50 | Compare PC vs Kali to the Internet | _fill in_ | _ruled out / confirmed_ |
| 2 | Switch default route missing | `show ip route` on 3750G (0.0.0.0/0 via 192.168.1.1) | _fill in_ | _ruled out / confirmed_ |
| 3 | NAT rule missing | `show nat detail` on ASA | _fill in_ | _ruled out / confirmed_ |
| 4 | ASA doesn't know how to reach 192.168.50.0/24 | `show route 192.168.50.10` on ASA | _fill in_ | _ruled out / confirmed_ |
| 5 | Simulate the flow | `packet-tracer input inside icmp 192.168.50.10 8 0 8.8.8.8 detailed` | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-02-evidence.png) -->

## Root cause

The ASA only has a connected route for `192.168.1.0/24`. Without the static route, replies to `192.168.50.0/24` follow the default route out the **outside** interface, so return traffic never comes back to VLAN 50. Hosts in VLAN 1 aren't affected because the network is directly connected to the ASA.

## Fix

```text
route inside 192.168.50.0 255.255.255.0 192.168.1.2
```

## Verification

| Test | Result |
| --- | --- |
| `show route 192.168.50.10` points to 192.168.1.2 via inside | ✅ |
| `packet-tracer` result: allow | ✅ |
| Kali → `8.8.8.8` | ✅ |

## Lesson

Routing is two-way: when a subnet sits behind a Layer 3 switch, every upstream device needs a route back to it. When only one subnet is broken, compare it with a working one to isolate the faulty device quickly.
