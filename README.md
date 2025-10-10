# Information Gathering
## 🔍 Information Gathering | การรวบรวมข้อมูล
Nmap
1. Check port และ service: nmap -sV -p- 192.168.1.1
3. Check Host 1 Up: ping 10.7.1.226
4. Check Host 254 Up: nmap -sP 10.7.1.0/24
5. Check Host Port Web TCP : sudo nmap -sT -p 80,443 10.7.1.0/24
6. STEALTH (หลบ IDS โดยจะไม่ได้ ACK กลับ) : sudo nmap -sS -p 80,443 10.7.1.0/24
7. Check Host Port Open : sudo nmap -sT 10.7.1.226
8. (หลบ IDS) : sudo nmap -sS 10.7.1.226
9. (Ver Host) : sudo nmap -O 10.7.1.226
10. (Ver DetailHost) : sudo nmap -A 10.7.1.226
11. (Decoy) : sudo nmap -sS -D 10.7.1.80 10.7.1.226
12. vuln : sudo nmap --script vuln 10.7.1.226
13. sudo nmap -v -Pn 10.7.1.226
14. cert : nmap -p 443 192.168.2.137 -sV --script=ssl-cert
Whois
1. whois facebook.com
2. 

2. whois [domain]
- ตรวจสอบข้อมูลเจ้าของโดเมน
ตัวอย่าง: whois example.com
3. dig [domain]
- ค้นหาข้อมูล DNS records
ตัวอย่าง: dig example.com
4. dirb [url]
- สแกนหาไดเรกทอรีและไฟล์บนเว็บ
ตัวอย่าง: dirb http://example.com/
5. nikto -h [host]
- สแกนหาช่องโหว่บนเว็บ
ตัวอย่าง: nikto -h example.com
6. dmitry -winsepfb [target]
- รวบรวมข้อมูลเป้าหมาย
ตัวอย่าง: dmitry -winsepfb example.com
7. recon-ng
- เครื่องมือรวบรวมข้อมูลแบบครบวงจร
ตัวอย่าง: recon-ng
8. theHarvester -d [domain] -l [limit] -b [source]
- รวบรวมอีเมลและโดเมนย่อย
ตัวอย่าง: theHarvester -d example.com -l 500 -b google
9. maltego
- เครื่องมือวิเคราะห์ความสัมพันธ์ของข้อมูล
ตัวอย่าง: maltego
10. fierce -dns [domain]
- สแกนหาโดเมนย่อย
ตัวอย่าง: fierce -dns example.com
🔓 Vulnerability Analysis | การวิเคราะห์ช่องโหว่
11. openvas
- สแกนหาช่องโหว่แบบครบวงจร
ตัวอย่าง: openvas-start
12. sqlmap -u [url]
- ทดสอบช่องโหว่ SQL Injection
ตัวอย่าง: sqlmap -u "http://example.com/page.php?id=1"
13. wpscan --url [url]
- สแกนหาช่องโหว่ WordPress
ตัวอย่าง: wpscan --url http://example.com
14. joomscan -u [url]
- สแกนหาช่องโหว่ Joomla
ตัวอย่าง: joomscan -u http://example.com
15. arachni [url]
- สแกนหาช่องโหว่เว็บแอพพลิเคชัน
ตัวอย่าง: arachni http://example.com

🚪 Exploitation Tools  | เครื่องมือสำหรับการโจมตี
16. metasploit
- Framework สำหรับทดสอบการโจมตี
ตัวอย่าง: msfconsole
17. set
- Social-Engineer Toolkit
ตัวอย่าง: setoolkit
18. beef-xss
- เครื่องมือทดสอบ XSS
ตัวอย่าง: beef-xss
19. searchsploit [keywords]
- ค้นหา exploits
ตัวอย่าง: searchsploit apache 2.4
20. armitage
- GUI สำหรับ Metasploit
ตัวอย่าง: armitage
🔑 Password Attacks | การโจมตีรหัสผ่าน
21. hashcat -m [mode] [hashfile] [wordlist]
- Crack password hashes
ตัวอย่าง: hashcat -m 0 hashes.txt wordlist.txt
22. john [hashfile]
- John the Ripper password cracker
ตัวอย่าง: john hashes.txt
23. hydra -l [user] -P [passlist] [service://server]
- Brute force online services
ตัวอย่าง: hydra -l admin -P passwords.txt ftp://192.168.1.1
24. crunch [min] [max] [characters] -o [output]
- สร้าง wordlist
ตัวอย่าง: crunch 8 8 abcdefgh -o wordlist.txt
25. medusa -h [host] -u [user] -P [passlist] -M [module]
- Brute force parallel login
ตัวอย่าง: medusa -h example.com -u admin -P passwords.txt -M http
🌐 Wireless Attacks | การโจมตีเครือข่ายไร้สาย
26. airmon-ng start [interface]
- เริ่มโหมด monitor
ตัวอย่าง: airmon-ng start wlan0
27. airodump-ng [interface]
- ดักจับ wireless traffic
ตัวอย่าง: airodump-ng wlan0mon
28. aireplay-ng -0 [deauths] -a [bssid] [interface]
- Deauthentication attack
ตัวอย่าง: aireplay-ng -0 5 -a 00:11:22:33:44:55 wlan0mon
29. wifite
- Automated wireless auditor
ตัวอย่าง: wifite
30. kismet
- Wireless network detector
ตัวอย่าง: kismet
🕸️ Web Application Analysis | การวิเคราะห์เว็บแอปพลิเคชัน
31. burpsuite
- Web application security testing
ตัวอย่าง: burpsuite
32. owasp-zap
- Web application vulnerability scanner
ตัวอย่าง: owasp-zap
33. skipfish -o [output] [url]
- Web application security scanner
ตัวอย่าง: skipfish -o results http://example.com
34. w3af
- Web Application Attack and Audit Framework
ตัวอย่าง: w3af
35. davtest -url [url]
- WebDAV testing tool
ตัวอย่าง: davtest -url http://example.com
🔭 Sniffing & Spoofing | การดักจับและปลอมแปลงข้อมูล
36. wireshark
- Network protocol analyzer
ตัวอย่าง: wireshark
37. ettercap -T -q -i [interface]
- Network sniffer/interceptor
ตัวอย่าง: ettercap -T -q -i eth0
38. mitmproxy
- Interactive HTTPS proxy
ตัวอย่าง: mitmproxy
39. dsniff
- Network sniffing tools
ตัวอย่าง: dsniff
40. scapy
- Packet manipulation
ตัวอย่าง: scapy
🚧 Post Exploitation | การดำเนินการหลังการโจมตี
41. meterpreter
- Advanced payload in Metasploit
ตัวอย่าง: use exploit/multi/handler
42. empire
- PowerShell post-exploitation
ตัวอย่าง: empire
43. mimikatz
- Windows credential dumper
ตัวอย่าง: mimikatz
44. powercat
- PowerShell netcat
ตัวอย่าง: powercat -c 192.168.1.1 -p 4444
45. privilege-escalation-awesome-scripts-suite
- Privilege escalation tools
ตัวอย่าง: ./LinEnum.sh
🗃️ Forensics Tools | เครื่องมือนิติวิทยาศาสตร์ดิจิทัล
46. autopsy
- Digital forensics platform
ตัวอย่าง: autopsy
47. binwalk [file]
- Firmware analysis tool
ตัวอย่าง: binwalk firmware.bin
48. foremost -i [file]
- File carver
ตัวอย่าง: foremost -i disk.img
49. volatility
- Memory forensics
ตัวอย่าง: volatility -f memory.dmp imageinfo
50. scalpel
- File carver
ตัวอย่าง: scalpel disk.img
🎯 Stress Testing | การทดสอบความทนทาน
51. siege -c [concurrent] -t [time] [url]
- HTTP load testing
ตัวอย่าง: siege -c 10 -t 30S http://example.com
52. slowhttptest -c [connections] -H -i [interval] -r [rate] -t [time] -u [url]
- Slow HTTP attack simulator
ตัวอย่าง: slowhttptest -c 1000 -H -i 10 -r 200 -t 60 -u http://example.com
53. t50 [target]
- Multi-protocol packet injector
ตัวอย่าง: t50 192.168.1.1
54. hping3 [options] [target]
- Network tool
ตัวอย่าง: hping3 -S -p 80 example.com
55. doona
- Network protocol fuzzer
ตัวอย่าง: doona
🔐 Cryptography & Steganography | การเข้ารหัสและการซ่อนข้อมูล
56. steghide embed -cf [container] -ef [file]
- Hide data in images
ตัวอย่าง: steghide embed -cf image.jpg -ef secret.txt
57. cryptcat
- Encrypted netcat
ตัวอย่าง: cryptcat -l -p 4444
58. truecrypt
- Disk encryption
ตัวอย่าง: truecrypt
59. ccrypt
- Encryption tool
ตัวอย่าง: ccrypt file.txt
60. outguess
- Steganographic tool
ตัวอย่าง: outguess -k "key" -d hidden.txt image.jpg
📱 Mobile Analysis | การวิเคราะห์อุปกรณ์มือถือ
61. apktool d [file.apk]
- Android APK analysis
ตัวอย่าง: apktool d application.apk
62. dex2jar [file.apk]
- Convert APK to JAR
ตัวอย่าง: d2j-dex2jar.sh application.apk
63. jd-gui
- Java decompiler
ตัวอย่าง: jd-gui
64. idb
- iOS app security assessment
ตัวอย่าง: idb
65. androguard
- Android application analysis
ตัวอย่าง: androguard
🌍 Reverse Engineering
66. gdb
- GNU debugger
ตัวอย่าง: gdb ./program
67. radare2
- Reverse engineering framework
ตัวอย่าง: r2 ./program
68. ida
- Interactive disassembler
ตัวอย่าง: ida
69. ollydbg
- x86 debugger
ตัวอย่าง: ollydbg
70. ghidra
- Software reverse engineering
ตัวอย่าง: ghidra
🛡️ Defensive Security | การป้องกันความปลอดภัย
71. snort
- IDS/IPS system
ตัวอย่าง: snort -dev -l ./log
72. ossec
- Host-based IDS
ตัวอย่าง: ossec-control start
73. lynis
- Security auditing
ตัวอย่าง: lynis audit system
74. rkhunter
- Rootkit detection
ตัวอย่าง: rkhunter --check
75. clamav
- Antivirus
ตัวอย่าง: clamscan -r /
🔧 System Services | บริการระบบ
76. postgresql start
- Start database service
ตัวอย่าง: service postgresql start
77. apache2ctl
- Apache web server control
ตัวอย่าง: apache2ctl start
78. ssh [user@host]
- Secure shell
ตัวอย่าง: ssh user@192.168.1.1
79. tor
- Anonymous networking
ตัวอย่าง: tor
80. privoxy
- Privacy proxy
ตัวอย่าง: privoxy
📊 Reporting Tools | เครื่องมือจัดทำรายงาน
81. maltreport
- Malware analysis report
ตัวอย่าง: maltreport sample.exe
82. faraday
- Collaborative penetration test
ตัวอย่าง: faraday
83. pipal
- Password analysis
ตัวอย่าง: pipal passwords.txt
84. metagoofil
- Metadata extraction
ตัวอย่าง: metagoofil -d example.com -t pdf -l 100 -n 25 -o output -f results.html
85. casefile
- Visual intelligence
ตัวอย่าง: casefile
🎮 Social Engineering | วิศวกรรมสังคม
86. gophish
- Phishing framework
ตัวอย่าง: gophish
87. king-phisher
- Phishing campaign toolkit
ตัวอย่าง: king-phisher
88. social-engineer-toolkit
- Social engineering tools
ตัวอย่าง: setoolkit
89. maltego
- Intelligence gathering
ตัวอย่าง: maltego
90. beef-xss
- Browser exploitation
ตัวอย่าง: beef-xss start
🧪 Miscellaneous | เครื่องมืออื่นๆ
91. proxychains [command]
- Run command through proxy
ตัวอย่าง: proxychains nmap 192.168.1.1
92. macchanger -r [interface]
- Change MAC address
ตัวอย่าง: macchanger -r eth0
93. etherape
- Graphical network monitor
ตัวอย่าง: etherape
94. netdiscover -r [range]
- Network address discovery
ตัวอย่าง: netdiscover -r 192.168.1.0/24
95. p0f
- Passive OS fingerprinting
ตัวอย่าง: p0f
96. dnsenum [domain]
- DNS enumeration
ตัวอย่าง: dnsenum example.com
97. theharvester
- E-mail harvester
ตัวอย่าง: theharvester -d example.com -l 500 -b google
98. responder
- LLMNR/NBT-NS/
