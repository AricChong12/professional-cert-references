# SRWE CCNA — Full Command Reference

All commands used across the SRWE curriculum, organized by module/topic.

---

## 1. Basic Device Configuration

```
enable
configure terminal
hostname SW1
no ip domain-lookup
banner motd #Authorized Access Only#
enable secret cisco12345
service password-encryption
line console 0
 password ciscoconpass
 login
 exec-timeout 5 0
 logging synchronous
line vty 0 15
 password ciscovtypass
 login
 transport input ssh
line vty 0 4
 login local
username admin privilege 15 secret cisco12345
!
crypto key generate rsa
ip ssh version 2
ip domain-name example.com
!
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
ip default-gateway 192.168.1.1        ! (Layer 2 switch only)
!
copy running-config startup-config
write memory
reload
erase startup-config
delete flash:vlan.dat
show running-config
show startup-config
show version
show flash
show ip interface brief
show interfaces
show mac address-table
show cdp neighbors
show cdp neighbors detail
show lldp neighbors
show ip route
show arp
show clock
show history
show terminal
show users
show sessions
show interfaces description
copy tftp flash
copy flash tftp
copy running-config tftp
```

---

## 2. Switch Interface / Duplex / Speed

```
interface fa0/1
 duplex full
 speed 100
 duplex auto
 speed auto
 description Link-to-Server
 shutdown
 no shutdown
show interfaces status
show interfaces fa0/1
show controllers
```

---

## 3. VLANs

```
vlan 10
 name SALES
vlan 20
 name IT
no vlan 10
!
interface fa0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 150
!
interface range fa0/1-5
 switchport access vlan 10
!
show vlan
show vlan brief
show vlan id 10
show interfaces vlan 1
show interfaces fa0/1 switchport
```

---

## 4. Trunking / DTP

```
interface fa0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q      ! (on switches supporting ISL/dot1q)
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30
 switchport trunk allowed vlan add 40
 switchport trunk allowed vlan remove 20
 switchport trunk allowed vlan all
 switchport nonegotiate
!
interface fa0/3
 switchport mode dynamic auto
 switchport mode dynamic desirable
!
show interfaces trunk
show dtp interface fa0/2
show interfaces fa0/2 switchport
```

---

## 5. Inter-VLAN Routing

**Router-on-a-stick (subinterfaces)**
```
interface g0/0/1
 no shutdown
interface g0/0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
interface g0/0/1.99
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0
```

**Layer 3 switch (SVI + routed port)**
```
ip routing
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface g1/0/1
 no switchport
 ip address 10.1.1.1 255.255.255.252
!
show ip route
show ip interface brief
show interfaces vlan 10
```

---

## 6. EtherChannel

```
interface range fa0/1-2
 channel-group 1 mode active            ! LACP
 channel-group 1 mode passive           ! LACP
 channel-group 1 mode desirable         ! PAgP
 channel-group 1 mode auto              ! PAgP
 channel-group 1 mode on                ! static, no protocol
!
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30
!
no channel-group 1
!
show etherchannel summary
show etherchannel port-channel
show etherchannel 1 port-detail
show interfaces port-channel 1
show run interface port-channel 1
show spanning-tree
```

---

## 7. Spanning Tree Protocol

```
spanning-tree mode pvst
spanning-tree mode rapid-pvst
spanning-tree mode mst
!
spanning-tree vlan 1 priority 4096
spanning-tree vlan 1 root primary
spanning-tree vlan 1 root secondary
spanning-tree vlan 1,10,20 root primary
!
interface fa0/1
 spanning-tree portfast
 spanning-tree portfast trunk
 spanning-tree bpduguard enable
 spanning-tree bpdufilter enable
 spanning-tree guard root
 spanning-tree guard loop
 spanning-tree cost 15
 spanning-tree vlan 10 cost 15
 spanning-tree vlan 10 port-priority 16
!
spanning-tree portfast default
spanning-tree portfast bpduguard default
spanning-tree extend system-id
spanning-tree backbonefast
spanning-tree uplinkfast
!
show spanning-tree
show spanning-tree vlan 10
show spanning-tree interface fa0/1
show spanning-tree summary
show spanning-tree root
show spanning-tree bridge
```

**Errdisable recovery**
```
errdisable recovery cause bpduguard
errdisable recovery interval 30
show errdisable recovery
show interfaces status err-disabled
```

---

## 8. First Hop Redundancy Protocols

**HSRP**
```
interface g0/0
 standby 1 ip 192.168.1.1
 standby 1 priority 150
 standby 1 preempt
 standby version 2
 standby 1 timers 1 3
 standby 1 timers msec 200 msec 750
 standby 1 authentication md5 key-string cisco123
 standby 1 track g0/1 decrement 60
show standby brief
show standby
```

**VRRP**
```
interface g0/0
 vrrp 1 ip 192.168.1.1
 vrrp 1 priority 150
 vrrp 1 preempt
 vrrp 1 timers advertise 1
show vrrp
show vrrp brief
```

**GLBP**
```
interface g0/0
 glbp 1 ip 192.168.1.1
 glbp 1 priority 150
 glbp 1 preempt
 glbp 1 load-balancing round-robin
show glbp
show glbp brief
```

---

## 9. Static & Default Routing

```
ip route 192.168.2.0 255.255.255.0 10.1.1.2
ip route 192.168.2.0 255.255.255.0 g0/0/1
ip route 192.168.2.0 255.255.255.0 10.1.1.2 200
ip route 0.0.0.0 0.0.0.0 10.1.1.1
ip route 0.0.0.0 0.0.0.0 g0/0/0
!
ipv6 unicast-routing
ipv6 route 2001:db8:1::/64 2001:db8:2::2
ipv6 route ::/0 2001:db8:2::1
!
show ip route
show ip route static
show ip route 192.168.2.0
show ipv6 route
```

---

## 10. OSPF (as covered in SRWE)

```
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 passive-interface g0/0
 default-information originate
!
interface g0/0
 ip ospf 1 area 0
 ip ospf priority 0
 ip ospf cost 10
!
show ip protocols
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
```

---

## 11. DHCPv4

```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool LAN-POOL-10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
 domain-name example.com
 lease 7 0 0
 lease infinite
no service dhcp
service dhcp
!
interface g0/0/0
 ip helper-address 192.168.10.5
!
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip dhcp server statistics
show running-config | section dhcp
```

**DHCPv4 client**
```
interface g0/0
 ip address dhcp
show dhcp lease
```

---

## 12. IPv6 Addressing / SLAAC / DHCPv6

```
ipv6 unicast-routing
interface g0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 address fe80::1 link-local
 ipv6 enable
 ipv6 nd other-config-flag
 ipv6 nd managed-config-flag
 ipv6 nd prefix default no-advertise
 ipv6 nd ra suppress all
!
show ipv6 interface brief
show ipv6 interface g0/0
show ipv6 route
show ipv6 neighbors
```

**Stateless / Stateful DHCPv6**
```
ipv6 dhcp pool LAN-POOL
 dns-server 2001:db8:1::5
 domain-name example.com
!
interface g0/0
 ipv6 dhcp server LAN-POOL
 ipv6 nd other-config-flag
!
ipv6 dhcp pool STATEFUL-POOL
 address prefix 2001:db8:1::/64
 dns-server 2001:db8:1::5
interface g0/0
 ipv6 dhcp server STATEFUL-POOL
 ipv6 nd managed-config-flag
!
show ipv6 dhcp pool
show ipv6 dhcp binding
show ipv6 dhcp interface
```

**DHCPv6 Relay**
```
interface g0/0
 ipv6 dhcp relay destination 2001:db8:1::5
```

---

## 13. Switch Port Security

```
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation protect
 switchport port-security violation restrict
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 switchport port-security mac-address 0011.2233.4455
 switchport port-security aging time 120
 switchport port-security aging type inactivity
 switchport port-security aging static
!
show port-security
show port-security interface fa0/1
show port-security address
!
shutdown
no shutdown          ! to recover an err-disabled port
```

---

## 14. DHCP Snooping / ARP Inspection / Other L2 Security

```
ip dhcp snooping
ip dhcp snooping vlan 10,20
ip dhcp snooping information option
no ip dhcp snooping information option
interface fa0/1
 ip dhcp snooping trust
 ip dhcp snooping limit rate 10
!
show ip dhcp snooping
show ip dhcp snooping binding
```

```
ip arp inspection vlan 10,20
interface fa0/1
 ip arp inspection trust
ip arp inspection validate src-mac dst-mac ip
show ip arp inspection
show ip arp inspection interfaces
show ip arp inspection statistics vlan 10
```

**VLAN/Native VLAN hardening**
```
interface fa0/24
 switchport mode access
 switchport access vlan 999
 shutdown
vlan 999
 name BLACKHOLE
!
interface trunk-port
 switchport trunk native vlan 999
```

**AAA / Local security**
```
aaa new-model
aaa authentication login default local
aaa authentication login default group radius local
radius server RAD1
 address ipv4 10.1.1.10 auth-port 1812 acct-port 1813
 key cisco123
```

---

## 15. WLAN / WLC (mostly GUI in SRWE, but CLI-adjacent items)

```
show wireless summary
show ap summary
show wlan summary
show controllers dot11Radio 0
```
(Most WLC config in SRWE is done through the WLC GUI: WLANs > Create New; Controller > Interfaces; Security > AAA; note SSID, VLAN mapping, Layer 2/3 security settings — CLI equivalents vary by AireOS vs IOS-XE.)

---

## 16. Troubleshooting / Diagnostic Commands (used throughout)

```
ping
ping 192.168.1.1
ping ipv6 2001:db8::1
traceroute 192.168.1.1
telnet 192.168.1.1
ssh -l admin 192.168.1.1
!
show interfaces
show ip interface brief
show ipv6 interface brief
show running-config
show running-config interface fa0/1
show cdp neighbors detail
show lldp neighbors detail
show mac address-table
show mac address-table dynamic
show mac address-table interface fa0/1
show arp
show logging
show debugging
debug spanning-tree events
debug ip dhcp server events
debug standby
no debug all
undebug all
!
show tech-support
show processes cpu
show memory
show environment
```

---

### Notes
- Commands shown in IOS-style syntax as used in Packet Tracer / CML labs for the SRWE course.
- Interface names (`fa0/1`, `g0/0/1`, etc.) vary by platform/model — substitute your actual interface IDs.
- Always pair a config change with the matching `show` command to verify.
