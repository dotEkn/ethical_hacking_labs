# Lab 2 - Information Gathering & Parallel Hash Attacks

## Overview

This lab introduces information gathering, reconnaissance, scanning and password cracking techniques commonly used during the early phases of an attack.
The lab follows the Reconnaissance and Vulnerability Identification of [Cyber Kill Chain](https://www.microsoft.com/sv-se/security/business/security-101/what-is-cyber-kill-chain).

The tasks focus on DNS Reconnaissance, network scanning, firewall filtering analysis, and hash cracking - using both rainbow tables and parallel brute-force techniques.

---

## Environment

- **Operating System:** Kali Linux
- **Platform:** WSL2
- **Tools:** DMitry, dig, nslookup, host, Nmap, Ophcrack and Hashcat

---

## 2.1 Information Gathering Using DNS Reconnaissance

### 2.1.1 Information Gathering with DMitry

[DMitry](https://github.com/jaygreig86/dmitry) us a reconnaissance tool designed to collect a wide range of information about a target domain or host. It combines multiple information-gathering techniques into a single utility.

DMitry can gather:
- WHOIS information
- Netcraft data
- DNS information
- Subdomain enumeration
- TCP port scan results
- Email address harvesting

This tool is useful in the early reconnaissance phase to quickly get an overview of a target.

---

### 2.1.2 DNS Zone Transfer Attempts

A [DNS zone transfer (AXFR)](https://learn.microsoft.com/en-us/windows-server/networking/dns/zone-types) attempts to retrieve the full dNS zone file from an authorative name server.

Zone transfer attemts were performed using the following tools:
- `dig`
- `nslookup`
- `host`
An example using `dig`
```
dig AXFR du.se @ns1.du.se
```
The zone transfer attempt was unsuccessful because the domain is protected using [DNSSEC](https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en)
and properly configured to restrict zone transfers.

### 2.1.3 Identifying Authorative DNS Servers Without DMitry
Authorative DNS servers can be identified using standard DNS utilities by querying Name Server (NS) records.

Example Command:
```
dig du.se NS
```

The returned NS records reveal the primary and secondary DNS servers to the domain. These servers can then be tested individually for misconfigs such as _open_ zone transfers.

### 2.1.4 Reverse DNS Brute Froce Enumeration
[Reverse DNS Brute force](https://www.geeksforgeeks.org/ethical-hacking/what-is-dns-enumeration/)
is a technique used to identify hostnames by resolving IP addresses back to DNS names when zone transfers are unavailable.

A CIDR network range was selected, and IP adresses within a `/24` subnet were enumerated.

IP addresses were generated using:
```
netenum <CIDR>/24 > netname-ips.txt
```
These addresses were then used as input for reverse DNS lookup.

### 2.1.5 Reverse DNS Script Analysis
The script `bf-dns.ph` automates the reverse DNS brute force by:
1. Reading a list of IP addresses.
2. Performing PTR records lookups for each address.
3. Returning valid hostnames.
The script performs the same task as repeated `host` or `dig -x` commands but in an automated manner.

### 2.1.6 Forward DNS Brute Force Enumeration
[Forward DNS Brute force enumeration](https://www.vaadata.com/blog/subdomain-enumeration-techniques-and-tools/) is used when DNS servers cannot be interrogated directly.

This technique works by:
- Guessing common hostnames.
- Appending them to target domain.
- Performing DNS lookups to identify valid subdomains.
Tools such as [patator](https://www.kali.org/tools/patator/), and custom wordlists can be used for this purpose.

### 2.1.7 Alternative DNS Reconnaissance Tools
Kali Linux includes several tools that simplify DNS Reconnaissance:
- `dnsenum`
- `dnsrecon`
- `fierce`
- `subfinder`
- Nmap NSE Scripts(`dns-zone-transfer`,`dns-brute`)
These tools automate enumeration and combine multiple reconnaissance techniques.

## 2.2 Target Scanning with Nmap
I used the `scanme.nmap.org` as a target, which is a widely used target for Nmap purposes. _Below are the commands I used for this task._
### TCP Connect Scan
A TCP Connect scan establishes a full TCP handshake with the target.
```
nmap -sT scanme.nmap.org
```
This scan is reliable but slower and easier to detect.

### TCP SYN Scan
A TCP SYN scan sends a SYN packet and analyzes the response without completing the handshake.
```
nmap -sS scanme.nmap.org
```
This scan is faster and more 'stealthy' than a TCP connect scan.

### UDP Scan
UDP scanning was performed using:
```
nmap -sU scanme.nmap.org
```
UDP scans are slower and less reliable because many services do not respond unless valid application data is received.

### OS Detection and Enumeration
Operating system fingerprinting and service enumeration were performed using:
```
nmap -O -A scanme.nmap.org
```
This enables OS detection, version detection, script scanning and traceroute.

### OS and Services Conclusion
Based on the scan results, the target appears to be a Linux-based system running commong network services. Open Ports and service banners provide insight into potential attack surfaces.

## 2.3 IP Filtering
Nmap was selected for firewall and filtering analysis due to it's flexibility and support for specialized scan types, as you'll see below.

The commands used:
```
nmap -sA scanme.nmap.org
nmap -sF scanme.nmap.org
nmap -sN scanme.nmap.org
nmap -sX scanme.nmap.org
```
Filtered ports did not respond to probes, indicating firewall rules blocking traffic. FIN, NULL and XMAS scans was tested to observe firewall behaviour when handling unusual TCP flags.

## 2.4 Cracking Hashes with Rainbow Tables
[Raibow tables](https://www.beyondidentity.com/glossary/rainbow-table-attack) were used to crack LM and NTLM hashes using [Ophcrack](https://ophcrack.sourceforge.io).

The hash file was in .pwdump format and was loaded into Ophcrack, which automatically separated the LM and NT hashes. The hashes were matched against the rainbow tables to recover the plaintext passwords.

```
hjo:sigurdbjornsson
hem:storfiskare11
mdo:TeaAtBreakfast
prb:vivafrance
saa:afkham-ebrahimi
tkv:gagnefROX
mig:pension2010
```
The difference between LM and NT hashes, LM hashes are legacy windows passwords hashes that split the password into two 7-character uppercase parts, making them highly vulnerable to rainbow table attacks. NTLM hashes are stronger and case-sensitive, using the full password, but they are still vulnearble to rainbow tables if unsalted.

## 2.5 Cracking Hashes with Parallel Computing (Hashcat)
Hashcas was used to perform accelerated password recovery using mask-based and brute-force attacks.

Hash identification was performed using:
```
hashcat --identify hashfile1.txt
```
At first I thought Hash 1 was an MD5 hash, but I was wrong - it was a Double-MD5, so the format was MD5(MD5(pass)). So for the first hash, we were tasked to use the rockyou.txt, which can be found in `/usr/share/wordlists/`.

So I ran hashcat with this command:
```
hashcat -a 0 -m 2600 hashfile1.txt rockyou.txt
```
This resulted in: `c042a646d3b38326ad0e3e8a7e12efc2`:`FandBow282926`. This took around 3s to crack.

For the second hash, it was a bit more difficult and I had to break down the task into several parts:
- Structure, tells us its exactly 12 characters.
- Starts with 1 capital letter.
- After that, all lowercase letters.
- We also have a special character `-` or `_`
- And it ends with a 4-digit year, 2000 and current year (2026).

Another clue was given _the hashing is a bit outdated - and the password is not itself random_.

I ran the same command as above to identify the hash:
```
hashcat --identify hashfile2.txt
```
I tried several different rulesets (given by the `man hashcat`). This still gave me nothing and I couldn't crack it, with the _NTLM_ hashmode.

Afterwards, I read up on [NTLM](https://www.tarlogic.com/cybersecurity-glossary/ntlm-hash/) and found out that it's basically built upon MD4, which gave me this command:
```
hashcat -m 900 -O hashfile2.txt -a 3 ?u?l?l?l?l?l?l20?d?d
```
And with that, I cracked it!
```
a1a7d311c5f1417694b07cbb47d08f00:Happier-2025
```
>[!NOTE]
_My group in the previous course used vulnerability scanners, so I didn't include it in this lab._
---
# Feedback
This lab provided a thorough instruction to reconnaissance, scanning, and password cracking techniques. The tasks demonstrated how attackers gather information and identify weaknesses in systems. The hand-on approach combined with theoretical context made the lab both informative and practical, thank you.
