# Web Application Security

The **Web Application Security** section covers the fundamental techniques used to assess, enumerate, exploit, and validate the security of modern web applications. Web applications are one of the primary attack surfaces during penetration tests and represent a significant portion of the **INE eJPT** exam.

This section follows the typical methodology used during a web application assessment, beginning with information gathering and directory enumeration, continuing through vulnerability discovery and exploitation, and ending with automated tools and traffic analysis.

The goal is not only to learn individual tools, but also to understand **when**, **why**, and **how** to use them together during a real penetration test.

---

## Learning Objectives

After completing this section, you should be able to:

- Understand the methodology of a web application penetration test.
- Discover hidden files and directories.
- Perform content discovery using multiple enumeration tools.
- Identify vulnerable web technologies.
- Exploit common web vulnerabilities.
- Analyze HTTP requests and responses.
- Use Burp Suite efficiently.
- Perform manual SQL Injection testing.
- Automate SQL Injection with SQLmap.
- Exploit Local File Inclusion vulnerabilities.
- Analyze network traffic with Wireshark.

---

# Topics Covered

## 01. Gobuster

Learn how to discover hidden directories, files, virtual hosts, and subdomains using Gobuster.

Topics include:

- Directory brute forcing
- File enumeration
- Virtual Host discovery
- DNS mode
- Common wordlists
- Best practices

---

## 02. Wfuzz

Learn how to fuzz web applications using custom payloads.

Topics include:

- Directory fuzzing
- Parameter fuzzing
- Header fuzzing
- Authentication testing
- Filtering responses
- Payload management

---

## 03. DNS Zone Transfer

Understand DNS enumeration and how insecure DNS servers can expose internal infrastructure.

Topics include:

- AXFR requests
- DNS records
- Nameservers
- Host discovery
- DNS reconnaissance

---

## 04. Directory Enumeration

Learn how to identify hidden resources exposed by web servers.

Topics include:

- Hidden directories
- Backup files
- Configuration files
- Robots.txt
- Common administrative panels
- Enumeration workflow

---

## 05. File Upload Vulnerabilities

Understand insecure file upload implementations.

Topics include:

- Upload bypasses
- Extension filtering
- MIME type bypass
- Magic bytes
- Upload restrictions
- Remote Code Execution

---

## 06. Web Shells

Learn how web shells work after successful exploitation.

Topics include:

- PHP shells
- ASPX shells
- Command execution
- File management
- Reverse shells
- Web shell detection

---

## 07. Burp Suite Basics

Learn the fundamentals of Burp Suite.

Topics include:

- Proxy
- Intercept
- HTTP history
- Repeater
- Intruder
- Decoder
- Comparer

---

## 08. Burp Repeater

Master manual HTTP request manipulation.

Topics include:

- Parameter modification
- Authentication testing
- Header manipulation
- Session analysis
- Manual vulnerability testing

---

## 09. Burp Intruder

Automate web application attacks.

Topics include:

- Sniper
- Battering Ram
- Pitchfork
- Cluster Bomb
- Payloads
- Response analysis

---

## 10. SQL Injection

Learn to manually identify and exploit SQL Injection vulnerabilities.

Topics include:

- Authentication bypass
- UNION attacks
- Boolean SQLi
- Error-based SQLi
- Time-based SQLi
- Database enumeration

---

## 11. SQLmap

Automate SQL Injection exploitation.

Topics include:

- Automatic detection
- Database enumeration
- Table dumping
- POST requests
- Burp integration
- OS shell

---

## 12. Local File Inclusion

Learn how insecure file inclusion can expose sensitive files and lead to Remote Code Execution.

Topics include:

- Directory traversal
- PHP wrappers
- Sensitive file disclosure
- Log poisoning
- LFI to RCE
- File inclusion bypasses

---

## 13. Wireshark

Analyze network traffic during web assessments.

Topics include:

- HTTP analysis
- DNS traffic
- FTP credentials
- Packet filtering
- TCP streams
- Traffic inspection

---

# Recommended Learning Order

```
Directory Enumeration

↓

Gobuster

↓

Wfuzz

↓

DNS Zone Transfer

↓

Burp Suite Basics

↓

Burp Repeater

↓

Burp Intruder

↓

File Upload Vulnerabilities

↓

Web Shells

↓

SQL Injection

↓

SQLmap

↓

Local File Inclusion

↓

Wireshark
```

---

# Skills You Will Develop

By completing this section you will be able to:

- Enumerate web servers efficiently.
- Discover hidden application content.
- Understand HTTP communication.
- Manipulate web requests manually.
- Automate repetitive testing.
- Identify common web vulnerabilities.
- Exploit SQL Injection vulnerabilities.
- Read sensitive files through LFI.
- Analyze captured network traffic.
- Combine multiple tools during a professional web application assessment.

---

# Tools Covered

- Gobuster
- Wfuzz
- Burp Suite
- SQLmap
- Wireshark
- cURL
- Firefox Developer Tools
- Browser Proxy Settings

---

# eJPT Focus

For the **INE eJPT** certification, web application testing is one of the core domains. You should be comfortable with:

- Directory and file enumeration.
- Manual HTTP request manipulation using Burp Suite.
- SQL Injection identification and exploitation.
- Automating SQL Injection with SQLmap.
- Exploiting Local File Inclusion vulnerabilities.
- Discovering sensitive information exposed by web servers.
- Uploading and interacting with web shells.
- Understanding HTTP methods, headers, cookies, and sessions.
- Analyzing web-related traffic using Wireshark.

---

# Prerequisites

Before studying this section, you should already understand:

- Networking Fundamentals
- TCP/IP
- HTTP and HTTPS
- Basic Linux commands
- Nmap
- Host discovery
- Service enumeration
- Reverse shells
- Basic penetration testing methodology

---

# Summary

Web applications are among the most common attack surfaces encountered during penetration tests. This section provides a practical introduction to the techniques, tools, and methodologies required to assess web applications effectively. By mastering enumeration, HTTP analysis, Burp Suite, SQL Injection, SQLmap, Local File Inclusion, web shells, and packet analysis with Wireshark, you will build the skills necessary to successfully perform web application assessments during both the **INE eJPT** exam and real-world penetration testing engagements.
