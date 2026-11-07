# TS-04 – VLAN deleted – SVI up/down

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L2/L3 · **Devices:** 3750G

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"All of VLAN 50 is unreachable from the management network after someone "cleaned up unused VLANs"."*

## Fault introduced

```text
no vlan 50
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| PC → `192.168.50.1` | ❌ |
| PC → Kali | ❌ |
| Kali → Metasploitable | ✅ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | SVI state | `show ip interface brief \| include Vlan50` → up/**down** | _fill in_ | _ruled out / confirmed_ |
| 2 | VLAN exists? | `show vlan brief` – VLAN 50 missing | _fill in_ | _ruled out / confirmed_ |
| 3 | Trunk forwarding | `show interfaces trunk` – 50 allowed but not in *active* list | _fill in_ | _ruled out / confirmed_ |
| 4 | Interface config intact? | `show run interface Vlan50` – IP still configured | _fill in_ | _ruled out / confirmed_ |
| 5 | Logs | `show logging \| include Vlan50` | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-04-evidence.png) -->

## Root cause

The VLAN was deleted from the VLAN database while the SVI and trunk configuration remained. With no VLAN, no port forwards VLAN 50 and the SVI goes **up/down**. The configuration looks correct when you only read `show run interface Vlan50`.

## Fix

```text
vlan 50
 name SECURITY-LAB
```

## Verification

| Test | Result |
| --- | --- |
| `show vlan brief` – VLAN 50 active | ✅ |
| Vlan50 up/up | ✅ |
| PC → Kali | ✅ |

## Lesson

up/down on an SVI means the VLAN has no active forwarding port: check `show vlan brief` and `show interfaces trunk` before touching IP settings.
