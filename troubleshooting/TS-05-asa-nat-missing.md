# TS-05 – NAT rule missing for a subnet

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L3/NAT · **Devices:** ASA 5505

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"Kali reaches every internal device, the ASA route is fine, but the Internet doesn't work."*

## Fault introduced

```text
object network LAB-NET
 no nat (inside,outside) dynamic interface
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| Kali → ASA inside | ✅ |
| Kali → `192.168.2.1` (Giga Hub) | ❌ |
| Kali → `8.8.8.8` | ❌ |
| PC → Internet | ✅ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | Route back present | `show route 192.168.50.10` | _fill in_ | _ruled out / confirmed_ |
| 2 | NAT rules | `show nat detail` – no rule for 192.168.50.0/24 | _fill in_ | _ruled out / confirmed_ |
| 3 | Translations | `show xlate` – none for 192.168.50.x | _fill in_ | _ruled out / confirmed_ |
| 4 | Simulation | `packet-tracer input inside icmp 192.168.50.10 8 0 8.8.8.8 detailed` – no NAT phase | _fill in_ | _ruled out / confirmed_ |
| 5 | What leaves the ASA | `capture CAPO interface outside match icmp any host 8.8.8.8` then `show capture CAPO` | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-05-evidence.png) -->

## Root cause

Without a NAT rule, the ASA forwards Kali's traffic to the outside **untranslated** (source `192.168.50.10`). The Giga Hub has no route to `192.168.50.0/24`, so replies never come back. The capture on the outside interface shows the private source address leaving the firewall.

## Fix

```text
object network LAB-NET
 nat (inside,outside) dynamic interface
```

## Verification

| Test | Result |
| --- | --- |
| `show xlate` shows 192.168.50.10 → 192.168.2.29 | ✅ |
| Kali → `8.8.8.8` | ✅ |
| `no capture CAPO` (cleanup) | ✅ |

## Lesson

Packet captures on both sides of a firewall show what really happens. Comparing the source address before and after the ASA proves whether NAT is applied.
