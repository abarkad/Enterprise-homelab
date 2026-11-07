# Device configurations

[← Back to README](../README.md)

Sanitized configurations exported from the lab devices.

| File | Source command |
| ---- | -------------- |
| `cisco-asa-5505.txt` | `show running-config` |
| `cisco-3750g.txt` | `show running-config` |
| `proxmox-interfaces.txt` | `cat /etc/network/interfaces` |

## Sanitization checklist

Before committing a configuration, remove or replace:

- [ ] `enable secret` / `enable password` lines
- [ ] `username ... password/secret` lines
- [ ] SNMP community strings
- [ ] Pre-shared keys and certificates
- [ ] Serial numbers and MAC addresses if you prefer not to publish them

Replace them with `<REDACTED>` rather than deleting the line, so the structure of the configuration stays readable.
