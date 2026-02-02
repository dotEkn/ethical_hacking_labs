# Lab 3.1 - Damn Vulnerable Web Application (DVWA)

## Overview
The objective of Lab 3.1 was to identify and exploit common web application vulnerabilties using **Damn Vulnerable Web Application (DVWA)**.\
DVWA is intentionally insecure and provides a safe environment for practicing real-world web attacks.

The lab focused on authentication weaknesses, injection vulnerabilites, file handling flaws, and cross-site scripting (XSS).

---

## Environment & Setup

### Tools Used
- Kali Linux
- Docker
- Damn Vulnerable Web Application (DVWA)
- Firefox (Developer Tools)
- OWASP ZAP
- Wordlists (dirb & rockyou.txt)
- John the Ripper
- Hashcat
- VirtualBox
---

### DVWA Deployment

DVWA was deployed using Docker:
```
docker compose up -d
```
Once running, DVWA was accessible at:
```
http://<ip_address>:4280
```
### Initial Configuration
I followed the instructions handed to me during the lab, so at first I navigated to the website, configured the `setup/php` where I created the database and later logged in with the default credentials:
```
username: admin
password: password
```
Once logged in I could configure it more, where the _DVWA Security Level_ was set to _Low_.

## Vulnerabilites Exploited

### Command Injection
**Vulnerability:**\
User input is passed directly to system commands without sanitization.\
**Payloads used:**
```
& whoami &
& id &
```
**Result:**\
Arbitrary system commands were executed on the server as the `www-data` user.

### Cross-Site Request Forgery (CRSF)
**Vulnerability:**\
Sensitive actions lack CSRF protection and rely only on session cookies.

**Observation:**\
Authenticated password-change requests could be replayed or forged without user interaction.

**Result:**\
State-changing actions could be executed without explicit user consent.

### File Inclusion (Local File Inclusion - LFI)
**Vulnerability:**\
User-controlled file paths are passed to PHP `include()` functions.

**Payloads used:**
```?
page=../../../../etc/passwd
```
**Result:**\
The `etc/passwd` file was succesfully disclosed, confirming LFI. The website exposed the root:daemon:user:etc:

### File Upload
**Vulnerability:**\
Uploaded files are not properly validated.

**Payload uploaded:**
```
<?php
phpinfo():
?>
```
The uploaded file was executed via the file inclusion vulnerability.

**Result:**
Server-side PHP code execution was confirmed.

### SQL Injection
**Vulnerability:**\
User input is concatenated directly into SQL queries without parameterization.

**Payloads used:**
```
' OR 1=1#
```
**Result:**
- Authentication Bypass
- Disclosure of database contents, including user credentials and hashes

### Cross-Site Scripting (DOM-Based XSS)
**Vulnerability:**\
Client-side JavaScript insers unsanitized input into DOM.

**Payload used:**
```
<script>alert(document.domain.concat(\"n").concat(window.origin))</script>
```
**Result:**
JavaScript executed in the victim's browser context.

### Cross-Site Scripting (Reflected)
**Vulnerability:**\
User input is reflected in server responses without output encoding.

**Payloads used:**
```
<script>alert('XSS')</script>
```
**Result:**
The script executed immediately when the request was processed.

### Cross-Site Scripting (Stored)
**Vulnerability:**\
User input is stored server-side and rendered to all users.

**Payload used:**
```
<script>alert('XSS')</script>
```
**Result:**
The script executed every time the affected page was loaded.

# Lab 3.2 - Privilege Escalation and Post-Exploitation Enumeration

## Overview

In this lab, the goal was to identify system vulnerabilities, escalate privileges, and perform post-exploitation enumeration to demonstrate full system compromise.
The target system was a Linux server containing a single non-root user (`bobby`) and a web application.

---

## Initial Access
Both Kali and WebApplicationLab VM were configured using **Host-Only Networking** in VirtualBox to esnure they were on the same subnet.

| System | IP Address |
|--------|------------|
| Kali (Attacker) | `192.168.56.102`|
| WebApplicationLab (Victim) | `192.168.56.101`|

Connectivity was verified using `ping` and browser access.

---

## Vulnerabilities Exploited

---

## SQL Injection - Guestbook Authentication Bypass

### Vulnerability

The Guestbook login functionality concatenated user input directly into the SQL queries without sanitization or parameterization.

### Method
The login request was intercepted using **OWASP ZAP**.
A SQL injection payload was injected into the authentication parameters.

**Payload used:**
```sql
' OR '1'='1' --
```
**Result:**\
Authentication bypass was achieved, access was granted to the password-protected guestbook page and the database-backed authentication logic was confirmed to be vulnerable.

## Local File Inclusion (LFI)
**Vulnerability**
The language selection feature (`loadLang.php`) passed user-controlled input directly to a PHP `include()` function.

### Method
Using OWASP ZAP, the `selectLang` POST parameter was modified to traverse the filesystem.

**Payload used:**
```
selectLang=/../etc/passwd
```
**Result:**
The contents of `/etc/passwd` were successfully discloseed, confirming Location File Inclusion.
