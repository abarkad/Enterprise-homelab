# TS-03 – Wrong default gateway on a host

[← Back to troubleshooting](README.md) · **Status:** 🔲 Not run yet · **Layer:** L3 · **Devices:** Kali VM

> Back up the configuration before breaking anything (see [labs/README.md](../labs/README.md#before-every-lab)).

## Scenario

*"Kali can reach Metasploitable but nothing outside its own subnet."*

## Fault introduced

```text
# on Kali
sudo ip route replace default via 192.168.50.254
```

## Expected symptoms

Confirm these when you run it; replace with what you actually saw.

| Test | Result |
| --- | --- |
| Kali → Metasploitable | ✅ |
| Kali → `192.168.50.1` | ✅ |
| Kali → `192.168.1.10` | ❌ |
| Kali → `8.8.8.8` | ❌ |

## Investigation

Work bottom-up and change nothing until the cause is proven.

| # | Hypothesis | Command | What I saw | Conclusion |
| - | --- | --- | --- | --- |
| 1 | Interface/IP problem | `ip a` on Kali | _fill in_ | _ruled out / confirmed_ |
| 2 | Default route wrong | `ip route` | _fill in_ | _ruled out / confirmed_ |
| 3 | Gateway resolves? | `ip neigh` – look for `192.168.50.254 FAILED` | _fill in_ | _ruled out / confirmed_ |
| 4 | Path | `traceroute -n 8.8.8.8` – no first hop | _fill in_ | _ruled out / confirmed_ |
| 5 | Switch side healthy | `show ip arp vlan 50` on the 3750G | _fill in_ | _ruled out / confirmed_ |

Key output:

```text
paste the output that proves the root cause
```

<!-- Add your screenshot, then uncomment: ![evidence](screenshots/ts-03-evidence.png) -->

## Root cause

The default route points to `192.168.50.254`, an address nobody owns. ARP for the gateway fails, so every packet for another subnet is dropped by Kali itself. Traffic inside `192.168.50.0/24` still works because it doesn't use the gateway.

## Fix

```text
sudo ip route replace default via 192.168.50.1
# make it permanent in NetworkManager / /etc/network/interfaces
```

## Verification

| Test | Result |
| --- | --- |
| `ip route` → default via 192.168.50.1 | ✅ |
| `traceroute -n 8.8.8.8` first hop 192.168.50.1 | ✅ |

## Lesson

"Local works, remote doesn't" points to the gateway. `ip route` plus `ip neigh` confirms it in seconds, without touching any network device.
