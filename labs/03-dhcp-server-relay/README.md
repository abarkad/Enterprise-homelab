# Lab 03 – DHCP server and DHCP relay

[← Back to labs](../README.md) · **Status:** 🔲 Planned · **Requires:** [Lab 01](../01-vlan-trunking/README.md)

## Objective

- **Part A:** make the 3750G the DHCP server for VLAN 20.
- **Part B:** move DHCP to a central Linux server in VLAN 1 and have the 3750G **relay** requests (`ip helper-address`), like an enterprise network.
- Capture and explain the **DORA** exchange (Discover, Offer, Request, Acknowledge).

## Part A – 3750G as DHCP server

```text
conf t
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp excluded-address 192.168.20.50
!
ip dhcp pool VLAN20-USERS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 1.1.1.1 8.8.8.8
 domain-name lab.local
 lease 0 8
end
```

Set the VLAN 20 test VM to DHCP (`dhclient -v` or NetworkManager).

| Check | Command | Expected | Evidence |
| ----- | ------- | -------- | -------- |
| Lease on client | `ip a`, `ip route` | `192.168.20.21`+ and default route `192.168.20.1` | |
| Bindings | `show ip dhcp binding` | Client MAC → IP | |
| Pool usage | `show ip dhcp pool` | Leased addresses count | |
| Conflicts | `show ip dhcp conflict` | Empty | |
| DORA | `debug ip dhcp server events` (turn off with `undebug all`) | DISCOVER, OFFER, REQUEST, ACK | |
| Packet capture | On the VM: `sudo tcpdump -ni eth0 port 67 or port 68` | The 4 DORA packets | |

## Part B – Central DHCP server + relay

| Item | Value |
| ---- | ----- |
| DHCP server | Linux VM in VLAN 1 (untagged on `vmbr0`) – `192.168.1.50` |
| Software | `isc-dhcp-server` or `dnsmasq` |

Example `dnsmasq` scope (`/etc/dnsmasq.d/vlan20.conf`):

```text
dhcp-range=set:vlan20,192.168.20.100,192.168.20.200,255.255.255.0,8h
dhcp-option=tag:vlan20,option:router,192.168.20.1
dhcp-option=tag:vlan20,option:dns-server,1.1.1.1
```

Disable the switch pool and enable relay:

```text
conf t
no ip dhcp pool VLAN20-USERS
interface Vlan20
 ip helper-address 192.168.1.50
end
```

| Check | Command | Expected | Evidence |
| ----- | ------- | -------- | -------- |
| Client lease | `ip a` on VM | `192.168.20.100`–`200` | |
| Relay working | On the DHCP server: `sudo tcpdump -ni eth0 port 67` | Unicast packets from `192.168.20.1` (giaddr) | |
| Server log | `journalctl -u dnsmasq` | DHCPACK to the client MAC | |

**Key point to explain:** the relay fills in the **giaddr** field with `192.168.20.1`, and that's how the server picks the right scope.

## Break / fix

- **Fault A:** remove `ip helper-address`. Explain why the broadcast never leaves VLAN 20.
- **Fault B:** point the helper at a wrong IP. Compare what `tcpdump` shows on the client vs the server.

Use the [break/fix template](../../troubleshooting/TEMPLATE.md).

## What I learned

_To fill in after completing the lab._
