# Practice Labs

This section contains hands-on labs designed to reinforce the concepts covered throughout this repository. Each lab simulates realistic penetration testing scenarios similar to those found in the **INE eJPT (Junior Penetration Tester)** certification exam.

The objective is to practice methodology rather than memorizing commands. Every lab should be approached as if it were a real penetration test, following a structured workflow from reconnaissance to privilege escalation.

---

# Labs Included

| Lab | Description |
|------|-------------|
| **01. WordPress Lab** | Perform a complete WordPress security assessment, including enumeration, brute force, plugin discovery, and exploitation. |
| **02. Brute Force Lab** | Practice password attacks against multiple services such as SSH, FTP, SMB, and HTTP using Hydra. |
| **03. FTP Anonymous Lab** | Enumerate and exploit anonymous FTP access, identify sensitive files, and transfer data between attacker and target. |
| **04. Full Pentest Practice** | Complete an end-to-end penetration test covering reconnaissance, enumeration, exploitation, privilege escalation, and reporting. |

---

# Recommended Workflow

Every lab should follow the same penetration testing methodology.

```
Information Gathering

↓

Host Discovery

↓

Port Scanning

↓

Service Enumeration

↓

Web Enumeration

↓

Vulnerability Assessment

↓

Exploitation

↓

Initial Access

↓

Privilege Escalation

↓

Post Exploitation

↓

Documentation
```

Following a consistent workflow will help build habits that are essential during the eJPT exam and in real-world engagements.

---

# Skills Practiced

By completing every lab in this section, you will practice:

- Information Gathering
- Network Enumeration
- Service Enumeration
- Nmap Scanning
- Directory Enumeration
- FTP Enumeration
- SMB Enumeration
- SSH Enumeration
- Web Application Testing
- WordPress Enumeration
- Password Attacks
- File Transfers
- Reverse Shells
- Metasploit Usage
- Manual Exploitation
- Linux Privilege Escalation
- Windows Privilege Escalation
- Credential Hunting
- Password Cracking
- Post Exploitation
- Professional Methodology

---

# Recommended Lab Platforms

These exercises can be completed using vulnerable machines from platforms such as:

- INE Skill Dive Labs
- TryHackMe
- Hack The Box
- VulnHub
- Metasploitable 2
- OWASP Broken Web Apps
- DVWA
- Damn Vulnerable Linux
- Kioptrix
- Mr. Robot
- Basic Pentesting

---

# Before Starting

Before beginning any lab, ensure that you can answer the following questions:

- Can I identify live hosts?
- Can I perform a complete Nmap scan?
- Can I enumerate every discovered service?
- Can I recognize common vulnerabilities?
- Can I search for public exploits?
- Can I obtain an initial shell?
- Can I stabilize my shell?
- Can I enumerate the operating system?
- Can I identify privilege escalation opportunities?
- Can I document my findings?

If the answer to any of these questions is **No**, consider reviewing the corresponding chapter before continuing.

---

# Lab Checklist

Use this checklist for every machine you solve.

- [ ] Host discovered
- [ ] Full TCP scan completed
- [ ] UDP scan (if required)
- [ ] Services identified
- [ ] Web directories enumerated
- [ ] CMS detected (if applicable)
- [ ] Users enumerated
- [ ] Credentials discovered
- [ ] Initial access obtained
- [ ] Shell upgraded
- [ ] Local enumeration completed
- [ ] Privilege escalation achieved
- [ ] Password hashes collected
- [ ] Credentials cracked
- [ ] Documentation completed

---

# eJPT Exam Advice

The eJPT exam is heavily methodology-based. During every lab:

- Enumerate before exploiting.
- Do not rely exclusively on Metasploit.
- Save every credential you discover.
- Test credentials against multiple services.
- Keep detailed notes.
- Stay organized.
- Verify every finding manually.
- Think like a professional penetration tester.

---

# Final Notes

Practice is the most important part of learning penetration testing. Reading commands is useful, but solving vulnerable machines repeatedly is what develops real-world skills.

These labs are designed to help you transition from understanding individual techniques to conducting complete penetration tests with confidence.

Once you can complete the **Full Pentest Practice** lab independently, you should be well prepared for the **eJPT** certification exam and ready to move on to more advanced challenges such as **eCPPT** or **OSCP**.
