# CCNA ITN — Command Cheat Sheet

## 1. CLI Modes
| Mode | Prompt | How to Enter |
|---|---|---|
| User EXEC | `Switch>` | Default login |
| Privileged EXEC | `Switch#` | `enable` |
| Global Config | `Switch(config)#` | `configure terminal` |
| Interface Config | `Switch(config-if)#` | `interface <type><number>` |
| Line Config | `Switch(config-line)#` | `line console 0` / `line vty 0 15` |

Exit one level: `exit`  |  Exit to privileged EXEC: `end` or `Ctrl+Z`

---

## 2. Basic Device Configuration
```
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret class
service password-encryption
banner motd #Authorized Access Only#
```

## 3. Passwords (Lines)
```
line console 0
 password cisco
 login
 exec-timeout 5 0
 logging synchronous
exit

line vty 0 15
 password cisco
 login
exit
```

## 4. SSH Setup
```
ip domain-name example.com
crypto key generate rsa
username admin secret cisco123
line vty 0 15
 transport input ssh
 login local
exit
ip ssh version 2
```

## 5. Interface Configuration (Router)
```
interface gigabitEthernet 0/0
 description Link to LAN
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit
```

## 6. Interface Configuration (Switch — Management SVI)
```
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.1.1
```

## 7. Switch Port Settings
```
interface fastEthernet 0/1
 description PC1 connection
 duplex full
 speed 100
 switchport mode access
exit
```

## 8. IPv6 Addressing
```
ipv6 unicast-routing
interface gigabitEthernet 0/0
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 address fe80::1 link-local
 no shutdown
exit
```

## 9. DHCP (Client on Interface)
```
interface gigabitEthernet 0/1
 ip address dhcp
 no shutdown
exit
```

## 10. Basic DHCPv4 Server (Router)
```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
exit
```

## 11. Saving / Managing Configuration
```
copy running-config startup-config     (or: write)
erase startup-config
reload
show running-config
show startup-config
copy running-config tftp
```

## 12. Verification / Show Commands
```
show ip interface brief
show ipv6 interface brief
show interfaces
show interfaces description
show version
show flash
show cdp neighbors
show cdp neighbors detail
show arp
show mac address-table
show ip route
show vlan brief
show sessions
show users
show ssh
show history
show terminal
```

## 13. CDP / LLDP
```
cdp run
no cdp run
show cdp neighbors detail
lldp run
show lldp neighbors
```

## 14. Testing Connectivity
```
ping 192.168.1.1
ping ipv6 2001:db8:acad:1::1
traceroute 192.168.1.1
tracert 192.168.1.1        (Windows equivalent)
telnet 192.168.1.1
ssh -l admin 192.168.1.1
```

## 15. Host / PC Commands (Windows CMD)
```
ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /displaydns
ipconfig /flushdns
arp -a
netstat -r
nslookup
```

## 16. Copying / Backing Up IOS
```
show flash
copy tftp flash
copy flash tftp
copy running-config tftp
copy startup-config tftp
```

## 17. Miscellaneous Useful Commands
```
no ip domain-lookup          (stops slow lookups on typos)
logging synchronous
exec-timeout 0 0              (disable timeout - lab use only)
clock set 14:00:00 8 Aug 2026
show clock
```

## 18. Keyboard Shortcuts
| Shortcut | Function |
|---|---|
| `Tab` | Auto-complete command |
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+R` | Redisplay line |
| `Ctrl+Shift+6` | Interrupt process (e.g. stop ping) |
| `Ctrl+Z` | Exit to privileged EXEC |
| `?` | Context-sensitive help |

---
**Tip:** Always verify with `show ip interface brief` after configuring interfaces — statuses should read `up / up`.
