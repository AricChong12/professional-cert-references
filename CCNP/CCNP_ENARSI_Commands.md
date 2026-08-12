# Layer 3 Technologies

```ios
! IPv4 Addressing
interface <INTERFACE>
 ip address <IP_ADDRESS> <MASK>
 no shutdown

! IPv6 Addressing
interface <INTERFACE>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>
 ipv6 enable
 no shutdown

! IPv4 Static Routes
ip route <NETWORK> <MASK> <NEXT_HOP_IP>
ip route <NETWORK> <MASK> <EXIT_INTERFACE>

! IPv6 Static Routes
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <NEXT_HOP_IPv6>
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <EXIT_INTERFACE>

! Default Routes
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP_IP>
ipv6 route ::/0 <NEXT_HOP_IPv6>

! Floating Static Routes
ip route <NETWORK> <MASK> <NEXT_HOP_IP> <AD_VALUE>
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <NEXT_HOP_IPv6> <AD_VALUE>

! Recursive Static Routes
ip route <NETWORK_A> <MASK_A> <IP_ADDRESS_B>
ip route <IP_ADDRESS_B> 255.255.255.255 <NEXT_HOP_IP_C>

! Policy-Based Routing (PBR)
ip access-list extended <ACL_NAME>
 permit tcp <SOURCE_IP> <WILDCARD> any eq <PORT>
exit
route-map <PBR_MAP_NAME> permit 10
 match ip address <ACL_NAME>
 set ip next-hop <IP_ADDRESS>
exit
interface <INTERFACE>
 ip policy route-map <PBR_MAP_NAME>

! VRF (Virtual Routing and Forwarding)
vrf definition <VRF_NAME>
 rd <ROUTE_DISTINGUISHER>
 address-family ipv4
 exit-address-family
 address-family ipv6
 exit-address-family
exit

! VRF-Lite Interface Assignment
interface <INTERFACE>
 vrf forwarding <VRF_NAME>
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPv6_ADDRESS>/<PREFIX_LEN>

! DHCP Server
ip dhcp excluded-address <START_IP> <END_IP>
ip dhcp pool <POOL_NAME>
 network <NETWORK> <MASK>
 default-router <IP_ADDRESS>
 dns-server <IP_ADDRESS>
 lease <DAYS> <HOURS> <MINUTES>

! DHCP Relay
interface <INTERFACE>
 ip helper-address <DHCP_SERVER_IP>

! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan <VLAN_ID>
interface <TRUSTED_INTERFACE>
 ip dhcp snooping trust
interface <UNTRUSTED_INTERFACE>
 ip dhcp snooping limit rate <RATE_VALUE>

! IP SLA
ip sla <SLA_ID>
 icmp-echo <TARGET_IP> source-ip <SOURCE_IP>
 frequency <SECONDS>
exit
ip sla schedule <SLA_ID> start-time now life forever

! Object Tracking
track <TRACK_ID> ip sla <SLA_ID> state

! HSRP
interface <INTERFACE>
 standby version 2
 standby <GROUP_ID> ip <VIRTUAL_IP>
 standby <GROUP_ID> priority <PRIORITY_VALUE>
 standby <GROUP_ID> preempt
 standby <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>

! VRRP
interface <INTERFACE>
 vrrp <GROUP_ID> ip <VIRTUAL_IP>
 vrrp <GROUP_ID> priority <PRIORITY_VALUE>
 vrrp <GROUP_ID> preempt
 vrrp <GROUP_ID> track <TRACK_ID> decrement <DECREMENT_VALUE>

! GLBP
interface <INTERFACE>
 glbp <GROUP_ID> ip <VIRTUAL_IP>
 glbp <GROUP_ID> priority <PRIORITY_VALUE>
 glbp <GROUP_ID> preempt
 glbp <GROUP_ID> load-balancing <round-robin|weighted|host-dependent>
```

# EIGRP

```ios
! Classic EIGRP IPv4
router eigrp <AS_NUMBER>
 eigrp router-id <ROUTER_ID>
 network <NETWORK> <WILDCARD>
 passive-interface <INTERFACE>
 no passive-interface <INTERFACE>
 variance <MULTIPLIER>
 traffic-share balanced

! Classic EIGRP IPv6
ipv6 router eigrp <AS_NUMBER>
 eigrp router-id <ROUTER_ID>
 passive-interface <INTERFACE>
 no shutdown
interface <INTERFACE>
 ipv6 eigrp <AS_NUMBER>

! Named EIGRP Configuration (IPv4 and IPv6)
router eigrp <VIRTUAL_INSTANCE_NAME>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  router-id <ROUTER_ID>
  network <NETWORK> <WILDCARD>
  af-interface default
   passive-interface
  exit-af-interface
  af-interface <INTERFACE>
   no passive-interface
   summary-address <IP_ADDRESS> <MASK>
   authentication mode md5
   authentication key-chain <KEY_CHAIN_NAME>
  exit-af-interface
  topology base
   variance <MULTIPLIER>
   maximum-paths <PATH_COUNT>
  exit-topology
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system <AS_NUMBER>
  router-id <ROUTER_ID>
  af-interface default
   passive-interface
  exit-af-interface
  af-interface <INTERFACE>
   no passive-interface
  exit-af-interface
  topology base
  exit-topology
 exit-address-family

! EIGRP Authentication (Classic Mode)
key chain <KEY_CHAIN_NAME>
 key <KEY_ID>
  key-string <KEY_VALUE>
interface <INTERFACE>
 ip authentication mode eigrp <AS_NUMBER> md5
 ip authentication key-chain eigrp <AS_NUMBER> <KEY_CHAIN_NAME>

! EIGRP Default Route Propagation
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP_IP>
router eigrp <AS_NUMBER>
 network 0.0.0.0

! EIGRP Stub Routing
router eigrp <AS_NUMBER>
 eigrp stub <receive-only|connected|static|summary|redistributed>

! EIGRP Route Filtering (Distribute-List)
ip prefix-list <PREFIX_LIST_NAME> permit <NETWORK>/<PREFIX_LEN>
router eigrp <AS_NUMBER>
 distribute-list prefix <PREFIX_LIST_NAME> <in|out> <INTERFACE>

! Offset Lists
access-list <ACL_NUMBER> permit <NETWORK> <WILDCARD>
router eigrp <AS_NUMBER>
 offset-list <ACL_NUMBER> <in|out> <OFFSET_VALUE> <INTERFACE>

! EIGRP Troubleshooting and Verification
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces
show ip eigrp traffic
show ipv6 eigrp neighbors
show ipv6 eigrp topology
```

# OSPF

```ios
! OSPFv2 (Single-Area and Multi-Area)
router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
 network <NETWORK> <WILDCARD> area <AREA_ID>
 passive-interface <INTERFACE>

! Interface-Based OSPFv2
interface <INTERFACE>
 ip ospf <PROCESS_ID> area <AREA_ID>

! OSPFv3 (IPv4 and IPv6)
router ospfv3 <PROCESS_ID>
 router-id <ROUTER_ID>
 address-family ipv4 unicast
  passive-interface <INTERFACE>
 exit-address-family
 address-family ipv6 unicast
  passive-interface <INTERFACE>
 exit-address-family
interface <INTERFACE>
 ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>
 ospfv3 <PROCESS_ID> ipv6 area <AREA_ID>

! OSPF Authentication (Interface Level)
interface <INTERFACE>
 ip ospf authentication message-digest
 ip ospf message-digest-key <KEY_ID> md5 <KEY_VALUE>

! OSPF Authentication (Area Level)
router ospf <PROCESS_ID>
 area <AREA_ID> authentication message-digest
interface <INTERFACE>
 ip ospf message-digest-key <KEY_ID> md5 <KEY_VALUE>

! OSPF Network Types
interface <INTERFACE>
 ip ospf network <broadcast|point-to-point|non-broadcast|point-to-multipoint>
 ip ospf priority <PRIORITY_VALUE>

! OSPF Cost and Reference Bandwidth
interface <INTERFACE>
 ip ospf cost <COST_VALUE>
router ospf <PROCESS_ID>
 auto-cost reference-bandwidth <BANDWIDTH_IN_MBPS>

! OSPF Summarization
! Inter-area (on ABR)
router ospf <PROCESS_ID>
 area <AREA_ID> range <NETWORK> <MASK>
! External (on ASBR)
router ospf <PROCESS_ID>
 summary-address <NETWORK> <MASK>

! Default Route Propagation
router ospf <PROCESS_ID>
 default-information originate
! Or force propagation:
 default-information originate always

! OSPF Area Types (Stub, Totally Stubby, NSSA, Totally NSSA)
router ospf <PROCESS_ID>
! Stub
 area <AREA_ID> stub
! Totally Stubby (ABR only)
 area <AREA_ID> stub no-summary
! NSSA
 area <AREA_ID> nssa
! Totally NSSA (ABR only)
 area <AREA_ID> nssa no-summary

! LSA-Related Configuration
router ospf <PROCESS_ID>
 timers throttle lsa all <START_MSEC> <HOLD_MSEC> <MAX_MSEC>
 max-lsa <NUMBER>

! OSPF Route Filtering
ip prefix-list <PREFIX_LIST_NAME> deny <NETWORK>/<PREFIX_LEN>
ip prefix-list <PREFIX_LIST_NAME> permit 0.0.0.0/0 le 32
router ospf <PROCESS_ID>
 distribute-list prefix <PREFIX_LIST_NAME> in
! Filter Type 3 LSA on ABR
 area <AREA_ID> filter-list prefix <PREFIX_LIST_NAME> <in|out>

! OSPF Route Selection (Tuning AD)
router ospf <PROCESS_ID>
 distance <AD_VALUE>

! OSPF Troubleshooting and Verification
show ip ospf neighbor
show ip ospf database
show ip ospf interface <INTERFACE>
show ip ospf border-routers
```

# BGP

```ios
! BGP Basic Peering
router bgp <AS_NUMBER>
 bgp router-id <ROUTER_ID>
 neighbor <IP_ADDRESS> remote-as <REMOTE_AS>

! BGP Source Interface Loopback Peering
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> update-source loopback <NUMBER>

! eBGP Multihop
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> ebgp-multihop <HOP_COUNT>

! BGP Authentication
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> password <PASSWORD>

! BGP Timers
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> timers <keepalive> <holdtime>

! BGP Next-Hop-Self
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> next-hop-self

! BGP Route Advertisement
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  network <NETWORK> mask <MASK>

! BGP Default Route
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> default-originate

! BGP Route Filtering (Prefix List / Route Map)
ip prefix-list <PREFIX_LIST_NAME> permit <NETWORK>/<PREFIX_LEN>
!
route-map <MAP_NAME> permit 10
 match ip address prefix-list <PREFIX_LIST_NAME>
exit
!
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> prefix-list <PREFIX_LIST_NAME> <in|out>
  neighbor <IP_ADDRESS> route-map <MAP_NAME> <in|out>

! AS-Path Access List
ip as-path access-list <LIST_NUMBER> permit <REGEX>
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> filter-list <LIST_NUMBER> <in|out>

! BGP Community Lists & Communities
ip community-list standard <COMMUNITY_LIST_NAME> permit <COMMUNITY_VALUE>
!
route-map <COMM_MAP> permit 10
 set community <no-advertise|no-export|local-as|COMMUNITY_VAL>
!
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> send-community <standard|extended|both>
  neighbor <IP_ADDRESS> route-map <COMM_MAP> out

! BGP Attribute Manipulation (Local Preference, Weight, MED, AS-Path Prepend)
route-map <ATTR_MAP> permit 10
 set local-preference <VALUE>
 set weight <VALUE>
 set metric <MED_VALUE>
 set as-path prepend <AS_NUMBER> <AS_NUMBER>
exit

! BGP Conditional Advertisement
ip prefix-list <EXIST_LIST> permit <NETWORK>/<PREFIX_LEN>
ip prefix-list <ADVERT_LIST> permit <NETWORK>/<PREFIX_LEN>
!
route-map <EXIST_MAP> permit 10
 match ip address prefix-list <EXIST_LIST>
exit
route-map <ADVERT_MAP> permit 10
 match ip address prefix-list <ADVERT_LIST>
exit
!
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> advertise-map <ADVERT_MAP> exist-map <EXIST_MAP>

! BGP Aggregation / Summarization
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  aggregate-address <SUMMARY_IP> <MASK> summary-only

! BGP Load Balancing (Maximum Paths)
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  maximum-paths <PATH_COUNT>

! BGP Peer Groups
router bgp <AS_NUMBER>
 neighbor <GROUP_NAME> peer-group
 neighbor <GROUP_NAME> remote-as <AS_NUMBER>
 neighbor <GROUP_NAME> update-source loopback0
 neighbor <GROUP_NAME> next-hop-self
 neighbor <PEER_IP> peer-group <GROUP_NAME>

! BGP Route Reflector
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  neighbor <PEER_IP> route-reflector-client

! BGP Confederations
router bgp <SUB_AS_NUMBER>
 bgp confederation identifier <PRIMARY_AS_NUMBER>
 bgp confederation peers <SUB_AS_LIST>
 neighbor <PEER_IP> remote-as <SUB_AS_NUMBER>

! Multiprotocol BGP (MP-BGP IPv6)
router bgp <AS_NUMBER>
 neighbor <IPv6_PEER_IP> remote-as <REMOTE_AS>
 address-family ipv6 unicast
  neighbor <IPv6_PEER_IP> activate
  network <IPv6_PREFIX>/<PREFIX_LEN>
```

# Redistribution

```ios
! Static to EIGRP
router eigrp <AS_NUMBER>
 redistribute static metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>

! Static to OSPF
router ospf <PROCESS_ID>
 redistribute static subnets

! Static to BGP
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  redistribute static

! EIGRP to OSPF
router ospf <PROCESS_ID>
 redistribute eigrp <AS_NUMBER> subnets

! OSPF to EIGRP
router eigrp <AS_NUMBER>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  topology base
   redistribute ospf <PROCESS_ID> metric 100000 10 255 1 1500

! EIGRP to BGP
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  redistribute eigrp <AS_NUMBER>

! OSPF to BGP
router bgp <AS_NUMBER>
 address-family ipv4 unicast
  redistribute ospf <PROCESS_ID>

! BGP to OSPF
router ospf <PROCESS_ID>
 redistribute bgp <AS_NUMBER> subnets

! BGP to EIGRP
router eigrp <AS_NUMBER>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  topology base
   redistribute bgp <AS_NUMBER> metric 100000 10 255 1 1500

! Redistribution with Route Maps, Prefix Lists, and Route Tagging
ip prefix-list <REDIST_PREFIX> permit <NETWORK>/<PREFIX_LEN>
!
route-map <TAG_MAP> permit 10
 match ip address prefix-list <REDIST_PREFIX>
 set tag <TAG_VALUE>
exit
route-map <TAG_MAP> deny 20
!
router ospf <PROCESS_ID>
 redistribute eigrp <AS_NUMBER> subnets route-map <TAG_MAP>

! Preventing Loops via Route Tag Filtering
route-map <FILTER_MAP> deny 10
 match tag <TAG_VALUE>
exit
route-map <FILTER_MAP> permit 20
exit
!
router eigrp <AS_NUMBER>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  topology base
   redistribute ospf <PROCESS_ID> route-map <FILTER_MAP>

! Administrative Distance Manipulation (Tuning AD to prevent loops)
router ospf <PROCESS_ID>
 distance ospf external <AD_VALUE>
!
router eigrp <AS_NUMBER>
 distance eigrp <INTERNAL_AD> <EXTERNAL_AD>
```

# MPLS

```ios
! Basic MPLS LDP Configuration
ip cef
interface <INTERFACE>
 mpls ip
 mpls label protocol ldp

! LDP Router ID
mpls ldp router-id loopback <NUMBER> force

! PE Router Configuration (L3VPN)
vrf definition <VRF_NAME>
 rd <ROUTE_DISTINGUISHER>
 route-target export <RT_VALUE>
 route-target import <RT_VALUE>
 address-family ipv4
 exit-address-family
!
interface <CE_FACING_INTERFACE>
 vrf forwarding <VRF_NAME>
 ip address <IP_ADDRESS> <MASK>
!
router bgp <AS_NUMBER>
 neighbor <PE_PEER_IP> remote-as <AS_NUMBER>
 neighbor <PE_PEER_IP> update-source loopback0
 !
 address-family vpnv4
  neighbor <PE_PEER_IP> activate
  neighbor <PE_PEER_IP> send-community extended
 exit-address-family
 !
 address-family ipv4 vrf <VRF_NAME>
  redistribute ospf <PROCESS_ID>
 exit-address-family

! CE Router Configuration (Standard Routing to PE)
interface <PE_FACING_INTERFACE>
 ip address <IP_ADDRESS> <MASK>
!
router ospf <PROCESS_ID>
 network <NETWORK> <WILDCARD> area 0

! MPLS Verification
show mpls interfaces
show mpls ldp neighbor
show mpls ldp discovery
show mpls ldp bindings
show mpls forwarding-table
show ip bgp vpnv4 all
show ip bgp vpnv4 all summary
```

# IPv6

```ios
! IPv6 Unicast Routing
ipv6 unicast-routing

! IPv6 Static and Default Routes
ipv6 route ::/0 <NEXT_HOP_IPv6>
ipv6 route <IPv6_PREFIX>/<PREFIX_LEN> <NEXT_HOP_IPv6>

! OSPFv3 Configuration
router ospfv3 <PROCESS_ID>
 router-id <ROUTER_ID>
 address-family ipv6 unicast
 exit-address-family
interface <INTERFACE>
 ospfv3 <PROCESS_ID> ipv6 area <AREA_ID>

! EIGRP for IPv6
router eigrp <VIRTUAL_INSTANCE_NAME>
 address-family ipv6 unicast autonomous-system <AS_NUMBER>
  router-id <ROUTER_ID>
  af-interface <INTERFACE>
   no passive-interface
  exit-af-interface
interface <INTERFACE>
 ipv6 enable

! IPv6 BGP (MP-BGP)
router bgp <AS_NUMBER>
 neighbor <IPv6_PEER_IP> remote-as <REMOTE_AS>
 address-family ipv6 unicast
  neighbor <IPv6_PEER_IP> activate
  network <IPv6_PREFIX>/<PREFIX_LEN>

! IPv6 Route Filtering
ipv6 prefix-list <IPv6_FILTER> permit <IPv6_PREFIX>/<PREFIX_LEN>
!
route-map <IPv6_MAP> permit 10
 match ipv6 address prefix-list <IPv6_FILTER>
exit
!
router bgp <AS_NUMBER>
 address-family ipv6 unicast
  neighbor <IPv6_PEER_IP> route-map <IPv6_MAP> in

! IPv6 Redistribution
router ospfv3 <PROCESS_ID>
 address-family ipv6 unicast
  redistribute eigrp <AS_NUMBER>
```

# Infrastructure Security

```ios
! Standard Named IPv4 ACL
ip access-list standard <ACL_NAME>
 permit <SOURCE_IP> <WILDCARD>
 deny any

! Extended Named IPv4 ACL
ip access-list extended <ACL_NAME>
 permit tcp <SOURCE_IP> <WILDCARD> <DEST_IP> <WILDCARD> eq ssh
 deny ip any any log

! IPv6 ACL
ipv6 access-list <IPv6_ACL_NAME>
 permit tcp any any eq 22
 deny ipv6 any any

! Applying ACL to Interfaces
interface <INTERFACE>
 ip access-group <ACL_NAME> <in|out>
 ipv6 traffic-filter <IPv6_ACL_NAME> <in|out>

! Prefix Lists
ip prefix-list <LIST_NAME> seq 10 permit <NETWORK>/<PREFIX_LEN> ge <MIN_LEN> le <MAX_LEN>

! Route Maps
route-map <MAP_NAME> permit 10
 match ip address prefix-list <LIST_NAME>
 set local-preference <VALUE>

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
  police 8000 conform-action transmit exceed-action drop
!
control-plane
 service-policy input <COPP_POLICY>

! uRPF (Unicast Reverse Path Forwarding)
interface <INTERFACE>
 ip verify unicast source reachable-via <rx|any>

! AAA Authentication (Local, TACACS+, RADIUS)
aaa new-model
tacacs server <TACACS_NAME>
 address ipv4 <IP_ADDRESS>
 key <KEY_STRING>
radius server <RADIUS_NAME>
 address ipv4 <IP_ADDRESS> auth-port 1812 acct-port 1813
 key <KEY_STRING>
!
aaa authentication login default group tacacs+ group radius local
aaa authorization exec default group tacacs+ local

! SSH
ip domain-name <DOMAIN_NAME>
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
 transport input ssh
 login authentication default

! SNMP Configuration
snmp-server community <STRING> RO
snmp-server host <IP_ADDRESS> version 2c <STRING>

! Syslog
logging host <IP_ADDRESS>
logging trap <SEVERITY>

! NTP Authentication
ntp authenticate
ntp authentication-key <KEY_ID> md5 <KEY_VALUE>
ntp trusted-key <KEY_ID>
ntp server <IP_ADDRESS> key <KEY_ID>
```

# Infrastructure Services

```ios
! DHCP Server
ip dhcp pool <POOL_NAME>
 network <NETWORK> <MASK>
 default-router <IP_ADDRESS>
 dns-server <IP_ADDRESS>

! DHCP Relay
interface <INTERFACE>
 ip helper-address <DHCP_SERVER_IP>

! DHCP Client
interface <INTERFACE>
 ip address dhcp

! DNS Configuration
ip domain-lookup
ip name-server <DNS_SERVER_IP>

! NTP Client & Server
ntp master <STRATUM_LEVEL>
ntp server <NTP_SERVER_IP>

! SPAN (Local)
monitor session 1 source interface <INTERFACE> both
monitor session 1 destination interface <INTERFACE>

! Flexible NetFlow (FNF)
flow record <RECORD_NAME>
 match ipv4 source address
 match ipv4 destination address
 collect counter bytes long
!
flow exporter <EXPORTER_NAME>
 destination <COLLECTOR_IP>
 transport udp 2055
!
flow monitor <MONITOR_NAME>
 record <RECORD_NAME>
 exporter <EXPORTER_NAME>
!
interface <INTERFACE>
 ip flow monitor <MONITOR_NAME> input
```

# Troubleshooting

```ios
! Interface Troubleshooting
show interfaces <INTERFACE>
show interfaces <INTERFACE> status
show controllers <INTERFACE>
clear counters <INTERFACE>

! IPv4/IPv6 Troubleshooting
show ip interface brief
show ipv6 interface brief
show ip route
show ipv6 route

! ARP and MAC Table
show arp
show mac address-table

! CEF (Cisco Express Forwarding)
show ip cef
show ip cef <IP_ADDRESS>

! EIGRP Neighbors and Topology
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces

! OSPF Neighbors, Database and Routes
show ip ospf neighbor
show ip ospf database
show ip ospf border-routers
show ip route ospf

! BGP Neighbors, Routes and Attributes
show ip bgp summary
show ip bgp neighbors <IP_ADDRESS>
show ip bgp neighbors <IP_ADDRESS> advertised-routes
show ip bgp

! Redistribution Verification
show ip protocols
show ip route

! ACL and VRF Verification
show access-lists
show vrf
show ip route vrf <VRF_NAME>

! MPLS LDP & L3VPN
show mpls interfaces
show mpls ldp neighbor
show mpls forwarding-table
show ip bgp vpnv4 all

! DHCP, NAT, IP SLA
show ip dhcp binding
show ip nat translations
show ip sla summary

! HSRP, VRRP, GLBP
show standby brief
show vrrp brief
show glbp brief

! Path/Packet Testing
ping <IP_ADDRESS> source <INTERFACE>
traceroute <IP_ADDRESS>
```

# Verification Commands

```ios
show running-config
show startup-config
show interfaces
show ip route
show ipv6 route
show ip protocols
show ip cef
show arp
show mac address-table
show access-lists
show ip eigrp neighbors
show ip ospf neighbor
show ip bgp summary
show bgp ipv4 unicast summary
show mpls ldp neighbor
show vrf
show ip sla summary
show track
show standby brief
show vrrp brief
show glbp brief
show ip dhcp binding
show logging
show ntp status
show ntp associations
show snmp community
show aaa sessions
show users
```

# Debug Commands

```ios
debug ip packet
debug ip routing
debug ip eigrp
debug ip eigrp notification
debug ip ospf adj
debug ip ospf events
debug ip bgp keepalives
debug ip bgp updates
debug ipv6 routing
debug ipv6 ospf
debug ip dhcp server events
debug standby events
debug vrrp events
debug glbp errors
debug ip sla error
debug mpls ldp packets
debug aaa authentication
debug aaa authorization
```

# ENARSI Quick Command Reference

```ios
! Core Routing & Services
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP_IP>
ipv6 route ::/0 <NEXT_HOP_IPv6>
ip routing
ipv6 unicast-routing
vrf definition <VRF_NAME>
 rd <RD>
 address-family ipv4
exit-address-family

! EIGRP
router eigrp <VIRTUAL_INSTANCE_NAME>
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  network <NETWORK> <WILDCARD>
  variance <MULTIPLIER>

! OSPF
router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
 network <NETWORK> <WILDCARD> area <AREA_ID>
 area <AREA_ID> range <NETWORK> <MASK>
 area <AREA_ID> stub no-summary

! BGP
router bgp <AS_NUMBER>
 neighbor <IP_ADDRESS> remote-as <REMOTE_AS>
 neighbor <IP_ADDRESS> update-source loopback0
 neighbor <IP_ADDRESS> next-hop-self
 address-family ipv4 unicast
  neighbor <IP_ADDRESS> activate
  network <NETWORK> mask <MASK>

! Redistribution
router ospf <PROCESS_ID>
 redistribute eigrp <AS_NUMBER> subnets route-map <MAP_NAME>

! MPLS
interface <INTERFACE>
 mpls ip
 mpls label protocol ldp

! IP SLA & Tracking
ip sla 1
 icmp-echo <TARGET_IP>
 frequency 5
ip sla schedule 1 start-time now life forever
track 1 ip sla 1 state
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP_IP> track 1

! Essential Verification
show ip interface brief
show ip route
show ip eigrp neighbors
show ip ospf neighbor
show ip bgp summary
show mpls ldp neighbor
show standby brief
```
