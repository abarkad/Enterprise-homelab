# Lab 05 – IPv6 dual stack

[← Back to labs](../README.md) · **Status:** 🔲 Planned · **Requires:** [Lab 01](../01-vlan-trunking/README.md)

## Objective

Run IPv4 and IPv6 side by side on the internal VLANs, with hosts getting addresses through **SLAAC** and the 3750G routing IPv6 between VLANs.

Scope: internal IPv6 only, using a Unique Local Address (ULA) prefix. Internet IPv6 through the ASA is out of scope.

## Addressing

| VLAN | IPv4 | IPv6 prefix | Gateway (3750G) |
| ---- | ---- | ----------- | --------------- |
| 20 USERS | `192.168.20.0/24` | `fd00:20::/64` | `fd00:20::1` |
| 50 SECURITY-LAB | `192.168.50.0/24` | `fd00:50::/64` | `fd00:50::1` |

## Configuration

### 1. Enable an IPv6-capable SDM template (needs a reload)

```text
show sdm prefer
conf t
sdm prefer dual-ipv4-and-ipv6 default
end
write memory
reload
```

> On the 3750G, IPv6 routing commands aren't available until the dual-stack SDM template is active.

### 2. IPv6 routing and SVIs

```text
conf t
ipv6 unicast-routing
!
interface Vlan20
 ipv6 address fd00:20::1/64
 ipv6 enable
!
interface Vlan50
 ipv6 address fd00:50::1/64
 ipv6 enable
end
```

SVIs send Router Advertisements by default, so Linux hosts will build a SLAAC address from the prefix.

## Verification

| Check | Command | Expected | Evidence |
| ----- | ------- | -------- | -------- |
| SDM template | `show sdm prefer` | dual-ipv4-and-ipv6 | |
| SVI addresses | `show ipv6 interface brief` | Global `fd00:..::1` + link-local `fe80::` | |
| Routes | `show ipv6 route` | Both `/64` connected | |
| SLAAC on host | `ip -6 addr` | `fd00:20::…/64` address | |
| Default route | `ip -6 route` | default via `fe80::…` (router link-local) | |
| Neighbours | `show ipv6 neighbors` | Host addresses | |
| Inter-VLAN | VLAN 20 VM: `ping -6 fd00:50::<kali>` | Replies | |
| NDP capture | `sudo tcpdump -ni eth0 icmp6` | RS / RA / NS / NA | |

## Break / fix

**Fault:** `no ipv6 unicast-routing`. Observe that inter-VLAN IPv6 pings fail immediately, the switch stops sending Router Advertisements, and hosts keep their IPv6 default route only until its router lifetime expires (`ip -6 route` shows the remaining time). Explain each of these.

Use the [break/fix template](../../troubleshooting/TEMPLATE.md).

## What I learned

_To fill in after completing the lab._
