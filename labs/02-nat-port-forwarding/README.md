# Lab 02 – NAT, PAT and port forwarding (ASA 5505)

[← Back to labs](../README.md) · **Status:** 🔲 Planned · **Requires:** [Lab 01](../01-vlan-trunking/README.md)

## Objective

1. Give the new VLAN 20 Internet access with **dynamic PAT**.
2. Publish a web server in VLAN 20 to the outside network with **static PAT (port forwarding)**.
3. Read and explain translations with `show xlate`, `show nat` and `packet-tracer`.

> Syntax below is for ASA 8.3+ (object NAT). Check your version with `show version`.

## Topology

| Item | Value |
| ---- | ----- |
| Web server | VM in VLAN 20 – `192.168.20.50` – nginx or `python3 -m http.server 80` |
| Published as | ASA outside `192.168.2.29` **TCP 8080** → `192.168.20.50:80` |
| Test client | Phone or laptop on the Bell Giga Hub Wi-Fi (`192.168.2.0/24`) |

> ⚠️ This publishes the server to the Giga Hub LAN only. **Don't** add a port forward on the Giga Hub, and **never** publish Metasploitable.

## Configuration

### 1. Return route for VLAN 20

```text
route inside 192.168.20.0 255.255.255.0 192.168.1.2
```

### 2. Dynamic PAT for VLAN 20

```text
object network USERS-NET
 subnet 192.168.20.0 255.255.255.0
 nat (inside,outside) dynamic interface
```

### 3. Static PAT (port forwarding)

```text
object network WEB-SRV
 host 192.168.20.50
 nat (inside,outside) static interface service tcp 80 8080
!
access-list OUTSIDE-IN extended permit tcp any object WEB-SRV eq 80
access-group OUTSIDE-IN in interface outside
```

> On ASA 8.3+, ACLs reference the **real** IP and port (`192.168.20.50:80`), not the translated ones.

## Verification

| Check | Command / action | Expected | Evidence |
| ----- | ---------------- | -------- | -------- |
| NAT rules | `show nat detail` | USERS-NET dynamic and WEB-SRV static rules, hit counts rising | |
| VLAN 20 → Internet | VM: `ping 8.8.8.8`, `curl -I https://example.com` | Success | |
| Translations | `show xlate` | PAT entries from `192.168.20.x` to `192.168.2.29` | |
| Simulated inbound | `packet-tracer input outside tcp 192.168.2.100 12345 192.168.2.29 8080` | UN-NAT to `192.168.20.50:80`, action **allow** | |
| Real inbound | Phone browser → `http://192.168.2.29:8080` | Web page loads | |
| ACL hits | `show access-list OUTSIDE-IN` | Hit count increases | |
| Connections | `show conn address 192.168.20.50` | TCP connection from the phone | |

## Break / fix

**Fault A:** remove the ACL (`no access-group OUTSIDE-IN in interface outside`). Use `packet-tracer` to find which phase drops the packet.

**Fault B:** write the ACL with the *mapped* port (`eq 8080`) instead of the real one. Explain why it fails.

Record both with the [break/fix template](../../troubleshooting/TEMPLATE.md).

## Cleanup

Remove the static NAT and ACL when you're done so nothing stays published:

```text
object network WEB-SRV
 no nat (inside,outside) static interface service tcp 80 8080
no access-group OUTSIDE-IN in interface outside
```

## What I learned

_To fill in after completing the lab._
