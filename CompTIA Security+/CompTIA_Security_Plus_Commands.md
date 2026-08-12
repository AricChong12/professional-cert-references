# 1. General Security Commands

## Windows
```powershell
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned -Force
whoami /all
net accounts
gpresult /h gpreport.html
```

## Linux
```bash
uname -a
whoami
id <USER>
last
w
```

## macOS
```bash
sw_vers
id -a
log show --predicate 'eventMessage contains "auth"' --last 1h
```

# 2. Networking and Security

```bash
# Windows
ipconfig /all
ipconfig /displaydns
ipconfig /flushdns
route print
arp -a
netstat -ano
netstat -r
nslookup <DOMAIN>
ping -n 4 <IP>
tracert <IP>
pathping <IP>
hostname

# Linux
ifconfig <INTERFACE>
ip addr show
ip link show
ip route show
arp -n
netstat -tulnp
ss -tulnp
ss -ap
nslookup <DOMAIN>
dig <DOMAIN> ANY
dig +trace <DOMAIN>
host <DOMAIN>
ping -c 4 <IP>
traceroute <IP>
hostname
whois <DOMAIN>
curl -I https://<DOMAIN>
wget -qO- https://<DOMAIN>
```

# 3. Network Troubleshooting

## DNS
```bash
# Windows
ipconfig /flushdns
nslookup <DOMAIN> <DNS_SERVER_IP>

# Linux
systemd-resolve --statistics
resolvectl query <DOMAIN>
dig @<DNS_SERVER_IP> <DOMAIN>
```

## DHCP
```bash
# Windows
ipconfig /release
ipconfig /renew

# Linux
dhclient -r <INTERFACE>
dhclient <INTERFACE>
```

## Routing
```bash
# Windows
route add 10.0.0.0 mask 255.0.0.0 <GATEWAY_IP>
route delete 10.0.0.0

# Linux
ip route add 10.0.0.0/8 via <GATEWAY_IP> dev <INTERFACE>
ip route del 10.0.0.0/8
```

## Interfaces
```bash
# Windows
netsh interface show interface
netsh interface set interface name="<INTERFACE>" admin=disabled

# Linux
ip link set dev <INTERFACE> down
ip link set dev <INTERFACE> up
```

## Ports and Connectivity
```bash
# Windows
Test-NetConnection -ComputerName <IP> -Port <PORT>

# Linux
nc -zv <IP> <PORT>
telnet <IP> <PORT>
```

# 4. Firewall Commands

## Windows Firewall
```powershell
Get-NetFirewallProfile
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
New-NetFirewallRule -DisplayName "Block Port 445" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Block
Enable-NetFirewallRule -DisplayName "Remote Desktop"
```

## Linux UFW
```bash
sudo ufw enable
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw deny from <IP> to any
```

## Linux iptables
```bash
sudo iptables -L -n -v
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -s <IP> -j DROP
```

## Linux nftables
```bash
sudo nft list ruleset
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
```

## firewalld
```bash
sudo firewall-cmd --state
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```

# 5. Windows Security

## PowerShell Security Commands
```powershell
Get-ExecutionPolicy -List
Set-ExecutionPolicy AllSigned
Get-AuthenticodeSignature <FILE>
```

## Windows Defender
```powershell
Get-MpComputerStatus
Start-MpScan -ScanType QuickScan
Update-MpSignature
Set-MpPreference -DisableRealtimeMonitoring $false
```

## Windows Event Logs
```powershell
Get-EventLog -LogName Security -Newest 50
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624} -MaxEvents 10
```

## Services and Processes
```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}
Stop-Service -Name "<SERVICE_NAME>"
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Stop-Process -Id <PID>
```

## Users and Groups
```powershell
net user
net localgroup Administrators
net user <USER> /active:no
```

## Local Security Configuration
```cmd
secedit /export /cfg C:\security_policy.inf
secedit /configure /db C:\Windows\security\local.sdb /cfg C:\security_policy.inf
gpupdate /force
```

# 6. Linux Security

## Users and Groups
```bash
cat /etc/passwd
cat /etc/shadow
cat /etc/group
users
groups <USER>
```

## Permissions and Sudo
```bash
ls -la <FILE>
sudo -l
sudo visudo
```

## Processes and Services
```bash
ps aux
systemctl list-units --type=service --state=running
sudo systemctl stop <SERVICE_NAME>
sudo systemctl disable <SERVICE_NAME>
```

## Logs
```bash
sudo tail -f /var/log/auth.log
sudo tail -f /var/log/secure
sudo grep "Failed password" /var/log/auth.log
```

## SSH Hardening
```bash
# Editing SSH Config file path: /etc/ssh/sshd_config
# Verification check:
sshd -t
sudo systemctl restart sshd
```

## File Ownership and Permissions
```bash
chown root:root <FILE>
chmod 700 <FILE>
chmod 644 <FILE>
```

# 7. Authentication and Access Control

## Linux
```bash
sudo useradd -m -s /bin/bash <USER>
sudo userdel -r <USER>
sudo passwd <USER>
sudo usermod -aG sudo <USER>
sudo chage -M 90 -m 7 -W 7 <USER>
```

## Windows Local Users & Groups
```powershell
New-LocalUser -Name "<USER>" -Description "Security Account" -NoPassword
Add-LocalGroupMember -Group "Administrators" -Member "<USER>"
Remove-LocalUser -Name "<USER>"
Set-LocalUser -Name "<USER>" -PasswordNeverExpires $false
```

# 8. File and Data Security

## File System Permissions
```bash
chmod 600 <FILE>
chmod u+s <FILE>
chmod +t <DIRECTORY>
chown <USER>:<GROUP> <FILE>
chgrp <GROUP> <FILE>
```

## ACLs (Linux)
```bash
getfacl <FILE>
setfacl -m u:<USER>:rwx <FILE>
setfacl -x u:<USER> <FILE>
```

## Encryption/Decryption Examples
```bash
gpg -c <FILE>
gpg -d <FILE>.gpg > <FILE>
```

## Hashing & File Integrity Verification
```bash
# Windows
Get-FileHash -Algorithm SHA256 <FILE>
certutil -hashfile <FILE> SHA256

# Linux
md5sum <FILE>
sha1sum <FILE>
sha256sum <FILE>
sha512sum <FILE>
sha256sum -c hashes.txt
```

# 9. Cryptography

## OpenSSL Key Generation
```bash
# Generate private RSA key
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:3072

# Generate ECC private key
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out ecc_private.pem
```

## CSR and Certificate Creation
```bash
# Generate CSR
openssl req -new -key private_key.pem -out request.csr -subj "/C=US/ST=State/L=City/O=Company/CN=<DOMAIN>"

# Create self-signed certificate
openssl req -x509 -new -nodes -key private_key.pem -sha256 -days 365 -out certificate.pem
```

## Certificate and CSR Inspection
```bash
openssl req -text -noout -verify -in request.csr
openssl x509 -text -noout -in certificate.pem
```

# 10. PKI and Certificates

```bash
# Inspect Certificate Chain
openssl s_client -connect <DOMAIN>:443 -showcerts

# Verify Certificate matches Private Key
openssl x509 -noout -modulus -in certificate.pem | openssl md5
openssl rsa -noout -modulus -in private_key.pem | openssl md5

# Convert DER to PEM
openssl x509 -inform der -in certificate.cer -out certificate.pem

# PKCS12 (.pfx/.p12) Export
openssl pkcs12 -export -out certificate.pfx -inkey private_key.pem -in certificate.pem
```

# 11. SSH

```bash
# Key Generation
ssh-keygen -t ed25519 -C "admin@<DOMAIN>"
ssh-keygen -t rsa -b 4096

# Copying Public Key to Server
ssh-copy-id -i ~/.ssh/id_ed25519.pub <USER>@<IP>

# Local Configuration and SSH Agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Secure Connection with specific options
ssh -i ~/.ssh/id_ed25519 -o Protocol=2 <USER>@<IP>

# Key Verification
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

# 12. Secure File Transfer

```bash
# SCP (Secure Copy)
scp -i ~/.ssh/id_ed25519 <FILE> <USER>@<IP>:/path/to/destination/
scp <USER>@<IP>:/path/to/remote/file <LOCAL_PATH>

# SFTP (Secure FTP)
sftp -i ~/.ssh/id_ed25519 <USER>@<IP>

# Rsync over SSH
rsync -avz -e "ssh -i ~/.ssh/id_ed25519" /src/dir/ <USER>@<IP>:/dest/dir/
```

# 13. Logs and Monitoring

## Linux System Logs
```bash
journalctl -xe
journalctl -u sshd --since "1 day ago"
sudo tail -n 100 /var/log/syslog
sudo dmesg | grep -i "error"
sudo grep -i "failed" /var/log/auth.log
```

## Windows Event Log (PowerShell)
```powershell
Get-WinEvent -LogName "System" -MaxEvents 50 | Where-Object {$_.LevelDisplayName -eq "Error"}
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4720}
```

# 14. Processes and Services

## Linux
```bash
ps -ef
ps aux --sort=-%cpu
top -b -n 1
htop
kill -9 <PID>
systemctl status <SERVICE_NAME>
systemctl restart <SERVICE_NAME>
```

## Windows
```powershell
tasklist /v
taskkill /PID <PID> /F
Get-Process -Name "<PROCESS_NAME>"
Stop-Process -Name "<PROCESS_NAME>" -Force
Get-Service -Name "<SERVICE_NAME>"
Stop-Service -Name "<SERVICE_NAME>"
```

# 15. Security Auditing

## Windows Audit Policy
```cmd
auditpol /get /category:*
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
```

## Linux Audit Daemon (auditd)
```bash
sudo systemctl status auditd
sudo auditctl -l
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo ausearch -k passwd_changes
sudo aureport -au
```

## File Integrity Checking (AIDE)
```bash
sudo aideinit
sudo aide --check
```

# 16. Vulnerability and Configuration Checking

```bash
# Windows Patch/Update Status
wmic qfe list brief /format:table
Get-HotFix

# Linux Update Check
sudo apt-get update && sudo apt-get --just-print upgrade
sudo yum check-update

# Open Ports and Running Services (Local Check)
# Windows
netstat -ab
# Linux
ss -tulpn
```

# 17. Malware and Incident Response

## Defensive Investigation
```bash
# Windows Defender Scan
Start-MpScan -ScanType FullScan

# Locate Suspicious Files and Hash (Linux)
find / -name "*.sh" -mtime -2 2>/dev/null
sha256sum /usr/bin/login

# Network Connection Check
ss -apn | grep -i "established"
netstat -naob

# Isolation/Disabling Interface
# Linux
sudo ip link set dev <INTERFACE> down
# Windows
Disable-NetAdapter -Name "<INTERFACE_NAME>" -Confirm:$false
```

# 18. Network Traffic Analysis

```bash
# Capture Traffic on Specific Interface (tcpdump)
sudo tcpdump -i <INTERFACE> -c 10

# Capture Traffic with filters (tcpdump)
sudo tcpdump -i <INTERFACE> port 80 or port 443

# TShark (Wireshark CLI)
tshark -i <INTERFACE> -c 20
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri
```

# 19. Packet Capture

```bash
# Capture raw packets and save to pcap file
sudo tcpdump -i <INTERFACE> -w output.pcap

# Read packets from pcap file
sudo tcpdump -r output.pcap

# Filtering by IP and Port
sudo tcpdump -r output.pcap host <IP> and port <PORT>

# Filtering by Protocol (TCP/UDP/ICMP)
sudo tcpdump -r output.pcap icmp
```

# 20. Digital Forensics

```bash
# File Hashes
sha256sum <FILE>
certutil -hashfile <FILE> SHA256

# Timestamp Verification
stat <FILE>

# Disk Info and Mounted Devices
df -h
lsblk
mount

# Finding files created in last 24h
find / -type f -mtime -1 2>/dev/null
```

# 21. Security Automation

## PowerShell: Log Failure Checker
```powershell
$events = Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 50
foreach ($e in $events) {
    Write-Output "Failed login: $($e.TimeCreated) - $($e.Properties[5].Value)"
}
```

## Bash: File Integrity Checker
```bash
#!/bin/bash
TARGET="/usr/bin/passwd"
KNOWN_HASH="<HASH_VALUE>"
CURRENT_HASH=$(sha256sum $TARGET | awk '{print $1}')
if [ "$KNOWN_HASH" != "$CURRENT_HASH" ]; then
    echo "Warning: Hash mismatch on $TARGET"
fi
```

## Python: Socket Connection Tester
```python
import socket
target = "<IP>"
port = 443
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(2.0)
result = s.connect_ex((target, port))
if result == 0:
    print(f"Port {port} is open")
else:
    print(f"Port {port} is closed")
s.close()
```

# 22. Cloud and Container Security

## Docker
```bash
docker ps --quiet | xargs docker inspect --format '{{ .Id }}: {{ .HostConfig.Privileged }}'
docker image inspect <IMAGE_ID>
docker logs <CONTAINER_ID>
docker stats --no-stream
```

## AWS CLI
```bash
aws iam list-users
aws iam get-credential-report
aws ec2 describe-security-groups --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}"
```

## Azure CLI
```bash
az ad user list --output table
az role assignment list --all --output table
az network nsg list --output table
```

# 23. Wireless Security

## Windows
```cmd
netsh wlan show interfaces
netsh wlan show profiles
netsh wlan show profile name="<SSID>" key=clear
```

## Linux
```bash
iwconfig
nmcli device wifi list
iw dev <INTERFACE> link
```

# 24. System Hardening

## Windows
```powershell
Set-Service -Name "RemoteRegistry" -StartupType Disabled
Stop-Service -Name "RemoteRegistry"
Set-LocalUser -Name "Administrator" -AccountExpires (Get-Date)
Set-NetFirewallProfile -All -Enabled True
```

## Linux
```bash
# Disable USB Storage
echo "install usb-storage /bin/true" | sudo tee /etc/modprobe.d/disable-usb-storage.conf

# Password Quality Control Config Check
cat /etc/pam.d/common-password

# Setting Sticky Bit on temp directory
sudo chmod +t /tmp
```

# 25. Backup and Recovery

## Windows
```powershell
wbadmin start backup -backupTarget:<BACKUP_DRIVE>: -include:C: -allCritical -quiet
```

## Linux
```bash
# Incremental backup using rsync
rsync -av --delete /src/dir/ /backup/dir/

# Compression using tar
tar -czvf backup.tar.gz /path/to/important_data/

# Restore command
tar -xzvf backup.tar.gz -C /restored_destination/
```

# 26. Useful Security+ Exam Commands

```bash
# Network Discovery and Mapping
ping -c 4 <IP>
nslookup <DOMAIN>
dig <DOMAIN>
traceroute <IP>

# Port Checking and Network Status
ss -tulnp
netstat -ano

# Packet Analysis
tcpdump -i <INTERFACE> -w file.pcap
tcpdump -r file.pcap

# Cryptography and PKI
openssl x509 -text -noout -in cert.pem
openssl req -new -key private.key -out req.csr

# File Integrity Checks
sha256sum <FILE>
Get-FileHash -Algorithm SHA256 <FILE>

# User Audits
whoami
id
last
net user
```
