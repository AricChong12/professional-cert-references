# CCNP Enterprise ENCOR (350-401) Command Cheat Sheet

## 1. Basic Device Configuration

```ios
! Hostname
configure terminal
hostname <HOSTNAME>

! Enable Secret
enable secret <PASSWORD>

! Console Line Configuration
line con 0
 password <PASSWORD>
 login
 exec-timeout <MINUTES> <SECONDS>
 logging synchronous
 exit

! VTY Lines Configuration (SSH only)
line vty 0 15
 login local
 transport input ssh
 exec-timeout <MINUTES> <SECONDS>
 logging synchronous
 exit

! Banners
banner motd #
<BANNER_TEXT>
#

! Local Users and Privilege Levels
username <USERNAME> privilege <PRIVILEGE_LEVEL> secret <PASSWORD>

! Password Encryption
service password-encryption

! Domain Name and SSH Configuration
ip domain-name <DOMAIN_NAME>
crypto key generate rsa general-keys modulus <KEYS_SIZE>
ip ssh version 2
ip ssh time-out <SECONDS>
ip ssh authentication-retries <NUMBER>

! NTP Configuration
ntp server <IP_ADDRESS> prefer
ntp master <STRATUM>
ntp authenticate
ntp authentication-key <KEY_ID> md5 <KEY_VALUE>
ntp trusted-key <KEY_ID>

! Clock Configuration
clock set <HH:MM:SS> <DAY> <MONTH> <YEAR>
clock timezone <ZONE> <OFFSET_HOURS>

! Logging and Timestamps
logging host <IP_ADDRESS>
logging trap <SEVERITY_LEVEL>
logging buffered <BYTES>
service timestamps debug datetime msec show-timezone
service timestamps log datetime msec show-timezone

! Verification
show running-config
show ip ssh
show ntp status
show ntp associations
show clock
show logging
```

## 2. Interface Configuration

```ios
! Access Interfaces
interface <INTERFACE>
 switchport mode access
 switchport access vlan <VLAN_ID>
 no shutdown

! Routed Ports
interface <INTERFACE>
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 no shutdown

! Loopback Interfaces
interface loopback <NUMBER>
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 exit

! Descriptions and Speed/Duplex
interface <INTERFACE>
 description <DESCRIPTION_TEXT>
 speed <10|100|1000|auto>
 duplex <half|full|auto>

! Secondary IP Addresses
interface <INTERFACE>
 ip address <IP_ADDRESS> <MASK> secondary

! Administrative Status
interface <INTERFACE>
 shutdown
no shutdown

! Interface Verification
show ip interface brief
show ipv6 interface brief
show interface <INTERFACE>
show interface status
show controllers <INTERFACE>
```

## 3. VLANs and Layer 2

```ios
! VLAN Creation and Naming
vlan <VLAN_ID>
 name <VLAN_NAME>
 exit

! Access Ports
interface <INTERFACE>
 switchport mode access
 switchport access vlan <VLAN_ID>

! Trunk Ports
interface <INTERFACE>
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan <VLAN_ID>
 switchport trunk allowed vlan <VLAN_LIST>

! Voice VLAN
interface <INTERFACE>
 switchport mode access
 switchport access vlan <VLAN_ID>
 switchport voice vlan <VOICE_VLAN_ID>

! Dynamic Trunking Protocol (DTP) Control
interface <INTERFACE>
 switchport nonegotiate
 switchport mode dynamic desirable
 switchport mode dynamic auto

! VLAN Trunking Protocol (VTP)
vtp domain <DOMAIN_NAME>
vtp password <PASSWORD>
vtp mode <server|client|transparent|off>
vtp version <1|2|3>

! EtherChannel Configuration (LACP)
interface range <INTERFACE_RANGE>
 channel-group <GROUP_ID> mode active
! Or mode passive

! EtherChannel Configuration (PAgP)
interface range <INTERFACE_RANGE>
 channel-group <GROUP_ID> mode desirable
! Or mode auto

! Static EtherChannel
interface range <INTERFACE_RANGE>
 channel-group <GROUP_ID> mode on

! Port-Channel logical interface configuration
interface port-channel <GROUP_ID>
 switchport mode trunk
 switchport trunk allowed vlan <VLAN_LIST>

! Layer 2 LACP System Priority and Interface Priority
lacp system-priority <PRIORITY>
interface <INTERFACE>
 lacp port-priority <PRIORITY>

! Layer 2 Verification
show vlan brief
show interfaces trunk
show switchport interface <INTERFACE>
show dtp interface <INTERFACE>
show vtp status
show etherchannel summary
show etherchannel port-channel
show lacp neighbor
show pagp neighbor
```

## 4. Spanning Tree

```ios
! STP Mode Configuration
spanning-tree mode <pvst|rapid-pvst|mst>

! Root Bridge Tuning (PVST+/Rapid-PVST+)
spanning-tree vlan <VLAN_ID> root primary
spanning-tree vlan <VLAN_ID> root secondary
spanning-tree vlan <VLAN_ID> priority <PRIORITY_VALUE>

! MST Configuration
spanning-tree mst configuration
 name <MST_NAME>
 revision <REVISION_NUMBER>
 instance <INSTANCE_ID> vlan <VLAN_LIST>
 exit
spanning-tree mst <INSTANCE_ID> root primary
spanning-tree mst <INSTANCE_ID> priority <PRIORITY_VALUE>

! Port Priority and Port Cost
interface <INTERFACE>
 spanning-tree vlan <VLAN_ID> port-priority <PRIORITY_VALUE>
 spanning-tree vlan <VLAN_ID> cost <COST_VALUE>

! PortFast
interface <INTERFACE>
 spanning-tree portfast
! Or spanning-tree portfast trunk

! BPDU Guard
interface <INTERFACE>
 spanning-tree bpduguard enable
! Global BPDU Guard
spanning-tree portfast edge bpduguard default

! BPDU Filter
interface <INTERFACE>
 spanning-tree bpdufilter enable
! Global BPDU Filter
spanning-tree portfast edge bpdufilter default

! Root Guard
interface <INTERFACE>
 spanning-tree guard root

! Loop Guard
interface <INTERFACE>
 spanning-tree guard loop
! Global Loop Guard
spanning-tree loopguard default

! UniDirectional Link Detection (UDLD)
udld enable
! Or udld aggressive
interface <INTERFACE>
 udld port
! Or udld port aggressive

! STP Verification
show spanning-tree
show spanning-tree vlan <VLAN_ID>
show spanning-tree summary
show spanning-tree detail
show spanning-tree mst
show udld
```

## 5. Inter-VLAN Routing

```ios
! Router-on-a-Stick (ROAS)
interface <INTERFACE>.<SUBINTERFACE_NUMBER>
 encapsulation dot1Q <VLAN_ID>
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 exit

! Switch SVI Configuration
interface vlan <VLAN_ID>
 description <DESCRIPTION_TEXT>
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 no shutdown
 exit

! Layer 3 Switch Routing Activation
ip routing
ipv6 unicast-routing

! Layer 3 Routed Switch Port
interface <INTERFACE>
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 no shutdown

! Inter-VLAN Verification
show ip route
show ipv6 route
show ip interface brief
show ipv6 interface brief
```

## 6. IPv4 and IPv6

```ios
! IPv4 Static and Default Routes
ip route <NETWORK> <MASK> <NEXT_HOP_IP>
ip route <NETWORK> <MASK> <EXIT_INTERFACE>
ip route 0.0.0.0 0.0.0.0 <DEFAULT_GATEWAY_IP>

! Floating Static Route (IPv4)
ip route <NETWORK> <MASK> <NEXT_HOP_IP> <ADMINISTRATIVE_DISTANCE>

! IPv6 Static and Default Routes
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <NEXT_HOP_IPv6>
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <EXIT_INTERFACE>
ipv6 route ::/0 <DEFAULT_GATEWAY_IPv6>

! Floating Static Route (IPv6)
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <NEXT_HOP_IPv6> <ADMINISTRATIVE_DISTANCE>

! Route Verification
show ip route
show ipv6 route
show ip route static
show ipv6 route static
show ip route <IP_ADDRESS>
show ipv6 route <IPv6_ADDRESS>
```

## 7. OSPF

```ios
! OSPFv2 (IPv4) - Network Statement Style
router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
 network <IP_ADDRESS> <WILDCARD_MASK> area <AREA_ID>
 passive-interface <INTERFACE>
 default-information originate
 exit

! OSPFv2 (IPv4) - Interface Level Style
interface <INTERFACE>
 ip ospf <PROCESS_ID> area <AREA_ID>
 ip ospf cost <COST_VALUE>
 ip ospf hello-interval <SECONDS>
 ip ospf dead-interval <SECONDS>

! Reference Bandwidth
router ospf <PROCESS_ID>
 auto-cost reference-bandwidth <SPEED_IN_MBPS>

! OSPFv3 (IPv4 and IPv6)
ipv6 router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
 passive-interface <INTERFACE>
 exit
interface <INTERFACE>
 ospfv3 <PROCESS_ID> ipv6 area <AREA_ID>
 ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>

! Area Configuration - Stub, Totally Stubby, NSSA
router ospf <PROCESS_ID>
 area <AREA_ID> stub
! Totally Stubby Area:
 area <AREA_ID> stub no-summary
! NSSA:
 area <AREA_ID> nssa
! NSSA Totally Stubby:
 area <AREA_ID> nssa no-summary

! OSPFv2 MD5 Authentication (Interface Level)
interface <INTERFACE>
 ip ospf authentication message-digest
 ip ospf message-digest-key <KEY_ID> md5 <KEY_VALUE>

! OSPFv2 MD5 Authentication (Area Level)
router ospf <PROCESS_ID>
 area <AREA_ID> authentication message-digest
interface <INTERFACE>
 ip ospf message-digest-key <KEY_ID> md5 <KEY_VALUE>

! OSPF Verification and Troubleshooting
show ip ospf
show ip ospf database
show ip ospf neighbor
show ip ospf interface <INTERFACE>
show ipv6 ospf
show ipv6 ospf neighbor
show ipv6 ospf interface
show ospfv3 neighbor
```

## 8. EIGRP

```ios
! Classic EIGRP (IPv4)
router eigrp <AS_NUMBER>
 eigrp router-id <ROUTER_ID>
 network <IP_ADDRESS> <WILDCARD_MASK>
 passive-interface <INTERFACE>
 variance <MULTIPLIER>
 auto-summary
 exit

! Named EIGRP Configuration (IPv4 and IPv6)
router eigrp <VIRTUAL_INSTANCE_NAME>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  router-id <ROUTER_ID>
  network <IP_ADDRESS> <WILDCARD_MASK>
  af-interface <INTERFACE>
   passive-interface
  exit-af-interface
  af-interface default
   summary-address <IP_ADDRESS> <MASK>
  exit-af-interface
  topology base
   variance <MULTIPLIER>
  exit-topology
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system <AS_NUMBER>
  router-id <ROUTER_ID>
  topology base
  exit-topology
 exit-address-family

! EIGRP Authentication (Classic)
ip key chain <KEY_CHAIN_NAME>
 key <KEY_ID>
  key-string <KEY_VALUE>
interface <INTERFACE>
 ip authentication mode eigrp <AS_NUMBER> md5
 ip authentication key-chain eigrp <AS_NUMBER> <KEY_CHAIN_NAME>

! EIGRP Authentication (Named Mode)
router eigrp <VIRTUAL_INSTANCE_NAME>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  af-interface <INTERFACE>
   authentication mode md5
   authentication key-chain <KEY_CHAIN_NAME>
  exit-af-interface
 exit-address-family

! EIGRP Verification
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces
show ip eigrp traffic
show ipv6 eigrp neighbors
show ipv6 eigrp topology
show eigrp address-family ipv4 neighbors
show eigrp address-family ipv6 neighbors
```

## 9. BGP

```ios
! BGP Basic Peering (eBGP and iBGP)
router bgp <LOCAL_AS>
 bgp router-id <ROUTER_ID>
 ! eBGP Peer
 neighbor <PEER_IP> remote-as <REMOTE_AS>
 ! iBGP Peer
 neighbor <INTERNAL_PEER_IP> remote-as <LOCAL_AS>

! Peering using Loopback Interfaces
router bgp <LOCAL_AS>
 neighbor <INTERNAL_PEER_IP> remote-as <LOCAL_AS>
 neighbor <INTERNAL_PEER_IP> update-source loopback <NUMBER>

! eBGP Multihop
router bgp <LOCAL_AS>
 neighbor <PEER_IP> ebgp-multihop <HOP_COUNT>

! Address Family Configuration & Network Advertisement
router bgp <LOCAL_AS>
 address-family ipv4 unicast
  neighbor <PEER_IP> activate
  network <IP_ADDRESS> mask <MASK>
  exit-address-family
 !
 address-family ipv6 unicast
  neighbor <IPv6_PEER_IP> activate
  network <IPv6_PREFIX>/<PREFIX_LEN>
  exit-address-family

! Default Route Advertisement
router bgp <LOCAL_AS>
 address-family ipv4 unicast
  neighbor <PEER_IP> default-originate
  exit-address-family

! BGP MD5 Authentication
router bgp <LOCAL_AS>
 neighbor <PEER_IP> password <PASSWORD>

! Path Attribute Manipulation via Route Maps
route-map <MAP_NAME> permit 10
 match ip address prefix-list <PREFIX_LIST_NAME>
 set local-preference <VALUE>
 set metric <MED_VALUE>
 set weight <VALUE>
 set as-path prepend <LOCAL_AS> <LOCAL_AS>

router bgp <LOCAL_AS>
 address-family ipv4 unicast
  neighbor <PEER_IP> route-map <MAP_NAME> <in|out>

! Prefix List and Route Filtering
ip prefix-list <LIST_NAME> seq 10 permit <NETWORK>/<PREFIX_LEN> ge <MIN_LEN> le <MAX_LEN>

router bgp <LOCAL_AS>
 address-family ipv4 unicast
  neighbor <PEER_IP> prefix-list <LIST_NAME> <in|out>

! BGP Communities Configuration
router bgp <LOCAL_AS>
 neighbor <PEER_IP> send-community <standard|extended|both>

route-map <COMMUNITY_MAP> permit 10
 set community <no-advertise|no-export|local-as|COMMUNITY_VAL>

! Clearing BGP Sessions (Soft and Hard Reset)
clear ip bgp *
clear ip bgp * soft
clear ip bgp <PEER_IP> soft in
clear ip bgp <PEER_IP> soft out

! BGP Verification
show ip bgp
show ip bgp summary
show ip bgp neighbors
show ip bgp neighbors <PEER_IP> received-routes
show ip bgp neighbors <PEER_IP> advertised-routes
show ip bgp neighbors <PEER_IP> routes
show bgp ipv6 unicast summary
```

## 10. IP Routing and Path Control

```ios
! Floating Static Route (Tuning Administrative Distance)
ip route <NETWORK> <MASK> <NEXT_HOP_IP> <AD_VALUE>

! Prefix List
ip prefix-list <LIST_NAME> seq <SEQ_NUM> permit <IP_ADDRESS>/<PREFIX_LEN>

! Route Map Creation
route-map <MAP_NAME> permit <SEQ_NUM>
 match ip address prefix-list <LIST_NAME>
 set ip next-hop <IP_ADDRESS>
 exit

! Distribute List (OSPF/EIGRP Route Filtering)
router ospf <PROCESS_ID>
 distribute-list prefix <LIST_NAME> in <INTERFACE>
router eigrp <AS_NUMBER>
 distribute-list prefix <LIST_NAME> out <INTERFACE>

! Route Redistribution (EIGRP into OSPF)
router ospf <PROCESS_ID>
 redistribute eigrp <AS_NUMBER> subnets route-map <MAP_NAME>

! Route Redistribution (OSPF into EIGRP)
router eigrp <AS_NUMBER>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  topology base
   redistribute ospf <PROCESS_ID> metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>

! Policy-Based Routing (PBR)
route-map <PBR_MAP> permit 10
 match ip address <ACL_NUMBER_OR_NAME>
 set ip next-hop <IP_ADDRESS>
 set interface <INTERFACE>
 exit
interface <INTERFACE>
 ip policy route-map <PBR_MAP>

! IP SLA Configuration
ip sla <SLA_ID>
 icmp-echo <TARGET_IP> source-ip <SOURCE_IP>
 frequency <SECONDS>
 exit
ip sla schedule <SLA_ID> start-time now life forever

! Object Tracking tied to IP SLA
track <TRACK_ID> ip sla <SLA_ID> state

! Static Route tracking
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP_IP> track <TRACK_ID>

! Verification
show ip policy
show route-map
show ip sla summary
show ip sla configuration
show track
```

## 11. NAT

```ios
! Static NAT
ip nat inside source static <LOCAL_IP> <GLOBAL_IP>

! Dynamic NAT Pool
ip nat pool <POOL_NAME> <START_IP> <END_IP> netmask <MASK>
access-list <ACL_ID> permit <LOCAL_SUBNET> <WILDCARD>
ip nat inside source list <ACL_ID> pool <POOL_NAME>

! Port Address Translation (PAT) using Pool
ip nat inside source list <ACL_ID> pool <POOL_NAME> overload

! Port Address Translation (PAT) using Interface
ip nat inside source list <ACL_ID> interface <INTERFACE> overload

! Assigning Inside/Outside Interfaces
interface <INTERFACE_INSIDE>
 ip nat inside
interface <INTERFACE_OUTSIDE>
 ip nat outside

! Verification
show ip nat translations
show ip nat translations verbose
show ip nat statistics
clear ip nat translation *
```

## 12. DHCP

```ios
! DHCP Exclusions
ip dhcp excluded-address <START_IP> <END_IP>
ip dhcp excluded-address <SINGLE_IP>

! DHCP Pool Configuration
ip dhcp pool <POOL_NAME>
 network <NETWORK_IP> <MASK>
 default-router <GATEWAY_IP>
 dns-server <DNS_IP_1> <DNS_IP_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
 option <OPTION_CODE> ip <IP_ADDRESS>
 option <OPTION_CODE> ascii "<STRING>"

! DHCP Client
interface <INTERFACE>
 ip address dhcp

! DHCP Relay Agent
interface <INTERFACE_RECEIVING_DHCP_DISCOVER>
 ip helper-address <DHCP_SERVER_IP>

! Verification
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics
show ip dhcp conflict
```

## 13. First Hop Redundancy

```ios
! HSRP Version 2 Configuration (IPv4)
interface <INTERFACE>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority <PRIORITY_VALUE>
 standby <GROUP_ID> preempt
 standby <GROUP_ID> name <GROUP_NAME>

! HSRP Tracking Interface
interface <INTERFACE>
 standby <GROUP_ID> track <INTERFACE_TO_TRACK> <DECREMENT_VALUE>

! HSRP Tracking Object (tied to IP SLA / Tracking)
interface <INTERFACE>
 standby <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>

! HSRP (IPv6)
interface <INTERFACE>
 standby version 2
 standby <GROUP_ID> ipv6 autoconfig
 standby <GROUP_ID> ipv6 <VIRTUAL_IPv6_ADDRESS>
 standby <GROUP_ID> preempt

! VRRP Configuration
interface <INTERFACE>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority <PRIORITY_VALUE>
 vrrp <GROUP_ID> preempt
 vrrp <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>

! GLBP Configuration
interface <INTERFACE>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority <PRIORITY_VALUE>
 glbp <GROUP_ID> preempt
 glbp <GROUP_ID> weighting <WEIGHT_VALUE>
 glbp <GROUP_ID> load-balancing <round-robin|weighted|host-dependent>

! Verification
show standby
show standby brief
show vrrp
show vrrp brief
show glbp
show glbp brief
```

## 14. Wireless LAN

```ios
! WLC AAA Server Configuration
wlan aaa-server radius host <RADIUS_SERVER_IP> auth-port 1812 acct-port 1813 key <SHARED_SECRET>

! Create WLAN and Mapping to Interface/VLAN
wlan <WLAN_NAME> <WLAN_ID> <SSID_NAME>
 client vlan <VLAN_NAME_OR_ID>
 no shutdown

! WLAN Security (WPA2-PSK)
wlan <WLAN_NAME> <WLAN_ID>
 security wpa
  security wpa wpa2
  security wpa wpa2 ciphers aes
  security wpa akm psk set-key ascii <PRE_SHARED_KEY>

! WLAN Security (WPA3-SAE)
wlan <WLAN_NAME> <WLAN_ID>
 security wpa
  security wpa wpa3
  security wpa akm sae

! WLAN Security (802.1X with AAA)
wlan <WLAN_NAME> <WLAN_ID>
 security wpa
  security wpa wpa2
  security wpa wpa2 ciphers aes
  security wpa akm dot1x
  authentication-list <AAA_METHOD_LIST>

! AP Group Configuration
ap tag default-tags
ap-group <GROUP_NAME>
 wlan <WLAN_NAME>

! Join Profile / AP Join Configuration
ap profile <PROFILE_NAME>
 hyperlocation
  exit
ap join-profile <AP_JOIN_PROFILE>
 tftp-server <IP_ADDRESS>

! Wireless Local Profiling
wireless profile policy <POLICY_NAME>
 vlan <VLAN_ID>
 central switching
 central dhcp
 central association
 no shutdown

! Verification
show wlan summary
show wlan id <WLAN_ID>
show ap summary
show wireless tag summary
```

## 15. Wireless Troubleshooting

```ios
! Controller and AP Information
show wlan summary
show ap summary
show ap config general
show ap cdp neighbors
show ap inventory

! Client Status and Debugging
show wireless client summary
show wireless client mac-address <MAC_ADDRESS> detail
show wireless client stats mac-address <MAC_ADDRESS>

! WLAN Config and Tagging Verification
show wireless tag summary
show wireless profile policy summary

! Debugging Commands
debug wireless client mac-address <MAC_ADDRESS>
debug wlan peer
debug aaa authentication
debug dot11
```

## 16. Network Security

```ios
! AAA TACACS+ and RADIUS Server Setup
tacacs server <TACACS_SERVER_NAME>
 address ipv4 <IP_ADDRESS>
 key <SHARED_SECRET>
!
radius server <RADIUS_SERVER_NAME>
 address ipv4 <IP_ADDRESS> auth-port 1812 acct-port 1813
 key <SHARED_SECRET>

! AAA Authentication Lists
aaa new-model
aaa authentication login default group tacacs+ local
aaa authentication login <METHOD_NAME> group radius local

! AAA Authorization Lists
aaa authorization exec default group tacacs+ local
aaa authorization commands <LEVEL> default group tacacs+ local

! AAA Accounting Lists
aaa accounting exec default start-stop group tacacs+
aaa accounting commands <LEVEL> default start-stop group tacacs+

! Access Control Lists (Standard IPv4)
ip access-list standard <ACL_NAME>
 <SEQUENCE_NUM> permit <SOURCE_IP> <WILDCARD>
 <SEQUENCE_NUM> deny any

! Access Control Lists (Extended IPv4)
ip access-list extended <ACL_NAME>
 <SEQUENCE_NUM> permit tcp <SOURCE_IP> <WILDCARD> <DEST_IP> <WILDCARD> eq <PORT>
 <SEQUENCE_NUM> permit udp <SOURCE_IP> <WILDCARD> <DEST_IP> <WILDCARD> eq <PORT>
 <SEQUENCE_NUM> permit icmp any any

! IPv6 ACL
ipv6 access-list <IPv6_ACL_NAME>
 permit ipv6 <SOURCE_PREFIX> <DEST_PREFIX>
 permit icmp any any

! Apply ACL to Interface
interface <INTERFACE>
 ip access-group <ACL_NAME> <in|out>
 ipv6 traffic-filter <IPv6_ACL_NAME> <in|out>

! Control Plane Policing (CoPP)
ip access-list extended <COPP_ACL>
 permit tcp any any eq ssh
 permit ospf any any
!
class-map match-all <COPP_CLASS>
 match access-group name <COPP_ACL>
!
policy-map <COPP_POLICY>
 class <COPP_CLASS>
  police <RATE_IN_BPS> conform-action transmit exceed-action drop
!
control-plane
 service-policy input <COPP_POLICY>

! Secure Management
ip http server
no ip http server
ip http secure-server
ip http authentication local

! Verification
show aaa method-lists all
show aaa sessions
show ip access-lists
show ipv6 access-list
show control-plane host open-ports
show policy-map control-plane
```

## 17. Infrastructure Security

```ios
! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan <VLAN_LIST>
no ip dhcp snooping information option
! Trusting interface
interface <INTERFACE_TO_DHCP_SERVER>
 ip dhcp snooping trust
! Limit Rate
interface <INTERFACE_TO_CLIENT>
 ip dhcp snooping limit rate <PACKETS_PER_SECOND>

! Dynamic ARP Inspection (DAI)
ip arp inspection vlan <VLAN_LIST>
! Trusting interface
interface <INTERFACE_TO_SWITCH_OR_ROUTER>
 ip arp inspection trust

! IP Source Guard (IPSG)
interface <INTERFACE_TO_CLIENT>
 ip verify source
! Or including MAC validation:
 ip verify source tracking port-security

! Port Security
interface <INTERFACE>
 switchport mode access
 switchport port-security
 switchport port-security maximum <MAX_MAC_COUNT>
 switchport port-security violation <protect|restrict|shutdown>
 switchport port-security mac-address sticky
 switchport port-security mac-address <MAC_ADDRESS>

! Storm Control
interface <INTERFACE>
 storm-control broadcast level <PERCENTAGE>
 storm-control multicast level <PERCENTAGE>
 storm-control action <shutdown|trap>

! Private VLANs
vlan <SECONDARY_VLAN_ID>
 private-vlan <community|isolated>
vlan <PRIMARY_VLAN_ID>
 private-vlan primary
 private-vlan association <SECONDARY_VLAN_ID>
! Host Port Configuration
interface <INTERFACE>
 switchport mode private-vlan host
 switchport private-vlan host-association <PRIMARY_VLAN_ID> <SECONDARY_VLAN_ID>
! Promiscuous Port Configuration
interface <INTERFACE>
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping <PRIMARY_VLAN_ID> <SECONDARY_VLAN_ID>

! 802.1X and MAB Port Authentication
aaa new-model
aaa authentication dot1x default group radius
dot1x system-auth-control
!
interface <INTERFACE>
 switchport mode access
 authentication port-control auto
 mab
 dot1x pae authenticator

! Cisco TrustSec (CTS) Basic Configuration
cts sga-enable
cts role-based enforcement
! Apply Security Group Tag (SGT) statically to port
interface <INTERFACE>
 cts manual
  policy static sgt <SGT_VALUE>

! Verification
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip verify source
show mac address-table count
show port-security
show port-security interface <INTERFACE>
show vlan private-vlan
show dot1x all
show cts security-group-table
```

## 18. Multicast

```ios
! Enable Multicast Routing globally
ip multicast-routing
ipv6 multicast-routing

! IGMP Snooping (enabled by default)
ip igmp snooping
ip igmp snooping vlan <VLAN_ID>

! IGMP Version on Interface
interface <INTERFACE>
 ip igmp version <1|2|3>

! PIM Sparse Mode and Dense Mode
interface <INTERFACE>
 ip pim sparse-mode
! Or dense-mode:
 ip pim dense-mode
! Or sparse-dense-mode:
 ip pim sparse-dense-mode

! Static Rendezvous Point (RP) Configuration
ip pim rp-address <RP_IP_ADDRESS> <ACL_OF_GROUPS>

! Auto-RP Configuration
ip pim send-rp-announce <INTERFACE> scope <TTL> group-list <ACL>
ip pim send-rp-discovery <INTERFACE> scope <TTL>
ip pim rp-candidate <INTERFACE>

! Bootstrap Router (BSR) Configuration
ip pim rp-candidate <INTERFACE> group-list <ACL>
ip pim bsr-candidate <INTERFACE>

! Verification
show ip igmp groups
show ip igmp interface
show ip pim neighbor
show ip pim interface
show ip pim rp mapping
show ip mroute
```

## 19. QoS

```ios
! Class Map: NBAR Classification
class-map match-any <CLASS_NAME>
 match protocol http
 match protocol secure-http

! Class Map: ACL-based Classification
class-map match-all <CLASS_NAME>
 match access-group name <ACL_NAME>

! Class Map: CoS or DSCP Marking Match
class-map match-any <CLASS_NAME>
 match ip dscp <DSCP_VALUE>
 match cos <CoS_VALUE>

! Policy Map Construction (Queuing, Shaping, Policing, LLQ)
policy-map <POLICY_NAME>
 class <REAL_TIME_CLASS>
  priority level 1
  police cir <BPS_RATE> conform-action transmit exceed-action drop
 class <DATA_CLASS>
  bandwidth remaining percent <PERCENT>
  random-detect
 class class-default
  shape average <BPS_RATE>

! Service Policy application to Interface
interface <INTERFACE>
 service-policy <input|output> <POLICY_NAME>

! Verification
show class-map
show policy-map
show policy-map interface <INTERFACE>
```

## 20. Network Virtualization

```ios
! VRF Definition
vrf definition <VRF_NAME>
 rd <ROUTE_DISTINGUISHER>
 route-target export <RT_VALUE>
 route-target import <RT_VALUE>
 !
 address-family ipv4
 exit-address-family
 !
 address-family ipv6
 exit-address-family

! Assigning VRF to Interface
interface <INTERFACE>
 vrf forwarding <VRF_NAME>
 ip address <IP_ADDRESS> <MASK>

! VRF Route Leaking (Global to VRF/VRF to VRF via Static Routes)
ip route vrf <VRF_NAME> <DEST_IP> <MASK> <NEXT_HOP_IP> global
ip route <DEST_IP> <MASK> <NEXT_HOP_IP> vrf <VRF_NAME>

! GRE Tunnel Configuration
interface tunnel <NUMBER>
 ip address <IP_ADDRESS> <MASK>
 tunnel source <SOURCE_INTERFACE_OR_IP>
 tunnel destination <DESTINATION_IP>
 tunnel mode gre ip

! IPsec Tunnel Configuration (Crypto Map Style)
crypto isakmp policy <POLICY_NUMBER>
 encr aes
 hash sha256
 authentication pre-share
 group 14
 exit
crypto isakmp key <SHARED_KEY> address <PEER_IP>
!
crypto ipsec transform-set <TRANSFORM_NAME> esp-aes esp-sha256-hmac
 mode tunnel
 exit
!
crypto map <MAP_NAME> <SEQ_NUM> ipsec-isakmp
 set peer <PEER_IP>
 set transform-set <TRANSFORM_NAME>
 match address <ACL_ID>
 exit
! Apply to physical interface
interface <INTERFACE>
 crypto map <MAP_NAME>

! VXLAN Basic Commands (NVE Interface Setup)
interface nve <NUMBER>
 no shutdown
 source-interface loopback <NUMBER>
 member vni <VNI_ID> mcast-group <MCAST_IP>

! Verification
show vrf
show ip route vrf <VRF_NAME>
show interfaces tunnel <NUMBER>
show crypto isakmp sa
show crypto ipsec sa
show nve interface
show nve vni
```

## 21. SD-WAN

```ios
! Cisco SD-WAN System CLI (vEdge/cEdge edge-router OS)
system
 system-ip <SYSTEM_IP>
 domain-id <DOMAIN_ID>
 site-id <SITE_ID>
 vbond <VBOND_IP_OR_DNS>
 organization-name "<ORG_NAME>"

! VPN Configuration (VPN 0 - Transport)
vpn 0
 interface <WAN_INTERFACE>
  ip address <IP_ADDRESS>/<PREFIX_LEN>
  tunnel-interface
   encapsulation ipsec
   color <biz-internet|mpls>
   no shutdown

! VPN Configuration (VPN 512 - Management)
vpn 512
 interface <MGMT_INTERFACE>
  ip address <IP_ADDRESS>/<PREFIX_LEN>
  no shutdown

! Service VPN (User Data VPN)
vpn <VPN_ID>
 router ospf
  redistribute omp
  area 0
   interface <LAN_INTERFACE>
 !
 interface <LAN_INTERFACE>
  ip address <IP_ADDRESS>/<PREFIX_LEN>
  no shutdown

! Verification and Troubleshooting
show control local-properties
show control connections
show control connections-history
show omp peers
show omp tlocs
show omp routes
```

## 22. SD-Access

```ios
! LISP Configuration on Map-Server/Map-Resolver (Control Plane Node)
router lisp
 site <SITE_NAME>
  authentication-key <SHARED_KEY>
  allowed-eid <EID_SUBNET>/<PREFIX_LEN>
  exit
 ipv4 map-server
 ipv4 map-resolver
 ipv4 database-mapping <EID_SUBNET>/<PREFIX_LEN> locator-set <LOCATOR_SET_NAME>

! VXLAN configuration on Edge Nodes
interface nve 1
 source-interface loopback 0
 member vni <VNI_ID> asymmetric

! Apply Security Group Tag (SGT) mappings
cts role-based sg-map vlan <VLAN_ID> sgt <SGT_VALUE>

! Device Verification
show lisp site
show lisp session
show cts role-based permissions
```

## 23. Network Programmability

```ios
! NETCONF Configuration
netconf-yang
netconf-yang cisco-ia auto-sync

! RESTCONF Configuration
ip http secure-server
restconf

! YANG Verification
show platform software yang-management process

! REST API testing using curl (executed from external terminal)
curl -k -u "<USERNAME>:<PASSWORD>" -H "Accept: application/yang-data+json" https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-native:native
curl -k -u "<USERNAME>:<PASSWORD>" -X GET https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces
```

## 24. Automation

```python
# Python: RESTCONF GET request to retrieve interface status
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

url = "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces"
headers = {
    "Accept": "application/yang-data+json",
    "Content-Type": "application/yang-data+json"
}
auth = ("<USERNAME>", "<PASSWORD>")

response = requests.get(url, headers=headers, auth=auth, verify=False)
print(response.json())
```

```yaml
# Ansible Playbook to retrieve system facts from IOS XE
- name: Get IOS XE Device Facts
  hosts: cisco_routers
  gather_facts: no
  connection: ansible.netcommon.network_cli
  tasks:
    - name: Gather facts
      cisco.ios.ios_facts:
        gather_subset: all
      register: device_facts

    - name: Debug Output
      ansible.builtin.debug:
        var: device_facts
```

```json
{
  "ietf-interfaces:interface": {
    "name": "GigabitEthernet1",
    "description": "Configured by RESTCONF",
    "type": "iana-if-type:ethernetCsmacd",
    "enabled": true,
    "ietf-ip:ipv4": {
      "address": [
        {
          "ip": "192.168.1.1",
          "netmask": "255.255.255.0"
        }
      ]
    }
  }
}
```

```yaml
# YAML interface payload
ietf-interfaces:interface:
  name: "GigabitEthernet1"
  description: "Configured by Ansible"
  type: "iana-if-type:ethernetCsmacd"
  enabled: true
  ietf-ip:ipv4:
    address:
      - ip: "192.168.1.1"
        netmask: "255.255.255.0"
```

```bash
# Git Commands
git init
git add <FILE_NAME>
git commit -m "<COMMIT_MESSAGE>"
git remote add origin <REMOTE_URL>
git push -u origin master
git clone <REPO_URL>
git checkout -b <BRANCH_NAME>

# Docker commands
docker build -t <IMAGE_NAME>:<TAG> .
docker run -d --name <CONTAINER_NAME> -p <HOST_PORT>:<CONTAINER_PORT> <IMAGE_NAME>:<TAG>
docker ps
docker images
docker exec -it <CONTAINER_NAME> /bin/bash
```

## 25. Monitoring and Telemetry

```ios
! SNMP Version 2c Configuration
snmp-server community <READ_ONLY_STRING> RO
snmp-server community <READ_WRITE_STRING> RW
snmp-server contact <CONTACT_INFO>
snmp-server host <SNMP_SERVER_IP> version 2c <COMMUNITY_STRING>
snmp-server enable traps

! SNMP Version 3 Configuration
snmp-server group <GROUP_NAME> v3 priv
snmp-server user <USER_NAME> <GROUP_NAME> v3 auth sha <AUTH_KEY> priv aes 128 <PRIV_KEY>

! Syslog Configuration
logging host <IP_ADDRESS>
logging trap <SEVERITY_LEVEL>
logging source-interface <INTERFACE>

! NetFlow Configuration (Traditional)
interface <INTERFACE>
 ip flow ingress
 ip flow egress
! Export details
ip flow-export destination <COLLECTOR_IP> <PORT>
ip flow-export version 9

! Flexible NetFlow (FNF) Configuration
flow record <RECORD_NAME>
 match ipv4 source address
 match ipv4 destination address
 collect counter bytes long
 collect counter packets long
!
flow exporter <EXPORTER_NAME>
 destination <COLLECTOR_IP>
 transport udp <PORT>
 template data timeout 60
!
flow monitor <MONITOR_NAME>
 record <RECORD_NAME>
 exporter <EXPORTER_NAME>
 cache timeout active 60
! Apply to interface
interface <INTERFACE>
 ip flow monitor <MONITOR_NAME> input

! SPAN (Local Mirroring)
monitor session 1 source interface <SOURCE_INTERFACE> both
monitor session 1 destination interface <DESTINATION_INTERFACE>

! ERSPAN (Remote Mirroring over IP)
! Source Switch
monitor session 1 type erspan-source
 source interface <SOURCE_INTERFACE>
 destination
  erspan-id <ID>
  ip address <DEST_SWITCH_IP>
  origin ip address <SOURCE_SWITCH_IP>
! Destination Switch
monitor session 1 type erspan-destination
 destination interface <DEST_INTERFACE>
 source
  erspan-id <ID>
  ip address <DEST_SWITCH_IP>

! Model-Driven/Streaming Telemetry (gRPC Dial-out)
telemetry receiver ip <RECEIVER_IP> port <PORT> protocol grpc-tcp
!
telemetry subscription 101
 selector-profile gRPC-dialout
 sensor-profile path <YANG_PATH>
 receiver ip <RECEIVER_IP> port <PORT>

! Verification
show snmp community
show logging
show ip flow export
show flow monitor
show monitor session 1
```

## 26. High Availability

```ios
! StackWise / StackWise Virtual Setup
stackwise-virtual
 domain <DOMAIN_ID>
! Assign Dual-Active Detection link
interface <INTERFACE>
 stackwise-virtual dual-active-detection

! Stateful Switchover (SSO)
redundancy
 mode sso

! Non-Stop Forwarding (NSF)
router ospf <PROCESS_ID>
 nsf
router eigrp <AS_NUMBER>
 nsf

! ISSU (In-Service Software Upgrade) Verification
issu load version <FLASH_PATH_TO_IMAGE>
issu runversion
issu commitversion

! Verification
show redundancy
show redundancy states
show switch
show switch stackwise-virtual
show switch stackwise-virtual dual-active-detection
```

## 27. Troubleshooting

```ios
! Interfaces Troubleshooting
show interfaces <INTERFACE> status
show interfaces <INTERFACE> accounting
show interface <INTERFACE> counters errors
clear counters <INTERFACE>

! VLANs & Trunks
show vlan brief
show interfaces trunk
show dtp interface <INTERFACE>

! EtherChannel
show etherchannel summary
show lacp neighbor
show pagp neighbor

! STP
show spanning-tree summary
show spanning-tree detail
show spanning-tree inconsistentports

! OSPF
show ip ospf neighbor
show ip ospf interface <INTERFACE>
show ip ospf database
show ip ospf neighbor detail

! EIGRP
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp traffic

! BGP
show ip bgp summary
show ip bgp neighbors <PEER_IP>
show ip bgp neighbors <PEER_IP> advertised-routes

! DHCP
show ip dhcp binding
show ip dhcp server statistics
show ip dhcp conflict

! NAT
show ip nat translations
show ip nat statistics

! HSRP
show standby brief
show standby

! ACLs
show ip access-lists
show access-list <ACL_NUMBER>

! AAA
show aaa method-lists all
show aaa sessions
debug aaa authentication

! Wireless
show wlan summary
show ap summary
show wireless client summary

! QoS
show policy-map interface <INTERFACE>

! Multicast
show ip mroute
show ip pim neighbor
show ip igmp groups

! VRF
show vrf
show ip route vrf <VRF_NAME>

! SD-WAN
show control connections
show omp peers

! SD-Access
show lisp site
show cts role-based permissions

! Automation
show platform software yang-management process

! CPU/Memory
show processes cpu sorted
show processes memory sorted
show memory statistics

! Logs
show logging
show logging | include <KEYWORD>
```

## 28. ENCOR Show Command Master List

```ios
! Show Commands
show ip interface brief
show ipv6 interface brief
show interface description
show interface status
show running-config
show startup-config
show version
show ip route
show ipv6 route
show vlan brief
show interfaces trunk
show spanning-tree
show spanning-tree summary
show spanning-tree mst
show etherchannel summary
show ip ospf neighbor
show ip ospf database
show ip eigrp neighbors
show ip eigrp topology
show ip bgp summary
show ip bgp neighbors
show ip nat translations
show ip nat statistics
show standby brief
show standby
show vrrp brief
show glbp brief
show ip dhcp binding
show ip dhcp server statistics
show wlan summary
show ap summary
show wireless client summary
show ip access-lists
show aaa sessions
show vrf
show ip mroute
show ip pim neighbor
show policy-map interface <INTERFACE>
show flow monitor
show redundancy states
show processes cpu sorted
show logging

! Debug Commands
debug ip packet
debug ip routing
debug ip ospf adj
debug ip ospf events
debug ip eigrp
debug ip bgp keepalives
debug ip bgp updates
debug ip dhcp server events
debug standby events
debug dot1x all
debug aaa authentication
debug aaa authorization
debug wireless client mac-address <MAC_ADDRESS>
debug lisp

! Clear Commands
clear counters
clear ip route *
clear ip route vrf <VRF_NAME> *
clear ip bgp * soft
clear ip nat translation *
clear ip dhcp binding *
clear mac address-table dynamic

! Ping and Traceroute
ping <IP_ADDRESS>
ping <IP_ADDRESS> source <INTERFACE>
ping ipv6 <IPv6_ADDRESS>
traceroute <IP_ADDRESS>
traceroute ipv6 <IPv6_ADDRESS>

! Test Commands
test aaa group tacacs+ <USERNAME> <PASSWORD> legacy
test aaa group radius <USERNAME> <PASSWORD> legacy
```

## 29. Complete Configuration Examples

### 1. Enterprise Layer 2 Switch

```ios
hostname SW-L2-ENTERPRISE
!
vlan 10
 name Data
vlan 20
 name Voice
vlan 99
 name Native-Mgmt
!
spanning-tree mode rapid-pvst
spanning-tree portfast edge bpduguard default
!
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
!
interface range GigabitEthernet0/1 - 2
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport mode trunk
 channel-group 1 mode active
!
interface range GigabitEthernet1/0/1 - 24
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 spanning-tree portfast
 spanning-tree bpduguard enable
!
interface vlan 99
 ip address 10.99.99.10 255.255.255.0
 no shutdown
!
ip default-gateway 10.99.99.1
```

### 2. Layer 3 Switch

```ios
hostname SW-L3-CORE
!
ip routing
ipv6 unicast-routing
!
vlan 10
 name Data
vlan 20
 name Voice
vlan 99
 name Native-Mgmt
!
interface vlan 10
 ip address 10.10.10.1 255.255.255.0
 ipv6 address 2001:db8:10::1/64
 no shutdown
!
interface vlan 20
 ip address 10.20.20.1 255.255.255.0
 ipv6 address 2001:db8:20::1/64
 no shutdown
!
interface vlan 99
 ip address 10.99.99.1 255.255.255.0
 ipv6 address 2001:db8:99::1/64
 no shutdown
!
interface GigabitEthernet0/1
 no switchport
 ip address 192.168.12.1 255.255.255.252
 no shutdown
```

### 3. OSPF Enterprise Network

```ios
hostname R-OSPF-ENT
!
router ospf 10
 router-id 1.1.1.1
 auto-cost reference-bandwidth 1000
 passive-interface default
 no passive-interface GigabitEthernet0/0
 area 0 authentication message-digest
!
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 10 area 0
!
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Cisco123
 ip ospf 10 area 0
 no passive-interface GigabitEthernet0/0
 no shutdown
```

### 4. EIGRP Enterprise Network

```ios
hostname R-EIGRP-ENT
!
router eigrp COMPANY-EIGRP
 address-family ipv4 unicast autonomous-system 100
  router-id 2.2.2.2
  network 10.0.0.0 0.255.255.255
  af-interface default
   passive-interface
  exit-af-interface
  af-interface GigabitEthernet0/0
   no passive-interface
   authentication mode md5
   authentication key-chain EIGRP-KEYS
  exit-af-interface
  topology base
   variance 2
  exit-topology
 exit-address-family
!
key chain EIGRP-KEYS
 key 1
  key-string CiscoEigrpPass
```

### 5. BGP Enterprise Network

```ios
hostname R-BGP-ENT
!
router bgp 65001
 bgp router-id 3.3.3.3
 neighbor 192.0.2.1 remote-as 65002
 neighbor 192.0.2.1 password BGPPassword123
 neighbor 10.255.255.2 remote-as 65001
 neighbor 10.255.255.2 update-source loopback0
 !
 address-family ipv4 unicast
  neighbor 192.0.2.1 activate
  neighbor 10.255.255.2 activate
  network 192.168.10.0 mask 255.255.255.0
  neighbor 192.0.2.1 route-map PREPEND-OUT out
 exit-address-family
!
ip prefix-list OUT-PREFIX seq 10 permit 192.168.10.0/24
!
route-map PREPEND-OUT permit 10
 match ip address prefix-list OUT-PREFIX
 set as-path prepend 65001 65001
```

### 6. HSRP Redundant Gateways

```ios
! Router 1 (Active)
interface GigabitEthernet0/1
 ip address 10.1.1.2 255.255.255.0
 standby version 2
 standby 10 ip 10.1.1.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track GigabitEthernet0/0 20

! Router 2 (Standby)
interface GigabitEthernet0/1
 ip address 10.1.1.3 255.255.255.0
 standby version 2
 standby 10 ip 10.1.1.1
 standby 10 priority 100
 standby 10 preempt
```

### 7. EtherChannel

```ios
! LACP Switch Side
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 channel-group 5 mode active
!
interface port-channel 5
 switchport mode trunk
```

### 8. DHCP Server

```ios
ip dhcp excluded-address 192.168.1.1 192.168.1.20
!
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8 8.8.4.4
 domain-name enterprise.local
 lease 7 0 0
```

### 9. NAT/PAT

```ios
interface GigabitEthernet0/0
 description INSIDE
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/1
 description OUTSIDE
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
!
ip access-list standard NAT-SUBNETS
 10 permit 192.168.10.0 0.0.0.255
!
ip nat inside source list NAT-SUBNETS interface GigabitEthernet0/1 overload
```

### 10. AAA with TACACS+

```ios
aaa new-model
!
tacacs server TACACS-SRV
 address ipv4 10.100.100.5
 key ServerPass123
!
aaa group server tacacs+ TACACS-GRP
 server TACACS-SRV
!
aaa authentication login default group TACACS-GRP local
aaa authorization exec default group TACACS-GRP local
aaa accounting exec default start-stop group TACACS-GRP
```

### 11. Secure Cisco Router

```ios
hostname SECURE-RTR
!
enable secret SuperSecurePassword123
!
service password-encryption
!
ip domain-name secure.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
!
username admin privilege 15 secret AdminPass123
!
line con 0
 exec-timeout 5 0
 logging synchronous
 login local
!
line vty 0 4
 exec-timeout 5 0
 logging synchronous
 login local
 transport input ssh
!
ip http secure-server
ip http authentication local
```

### 12. QoS Policy

```ios
ip access-list extended VOICE-TRAFFIC
 permit udp any any range 16384 32767
!
class-map match-all VOICE-CLASS
 match access-group name VOICE-TRAFFIC
!
policy-map WAN-EDGE-QOS
 class VOICE-CLASS
  priority level 1
  police cir 128000 conform-action transmit exceed-action drop
 class class-default
  shape average 10000000
!
interface GigabitEthernet0/0
 service-policy output WAN-EDGE-QOS
```

### 13. VRF

```ios
vrf definition CUSTOMER-A
 rd 100:1
 route-target export 100:1
 route-target import 100:1
 !
 address-family ipv4
 exit-address-family
!
interface GigabitEthernet0/1
 vrf forwarding CUSTOMER-A
 ip address 10.200.1.1 255.255.255.0
 no shutdown
!
ip route vrf CUSTOMER-A 0.0.0.0 0.0.0.0 10.200.1.254
```

### 14. Multicast

```ios
ip multicast-routing
!
ip pim rp-address 10.10.10.10
!
interface GigabitEthernet0/0
 ip address 10.1.1.1 255.255.255.0
 ip pim sparse-mode
!
interface GigabitEthernet0/1
 ip address 10.2.2.1 255.255.255.0
 ip pim sparse-mode
```

### 15. Network Automation using RESTCONF

```ios
! Cisco Device Setup
ip http secure-server
restconf
!
username restapi privilege 15 secret RESTCONFpass!
!
! External execution payload to update configuration
! URI: https://192.168.1.1/restconf/data/ietf-interfaces:interfaces/interface=Loopback99
! Method: PUT
! Body:
! {
!   "ietf-interfaces:interface": {
!     "name": "Loopback99",
!     "description": "Created via RESTCONF API",
!     "type": "iana-if-type:softwareLoopback",
!     "enabled": true,
!     "ietf-ip:ipv4": {
!       "address": [
!         {
!           "ip": "99.99.99.99",
!           "netmask": "255.255.255.255"
!         }
!       ]
!     }
!   }
! }
```
