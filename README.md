# Enterprise Cybersecurity Homelab

Enterprise-style cybersecurity homelab built using Cisco networking equipment, a Dell PowerEdge server, Proxmox VE, Kali Linux, and intentionally vulnerable systems.

The goal of this project is to gain hands-on experience in:

- Network infrastructure
- VLAN segmentation
- Layer 2/Layer 3 networking
- Cisco ASA firewall administration
- NAT/PAT
- Virtualization
- Linux system administration
- Penetration testing
- Vulnerability assessment
- Network reconnaissance and enumeration
- Security lab design
- Cybersecurity documentation

---

# Project Overview

This project documents the design, deployment, configuration, troubleshooting, and security testing of my personal enterprise-style cybersecurity homelab.

The environment combines physical enterprise networking equipment with virtual machines running on a Dell PowerEdge R320.


The lab simulates a small enterprise network where you can practice networking, firewalling, virtualization, offensive security, and defensive security in an isolated environment.

---

# Lab Architecture

The current environment consists of:

```text
                         Internet
                            |
                    Bell Giga Hub
                    192.168.2.1
                            |
                            |
                    Cisco ASA 5505
                 Outside: 192.168.2.29
                 Inside:  192.168.1.1
                            |
                            |
                    Cisco 3750G L3
                 VLAN 1: 192.168.1.0/24
                            |
              +-------------+-------------+
              |                           |
          Desktop PC                 Dell R320
                                      Proxmox VE
                                          |
                                        vmbr0
                                          |
                                802.1Q Trunk
                                          |
                              VLAN 50 SECURITY-LAB
                                192.168.50.0/24
                                          |
                         +----------------+----------------+
                         |                                 |
                      Kali Linux                    Metasploitable 2
                    192.168.50.10                     192.168.50.20   
```

Network Design
````
VLAN 1 - Management / Internal Network
| Component        | Address           |
| ---------------- | ----------------- |
| Cisco ASA Inside | `192.168.1.1/24`  |
| Cisco 3750G      | `192.168.1.2/24`  |
| Proxmox          | `192.168.1.10/24` |
| Network          | `192.168.1.0/24`  |
````
The Cisco ASA provides the default gateway for the VLAN 1 internal
network and serves as the Internet edge firewall.

The Cisco 3750G provides the Layer 3 SVI for VLAN 50 and performs
inter-VLAN routing between the security lab and the internal network.
````
VLAN 50 - Security Lab
VLAN Name: SECURITY-LAB
| Component        | Address           |
| ---------------- | ----------------- |
| VLAN 50 SVI      | `192.168.50.1/24` |
| Kali Linux       | `192.168.50.10`   |
| Metasploitable 2 | `192.168.50.20`   |
| Network          | `192.168.50.0/24` |
````
VLAN 50 is used for penetration-testing exercises and intentionally vulnerable systems.
Metasploitable 2 is intentionally kept without a default Internet gateway so vulnerable services remain isolated from the Internet.
````
Hardware
Server
Dell PowerEdge R320
2 × 1 TB HDD
RAID 1
Proxmox VE
Networking
Cisco ASA 5505 Firewall
Cisco WS-C3750G-24TS-1U Layer 3 Switch
Bell Giga Hub
Desktop PC
Virtualization
Proxmox VE
````
Proxmox VE is used as the virtualization platform for the security lab.

Host Network
````
vmbr0
IP: 192.168.1.10/24
Gateway: 192.168.1.1
Physical interface: nic0
````
The Proxmox bridge carries the management network and tagged VLAN traffic to virtual machines.

Virtual Machines
````
| VM ID | System           | IP Address      | Purpose             |
| ----- | ---------------- | --------------- | ------------------- |
| 101   | Kali Linux       | `192.168.50.10` | Penetration testing |
| 102   | Metasploitable 2 | `192.168.50.20` | Vulnerable target   |
````
Cisco 3750G Layer 3 Switch

The Cisco 3750G provides:
````
VLAN segmentation
Inter-VLAN routing
Default routing toward the firewall
802.1Q trunking
Layer 3 switching
````
VLANs
````
VLAN 1
Network: 192.168.1.0/24
Gateway: 192.168.1.1
````
VLAN 50
````
Name: SECURITY-LAB
Network: 192.168.50.0/24
Gateway: 192.168.50.1
````
Routing
````
The switch uses the Cisco ASA as its default route:
0.0.0.0/0 -> 192.168.1.1
````
Proxmox Trunk
The connection between the Cisco 3750G and Proxmox uses an 802.1Q trunk.
````
Cisco Gi1/0/24
       |
       | 802.1Q
       |
Proxmox vmbr0
````
Allowed VLANs:
````
VLAN 1
VLAN 50
````
Cisco ASA 5505

The ASA provides the firewall and Internet edge for the lab.

Interfaces
Inside
````
IP: 192.168.1.1/24
Security Level: 100
````
Outside
````
IP: 192.168.2.29
Gateway: 192.168.2.1
Security Level: 0
````
The outside interface receives its address from the Bell Giga Hub network.

NAT / PAT

The ASA is configured with PAT to provide internal networks with Internet access.

Internal networks include:
````
192.168.1.0/24
192.168.50.0/24
````
The ASA translates internal traffic to its outside interface:
````
192.168.50.x
      |
      v
Cisco ASA
      |
      v
192.168.2.29
      |
      v
Internet
````
Connectivity was validated using Cisco ASA packet-tracer.

Example:
````
packet-tracer input inside icmp 192.168.50.10 8 0 8.8.8.8
````
The packet-tracer result showed:
````
NAT ALLOW
Action: allow
````
#Kali Linux
Kali Linux is deployed as VM 101 on Proxmox.
````
Configuration
IP Address: 192.168.50.10
Subnet:     255.255.255.0
Gateway:    192.168.50.1
VLAN:       50
Bridge:     vmbr0
````
Kali is used as the penetration-testing workstation.

Tools used include:

Nmap
````
Metasploit Framework
SearchSploit
FTP enumeration tools
Network troubleshooting utilities
Metasploitable 2
````
Metasploitable 2 is deployed as VM 102.

It is intentionally vulnerable and is used as a controlled target for penetration-testing exercises.

Configuration
````
IP Address: 192.168.50.20
Subnet:     255.255.255.0
VLAN:       50
````
The machine does not use an Internet default gateway.

Penetration Testing Lab

The first penetration-testing exercise focused on the vulnerable FTP service running:
````
vsftpd 2.3.4
Target
192.168.50.20
Attacker
192.168.50.10
````
The exercise followed a structured penetration-testing methodology:
````
Reconnaissance
      ↓
Enumeration
      ↓
Vulnerability Research
      ↓
Exploitation
      ↓
Post-Exploitation Enumeration
      ↓
Documentation
Exercise 01 - vsftpd 2.3.4
Reconnaissance
````
Initial Nmap scan:
````
nmap 192.168.50.20
````
The scan identified multiple open TCP services.

Important examples included:
````
21/tcp    FTP
22/tcp    SSH
23/tcp    Telnet
25/tcp    SMTP
80/tcp    HTTP
139/tcp   NetBIOS
445/tcp   SMB
3306/tcp  MySQL
5432/tcp  PostgreSQL
5900/tcp  VNC
8180/tcp  HTTP
Service Enumeration
````
Version detection was performed with:
````
nmap -sV 192.168.50.20
````
The FTP service was identified as:
````
vsftpd 2.3.4
FTP Enumeration
````
The FTP service was further enumerated:
````
nmap -p21 --script ftp-anon,ftp-syst 192.168.50.20
````
The scan identified:
````
vsftpd 2.3.4
Anonymous FTP login allowed
Vulnerability Research
````
SearchSploit was used to identify known public exploits associated with the service:
````
searchsploit vsftpd 2.3.4
````
The search identified the well-known:
````
vsftpd 2.3.4 - Backdoor Command Execution
Exploitation
````
The exercise was performed against the intentionally vulnerable Metasploitable 2 VM inside the isolated security VLAN.

Metasploit was used to validate the vulnerability:
````
exploit/unix/ftp/vsftpd_234_backdoor
````
Attacker:
````
LHOST = 192.168.50.10
````
Target:
````
192.168.50.20
````
The exploit successfully opened a Meterpreter session.

Post-Exploitation

The resulting session was verified with:
````
getuid
````
Result:
````
Server username: root
````
System information was collected with:
````
sysinfo
````
The target was identified as:
````
Ubuntu 8.04
Linux 2.6.24-16-server
i686
````
The current working directory was checked:

pwd

Result:

/root

The Meterpreter process was also identified using:

getpid

Result:

5034

This confirmed successful command execution with root-level privileges on the intentionally vulnerable target.

Security Lessons

This exercise demonstrated several important security concepts:
````
Service discovery
Port scanning
Service version enumeration
Anonymous FTP access
Vulnerability research
Exploit validation
Metasploit usage
Meterpreter sessions
Privilege context verification
Post-exploitation enumeration
Network segmentation
Isolated security testing
Troubleshooting Experience
````
The lab was built through multiple troubleshooting stages.

Physical Connectivity

Initial Proxmox management access failed because the Ethernet connection was not physically connected.

Resolution:
````
Verify physical link
↓
Connect Ethernet cable
↓
Verify interface status
↓
Test connectivity
Network Topology
````
The desktop computer was initially not connected through the intended firewall/switch topology.

The topology was corrected so management traffic could reach the internal network.

ASA Inside / Outside Interfaces

A major troubleshooting lesson was understanding the difference between the ASA interfaces.

Outside
````
192.168.2.29
      |
      v
   Internet
````
and:

Inside
````
192.168.1.1
      |
      v
Internal Network
````
Internal devices use the ASA inside interface as the default gateway.

Proxmox Management

The Proxmox management address was configured as:
````
192.168.1.10/24
````
Connectivity was then verified between:

PC
 ↓
Cisco 3750G
 ↓
Cisco ASA
 ↓
Internal Network
 ↓
Proxmox
Connectivity Validation

The following connectivity was successfully tested:

PC → ASA
PC → Proxmox
Proxmox → ASA
ASA → Cisco 3750G
ASA → VLAN 50
Kali → Metasploitable
Kali → Internet
Troubleshooting Methodology

This project reinforced a structured troubleshooting methodology:
````
Layer 1
Physical connectivity
        ↓
Layer 2
Switching / VLANs
        ↓
Layer 3
IP addressing / Routing
        ↓
Layer 4
TCP / UDP connectivity
        ↓
Layer 7
Application / Services
````
Instead of changing multiple configurations at once, I tested connectivity one hop at a time.
````
Skills Demonstrated
Networking
Cisco IOS
VLAN configuration
802.1Q trunking
Layer 2 switching
Layer 3 switching
Inter-VLAN routing
Static routing
Default gateways
IP addressing
Network troubleshooting
Firewall
Cisco ASA 5505
Inside / Outside interfaces
Security levels
Routing
NAT
PAT
Packet Tracer
Internet edge configuration
Virtualization
Proxmox VE
Virtual machine deployment
Virtual networking
VLAN-aware VM networking
Linux bridge configuration
Virtual disk management
Linux
Kali Linux
Linux networking
IP configuration
Service enumeration
Command-line administration
Cybersecurity
Reconnaissance
Nmap
Service enumeration
SearchSploit
Metasploit
Meterpreter
Vulnerability validation
Post-exploitation enumeration
Security lab isolation
Repository Structure
````
The documentation is organized into the following areas:
````
Enterprise-homelab/
│
├── README.md
│
├── diagrams/
│   ├── physical-topology.md
│   ├── logical-topology.md
│   └── vlan-design.md
│
├── infrastructure/
│   ├── proxmox/
│   │   ├── installation.md
│   │   ├── networking.md
│   │   └── storage.md
│   │
│   ├── cisco-asa/
│   │   ├── interfaces.md
│   │   ├── routing.md
│   │   └── nat-pat.md
│   │
│   └── cisco-3750/
│       ├── vlan.md
│       ├── trunk.md
│       └── inter-vlan-routing.md
│
├── security-lab/
│   ├── kali-linux/
│   │   └── deployment.md
│   │
│   └── metasploitable2/
│       └── deployment.md
│
├── penetration-testing/
│   └── 01-vsftpd-2.3.4/
│       ├── README.md
│       ├── reconnaissance.md
│       ├── enumeration.md
│       ├── exploitation.md
│       └── screenshots/
│
└── screenshots/
````
Project Status
Completed
 Physical infrastructure deployed
 Dell PowerEdge R320 deployed
 Proxmox VE deployed
 Proxmox management networking configured
 Cisco ASA configured
 Cisco 3750G Layer 3 switch configured
 VLAN 50 created
 Inter-VLAN routing configured
 802.1Q trunk configured
 ASA routing configured
 NAT/PAT configured
 Internet connectivity validated
 Kali Linux deployed
 Metasploitable 2 deployed
 Security lab network established
 Nmap reconnaissance performed
 Service enumeration performed
 vsftpd 2.3.4 vulnerability researched
 Vulnerability exploited in isolated lab
 Meterpreter session obtained
 Root-level access verified
 Initial post-exploitation enumeration completed
In Progress
 Document complete infrastructure configuration
 Create detailed network diagrams
 Expand penetration-testing exercises
 Build additional vulnerable targets
 Develop defensive security exercises
 Create custom Python security tools
Future
 Windows Server
 Active Directory
 Enterprise DNS/DHCP environment
 Additional VLAN segmentation
 More penetration-testing scenarios
 Detection and monitoring exercises
 Security automation
 Python security tooling
Security Disclaimer

All penetration-testing activities documented in this repository are performed against systems that I own or intentionally vulnerable systems deployed inside my isolated homelab.

The techniques demonstrated here are intended for authorized security testing, education, and cybersecurity skill development.

Project Goals

The long-term goal of this project is to build a realistic enterprise-style cybersecurity environment where I can continuously practice:
````
Networking
    +
Systems Administration
    +
Virtualization
    +
Offensive Security
    +
Defensive Security
    +
Security Automation
````
The lab will continue to evolve as I add new technologies, systems, and security scenarios.
````
