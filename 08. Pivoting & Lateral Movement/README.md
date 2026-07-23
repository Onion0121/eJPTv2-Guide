# Pivoting & Lateral Movement

> **Difficulty:** Beginner - Intermediate  
> **Estimated Reading Time:** 3-4 Hours

---

# Table of Contents

- [Introduction](#introduction)
- [What is Pivoting?](#what-is-pivoting)
- [What is Lateral Movement?](#what-is-lateral-movement)
- [Why Pivoting & Lateral Movement Matter](#why-pivoting--lateral-movement-matter)
- [Pivoting Methodology](#pivoting-methodology)
- [Network Mapping](#network-mapping)
- [Windows Pivoting](#windows-pivoting)
- [Linux Pivoting](#linux-pivoting)
- [SSH Pivoting](#ssh-pivoting)
- [Metasploit Routing](#metasploit-routing)
- [Lateral Movement Basics](#lateral-movement-basics)
- [Common Tools](#common-tools)
- [Attack Workflow](#attack-workflow)
- [Defense & Mitigation](#defense--mitigation)
- [eJPT Exam Tips](#ejpt-exam-tips)
- [Summary](#summary)
- [Next Section](#next-section)

---

# Introduction

Pivoting and lateral movement are advanced stages of a penetration test that occur after gaining initial access to a target system. Once a tester compromises one machine, the next objective is usually not to stop there. The compromised system can become a bridge to internal networks, additional hosts, sensitive services, and critical infrastructure. The goal is to understand how an attacker can move deeper into an environment.

Typical attack path:

```
External Reconnaissance

↓

Initial Exploitation

↓

Compromised Host

↓

Pivot Into Internal Network

↓

Lateral Movement

↓

Compromise Additional Systems
```

# What is Pivoting?

Pivoting is the process of using a compromised machine to access networks or systems that are not directly reachable from the attacker machine. The compromised host becomes a gateway between the attacker and internal resources.

Example:

```
Attacker

192.168.1.50


↓

Compromised Server

192.168.1.100

10.10.10.5


↓

Internal Database Server

10.10.10.20
```

The attacker cannot directly access the internal server:

```
10.10.10.20
```

but can route traffic through the compromised machine:

```
192.168.1.100
```

# What is Lateral Movement?

Lateral movement is the process of moving from one compromised system to another inside the same network. After gaining access to one machine, attackers search for additional credentials, internal services, domain accounts, and sensitive systems.

Example:

```
Web Server

↓

File Server

↓

Domain Controller

↓

Database Server
```

The objective is to expand access and reach more valuable targets.

# Why Pivoting & Lateral Movement Matter

Modern enterprise networks are usually segmented into different zones.

Example:

```
Internet

↓

Public Web Server

↓

Internal Network

↓

Active Directory

↓

Database Servers
```

A public-facing server may only provide the initial entry point. Attackers use pivoting and lateral movement to:

- Discover hidden systems
- Access internal applications
- Obtain credentials
- Escalate privileges
- Reach critical infrastructure

# Pivoting Methodology

A professional pivoting workflow:

```
Compromise Initial Host

↓

Enumerate Network Information

↓

Identify Internal Networks

↓

Discover Internal Hosts

↓

Create Tunnel

↓

Access Internal Services

↓

Perform Lateral Movement
```

# Network Mapping

Before pivoting, the attacker must understand the network environment.

## Windows Network Enumeration

View network interfaces:

```cmd
ipconfig /all
```

View routes:

```cmd
route print
```

View ARP table:

```cmd
arp -a
```

## Linux Network Enumeration

View interfaces:

```bash
ip addr
```

View routing information:

```bash
ip route
```

View neighbors:

```bash
ip neigh
```

Discover hosts:

```bash
nmap -sn 10.10.10.0/24
```

# Windows Pivoting

Windows systems are common pivot points in enterprise environments.

Common Windows pivoting techniques:

- Meterpreter routing
- SOCKS proxies
- Port forwarding
- Chisel tunnels
- Ligolo-ng tunnels

Example:

```
Kali Linux

↓

Windows Pivot Host

↓

Internal Network
```

Important commands:

```cmd
ipconfig
route print
arp -a
```

These commands reveal:

- Internal IP addresses
- Network interfaces
- Available routes
- Nearby systems

# Linux Pivoting

Linux servers are frequently exposed to the internet and can become pivot points.

Common Linux pivoting techniques:

- SSH tunneling
- SSHuttle
- SOCKS proxies
- Chisel
- Ligolo-ng

Important commands:

```bash
ip addr

ip route

arp -a
```

These commands help identify reachable networks.

# SSH Pivoting

SSH is one of the most common pivoting methods.

SSH supports:

- Local port forwarding
- Remote port forwarding
- Dynamic SOCKS proxies

## Local Port Forwarding

Example:

```bash
ssh -L 8080:10.10.10.20:80 user@pivot
```

Traffic flow:

```
Browser

↓

localhost:8080

↓

SSH Tunnel

↓

Pivot Host

↓

Internal Web Server
```

## SOCKS Proxy

Create a SOCKS proxy:

```bash
ssh -D 9050 user@pivot
```

Proxy:

```
127.0.0.1:9050
```

Use with Proxychains:

```bash
proxychains nmap -sT TARGET
```

# Metasploit Routing

Metasploit supports pivoting through Meterpreter sessions.

Workflow:

```
Exploit Target

↓

Meterpreter Session

↓

Add Route

↓

Scan Internal Network
```

Start Metasploit:

```bash
msfconsole
```

View sessions:

```text
sessions
```

Interact with a session:

```text
sessions -i 1
```

Add route:

```text
run autoroute -s 10.10.10.0/24
```

View routes:

```text
route print
```

After adding the route, Metasploit modules can communicate with internal targets.

# Lateral Movement Basics

After accessing an internal network, attackers attempt to compromise additional machines.

Common methods include:

- Credential reuse
- Password attacks
- SMB authentication
- SSH access
- Remote Desktop
- Windows administration services

# Common Lateral Movement Techniques

## Windows

Common techniques:

- SMB attacks
- Pass-the-Hash
- RDP access
- WinRM
- PsExec

## Linux

Common techniques:

- SSH key reuse
- Shared credentials
- Private key theft
- Weak permissions

# Common Tools

## Network Discovery

- Nmap
- Netdiscover
- arp-scan

## Tunneling

- SSH
- Chisel
- Ligolo-ng
- SSHuttle

## Exploitation Frameworks

- Metasploit
- Meterpreter

## Proxy Tools

- Proxychains
- SOCKS proxies

# Attack Workflow

Complete pivoting and lateral movement workflow:

```
Initial Compromise

↓

Shell Access

↓

System Enumeration

↓

Network Enumeration

↓

Identify Internal Range

↓

Create Tunnel

↓

Scan Internal Hosts

↓

Enumerate Services

↓

Exploit New Targets

↓

Move Laterally
```

# Defense & Mitigation

Organizations can reduce pivoting and lateral movement risks by:

- Segmenting networks
- Applying least privilege
- Using strong authentication
- Implementing MFA
- Monitoring internal traffic
- Restricting remote services
- Protecting credentials
- Detecting suspicious tunnels

# eJPT Exam Tips

For the eJPT certification, focus on understanding:

## Network Enumeration

Commands:

```bash
ip addr

ip route

arp -a
```

## SSH Pivoting

Local forwarding:

```bash
ssh -L
```

Remote forwarding:

```bash
ssh -R
```

SOCKS proxy:

```bash
ssh -D
```

## Metasploit Routing

Important commands:

```text
sessions

autoroute

route print

portfwd
```

Main concept:

```
The first compromised machine is usually not the final target.
```

Always check:

- Network interfaces
- Routes
- Internal networks
- Available services

# Summary

Pivoting and lateral movement are essential penetration testing skills that allow attackers and security professionals to move beyond the initial compromised system.

In this section, you learned:

- Pivoting concepts
- Lateral movement concepts
- Network mapping
- Windows pivoting
- Linux pivoting
- SSH tunneling
- Metasploit routing
- Internal network access
- Movement between compromised systems

These skills are fundamental for:

- eJPT
- eCPPT
- OSCP preparation
- Internal penetration testing
- Red team operations

# Next Section

➡ **09. Reporting & Methodology**

The next section covers:

- Penetration testing methodology
- Documentation
- Evidence collection
- Professional reporting
- Risk classification
- Remediation recommendations
