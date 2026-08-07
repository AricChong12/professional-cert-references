# CCNA ENSA (Enterprise Networking, Security & Automation) Cheat Sheet

Covers: Single/Multi-area OSPFv2 & OSPFv3, STP tuning, EtherChannel, WLAN, ACLs, NAT, QoS basics, WAN, Network Security, SNMP/Syslog/NTP, and Network Automation/Programmability.

---

## 1. Single-Area OSPFv2

```
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 passive-interface gi0/1
 passive-interface default          # make all passive, then...
 no passive-interface gi0/0         # ...unpassive the ones facing neighbors
 default-information originate
 auto-cost reference-bandwidth 10000

interface gi0/0
 ip ospf 1 area 0                   # interface-based method
 ip ospf priority 255               # higher = preferred DR
 ip ospf cost 5
 ip ospf hello-interval 10
 ip ospf dead-interval 40
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 <key>

show ip ospf neighbor
show ip ospf interface brief
show ip ospf interface gi0/0
show ip protocols
show ip route ospf
show ip ospf
```

---

## 2. Multiarea OSPFv2

```
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 1
 network 192.168.20.0 0.0.0.255 area 2

# On ABR - route summarization
 area 1 range 192.168.16.0 255.255.240.0

# Stub area (blocks external LSAs)
 area 1 stub

# Totally stubby area (blocks external + summary LSAs) - config on ABR
 area 1 stub no-summary

show ip ospf database
show ip ospf border-routers
show ip ospf summary-address
```

---

## 3. OSPFv3 (IPv6)

```
ipv6 unicast-routing
interface gi0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 ospf 1 area 0

router ospfv3 1
 router-id 1.1.1.1
 log-adjacency-changes

show ipv6 ospf neighbor
show ipv6 ospf interface brief
show ipv6 route ospf
```

---

## 4. Spanning Tree (PVST+, Rapid PVST+, Tuning)

```
spanning-tree mode pvst
spanning-tree mode rapid-pvst
spanning-tree vlan 1,10,20 root primary
spanning-tree vlan 1,10,20 root secondary
spanning-tree vlan 10 priority 4096

interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
 spanning-tree guard root
 spanning-tree link-type point-to-point

show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree interface fa0/1 detail
```

---

## 5. EtherChannel (LACP / PAgP)

```
interface range fa0/1 - 2
 channel-group 1 mode active     # LACP - active
 channel-group 1 mode passive    # LACP - passive
 channel-group 1 mode desirable  # PAgP - desirable
 channel-group 1 mode auto       # PAgP - auto
 channel-group 1 mode on         # static (no negotiation)

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20

show etherchannel summary
show etherchannel port-channel
show interfaces port-channel 1 etherchannel
```

---

## 6. WLAN (Wireless)

```
# WLC (Wireless LAN Controller) - GUI-driven, but key CLI/verification:
show wlan summary
show ap summary
show client summary
show interface summary

# Autonomous AP basics
interface dot11radio 0
 ssid MY_SSID
 encryption mode ciphers aes-ccm
 authentication open
 authentication key-management wpa version 2
 wpa-psk ascii <passphrase>

# Key concepts to know (exam-focused, not CLI-heavy):
# - WEP, WPA, WPA2, WPA3
# - PSK vs Enterprise (802.1X/RADIUS)
# - Autonomous vs Lightweight AP (CAPWAP tunnel to WLC)
# - 2.4GHz vs 5GHz, channel bonding, SSID broadcast
```

---

## 7. ACLs (Standard, Extended, Named)

```
# Standard (1-99 / 1300-1999) - source IP only, place close to destination
access-list 10 permit 192.168.1.0 0.0.0.255
access-list 10 deny any

# Extended (100-199 / 2000-2699) - src/dst IP, protocol, port, place close to source
access-list 110 permit tcp 192.168.1.0 0.0.0.255 any eq 80
access-list 110 permit tcp any any established
access-list 110 deny ip any any

# Named ACL with sequence numbers
ip access-list extended WEB_FILTER
 10 permit tcp any any eq 443
 20 permit tcp any any eq 80
 30 deny ip any any

interface gi0/0
 ip access-group 110 in

# VTY access control
line vty 0 15
 access-class 10 in

show access-lists
show ip interface gi0/0
```

---

## 8. NAT (Static, Dynamic, PAT)

```
# Static NAT
ip nat inside source static 192.168.1.10 203.0.113.10

# Dynamic NAT
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat pool MYPOOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
ip nat inside source list 1 pool MYPOOL

# PAT / NAT Overload
ip nat inside source list 1 interface gi0/1 overload

interface gi0/0
 ip nat inside
interface gi0/1
 ip nat outside

show ip nat translations
show ip nat statistics
clear ip nat translation *
debug ip nat
```

---

## 9. QoS Basics (Concepts + Light Config)

```
# Key concepts for exam:
# - Classification & marking (DSCP, CoS, ToS)
# - Congestion management (queuing: FIFO, WFQ, CBWFQ, LLQ)
# - Congestion avoidance (WRED)
# - Policing vs Shaping (drop vs delay/buffer excess traffic)
# - Trust boundary

# Basic MQC (Modular QoS CLI) example
class-map MATCH_VOICE
 match protocol rtp
policy-map QOS_POLICY
 class MATCH_VOICE
  priority percent 20
interface gi0/0
 service-policy output QOS_POLICY

show policy-map interface gi0/0
show class-map
```

---

## 10. WAN Concepts

```
# Key concepts:
# - Leased line, MPLS, Metro Ethernet, DSL, Cable, VSAT
# - VPN types: site-to-site (IPsec), client (remote access), GRE tunnel

# GRE Tunnel example
interface tunnel0
 ip address 10.0.0.1 255.255.255.252
 tunnel source gi0/0
 tunnel destination 203.0.113.5
 tunnel mode gre ip

show interfaces tunnel0
show ip interface brief
```

---

## 11. Network Security Concepts & Device Hardening

```
# AAA (local)
username admin privilege 15 secret <password>
aaa new-model
aaa authentication login default local
aaa authorization exec default local

# AAA with RADIUS/TACACS+
radius server RAD1
 address ipv4 192.168.1.100 auth-port 1812 acct-port 1813
 key <sharedsecret>
aaa authentication login default group radius local

tacacs server TAC1
 address ipv4 192.168.1.101
 key <sharedsecret>
aaa authentication login default group tacacs+ local

# DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10
interface gi0/1
 ip dhcp snooping trust
interface fa0/1
 ip dhcp snooping limit rate 10

# Dynamic ARP Inspection
ip arp inspection vlan 10
interface gi0/1
 ip arp inspection trust

# Port Security (recap)
interface fa0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict

show aaa sessions
show dhcp snooping
show ip dhcp snooping
show ip arp inspection
show port-security
```

---

## 12. SNMP, Syslog, NTP (Network Management)

```
# NTP
ntp server 192.168.1.100
ntp master 3
clock timezone EST -5
show ntp status
show ntp associations

# Syslog
logging host 192.168.1.50
logging trap informational      # levels 0-7 (0=emergency ... 7=debugging)
logging source-interface gi0/0
show logging

# SNMP
snmp-server community public RO
snmp-server community private RW
snmp-server location HQ-Rack3
snmp-server contact netadmin@example.com
snmp-server enable traps
snmp-server host 192.168.1.50 version 2c public
show snmp
```

---

## 13. Network Automation & Programmability (Concepts)

```
# Key exam concepts (mostly theory, light CLI):
# - Traditional network mgmt vs SDN (control plane / data plane separation)
# - Northbound API (app <-> controller) vs Southbound API (controller <-> device)
# - Controllers: Cisco DNA Center, APIC-EM
# - REST APIs use HTTP methods: GET, POST, PUT, DELETE
# - Data formats: JSON, XML, YAML
# - Configuration mgmt tools: Ansible (agentless, SSH, YAML playbooks),
#   Puppet (agent-based, Ruby), Chef (agent-based, Ruby, "recipes")
# - Cisco DNA Center automates via REST APIs; uses Intent-Based Networking (IBN)

# Example JSON structure to recognize:
# {
#   "hostname": "R1",
#   "interfaces": [
#     {"name": "Gi0/0", "ip": "192.168.1.1"}
#   ]
# }
```

---

## 14. Quick Verification Command Index

| Purpose | Command |
|---|---|
| OSPF neighbors | `show ip ospf neighbor` |
| OSPF interfaces | `show ip ospf interface brief` |
| OSPF database | `show ip ospf database` |
| STP status | `show spanning-tree summary` |
| EtherChannel | `show etherchannel summary` |
| ACLs applied | `show ip interface <int>` |
| ACL contents | `show access-lists` |
| NAT translations | `show ip nat translations` |
| DHCP snooping | `show ip dhcp snooping` |
| ARP inspection | `show ip arp inspection` |
| Port security | `show port-security` |
| NTP status | `show ntp status` |
| Syslog buffer | `show logging` |
| SNMP status | `show snmp` |
| Routing table | `show ip route` |
| IPv6 OSPF | `show ipv6 ospf neighbor` |

---

*Tip: For ENSA, exam weight leans heavily on OSPF (single & multi-area), security (ACLs, DHCP snooping, DAI, AAA), and automation concepts (SDN, APIs, JSON/Ansible) — prioritize those.*
