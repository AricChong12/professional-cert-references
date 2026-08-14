# CCIE / Cisco Commands & Configuration Reference

A consolidated reference of commonly used Cisco IOS/IOS-XE commands for CCIE-level study (R&S / Enterprise Infrastructure track). Covers device basics, Layer 2, Layer 3 routing protocols, redundancy, security, QoS, and troubleshooting.

---

## Table of Contents
1. [Basic Device Configuration](#basic-device-configuration)
2. [Interface Configuration](#interface-configuration)
3. [VLANs & Trunking](#vlans--trunking)
4. [Spanning Tree Protocol (STP)](#spanning-tree-protocol-stp)
5. [EtherChannel](#etherchannel)
6. [First Hop Redundancy (HSRP/VRRP/GLBP)](#first-hop-redundancy-hsrpvrrpglbp)
7. [Static Routing](#static-routing)
8. [OSPF](#ospf)
9. [EIGRP](#eigrp)
10. [BGP](#bgp)
11. [Route Redistribution & Filtering](#route-redistribution--filtering)
12. [ACLs](#acls)
13. [NAT](#nat)
14. [VRF / MPLS](#vrf--mpls)
15. [QoS](#qos)
16. [Security (AAA, SSH, Port Security)](#security-aaa-ssh-port-security)
17. [Multicast](#multicast)
18. [Troubleshooting & Verification](#troubleshooting--verification)

---

## Basic Device Configuration

```
enable
configure terminal
hostname R1
enable secret cisco123
no ip domain-lookup
service password-encryption
banner motd # Authorized Access Only #

line console 0
 password cisco
 login
 logging synchronous

line vty 0 4
 password cisco
 login
 transport input ssh

do write memory
copy running-config startup-config
```

---

## Interface Configuration

```
interface GigabitEthernet0/0
 description LINK-TO-R2
 ip address 10.1.1.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 no ip address
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
```

Verification:
```
show ip interface brief
show interfaces status
show interfaces GigabitEthernet0/0
show run interface GigabitEthernet0/0
```

---

## VLANs & Trunking

```
vlan 10
 name USERS
vlan 20
 name VOICE

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 99
```

VTP:
```
vtp mode transparent
vtp domain CCIE
vtp password cisco123
```

Verification:
```
show vlan brief
show interfaces trunk
show vtp status
```

---

## Spanning Tree Protocol (STP)

```
spanning-tree mode rapid-pvst
spanning-tree vlan 10 root primary
spanning-tree vlan 10 priority 4096

interface FastEthernet0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
 spanning-tree guard root
```

MST:
```
spanning-tree mode mst
spanning-tree mst configuration
 name REGION1
 revision 1
 instance 1 vlan 10-20
```

Verification:
```
show spanning-tree
show spanning-tree vlan 10
show spanning-tree summary
```

---

## EtherChannel

```
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode active
 
interface Port-channel1
 switchport mode trunk
```

Modes: `active` (LACP), `passive` (LACP), `desirable`/`auto` (PAgP), `on` (static).

Verification:
```
show etherchannel summary
show etherchannel port-channel
```

---

## First Hop Redundancy (HSRP/VRRP/GLBP)

**HSRP:**
```
interface GigabitEthernet0/0
 ip address 10.1.1.2 255.255.255.0
 standby 1 ip 10.1.1.1
 standby 1 priority 110
 standby 1 preempt
 standby 1 authentication md5 key-string cisco123
 standby 1 track GigabitEthernet0/1 decrement 20
```

**VRRP:**
```
interface GigabitEthernet0/0
 vrrp 1 ip 10.1.1.1
 vrrp 1 priority 110
 vrrp 1 preempt
```

**GLBP:**
```
interface GigabitEthernet0/0
 glbp 1 ip 10.1.1.1
 glbp 1 priority 110
 glbp 1 load-balancing round-robin
```

Verification:
```
show standby brief
show vrrp brief
show glbp brief
```

---

## Static Routing

```
ip route 192.168.10.0 255.255.255.0 10.1.1.2
ip route 0.0.0.0 0.0.0.0 10.1.1.254
ip route 192.168.20.0 255.255.255.0 10.1.1.2 250   ! floating static
```

---

## OSPF

```
router ospf 1
 router-id 1.1.1.1
 network 10.1.1.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 1
 passive-interface GigabitEthernet0/1
 auto-cost reference-bandwidth 10000

interface GigabitEthernet0/0
 ip ospf 1 area 0
 ip ospf priority 100
 ip ospf cost 10
 ip ospf hello-interval 5
 ip ospf dead-interval 20
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 cisco123
```

Area types:
```
area 1 stub
area 1 stub no-summary       ! totally stubby
area 2 nssa
area 2 nssa no-summary
```

Verification:
```
show ip ospf neighbor
show ip ospf interface brief
show ip ospf database
show ip route ospf
show ip protocols
```

---

## EIGRP

```
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 100
  network 10.1.1.0 0.0.0.255
  eigrp router-id 1.1.1.1
  af-interface GigabitEthernet0/0
   authentication mode md5
   authentication key-chain CCIE-KEYS
  exit-af-interface
 exit-address-family
```

Classic (legacy) config:
```
router eigrp 100
 network 10.1.1.0 0.0.0.255
 no auto-summary
 variance 2
 eigrp stub connected
```

Verification:
```
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp
show ip protocols
```

---

## BGP

```
router bgp 65001
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 neighbor 10.1.1.2 remote-as 65002
 neighbor 10.1.1.2 description EBGP-TO-ISP
 neighbor 10.1.1.2 password cisco123
 network 192.168.10.0 mask 255.255.255.0

 address-family ipv4
  neighbor 10.1.1.2 activate
  neighbor 10.1.1.2 soft-reconfiguration inbound
  neighbor 10.1.1.2 route-map SET-LOCAL-PREF in
 exit-address-family
```

Route reflector / iBGP:
```
neighbor 10.1.1.3 remote-as 65001
neighbor 10.1.1.3 update-source Loopback0
neighbor 10.1.1.3 next-hop-self
neighbor 10.1.1.3 route-reflector-client
```

Verification:
```
show ip bgp summary
show ip bgp
show ip bgp neighbors 10.1.1.2
show ip route bgp
```

---

## Route Redistribution & Filtering

```
route-map REDIST-TO-OSPF permit 10
 match tag 100
 set metric-type type-1

router ospf 1
 redistribute eigrp 100 subnets route-map REDIST-TO-OSPF

router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
```

Prefix lists / distribute lists:
```
ip prefix-list PL-DENY seq 5 deny 192.168.99.0/24
ip prefix-list PL-DENY seq 10 permit 0.0.0.0/0 le 32

router ospf 1
 distribute-list prefix PL-DENY in
```

---

## ACLs

```
ip access-list standard MGMT-ACCESS
 permit 10.1.1.0 0.0.0.255
 deny any log

ip access-list extended BLOCK-TELNET
 deny tcp any any eq 23
 permit ip any any

interface GigabitEthernet0/0
 ip access-group BLOCK-TELNET in
```

Verification:
```
show access-lists
show ip interface GigabitEthernet0/0 | include access list
```

---

## NAT

```
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside

ip nat pool NAT-POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
ip access-list standard NAT-SRC
 permit 10.1.1.0 0.0.0.255

ip nat inside source list NAT-SRC pool NAT-POOL overload
ip nat inside source static 10.1.1.5 203.0.113.5
```

Verification:
```
show ip nat translations
show ip nat statistics
```

---

## VRF / MPLS

```
ip vrf CUSTOMER_A
 rd 65001:1
 route-target export 65001:1
 route-target import 65001:1

interface GigabitEthernet0/0
 ip vrf forwarding CUSTOMER_A
 ip address 10.1.1.1 255.255.255.0
```

MPLS basics:
```
mpls ip
mpls label protocol ldp
interface GigabitEthernet0/0
 mpls ip
```

Verification:
```
show ip vrf
show mpls ldp neighbor
show mpls forwarding-table
```

---

## QoS

```
class-map match-all VOICE
 match dscp ef

policy-map WAN-POLICY
 class VOICE
  priority percent 20
 class class-default
  fair-queue

interface GigabitEthernet0/0
 service-policy output WAN-POLICY
```

Verification:
```
show policy-map interface GigabitEthernet0/0
show class-map
```

---

## Security (AAA, SSH, Port Security)

```
ip domain-name ccie.lab
crypto key generate rsa modulus 2048
ip ssh version 2

username admin privilege 15 secret cisco123

aaa new-model
aaa authentication login default local
aaa authorization exec default local

interface FastEthernet0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

---

## Multicast

```
ip multicast-routing

interface GigabitEthernet0/0
 ip pim sparse-mode

ip pim rp-address 10.1.1.1 1
```

Verification:
```
show ip mroute
show ip pim neighbor
show ip igmp groups
```

---

## Troubleshooting & Verification

```
show running-config
show version
show ip route
show cdp neighbors detail
show lldp neighbors detail
ping 10.1.1.1
traceroute 10.1.1.1
debug ip routing
debug ip ospf adj
debug eigrp packets
debug ip bgp updates
show processes cpu sorted
show logging
show tech-support
```

---

### Notes
- Always use `terminal monitor` when debugging over a Telnet/SSH session to see debug output.
- Use `no debug all` (or `undebug all`) to disable all debugging.
- Save configs frequently: `copy running-config startup-config` or `write memory`.
- This reference is a study aid, not a substitute for the official Cisco CCIE Enterprise Infrastructure blueprint and documentation.
