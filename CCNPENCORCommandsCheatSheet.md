CCNPENCORCommandsCheatSheet.md

Basic Device Configuration

enable
configure terminal
hostname <HOSTNAME>
no ip domain lookup
ip domain name <DOMAIN_NAME>
enable secret <SECRET_PLACEHOLDER>
service password-encryption
security passwords min-length 8


! Console line
line console 0
 exec-timeout 5 0
 logging synchronous
 login local
 exit



! VTY lines
line vty 0 15
 exec-timeout 10 0
 logging synchronous
 transport input ssh
 transport output ssh
 login local
 exit



! Banners
banner motd ^C Authorized access only. ^C
banner login ^C Login required. ^C
banner exec ^C Session established. ^C



! Local users and privilege levels
username <USERNAME> privilege 15 algorithm-type scrypt secret <SECRET_PLACEHOLDER>
username <READONLYUSER> privilege 5 algorithm-type scrypt secret <SECRETPLACEHOLDER>
privilege exec level 5 show running-config
enable algorithm-type scrypt secret level 5 <SECRET_PLACEHOLDER>



! SSH
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
ip ssh dh min size 2048
ip ssh source-interface Loopback0



! Clock and NTP
clock timezone <TZNAME> <UTCOFFSET>
clock summer-time <TZ_NAME> recurring
ntp server <IP_ADDRESS> prefer
ntp server <IPADDRESS2>
ntp source Loopback0
ntp authenticate
ntp authentication-key 1 md5 <NTPKEYPLACEHOLDER>
ntp trusted-key 1
ntp update-calendar
ntp master 3



! Logging and timestamps
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
logging buffered 65536 informational
logging console warnings
logging monitor informational
logging host <IP_ADDRESS>
logging trap informational
logging source-interface Loopback0
logging origin-id hostname
archive
 log config
  logging enable
  notify syslog contenttype plaintext
  hidekeys



! Verification
show version
show running-config
show startup-config
show clock detail
show ntp status
show ntp associations
show users
show ssh
show ip ssh
show logging
show privilege
show license summary
show inventory
show environment all



! Save / reload
copy running-config startup-config
write memory
reload in 10
reload cancel


Interface Configuration


interface <INTERFACE>
 description <TEXT_DESCRIPTION>
 no shutdown
 exit



! Layer 2 access interface
interface GigabitEthernet1/0/10
 description ACCESS-PORT-USER
 switchport
 switchport mode access
 switchport access vlan <VLAN_ID>
 spanning-tree portfast
 no shutdown



! Routed port on Layer 3 switch
interface GigabitEthernet1/0/24
 description L3-UPLINK
 no switchport
 ip address <IP_ADDRESS> <MASK>
 no shutdown



! Loopback
interface Loopback0
 description ROUTER-ID-MGMT
 ip address <ROUTER_ID> 255.255.255.255
 ipv6 address <IPV6_ADDRESS>/128



! Speed / duplex / MTU
interface <INTERFACE>
 speed 1000
 duplex full
 no negotiation auto
 mtu 9216
 ip mtu 1500
 load-interval 30



! IPv4 addressing
interface <INTERFACE>
 ip address <IP_ADDRESS> <MASK>
 ip address <SECONDARY_IP> <MASK> secondary
 ip helper-address <DHCPSERVERIP>
 no ip redirects
 no ip proxy-arp



! IPv6 addressing
ipv6 unicast-routing
interface <INTERFACE>
 ipv6 enable
 ipv6 address <IPV6_PREFIX>::1/64
 ipv6 address FE80::1 link-local
 ipv6 address autoconfig
 ipv6 address dhcp
 no ipv6 redirects



! Verification
show ip interface brief
show ipv6 interface brief
show interfaces <INTERFACE>
show interfaces status
show interfaces description
show interfaces trunk
show interfaces counters errors
show interfaces <INTERFACE> switchport
show ip interface <INTERFACE>
show ipv6 interface <INTERFACE>
show controllers <INTERFACE>
show idprom interface <INTERFACE>
clear counters <INTERFACE>


VLANs and Layer 2


! VLAN creation and naming
vlan <VLAN_ID>
 name <VLAN_NAME>
vlan 10
 name DATA
vlan 20
 name VOICE
vlan 99
 name NATIVE
vlan 999
 name PARKING-LOT



! Access ports
interface range GigabitEthernet1/0/1 - 12
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 switchport nonegotiate
 spanning-tree portfast
 spanning-tree bpduguard enable



! Trunk ports
interface GigabitEthernet1/0/48
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk allowed vlan add 40
 switchport trunk allowed vlan remove 30
 switchport nonegotiate



! DTP modes
switchport mode dynamic auto
switchport mode dynamic desirable
switchport mode access
switchport mode trunk
switchport nonegotiate



! VTP
vtp domain <VTP_DOMAIN>
vtp version 3
vtp mode server
vtp mode transparent
vtp mode client
vtp mode off
vtp password <VTPPASSWORDPLACEHOLDER>
vtp pruning
! VTPv3 primary server (exec mode)
vtp primary vlan force



! Layer 2 EtherChannel - LACP
interface range GigabitEthernet1/0/47 - 48
 switchport
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
 channel-protocol lacp
 channel-group 1 mode active
 exit
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99



! EtherChannel - PAgP
interface range GigabitEthernet1/0/45 - 46
 channel-protocol pagp
 channel-group 2 mode desirable



! Static EtherChannel
interface range GigabitEthernet1/0/43 - 44
 channel-group 3 mode on



! Layer 3 EtherChannel
interface range TenGigabitEthernet1/0/1 - 2
 no switchport
 channel-group 10 mode active
 exit
interface Port-channel10
 no switchport
 ip address <IP_ADDRESS> <MASK>



! Load balancing and LACP tuning
port-channel load-balance src-dst-ip
lacp system-priority 100
interface <INTERFACE>
 lacp port-priority 100
 lacp rate fast



! Verification
show vlan brief
show vlan id <VLAN_ID>
show vtp status
show vtp password
show interfaces trunk
show interfaces <INTERFACE> switchport
show etherchannel summary
show etherchannel detail
show etherchannel port-channel
show etherchannel load-balance
show lacp neighbor
show lacp sys-id
show pagp neighbor
show mac address-table
show mac address-table vlan <VLAN_ID>
show mac address-table dynamic interface <INTERFACE>
clear mac address-table dynamic
show cdp neighbors detail
show lldp neighbors detail


Spanning Tree


! Mode selection
spanning-tree mode rapid-pvst
spanning-tree mode mst
spanning-tree mode pvst



! Root bridge election
spanning-tree vlan 1-100 root primary
spanning-tree vlan 1-100 root secondary
spanning-tree vlan <VLAN_ID> priority 4096
spanning-tree vlan <VLAN_ID> priority 8192
spanning-tree extend system-id



! Port cost and priority
interface <INTERFACE>
 spanning-tree vlan <VLAN_ID> cost 10000
 spanning-tree vlan <VLAN_ID> port-priority 64
 spanning-tree cost 20000
 spanning-tree port-priority 112
 spanning-tree link-type point-to-point



! Timers (configure on root only)
spanning-tree vlan <VLAN_ID> hello-time 2
spanning-tree vlan <VLAN_ID> forward-time 15
spanning-tree vlan <VLAN_ID> max-age 20
spanning-tree pathcost method long



! MST
spanning-tree mode mst
spanning-tree mst configuration
 name <MSTREGIONNAME>
 revision 1
 instance 1 vlan 1-100
 instance 2 vlan 101-200
 show pending
 exit
spanning-tree mst 1 root primary
spanning-tree mst 2 root secondary
spanning-tree mst 1 priority 8192
interface <INTERFACE>
 spanning-tree mst 1 cost 20000
 spanning-tree mst 1 port-priority 64



! Protection features
spanning-tree portfast default
spanning-tree portfast bpduguard default
spanning-tree portfast bpdufilter default
spanning-tree loopguard default
errdisable recovery cause bpduguard
errdisable recovery cause udld
errdisable recovery interval 300

interface <INTERFACE>
 spanning-tree portfast
 spanning-tree portfast trunk
 spanning-tree bpduguard enable
 spanning-tree bpdufilter enable
 spanning-tree guard root
 spanning-tree guard loop
 spanning-tree guard none



! UDLD
udld enable
udld aggressive
udld message time 15
interface <INTERFACE>
 udld port
 udld port aggressive
udld reset



! Verification
show spanning-tree
show spanning-tree summary
show spanning-tree root
show spanning-tree vlan <VLAN_ID>
show spanning-tree vlan <VLAN_ID> detail
show spanning-tree interface <INTERFACE> detail
show spanning-tree blockedports
show spanning-tree inconsistentports
show spanning-tree mst
show spanning-tree mst configuration
show spanning-tree mst 1 detail
show spanning-tree bridge
show udld <INTERFACE>
show errdisable recovery
clear spanning-tree detected-protocols
debug spanning-tree events


Inter-VLAN Routing


! Router-on-a-stick
interface GigabitEthernet0/0/1
 no ip address
 no shutdown
 exit
interface GigabitEthernet0/0/1.10
 encapsulation dot1Q 10
 ip address <IP_ADDRESS> <MASK>
 exit
interface GigabitEthernet0/0/1.20
 encapsulation dot1Q 20
 ip address <IP_ADDRESS> <MASK>
 exit
interface GigabitEthernet0/0/1.99
 encapsulation dot1Q 99 native
 ip address <IP_ADDRESS> <MASK>



! Layer 3 switch SVIs
ip routing
ipv6 unicast-routing
vlan 10,20,30
interface Vlan10
 description USER-DATA-GW
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPV6_PREFIX>::1/64
 no shutdown
interface Vlan20
 ip address <IP_ADDRESS> <MASK>
 no shutdown



! SVI autostate
interface Vlan30
 no autostate



! Routed port
interface GigabitEthernet1/0/24
 no switchport
 ip address <IP_ADDRESS> <MASK>



! Verification
show ip route
show ip route connected
show vlan brief
show interfaces vlan <VLAN_ID>
show ip interface brief | exclude unassigned
show ipv6 route
show sdm prefer


IPv4 and IPv6


! IPv4 static / default / floating static
ip route <NETWORK> <MASK> <NEXTHOPIP>
ip route <NETWORK> <MASK> <INTERFACE> <NEXTHOPIP>
ip route <NETWORK> <MASK> <NEXTHOPIP> name <ROUTE_NAME>
ip route 0.0.0.0 0.0.0.0 <NEXTHOPIP>
ip route 0.0.0.0 0.0.0.0 <BACKUPNEXTHOP> 200
ip route <NETWORK> <MASK> Null0
ip route <NETWORK> <MASK> <NEXTHOPIP> track 1



! IPv6 static / default
ipv6 unicast-routing
ipv6 route <IPV6PREFIX>/64 <IPV6NEXT_HOP>
ipv6 route <IPV6_PREFIX>/64 <INTERFACE> FE80::2
ipv6 route ::/0 <IPV6NEXTHOP>
ipv6 route ::/0 <INTERFACE> FE80::2 200
ipv6 route <IPV6_PREFIX>/64 Null0



! CEF / load sharing
ip cef
ipv6 cef
ip cef load-sharing algorithm include-ports source destination



! Verification
show ip route
show ip route <IP_ADDRESS>
show ip route summary
show ip route static
show ip route 0.0.0.0
show ipv6 route
show ipv6 route static
show ipv6 neighbors
show ip arp
show ip cef <IP_ADDRESS>
show ip cef exact-route <SRCIP> <DSTIP>
show adjacency detail
ping <IP_ADDRESS> source <INTERFACE> repeat 100
ping ipv6 <IPV6_ADDRESS>
traceroute <IP_ADDRESS> source <INTERFACE>


OSPF


! OSPFv2 basic
router ospf 1
 router-id <ROUTER_ID>
 network <IPADDRESS> <WILDCARD> area <AREAID>
 network 10.0.0.0 0.255.255.255 area 0
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
 auto-cost reference-bandwidth 100000
 log-adjacency-changes detail
 maximum-paths 4



! Interface-level OSPF (preferred on IOS XE)
interface <INTERFACE>
 ip ospf 1 area <AREA_ID>
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 255
 ip ospf hello-interval 10
 ip ospf dead-interval 40
 ip ospf mtu-ignore
 ip ospf retransmit-interval 5



! Default route origination
router ospf 1
 default-information originate
 default-information originate always metric 20 metric-type 1



! Area types
router ospf 1
 area 1 stub
 area 1 stub no-summary
 area 2 nssa
 area 2 nssa no-summary
 area 2 nssa default-information-originate
 area 3 range <IP_ADDRESS> <MASK>
 summary-address <IP_ADDRESS> <MASK>
 area 1 default-cost 10



! Virtual link
router ospf 1
 area 1 virtual-link <REMOTEROUTERID>



! Authentication - interface (SHA / MD5)
key chain OSPF-KC
 key 1
  key-string <KEY_PLACEHOLDER>
  cryptographic-algorithm hmac-sha-256
interface <INTERFACE>
 ip ospf authentication key-chain OSPF-KC



! Authentication - MD5 legacy
interface <INTERFACE>
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 <KEY_PLACEHOLDER>
router ospf 1
 area 0 authentication message-digest



! OSPFv3 (address-family / IPv6)
ipv6 unicast-routing
router ospfv3 1
 router-id <ROUTER_ID>
 address-family ipv6 unicast
  passive-interface default
  no passive-interface GigabitEthernet0/0/0
  default-information originate always
  exit-address-family
 address-family ipv4 unicast
  exit-address-family

interface <INTERFACE>
 ospfv3 1 ipv6 area 0
 ospfv3 1 ipv4 area 0
 ospfv3 cost 10
 ospfv3 network point-to-point
 ospfv3 authentication ipsec spi 500 sha1 <KEY_PLACEHOLDER>



! Verification and troubleshooting
show ip ospf
show ip ospf interface brief
show ip ospf interface <INTERFACE>
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip ospf database summary
show ip ospf database external
show ip ospf database nssa-external
show ip ospf border-routers
show ip ospf statistics
show ip ospf rib
show ip route ospf
show ipv6 ospf
show ipv6 ospf neighbor
show ipv6 ospf interface
show ipv6 ospf database
show ipv6 route ospf
clear ip ospf process
clear ip ospf redistribution
debug ip ospf adj
debug ip ospf hello
debug ip ospf events
debug ip ospf packet


EIGRP


! Classic EIGRP
router eigrp <AS_NUMBER>
 eigrp router-id <ROUTER_ID>
 network <IP_ADDRESS> <WILDCARD>
 network 10.0.0.0 0.255.255.255
 no auto-summary
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
 variance 4
 maximum-paths 6
 eigrp stub connected summary
 redistribute static metric 10000 100 255 1 1500



! Named EIGRP
router eigrp ENCOR-EIGRP
 address-family ipv4 unicast autonomous-system <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  network 10.0.0.0 0.255.255.255
  af-interface default
   passive-interface
   exit-af-interface
  af-interface GigabitEthernet0/0/0
   no passive-interface
   hello-interval 5
   hold-time 15
   bandwidth-percent 50
   summary-address <IP_ADDRESS> <MASK>
   authentication mode hmac-sha-256 <KEY_PLACEHOLDER>
   exit-af-interface
  topology base
   variance 4
   maximum-paths 6
   distribute-list prefix <PREFIXLISTNAME> out
   exit-af-topology
  exit-address-family
 address-family ipv6 unicast autonomous-system <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  exit-address-family



! Classic authentication (MD5)
key chain EIGRP-KC
 key 1
  key-string <KEY_PLACEHOLDER>
  cryptographic-algorithm hmac-sha-256
interface <INTERFACE>
 ip authentication mode eigrp <AS_NUMBER> md5
 ip authentication key-chain eigrp <AS_NUMBER> EIGRP-KC



! Metrics, timers, summarization, default route
interface <INTERFACE>
 bandwidth 100000
 delay 100
 ip hello-interval eigrp <AS_NUMBER> 5
 ip hold-time eigrp <AS_NUMBER> 15
 ip bandwidth-percent eigrp <AS_NUMBER> 50
 ip summary-address eigrp <ASNUMBER> <IPADDRESS> <MASK>
 no ip split-horizon eigrp <AS_NUMBER>

router eigrp <AS_NUMBER>
 metric weights 0 1 0 1 0 0
 network 0.0.0.0
ip route 0.0.0.0 0.0.0.0 <NEXTHOPIP>



! EIGRP for IPv6 (classic)
ipv6 router eigrp <AS_NUMBER>
 eigrp router-id <ROUTER_ID>
 no shutdown
interface <INTERFACE>
 ipv6 eigrp <AS_NUMBER>



! Verification
show ip eigrp neighbors
show ip eigrp neighbors detail
show ip eigrp interfaces
show ip eigrp interfaces detail
show ip eigrp topology
show ip eigrp topology all-links
show ip eigrp topology <NETWORK>/<PREFIX>
show ip eigrp traffic
show ip protocols
show ip route eigrp
show eigrp address-family ipv4 neighbors
show eigrp address-family ipv4 topology
show eigrp protocols
show ipv6 eigrp neighbors
show ipv6 route eigrp
clear ip eigrp neighbors
debug eigrp packets
debug ip eigrp notifications


BGP


! Base configuration
router bgp <AS_NUMBER>
 bgp router-id <ROUTER_ID>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 bgp deterministic-med
 bgp bestpath as-path multipath-relax
 timers bgp 10 30



! eBGP neighbor
router bgp 65001
 neighbor <PEER_IP> remote-as 65002
 neighbor <PEER_IP> description eBGP-to-ISP
 neighbor <PEERIP> password <BGPKEY_PLACEHOLDER>
 neighbor <PEER_IP> ebgp-multihop 2
 neighbor <PEER_IP> update-source Loopback0
 address-family ipv4 unicast
  neighbor <PEER_IP> activate
  neighbor <PEER_IP> soft-reconfiguration inbound
  exit-address-family



! iBGP with loopback peering
router bgp 65001
 neighbor <IBGPPEERIP> remote-as 65001
 neighbor <IBGPPEERIP> update-source Loopback0
 address-family ipv4 unicast
  neighbor <IBGPPEERIP> activate
  neighbor <IBGPPEERIP> next-hop-self all
  exit-address-family



! Peer groups / templates
router bgp 65001
 neighbor IBGP-PEERS peer-group
 neighbor IBGP-PEERS remote-as 65001
 neighbor IBGP-PEERS update-source Loopback0
 neighbor <IBGPPEERIP> peer-group IBGP-PEERS



! Network advertisement / aggregation / default route
router bgp 65001
 address-family ipv4 unicast
  network <NETWORK> mask <MASK>
  aggregate-address <NETWORK> <MASK> summary-only
  aggregate-address <NETWORK> <MASK> as-set
  redistribute ospf 1 route-map OSPF-TO-BGP
  neighbor <PEER_IP> default-originate
  neighbor <PEER_IP> default-originate route-map DEFAULT-COND
  exit-address-family



! Prefix lists
ip prefix-list PL-ALLOW seq 5 permit <NETWORK>/<PREFIX>
ip prefix-list PL-ALLOW seq 10 permit 0.0.0.0/0
ip prefix-list PL-BLOCK seq 5 deny <NETWORK>/<PREFIX> le 32
ip prefix-list PL-SUMMARY seq 5 permit 10.0.0.0/8 ge 16 le 24
ipv6 prefix-list PL6-ALLOW seq 5 permit <IPV6_PREFIX>/48 le 64



! AS-path filtering / communities / route maps
ip as-path access-list 1 permit ^$
ip as-path access-list 10 permit 65002
ip community-list standard CL-CUST permit 65001:100
ip community-list expanded CL-EXP permit 65001:1[0-9][0-9]

route-map RM-IN permit 10
 match ip address prefix-list PL-ALLOW
 set local-preference 200
 set community 65001:100 additive
route-map RM-IN permit 20
 set local-preference 90

route-map RM-OUT permit 10
 match as-path 1
 set as-path prepend 65001 65001 65001
 set metric 100
route-map RM-OUT deny 20

route-map RM-WEIGHT permit 10
 match ip address prefix-list PL-ALLOW
 set weight 500



! Applying policy
router bgp 65001
 address-family ipv4 unicast
  neighbor <PEER_IP> route-map RM-IN in
  neighbor <PEER_IP> route-map RM-OUT out
  neighbor <PEER_IP> prefix-list PL-ALLOW in
  neighbor <PEER_IP> filter-list 10 in
  neighbor <PEER_IP> send-community both
  neighbor <PEER_IP> maximum-prefix 100000 80 restart 15
  neighbor <PEER_IP> weight 100
  neighbor <PEER_IP> route-reflector-client
  neighbor <PEER_IP> allowas-in 1
  neighbor <PEER_IP> as-override
  exit-address-family



! MP-BGP IPv6
router bgp 65001
 neighbor <IPV6_PEER> remote-as 65002
 address-family ipv6 unicast
  neighbor <IPV6_PEER> activate
  network <IPV6_PREFIX>/48
  exit-address-family



! Verification
show ip bgp summary
show bgp ipv4 unicast summary
show ip bgp
show ip bgp <NETWORK>/<PREFIX>
show ip bgp neighbors <PEER_IP>
show ip bgp neighbors <PEER_IP> advertised-routes
show ip bgp neighbors <PEER_IP> routes
show ip bgp neighbors <PEER_IP> received-routes
show ip bgp regexp 65002
show ip bgp community 65001:100
show ip bgp filter-list 10
show ip bgp paths
show ip bgp rib-failure
show bgp ipv6 unicast summary
show ip route bgp
show ip protocols



! Troubleshooting
clear ip bgp 
clear ip bgp <PEER_IP> soft
clear ip bgp <PEER_IP> soft in
clear ip bgp <PEER_IP> soft out
clear bgp ipv4 unicast <PEER_IP> in
debug ip bgp
debug ip bgp updates
debug ip bgp <PEER_IP> updates
debug ip bgp events


IP Routing and Path Control


! Administrative distance
router ospf 1
 distance 115
 distance ospf external 130
router eigrp <AS_NUMBER>
 distance eigrp 90 170
router bgp <AS_NUMBER>
 distance bgp 20 200 200
ip route <NETWORK> <MASK> <NEXTHOPIP> 250



! Redistribution
router ospf 1
 redistribute eigrp <AS_NUMBER> subnets metric 100 metric-type 1 route-map RM-EIGRP-TO-OSPF
 redistribute static subnets
 redistribute connected subnets
 redistribute bgp <AS_NUMBER> subnets

router eigrp <AS_NUMBER>
 redistribute ospf 1 metric 1000000 10 255 1 1500 route-map RM-OSPF-TO-EIGRP
 redistribute connected metric 1000000 10 255 1 1500
 default-metric 1000000 10 255 1 1500

router bgp <AS_NUMBER>
 address-family ipv4 unicast
  redistribute ospf 1 match internal external 1 external 2



! Route maps with tags (loop prevention)
route-map RM-OSPF-TO-EIGRP deny 10
 match tag 100
route-map RM-OSPF-TO-EIGRP permit 20
 set tag 200

route-map RM-EIGRP-TO-OSPF permit 10
 match ip address prefix-list PL-ALLOW
 set metric-type type-1
 set tag 100



! Distribute lists
router ospf 1
 distribute-list prefix PL-BLOCK in
 distribute-list route-map RM-FILTER in GigabitEthernet0/0/0
router eigrp <AS_NUMBER>
 distribute-list prefix PL-ALLOW out GigabitEthernet0/0/1



! Policy-Based Routing
ip access-list extended ACL-PBR-USERS
 permit ip 10.10.10.0 0.0.0.255 any

route-map PBR-USERS permit 10
 match ip address ACL-PBR-USERS
 set ip next-hop verify-availability <NEXTHOPIP> 10 track 1
 set ip next-hop <NEXTHOPIP>
 set interface <INTERFACE>
 set ip precedence 5
route-map PBR-USERS permit 20

interface <INTERFACE>
 ip policy route-map PBR-USERS
ip local policy route-map PBR-USERS



! IP SLA and object tracking
ip sla 1
 icmp-echo <TARGET_IP> source-interface <INTERFACE>
 frequency 5
 threshold 500
 timeout 1000
ip sla schedule 1 life forever start-time now

ip sla 2
 http get http://<TARGET_IP>
ip sla schedule 2 life forever start-time now

track 1 ip sla 1 reachability
track 2 ip sla 1 state
track 10 interface <INTERFACE> line-protocol
track 20 ip route <NETWORK> <MASK> reachability
track 30 list boolean and
 object 1
 object 10



! Floating static with tracking
ip route 0.0.0.0 0.0.0.0 <PRIMARYNEXTHOP> track 1
ip route 0.0.0.0 0.0.0.0 <BACKUPNEXTHOP> 200



! Verification
show ip route
show ip protocols
show route-map
show route-map PBR-USERS
show ip policy
show ip prefix-list
show ip prefix-list detail PL-ALLOW
show ip access-lists
show ip sla configuration
show ip sla statistics
show ip sla summary
show track
show track brief
debug ip policy


NAT


! Interfaces
interface GigabitEthernet0/0/0
 description OUTSIDE
 ip address <PUBLIC_IP> <MASK>
 ip nat outside
interface GigabitEthernet0/0/1
 description INSIDE
 ip address <IP_ADDRESS> <MASK>
 ip nat inside



! Static NAT
ip nat inside source static <INSIDELOCALIP> <INSIDEGLOBALIP>
ip nat inside source static tcp <INSIDELOCALIP> 443 <INSIDEGLOBALIP> 443 extendable
ip nat outside source static <OUTSIDEGLOBALIP> <OUTSIDELOCALIP>



! Dynamic NAT with pool
ip nat pool NAT-POOL <STARTIP> <ENDIP> netmask <MASK>
ip access-list standard ACL-NAT
 permit 10.0.0.0 0.255.255.255
ip nat inside source list ACL-NAT pool NAT-POOL



! PAT
ip nat inside source list ACL-NAT pool NAT-POOL overload
ip nat inside source list ACL-NAT interface GigabitEthernet0/0/0 overload



! Route-map based NAT
route-map RM-NAT permit 10
 match ip address ACL-NAT
 match interface GigabitEthernet0/0/0
ip nat inside source route-map RM-NAT interface GigabitEthernet0/0/0 overload



! Timers
ip nat translation timeout 3600
ip nat translation tcp-timeout 3600
ip nat translation udp-timeout 300
ip nat translation icmp-timeout 60
ip nat translation max-entries 20000



! Verification / troubleshooting
show ip nat translations
show ip nat translations verbose
show ip nat statistics
show ip nat pool name NAT-POOL
clear ip nat translation 
clear ip nat translation inside <INSIDEGLOBALIP> <INSIDELOCALIP>
clear ip nat statistics
debug ip nat
debug ip nat detailed


DHCP


! DHCP server
ip dhcp excluded-address <STARTIP> <ENDIP>
ip dhcp excluded-address <IP_ADDRESS>

ip dhcp pool VLAN10-POOL
 network <NETWORK> <MASK>
 default-router <GATEWAY_IP>
 dns-server <DNSIP1> <DNSIP2>
 domain-name <DOMAIN_NAME>
 lease 7 0 0
 option 150 ip <TFTPSERVERIP>
 option 43 hex f104.0a0a.0a05
 netbios-name-server <WINS_IP>



! DHCP reservation
ip dhcp pool STATIC-HOST
 host <IP_ADDRESS> <MASK>
 client-identifier 0100.1122.3344.55
 hardware-address 0011.2233.4455
 default-router <GATEWAY_IP>



! DHCP relay
interface Vlan10
 ip helper-address <DHCPSERVERIP>
 ip helper-address <DHCPSERVERIP_2>
ip forward-protocol udp bootps
ip dhcp relay information trust-all



! DHCP client
interface <INTERFACE>
 ip address dhcp
 ipv6 address dhcp
 ipv6 enable



! DHCPv6 server / stateless
ipv6 dhcp pool DHCPV6-POOL
 address prefix <IPV6_PREFIX>::/64
 dns-server <IPV6_DNS>
 domain-name <DOMAIN_NAME>
interface <INTERFACE>
 ipv6 dhcp server DHCPV6-POOL
 ipv6 nd other-config-flag
 ipv6 nd managed-config-flag



! Verification
show ip dhcp binding
show ip dhcp pool
show ip dhcp pool VLAN10-POOL
show ip dhcp conflict
show ip dhcp server statistics
show ip dhcp relay information trusted-sources
show ipv6 dhcp pool
show ipv6 dhcp binding
clear ip dhcp binding 
clear ip dhcp conflict 
debug ip dhcp server events
debug ip dhcp server packet


First Hop Redundancy


! HSRPv2 IPv4
interface Vlan10
 ip address <IP_ADDRESS> <MASK>
 standby version 2
 standby 10 ip <VIRTUAL_IP>
 standby 10 priority 110
 standby 10 preempt delay minimum 60
 standby 10 timers msec 250 msec 750
 standby 10 authentication md5 key-string <KEY_PLACEHOLDER>
 standby 10 name HSRP-VLAN10
 standby 10 track 1 decrement 20
 standby 10 track <INTERFACE> 30



! HSRP IPv6
interface Vlan10
 standby version 2
 standby 20 ipv6 autoconfig
 standby 20 priority 110
 standby 20 preempt



! VRRPv3
fhrp version vrrp v3
interface Vlan20
 ip address <IP_ADDRESS> <MASK>
 vrrp 20 address-family ipv4
  address <VIRTUAL_IP>
  priority 120
  preempt delay minimum 30
  timers advertise 100
  vrrs leader VRRP-VLAN20
  exit
interface Vlan20
 vrrp 20 address-family ipv6
  address FE80::20 primary
  address <IPV6VIRTUALADDRESS>



! Legacy VRRPv2
interface Vlan20
 vrrp 20 ip <VIRTUAL_IP>
 vrrp 20 priority 120
 vrrp 20 preempt
 vrrp 20 authentication md5 key-string <KEY_PLACEHOLDER>
 vrrp 20 track 1 decrement 30



! GLBP
interface Vlan30
 ip address <IP_ADDRESS> <MASK>
 glbp 30 ip <VIRTUAL_IP>
 glbp 30 priority 110
 glbp 30 preempt
 glbp 30 load-balancing host-dependent
 glbp 30 weighting 100 lower 80 upper 100
 glbp 30 weighting track 1 decrement 30
 glbp 30 authentication md5 key-string <KEY_PLACEHOLDER>
 glbp 30 timers 3 10
 glbp 30 forwarder preempt delay minimum 30



! Verification
show standby
show standby brief
show standby vlan10
show standby all
show vrrp
show vrrp brief
show vrrp all
show glbp
show glbp brief
show fhrp all
show track
debug standby
debug standby events terse
debug vrrp all
debug glbp events


Wireless LAN


! AP CAPWAP join (AP CLI - CAPWAP AP mode)
capwap ap hostname <AP_NAME>
capwap ap ip <IPADDRESS> <MASK> <GATEWAYIP>
capwap ap primary-base <WLCNAME> <WLCMGMT_IP>
capwap ap secondary-base <WLCNAME2> <WLCMGMT_IP2>
capwap ap controller ip address <WLCMGMTIP>
capwap ap erase all
capwap ap restart



! Catalyst 9800 WLC (IOS XE) - AAA for wireless
aaa new-model
radius server ISE-1
 address ipv4 <RADIUS_IP> auth-port 1812 acct-port 1813
 key <RADIUSKEYPLACEHOLDER>
 automate-tester username <PROBE_USER> probe-on
aaa group server radius RAD-GRP
 server name ISE-1
 deadtime 5
aaa authentication dot1x DOT1X-LIST group RAD-GRP
aaa authorization network DOT1X-LIST group RAD-GRP
aaa accounting identity DOT1X-LIST start-stop group RAD-GRP
radius-server attribute 6 on-for-login-auth
radius-server attribute 31 mac format ietf upper-case



! WLAN with WPA2-Enterprise (802.1X)
wlan CORP-WLAN 1 <SSID_NAME>
 security wpa
 security wpa wpa2
 security wpa wpa2 ciphers aes
 security dot1x authentication-list DOT1X-LIST
 security pmf optional
 no shutdown



! WLAN with WPA2-PSK
wlan GUEST-WLAN 2 <SSID_NAME>
 no security wpa akm dot1x
 security wpa akm psk set-key ascii 0 <PSK_PLACEHOLDER>
 security wpa wpa2 ciphers aes
 no shutdown



! WLAN with WPA3-SAE
wlan SECURE-WLAN 3 <SSID_NAME>
 security wpa psk set-key ascii 0 <PSK_PLACEHOLDER>
 security wpa akm sae
 security wpa wpa3
 security wpa wpa2 ciphers gcmp256
 security pmf mandatory
 no shutdown



! Open / broadcast control
wlan OPEN-WLAN 4 <SSID_NAME>
 no security wpa
 no security wpa akm dot1x
 no security wpa wpa2
 broadcast-ssid
 no broadcast-ssid
 client vlan <VLAN_ID>
 no shutdown



! Policy profile (VLAN mapping, central switching)
wireless profile policy POLICY-CORP
 vlan <VLAN_ID>
 central switching
 central authentication
 central dhcp
 aaa-override
 nac
 accounting-list DOT1X-LIST
 session-timeout 86400
 idle-timeout 300
 no shutdown



! RF and Flex profiles
wireless profile rf RF-CORP
wireless profile flex FLEX-BRANCH
 native-vlan-id <VLAN_ID>
 vlan-name DATA
  vlan-id <VLAN_ID>



! Tags
wireless tag policy TAG-POLICY-CAMPUS
 wlan CORP-WLAN policy POLICY-CORP
 wlan GUEST-WLAN policy POLICY-GUEST
wireless tag site TAG-SITE-CAMPUS
 no local-site
wireless tag rf TAG-RF-CAMPUS
 5ghz-rf-policy RF-CORP
 24ghz-rf-policy RF-CORP

ap <APMACADDRESS>
 policy-tag TAG-POLICY-CAMPUS
 site-tag TAG-SITE-CAMPUS
 rf-tag TAG-RF-CAMPUS



! Controller interfaces / management
wireless management interface Vlan<VLAN_ID>
wireless management trustpoint <TRUSTPOINT_NAME>
wireless country <COUNTRY_CODE>
wireless mobility group name <MOBILITY_GROUP>
wireless mobility group member ip <PEERWLCIP> public-ip <PEERWLCIP> group <MOBILITY_GROUP>
wireless mobility mac-address <MAC_ADDRESS>



! RRM / roaming assist
ap dot11 5ghz rrm channel dca interval 12
ap dot11 5ghz rrm tpc-threshold -70
ap dot11 24ghz rrm ccx location-measurement 60
wireless profile policy POLICY-CORP
 ipv4 dhcp required
 mobility anchor <ANCHORWLCIP> priority 1
ap dot11 5ghz cleanair
ap dot11 5ghz rrm group-mode auto



! Verification
show wireless summary
show wlan summary
show wlan id 1
show wireless profile policy summary
show wireless tag summary
show ap summary
show ap config general
show wireless client summary
show wireless mobility summary
show wireless management trustpoint
show wireless country configured


Wireless Troubleshooting


! Controller and AP state
show wireless summary
show wireless stats client detail
show wireless stats ap join summary
show ap summary
show ap status
show ap uptime
show ap join stats summary
show ap config general
show ap name <AP_NAME> config general
show ap name <AP_NAME> config dot11 5ghz
show ap dot11 5ghz summary
show ap dot11 24ghz summary
show ap dot11 5ghz channel
show ap dot11 5ghz load-info
show ap image
show ap cdp neighbors
show ap tag summary
show ap name <AP_NAME> tag detail



! Clients
show wireless client summary
show wireless client mac-address <CLIENT_MAC> detail
show wireless client mac-address <CLIENT_MAC> stats
show wireless client mac-address <CLIENT_MAC> mobility history
show wireless client mac-address <CLIENT_MAC> dot11
show wireless client device summary
show wireless exclusionlist



! WLAN / policy / RF
show wlan summary
show wlan id <WLAN_ID>
show wlan name <WLAN_NAME> client stats
show wireless profile policy detailed POLICY-CORP
show wireless profile flex detailed FLEX-BRANCH
show wireless tag policy detailed TAG-POLICY-CAMPUS
show wireless dot11 5ghz network
show wireless mobility summary
show wireless mobility peer ip <PEERWLCIP>
show wireless mobility ap-list



! AAA / security
show aaa servers
show radius statistics
show wireless client mac-address <CLIENT_MAC> detail | include Policy|VLAN|State
test aaa group radius server <RADIUSIP> <USERNAME> <PASSWORDPLACEHOLDER> new-code



! Trace / debug (9800)
debug wireless mac <CLIENT_MAC> monitor-time 600
debug wireless ap name <AP_NAME> monitor-time 600
no debug wireless mac <CLIENT_MAC>
show wireless loganalyzer summary
show logging profile wireless start last 30 minutes
show logging profile wireless filter mac <CLIENT_MAC> to-file bootflash:client.txt
debug capwap error
debug capwap events
debug client <CLIENT_MAC>
debug dot11 all



! Packet capture on WLC
monitor capture MYCAP interface <INTERFACE> both
monitor capture MYCAP match any
monitor capture MYCAP buffer size 100
monitor capture MYCAP start
monitor capture MYCAP stop
monitor capture MYCAP export bootflash:mycap.pcap
show monitor capture MYCAP buffer brief


Network Security


! AAA base
aaa new-model
aaa session-id common



! TACACS+
tacacs server TAC-1
 address ipv4 <TACACS_IP>
 key <TACACSKEYPLACEHOLDER>
 timeout 5
 single-connection
aaa group server tacacs+ TAC-GRP
 server name TAC-1
 ip tacacs source-interface Loopback0



! RADIUS
radius server RAD-1
 address ipv4 <RADIUS_IP> auth-port 1812 acct-port 1813
 key <RADIUSKEYPLACEHOLDER>
 retransmit 3
 timeout 5
aaa group server radius RAD-GRP
 server name RAD-1
 ip radius source-interface Loopback0



! Authentication / authorization / accounting
aaa authentication login default group TAC-GRP local
aaa authentication login CONSOLE local
aaa authentication enable default group TAC-GRP enable
aaa authorization config-commands
aaa authorization exec default group TAC-GRP local if-authenticated
aaa authorization commands 15 default group TAC-GRP local if-authenticated
aaa accounting exec default start-stop group TAC-GRP
aaa accounting commands 15 default start-stop group TAC-GRP
aaa accounting system default start-stop group TAC-GRP

line console 0
 login authentication CONSOLE
line vty 0 15
 login authentication default
 authorization exec default
 transport input ssh



! Login hardening
login block-for 120 attempts 3 within 60
login quiet-mode access-class ACL-MGMT
login delay 2
login on-failure log
login on-success log
service tcp-keepalives-in
service tcp-keepalives-out



! Standard ACL
ip access-list standard ACL-MGMT
 permit 10.10.10.0 0.0.0.255
 deny   any log
line vty 0 15
 access-class ACL-MGMT in vrf-also



! Extended ACL
ip access-list extended ACL-EDGE-IN
 permit tcp any host <IP_ADDRESS> eq 443
 permit tcp any host <IP_ADDRESS> eq www
 permit udp any host <IP_ADDRESS> eq domain
 permit icmp any any echo-reply
 deny   ip 10.0.0.0 0.255.255.255 any log
 deny   ip any any log
interface <INTERFACE>
 ip access-group ACL-EDGE-IN in



! Time-based / object groups
time-range WORK-HOURS
 periodic weekdays 08:00 to 18:00
object-group network OG-SERVERS
 host <IP_ADDRESS>
 <NETWORK> <MASK>
object-group service OG-WEB
 tcp eq 80
 tcp eq 443
ip access-list extended ACL-OG
 permit object-group OG-WEB any object-group OG-SERVERS time-range WORK-HOURS



! IPv6 ACL
ipv6 access-list ACL6-EDGE
 permit tcp any host <IPV6_ADDRESS> eq 443
 permit icmp any any nd-na
 permit icmp any any nd-ns
 deny ipv6 any any log
interface <INTERFACE>
 ipv6 traffic-filter ACL6-EDGE in
line vty 0 15
 ipv6 access-class ACL6-MGMT in



! Control Plane Policing
ip access-list extended ACL-CoPP-SSH
 permit tcp <MGMT_NET> <WILDCARD> any eq 22
class-map match-all CM-CoPP-SSH
 match access-group name ACL-CoPP-SSH
class-map match-all CM-CoPP-ICMP
 match protocol icmp
policy-map PM-CoPP
 class CM-CoPP-SSH
  police 128000 conform-action transmit exceed-action drop
 class CM-CoPP-ICMP
  police 32000 conform-action transmit exceed-action drop
 class class-default
  police 512000 conform-action transmit exceed-action drop
control-plane
 service-policy input PM-CoPP



! Device hardening
no ip http server
ip http secure-server
ip http authentication aaa
ip http secure-ciphersuite aes-128-cbc-sha aes-256-cbc-sha
no cdp run
no lldp run global
no service pad
no ip source-route
no ip bootp server
no ip domain-lookup
no service config
service sequence-numbers
memory free low-watermark processor 65536
exception crashinfo maximum files 2



! Verification
show aaa servers
show aaa sessions
show aaa method-lists all
show tacacs
show radius statistics
show ip access-lists
show ipv6 access-list
show access-lists
show policy-map control-plane
show login failures
debug aaa authentication
debug aaa authorization
debug radius authentication
debug tacacs
test aaa group TAC-GRP <USERNAME> <PASSWORD_PLACEHOLDER> legacy


Infrastructure Security


! Port security
interface GigabitEthernet1/0/5
 switchport mode access
 switchport access vlan <VLAN_ID>
 switchport port-security
 switchport port-security maximum 3
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security mac-address <MAC_ADDRESS>
 switchport port-security aging time 10
 switchport port-security aging type inactivity
errdisable recovery cause psecure-violation



! DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30
no ip dhcp snooping information option
ip dhcp snooping database flash:dhcp-snoop.db
interface GigabitEthernet1/0/48
 ip dhcp snooping trust
interface range GigabitEthernet1/0/1 - 24
 ip dhcp snooping limit rate 15



! Dynamic ARP Inspection
ip arp inspection vlan 10,20,30
ip arp inspection validate src-mac dst-mac ip
arp access-list ARP-STATIC
 permit ip host <IPADDRESS> mac host <MACADDRESS>
ip arp inspection filter ARP-STATIC vlan 10
interface GigabitEthernet1/0/48
 ip arp inspection trust
interface range GigabitEthernet1/0/1 - 24
 ip arp inspection limit rate 15



! IP Source Guard
interface range GigabitEthernet1/0/1 - 24
 ip verify source
 ip verify source port-security
ip source binding <MACADDRESS> vlan <VLANID> <IP_ADDRESS> interface GigabitEthernet1/0/5



! Storm control
interface range GigabitEthernet1/0/1 - 24
 storm-control broadcast level pps 1k
 storm-control multicast level 5.00
 storm-control unicast level 60.00 50.00
 storm-control action shutdown
 storm-control action trap



! Private VLANs
vtp mode transparent
vlan 100
 private-vlan primary
 private-vlan association 101,102
vlan 101
 private-vlan isolated
vlan 102
 private-vlan community

interface GigabitEthernet1/0/2
 switchport mode private-vlan host
 switchport private-vlan host-association 100 101
interface GigabitEthernet1/0/48
 switchport mode private-vlan promiscuous
 switchport private-vlan mapping 100 101,102
interface Vlan100
 private-vlan mapping 101,102

! Protected port (alternative)
interface GigabitEthernet1/0/3
 switchport protected



! 802.1X and MAB (IBNS 2.0)
aaa new-model
dot1x system-auth-control
aaa authentication dot1x default group RAD-GRP
aaa authorization network default group RAD-GRP
aaa accounting dot1x default start-stop group RAD-GRP
radius-server attribute 6 on-for-login-auth
radius-server attribute 8 include-in-access-req
radius-server attribute 25 access-request include

policy-map type control subscriber PM-DOT1X-MAB
 event session-started match-all
  10 class always do-until-failure
   10 authenticate using dot1x priority 10
   20 authenticate using mab priority 20

interface range GigabitEthernet1/0/1 - 24
 switchport mode access
 switchport access vlan <VLAN_ID>
 access-session host-mode multi-domain
 access-session closed
 access-session port-control auto
 mab
 dot1x pae authenticator
 dot1x timeout tx-period 7
 authentication periodic
 authentication timer reauthenticate server
 service-policy type control subscriber PM-DOT1X-MAB
 spanning-tree portfast



! Cisco TrustSec
cts credentials id <DEVICEID> password <CTSPASSWORD_PLACEHOLDER>
cts authorization list DOT1X-LIST
cts role-based enforcement
cts role-based enforcement vlan-list 10-20
cts role-based sgt-map <IP_ADDRESS> sgt 100
cts role-based sgt-map vlan-list 10 sgt 10
cts sxp enable
cts sxp default password <SXPPASSWORDPLACEHOLDER>
cts sxp default source-ip <IP_ADDRESS>
cts sxp connection peer <PEER_IP> password default mode local listener
interface <INTERFACE>
 cts manual
  policy static sgt 100 trusted
  propagate sgt



! Verification
show port-security
show port-security interface <INTERFACE>
show port-security address
show ip dhcp snooping
show ip dhcp snooping binding
show ip dhcp snooping statistics
show ip arp inspection
show ip arp inspection interfaces
show ip arp inspection statistics vlan <VLAN_ID>
show ip verify source
show ip source binding
show storm-control broadcast
show vlan private-vlan
show interfaces <INTERFACE> switchport
show dot1x all
show dot1x interface <INTERFACE> details
show authentication sessions
show access-session interface <INTERFACE> details
show mab all
show cts environment-data
show cts role-based permissions
show cts role-based sgt-map all
show cts sxp connections brief
show cts interface <INTERFACE>
clear access-session interface <INTERFACE>
debug dot1x all
debug access-session all


Multicast


! Enable multicast routing
ip multicast-routing
ip multicast-routing distributed
ipv6 multicast-routing



! PIM on interfaces
interface <INTERFACE>
 ip pim sparse-mode
 ip pim dense-mode
 ip pim sparse-dense-mode
 ip pim query-interval 30
 ip pim dr-priority 100
 ip igmp version 3
 ip igmp query-interval 60
 ip igmp join-group <MCAST_GROUP>
 ip igmp static-group <MCAST_GROUP>
 ip igmp access-group <ACL_NAME>



! Static RP
ip pim rp-address <RPIPADDRESS>
ip pim rp-address <RPIPADDRESS> ACL-MCAST-GROUPS override
ip access-list standard ACL-MCAST-GROUPS
 permit 239.1.0.0 0.0.255.255



! Auto-RP
ip pim send-rp-announce Loopback0 scope 16 group-list ACL-MCAST-GROUPS
ip pim send-rp-discovery Loopback0 scope 16
ip pim autorp listener
no ip pim dm-fallback



! BSR
ip pim bsr-candidate Loopback0 30
ip pim rp-candidate Loopback0 group-list ACL-MCAST-GROUPS priority 10



! Anycast RP with MSDP
interface Loopback1
 ip address <ANYCASTRPIP> 255.255.255.255
ip pim rp-address <ANYCASTRPIP>
ip msdp peer <PEERRPIP> connect-source Loopback0
ip msdp originator-id Loopback0



! SSM
ip pim ssm default
ip pim ssm range ACL-SSM-RANGE



! IGMP snooping (Layer 2)
ip igmp snooping
ip igmp snooping vlan <VLAN_ID>
ip igmp snooping vlan <VLAN_ID> immediate-leave
ip igmp snooping querier
ip igmp snooping vlan <VLANID> querier address <IPADDRESS>
ip igmp snooping vlan <VLAN_ID> mrouter interface <INTERFACE>
no ip igmp snooping vlan <VLAN_ID>



! Verification
show ip mroute
show ip mroute summary
show ip mroute <MCAST_GROUP>
show ip mroute count
show ip mroute active
show ip pim interface
show ip pim neighbor
show ip pim rp
show ip pim rp mapping
show ip pim bsr-router
show ip pim autorp
show ip igmp groups
show ip igmp interface
show ip igmp membership
show ip igmp snooping
show ip igmp snooping groups
show ip igmp snooping mrouter
show ip msdp peer
show ip msdp summary
show ip rpf <SOURCE_IP>
clear ip mroute 
clear ip igmp group
mtrace <SOURCEIP> <RECEIVERIP> <MCAST_GROUP>
debug ip pim
debug ip igmp
debug ip mpacket


QoS


! Class maps - classification
class-map match-any CM-VOICE
 match dscp ef
 match protocol rtp audio
class-map match-any CM-VIDEO
 match dscp af41 af42 af43
 match protocol cisco-phone
class-map match-all CM-SIGNALING
 match dscp cs3
class-map match-any CM-CRITICAL-DATA
 match access-group name ACL-CRITICAL
 match protocol http url "erp.example.com"
class-map match-any CM-SCAVENGER
 match dscp cs1
 match protocol bittorrent
class-map match-all CM-COS5
 match cos 5
class-map match-any CM-VLAN
 match vlan 10



! NBAR2
ip nbar protocol-discovery
interface <INTERFACE>
 ip nbar protocol-discovery ipv4
class-map match-any CM-NBAR-BUSINESS
 match protocol attribute business-relevance business-relevant
 match protocol attribute traffic-class transactional-data



! ACL-based classification
ip access-list extended ACL-CRITICAL
 permit tcp any any eq 1521
 permit tcp any host <IP_ADDRESS> eq 443



! Marking policy
policy-map PM-MARK-IN
 class CM-VOICE
  set dscp ef
  set cos 5
 class CM-VIDEO
  set dscp af41
 class CM-SIGNALING
  set dscp cs3
 class CM-SCAVENGER
  set dscp cs1
 class class-default
  set dscp default



! Queuing / LLQ / shaping / policing
policy-map PM-QUEUE-OUT
 class CM-VOICE
  priority level 1 percent 10
  police cir percent 10 conform-action transmit exceed-action drop
 class CM-VIDEO
  bandwidth remaining percent 30
  random-detect dscp-based
 class CM-CRITICAL-DATA
  bandwidth remaining percent 30
  fair-queue
 class CM-SCAVENGER
  bandwidth remaining percent 1
 class class-default
  bandwidth remaining percent 39
  fair-queue
  random-detect

policy-map PM-SHAPE-PARENT
 class class-default
  shape average 100000000
  service-policy PM-QUEUE-OUT



! Policing (single/dual rate)
policy-map PM-POLICE-IN
 class CM-SCAVENGER
  police cir 5000000 bc 156250 conform-action transmit exceed-action drop
 class class-default
  police cir 50000000 pir 100000000
   conform-action transmit
   exceed-action set-dscp-transmit af11
   violate-action drop



! Applying service policies
interface <INTERFACE>
 service-policy input PM-MARK-IN
 service-policy output PM-SHAPE-PARENT
 bandwidth 100000



! Trust and switch QoS (Catalyst IOS XE)
interface <INTERFACE>
 auto qos trust dscp
 auto qos voip cisco-phone
 trust device cisco-phone
 priority-queue out
 srr-queue bandwidth share 1 30 35 5



! Verification
show policy-map
show policy-map PM-QUEUE-OUT
show policy-map interface <INTERFACE>
show policy-map interface <INTERFACE> output
show class-map
show mls qos interface <INTERFACE>
show platform hardware qos interface <INTERFACE> output
show ip nbar protocol-discovery
show ip nbar protocol-discovery interface <INTERFACE> top-n 10
show table-map
show auto qos
clear counters
debug qos


Network Virtualization


! VRF-Lite (multi-AF definition)
vrf definition RED
 rd 65001:100
 address-family ipv4
  route-target export 65001:100
  route-target import 65001:100
  exit-address-family
 address-family ipv6
  exit-address-family

interface <INTERFACE>
 vrf forwarding RED
 ip address <IP_ADDRESS> <MASK>
 ipv6 address <IPV6_PREFIX>::1/64



! VRF routing
ip route vrf RED <NETWORK> <MASK> <NEXTHOPIP>
ipv6 route vrf RED <IPV6PREFIX>/64 <IPV6NEXT_HOP>

router ospf 10 vrf RED
 router-id <ROUTER_ID>
 network <IP_ADDRESS> <WILDCARD> area 0

router eigrp NAMED-EIGRP
 address-family ipv4 unicast vrf RED autonomous-system <AS_NUMBER>
  network 10.0.0.0 0.255.255.255
  exit-address-family

router bgp <AS_NUMBER>
 address-family ipv4 vrf RED
  neighbor <PEERIP> remote-as <REMOTEAS>
  neighbor <PEER_IP> activate
  redistribute ospf 10
  exit-address-family



! Route leaking between VRFs (BGP RT import/export)
vrf definition BLUE
 rd 65001:200
 address-family ipv4
  route-target export 65001:200
  route-target import 65001:100
  exit-address-family

! Static route leaking
ip route vrf RED <NETWORK> <MASK> <NEXTHOPIP> global
ip route <NETWORK> <MASK> <INTERFACE> vrf RED



! Management VRF
vrf definition Mgmt-vrf
interface GigabitEthernet0
 vrf forwarding Mgmt-vrf
 ip address <IP_ADDRESS> <MASK>
ip route vrf Mgmt-vrf 0.0.0.0 0.0.0.0 <GATEWAY_IP>
ip tftp source-interface GigabitEthernet0



! GRE tunnel
interface Tunnel0
 description GRE-TO-BRANCH
 ip address <TUNNEL_IP> <MASK>
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source <INTERFACE>
 tunnel destination <REMOTEPUBLICIP>
 tunnel mode gre ip
 tunnel vrf Mgmt-vrf
 keepalive 5 3



! IPsec (IKEv2) protecting GRE
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
crypto ikev2 policy IKEV2-POL
 proposal IKEV2-PROP
crypto ikev2 keyring IKEV2-KR
 peer BRANCH
  address <REMOTEPUBLICIP>
  pre-shared-key <PSK_PLACEHOLDER>
crypto ikev2 profile IKEV2-PROF
 match identity remote address <REMOTEPUBLICIP> 255.255.255.255
 identity local address <LOCALPUBLICIP>
 authentication local pre-share
 authentication remote pre-share
 keyring local IKEV2-KR

crypto ipsec transform-set TS-AES esp-aes 256 esp-sha256-hmac
 mode transport
crypto ipsec profile IPSEC-PROF
 set transform-set TS-AES
 set ikev2-profile IKEV2-PROF

interface Tunnel0
 tunnel protection ipsec profile IPSEC-PROF



! IPsec VTI
interface Tunnel1
 ip address <TUNNEL_IP> <MASK>
 tunnel source <INTERFACE>
 tunnel destination <REMOTEPUBLICIP>
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile IPSEC-PROF



! VXLAN (BGP EVPN basics)
l2vpn evpn
 replication-type ingress
 router-id Loopback0
l2vpn evpn instance 10 vlan-based
 encapsulation vxlan
vlan configuration 10
 member evpn-instance 10 vni 10010
vlan configuration 100
 member vni 50000

interface nve1
 no ip address
 source-interface Loopback1
 host-reachability protocol bgp
 member vni 10010 ingress-replication
 member vni 50000 vrf RED

router bgp <AS_NUMBER>
 address-family l2vpn evpn
  neighbor <SPINE_IP> activate
  neighbor <SPINE_IP> send-community both
  exit-address-family



! Verification
show vrf
show vrf detail
show ip route vrf RED
show ip vrf interfaces
show ipv6 route vrf RED
ping vrf RED <IP_ADDRESS>
traceroute vrf RED <IP_ADDRESS>
show ip bgp vpnv4 vrf RED
show interfaces Tunnel0
show ip interface brief | include Tunnel
show crypto ikev2 sa detailed
show crypto ipsec sa
show crypto session detail
show crypto engine connections active
clear crypto sa
clear crypto ikev2 sa
show nve peers
show nve vni
show nve interface nve1 detail
show bgp l2vpn evpn summary
show l2vpn evpn evi
show l2route evpn mac
debug crypto ikev2
debug crypto ipsec


SD-WAN


! cEdge (IOS XE SD-WAN) system configuration
config-transaction
 system
  system-ip <SYSTEM_IP>
  site-id <SITE_ID>
  organization-name "<ORG_NAME>"
  vbond <VBOND_IP>
  host-name <HOSTNAME>
  admin-tech-on-failure
  sp-organization-name "<ORG_NAME>"
 !
 commit



! VPN 0 (transport / underlay)
config-transaction
 vrf definition 65528
 interface GigabitEthernet0/0/0
  no shutdown
  ip address <IP_ADDRESS> <MASK>
  exit
 sdwan
  interface GigabitEthernet0/0/0
   tunnel-interface
    encapsulation ipsec
    color mpls restrict
    allow-service all
    allow-service sshd
    allow-service netconf
    no allow-service bgp
   exit
  exit
 ip route 0.0.0.0 0.0.0.0 <NEXTHOPIP>
 commit



! Service VPN (VRF) and routing
config-transaction
 vrf definition 10
  rd 1:10
  address-family ipv4
   route-target export 1:10
   route-target import 1:10
   exit-address-family
 interface GigabitEthernet0/0/1
  vrf forwarding 10
  ip address <IP_ADDRESS> <MASK>
  no shutdown
 router ospf 10 vrf 10
  router-id <ROUTER_ID>
  network <IP_ADDRESS> <WILDCARD> area 0
 router bgp <AS_NUMBER>
  address-family ipv4 vrf 10
   neighbor <PEERIP> remote-as <REMOTEAS>
   neighbor <PEER_IP> activate
 commit



! VPN 512 (out-of-band management)
config-transaction
 vrf definition Mgmt-intf
 interface GigabitEthernet0
  vrf forwarding Mgmt-intf
  ip address <IP_ADDRESS> <MASK>
  no shutdown
 ip route vrf Mgmt-intf 0.0.0.0 0.0.0.0 <GATEWAY_IP>
 commit



! Localized policy / QoS / ACL on cEdge
config-transaction
 policy
  class-map VOICE
   match qos-group 0
  qos-map QOS-MAP
   qos-scheduler VOICE-SCHED
    class VOICE
    bandwidth-percent 20
    scheduling llq
    drops tail-drop
 commit



! Controller-side (vManage/vSmart CLI - viptela OS)
show control connections
show control local-properties
show control connections-history
show orchestrator connections
show omp peers
show omp routes
show omp tlocs
show running-config apply-policy



! cEdge verification
show sdwan running-config
show sdwan system status
show sdwan control connections
show sdwan control local-properties
show sdwan control connection-history
show sdwan omp peers
show sdwan omp routes
show sdwan omp tlocs
show sdwan omp summary
show sdwan bfd sessions
show sdwan bfd summary
show sdwan ipsec local-sa
show sdwan ipsec outbound-connections
show sdwan policy from-vsmart
show sdwan policy service-path vpn 10 interface <INTERFACE> source-ip <IPADDRESS> dest-ip <IPADDRESS> protocol 6 all
show sdwan tunnel statistics
show sdwan app-route stats
show sdwan certificate serial
show sdwan certificate installed
show sdwan reboot history
show sdwan software



! SD-WAN troubleshooting
request platform software sdwan admin-tech
request platform software sdwan port_hop color mpls
request platform software sdwan software activate <VERSION>
clear sdwan control connections
clear sdwan omp all
clear sdwan bfd transitions
debug platform software sdwan vdaemon all
monitor capture SDWANCAP interface <INTERFACE> both
show tech-support sdwan bfd


SD-Access


! Underlay (IS-IS example) and loopbacks
interface Loopback0
 ip address <ROUTER_ID> 255.255.255.255
 ip router isis
router isis
 net 49.0001.0100.0100.1001.00
 is-type level-2-only
 metric-style wide
 log-adjacency-changes
 nsf ietf
interface <INTERFACE>
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ip router isis
 isis network point-to-point
 ip pim sparse-mode



! Control Plane Node (LISP MS/MR)
router lisp
 locator-table default
 locator-set RLOC-SET
  IPv4-interface Loopback0 priority 10 weight 10
  exit-locator-set
 service ipv4
  encapsulation vxlan
  map-server
  map-resolver
  exit-service-ipv4
 service ethernet
  map-server
  map-resolver
  exit-service-ethernet
 site SITE-FABRIC
  authentication-key <LISPKEYPLACEHOLDER>
  eid-record any-mac
  eid-record instance-id 4099 <NETWORK>/<PREFIX> accept-more-specifics
  exit-site



! Fabric Edge Node
vrf definition CAMPUS
 rd 1:4099
 address-family ipv4
  route-target export 1:4099
  route-target import 1:4099

router lisp
 locator-set RLOC-EDGE
  IPv4-interface Loopback0 priority 10 weight 10
 instance-id 4099
  dynamic-eid DEFAULT-ETHER
   database-mapping <NETWORK>/<PREFIX> locator-set RLOC-EDGE
   exit-dynamic-eid
  service ipv4
   eid-table vrf CAMPUS
   map-cache 0.0.0.0/0 map-request
   itr map-resolver <CPNODEIP>
   etr map-server <CPNODEIP> key <LISPKEYPLACEHOLDER>
   itr
   etr
   exit-service-ipv4

interface LISP0
interface Vlan1021
 vrf forwarding CAMPUS
 ip address <ANYCASTGWIP> <MASK>
 lisp mobility DEFAULT-ETHER
 no ip redirects
 ip helper-address <DHCPSERVERIP>
 mac-address 0000.0c9f.f001



! Border Node (with BGP handoff)
router lisp
 instance-id 4099
  service ipv4
   eid-table vrf CAMPUS
   map-server
   map-resolver
   proxy-etr
   proxy-itr <RLOC_IP>
   route-export site-registrations
   distance site-registrations 250

router bgp <AS_NUMBER>
 address-family ipv4 vrf CAMPUS
  neighbor <FUSIONPEERIP> remote-as <REMOTE_AS>
  neighbor <FUSIONPEERIP> activate
  redistribute lisp metric 10



! SGT / TrustSec in fabric
cts authorization list SDA-LIST
cts role-based enforcement
cts credentials id <DEVICEID> password <CTSPASSWORD_PLACEHOLDER>
cts sxp enable
cts sxp connection peer <ISE_IP> password default mode local speaker



! Host onboarding authentication
aaa new-model
aaa authentication dot1x default group RAD-GRP
aaa authorization network default group RAD-GRP
dot1x system-auth-control
device-tracking policy IPDT-POLICY
 tracking enable
interface <INTERFACE>
 device-tracking attach-policy IPDT-POLICY
 access-session host-mode multi-auth
 access-session closed
 access-session port-control auto
 mab
 dot1x pae authenticator



! Verification
show lisp session
show lisp instance-id 4099 ipv4 summary
show lisp instance-id 4099 ipv4 database
show lisp instance-id 4099 ipv4 map-cache
show lisp instance-id 4099 ipv4 server
show lisp instance-id 4099 ipv4 server summary
show lisp instance-id 4099 ethernet database
show lisp site
show lisp site detail
show lisp dynamic-eid detail
show device-tracking database
show ip route vrf CAMPUS
show cts environment-data
show cts role-based sgt-map all
show cts role-based counters
show access-session interface <INTERFACE> details
show authentication sessions
show ip pim vrf CAMPUS neighbor
show isis neighbors
show bgp vpnv4 unicast all summary
debug lisp control-plane all


Network Programmability


! Enable NETCONF / RESTCONF on IOS XE
configure terminal
 netconf-yang
 netconf-yang feature candidate-datastore
 restconf
 ip http secure-server
 no ip http server
 ip http authentication local
 username <APIUSER> privilege 15 algorithm-type scrypt secret <SECRETPLACEHOLDER>
 aaa new-model
 aaa authentication login default local
 aaa authorization exec default local
end



! NETCONF tuning / verification
netconf-yang ssh port 830
show netconf-yang sessions
show netconf-yang sessions detail
show netconf-yang datastores
show netconf-yang statistics
show platform software yang-management process
show platform software yang-management process monitor
show restconf
show running-config | section netconf-yang
debug netconf-yang level debug



! gNMI / gRPC
configure terminal
 gnxi
 gnxi server
 gnxi secure-server
 gnxi secure-port 9339
 gnxi secure-trustpoint <TRUSTPOINT_NAME>
end
show gnxi state detail


NETCONF hello over SSH
ssh -s <APIUSER>@<IPADDRESS> -p 830 netconf

RESTCONF - capabilities
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/netconf-state/capabilities" \
  -H "Accept: application/yang-data+json"

RESTCONF - get all interfaces
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces" \
  -H "Accept: application/yang-data+json"

RESTCONF - get one interface
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces/interface=Loopback100" \
  -H "Accept: application/yang-data+json"

RESTCONF - create loopback (POST)
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X POST "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces" \
  -H "Content-Type: application/yang-data+json" \
  -H "Accept: application/yang-data+json" \
  -d '{
    "ietf-interfaces:interface": {
      "name": "Loopback100",
      "description": "Created via RESTCONF",
      "type": "iana-if-type:softwareLoopback",
      "enabled": true,
      "ietf-ip:ipv4": {
        "address": [{"ip": "<IP_ADDRESS>", "netmask": "<MASK>"}]
      }
    }
  }'

RESTCONF - replace (PUT) and delete (DELETE)
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X PUT "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces/interface=Loopback100" \
  -H "Content-Type: application/yang-data+json" \
  -d '{"ietf-interfaces:interface":{"name":"Loopback100","type":"iana-if-type:softwareLoopback","enabled":false}}'

curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X DELETE "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces/interface=Loopback100"

RESTCONF - native model and operational data
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-native:native/hostname" \
  -H "Accept: application/yang-data+json"

curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-interfaces-oper:interfaces" \
  -H "Accept: application/yang-data+json"

YANG tooling
pyang -f tree Cisco-IOS-XE-native.yang
pyang -f tree --tree-path=/native/interface ietf-interfaces.yang
pyang -f jstree -o model.html ietf-interfaces.yang
yanglint -f tree ietf-interfaces.yang

ncclient quick test
python3 -c "from ncclient import manager; m=manager.connect(host='<IPADDRESS>',port=830,username='<APIUSER>',password='<PASSWORDPLACEHOLDER>',hostkeyverify=False); print(m.getconfig('running').xml[:500]); m.closesession()"


! Guest Shell / on-box Python (IOS XE)
iox
app-hosting appid guestshell
guestshell enable
guestshell run bash
guestshell run python3
guestshell destroy
show iox-service
show app-hosting list



! EEM applet (on-box automation)
event manager applet LOG-INTF-DOWN
 event syslog pattern "Interface GigabitEthernet0/0/1, changed state to down"
 action 1.0 cli command "enable"
 action 2.0 cli command "show ip interface brief"
 action 3.0 syslog msg "Uplink down detected"
show event manager policy registered
show event manager history events


Automation

Python environment
python3 --version
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install netmiko napalm ncclient requests pyats genie paramiko nornir ansible-pylibssh
pip list
pip freeze > requirements.txt
python3 script.py

Netmiko one-liner
python3 -c "from netmiko import ConnectHandler; d={'devicetype':'ciscoxe','host':'<IPADDRESS>','username':'<USERNAME>','password':'<PASSWORDPLACEHOLDER>'}; c=ConnectHandler(d); print(c.send_command('show ip interface brief')); c.disconnect()"

pyATS / Genie
pyats version check
genie parse "show ip interface brief" --testbed-file testbed.yaml --devices <DEVICE_NAME>
genie learn ospf bgp interface --testbed-file testbed.yaml --output snapshot_before
genie diff snapshotbefore snapshotafter
pyats run job job.py --testbed-file testbed.yaml

Ansible
ansible --version
ansible-galaxy collection install cisco.ios
ansible-galaxy collection install cisco.iosxr
ansible-inventory -i inventory.yml --list
ansible all -i inventory.yml -m ping
ansible iosswitches -i inventory.yml -m cisco.ios.ioscommand -a "commands='show version'"
ansible-playbook -i inventory.yml playbook.yml
ansible-playbook -i inventory.yml playbook.yml --check --diff
ansible-playbook -i inventory.yml playbook.yml --limit <HOSTNAME> -vvv
ansible-playbook playbook.yml --tags vlan --skip-tags ospf
ansible-vault encrypt group_vars/all/vault.yml
ansible-vault view group_vars/all/vault.yml
ansible-lint playbook.yml

inventory.yml
all:
  children:
    ios_switches:
      hosts:
        SW1:
          ansiblehost: <IPADDRESS>
      vars:
        ansiblenetworkos: cisco.ios.ios
        ansibleconnection: ansible.netcommon.networkcli
        ansible_user: "<USERNAME>"
        ansiblepassword: "<PASSWORDPLACEHOLDER>"

playbook.yml
- name: Configure VLANs
  hosts: ios_switches
  gather_facts: false
  tasks:
    - name: Create VLAN
      cisco.ios.ios_vlans:
        config:
          - vlan_id: 10
            name: DATA
        state: merged

    - name: Save config
      cisco.ios.ios_config:
        save_when: modified

JSON / YAML tooling
python3 -m json.tool response.json
cat response.json | jq '.'
cat response.json | jq '."ietf-interfaces:interfaces".interface[].name'
python3 -c "import json,sys; print(json.load(open('data.json')))"
python3 -c "import yaml,json; print(json.dumps(yaml.safe_load(open('vars.yml')), indent=2))"
yamllint playbook.yml
yq '.all.children' inventory.yml

REST API testing
curl -k -X GET "https://<IPADDRESS>/restconf/data/Cisco-IOS-XE-native:native" -u <APIUSER>:<PASSWORD_PLACEHOLDER> -H "Accept: application/yang-data+json"
curl -k -i -X POST "https://<DNACIP>/dna/system/api/v1/auth/token" -u <APIUSER>:<PASSWORD_PLACEHOLDER>
curl -k -X GET "https://<DNACIP>/dna/intent/api/v1/network-device" -H "X-Auth-Token: <TOKENPLACEHOLDER>"
curl -k -X GET "https://<WLCIP>/restconf/data/Cisco-IOS-XE-wireless-access-point-oper:access-point-oper-data" -u <APIUSER>:<PASSWORD_PLACEHOLDER> -H "Accept: application/yang-data+json"
http --verify=no GET https://<IPADDRESS>/restconf/data/ietf-interfaces:interfaces --auth <APIUSER>:<PASSWORD_PLACEHOLDER>

Git
git init
git clone <REPO_URL>
git status
git add .
git commit -m "Add ENCOR configs"
git log --oneline --graph --decorate
git branch feature/ospf
git checkout -b feature/bgp
git switch main
git merge feature/ospf
git diff HEAD~1
git remote add origin <REPO_URL>
git push -u origin main
git pull --rebase
git stash
git stash pop
git revert <COMMIT_HASH>
git reset --hard <COMMIT_HASH>
git tag -a v1.0 -m "Baseline"

Docker
docker --version
docker pull python:3.12-slim
docker images
docker build -t netauto:1.0 .
docker run -it --rm -v $(pwd):/app netauto:1.0 bash
docker run -d --name ansible-runner netauto:1.0
docker ps -a
docker exec -it ansible-runner /bin/bash
docker logs -f ansible-runner
docker stop ansible-runner
docker rm ansible-runner
docker rmi netauto:1.0
docker network ls
docker network create --driver bridge netlab
docker compose up -d
docker compose down

dockerfile
Dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python3", "script.py"]


Monitoring and Telemetry


! SNMPv2c (read-only, restricted)
ip access-list standard ACL-SNMP
 permit <NMS_IP>
snmp-server community <ROCOMMUNITYPLACEHOLDER> RO ACL-SNMP
snmp-server location <LOCATION_TEXT>
snmp-server contact <CONTACT_TEXT>
snmp-server host <NMSIP> version 2c <ROCOMMUNITY_PLACEHOLDER>
snmp-server enable traps snmp linkdown linkup coldstart warmstart
snmp-server enable traps config
snmp-server enable traps cpu threshold
snmp-server source-interface informs Loopback0



! SNMPv3
snmp-server view VIEW-ALL iso included
snmp-server group SNMP-GRP v3 priv read VIEW-ALL access ACL-SNMP
snmp-server user <SNMPUSER> SNMP-GRP v3 auth sha <AUTHKEYPLACEHOLDER> priv aes 256 <PRIVKEY_PLACEHOLDER>
snmp-server host <NMSIP> version 3 priv <SNMPUSER>
show snmp
show snmp user
show snmp group
show snmp host



! Syslog
logging buffered 128000 informational
logging host <SYSLOG_IP> transport udp port 514
logging host <SYSLOG_IP2> transport tcp port 601
logging trap informational
logging facility local6
logging source-interface Loopback0
logging rate-limit 100 except errors
logging discriminator NOLINK severity drops 6 msg-body drops LINEPROTO
logging buffered discriminator NOLINK
service timestamps log datetime msec localtime show-timezone
show logging
clear logging



! Flexible NetFlow
flow record FR-IPV4
 match ipv4 source address
 match ipv4 destination address
 match ipv4 protocol
 match transport source-port
 match transport destination-port
 match ipv4 tos
 match interface input
 collect interface output
 collect counter bytes long
 collect counter packets long
 collect timestamp absolute first
 collect timestamp absolute last
 collect application name

flow exporter FE-COLLECTOR
 destination <COLLECTOR_IP>
 source Loopback0
 transport udp 2055
 template data timeout 60
 option interface-table
 option application-table

flow monitor FM-IPV4
 record FR-IPV4
 exporter FE-COLLECTOR
 cache timeout active 60
 cache timeout inactive 15

interface <INTERFACE>
 ip flow monitor FM-IPV4 input
 ip flow monitor FM-IPV4 output



! Flexible NetFlow verification
show flow record
show flow exporter
show flow exporter statistics
show flow monitor
show flow monitor FM-IPV4 cache
show flow monitor FM-IPV4 cache format table
show flow monitor FM-IPV4 statistics
show flow interface <INTERFACE>
clear flow monitor FM-IPV4 cache



! SPAN
monitor session 1 source interface GigabitEthernet1/0/1 - 2 both
monitor session 1 source vlan <VLAN_ID> rx
monitor session 1 destination interface GigabitEthernet1/0/48 encapsulation replicate
monitor session 1 filter vlan 10
no monitor session 1



! RSPAN
vlan 400
 name RSPAN-VLAN
 remote-span
monitor session 2 source interface GigabitEthernet1/0/3
monitor session 2 destination remote vlan 400



! ERSPAN
monitor session 3 type erspan-source
 source interface GigabitEthernet1/0/5 both
 no shutdown
 destination
  erspan-id 101
  mtu 1464
  ip address <ERSPANDESTIP>
  origin ip address <SOURCE_IP>

monitor session 4 type erspan-destination
 destination interface GigabitEthernet1/0/6
 no shutdown
 source
  erspan-id 101
  ip address <ERSPANDESTIP>

show monitor session all
show monitor session 3 detail
show capability feature monitor erspan-source



! Embedded packet capture
monitor capture CAP interface <INTERFACE> both
monitor capture CAP match ipv4 host <IP_ADDRESS> any
monitor capture CAP buffer size 10 circular
monitor capture CAP limit duration 60
monitor capture CAP start
monitor capture CAP stop
show monitor capture CAP buffer brief
monitor capture CAP export bootflash:cap.pcap
no monitor capture CAP



! Model-Driven Telemetry - dial-out (gRPC)
telemetry ietf subscription 101
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface/statistics
 stream yang-push
 update-policy periodic 3000
 receiver ip address <COLLECTOR_IP> 57500 protocol grpc-tcp
 source-address <IP_ADDRESS>

telemetry ietf subscription 102
 encoding encode-kvgpb
 filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
 stream yang-push
 update-policy on-change
 receiver ip address <COLLECTOR_IP> 57500 protocol grpc-tcp



! Telemetry verification
show telemetry ietf subscription all
show telemetry ietf subscription all brief
show telemetry ietf subscription 101 detail
show telemetry ietf subscription 101 receiver
show telemetry connection all
show telemetry internal connection
debug telemetry level debug



! Device performance monitoring
show processes cpu sorted 5min
show processes cpu history
show processes memory sorted
show platform resources
show platform hardware qfp active datapath utilization
show memory statistics
show interfaces | include rate|errors
show ip traffic
show snmp mib ifmib ifindex


High Availability


! StackWise (Catalyst 9000)
switch 1 priority 15
switch 2 priority 14
switch 1 renumber 2
switch 2 provision c9300-48p
stack-mac persistent timer 0
show switch
show switch detail
show switch stack-ports summary
show switch stack-ring speed
show platform stack-manager all
reload slot 2



! StackWise Virtual
stackwise-virtual
 domain 100
interface range TenGigabitEthernet1/0/1 - 2
 stackwise-virtual link 1
interface TenGigabitEthernet1/0/3
 stackwise-virtual dual-active-detection
exit
! Reload both switches to form SVL
show stackwise-virtual
show stackwise-virtual link
show stackwise-virtual neighbors
show stackwise-virtual dual-active-detection
show stackwise-virtual bandwidth



! Redundancy / SSO / NSF
redundancy
 mode sso
 main-cpu
  standby console enable

router ospf 1
 nsf ietf
 nsf cisco
router eigrp <AS_NUMBER>
 nsf
router bgp <AS_NUMBER>
 bgp graceful-restart
 bgp graceful-restart restart-time 120



! Redundancy verification
show redundancy
show redundancy states
show redundancy switchover history
show redundancy config-sync failures mcl
show platform software iomd redundancy
redundancy force-switchover
redundancy reload peer



! ISSU / software management (install mode)
show install summary
show install log
show version | include INSTALL|BOOT
install add file bootflash:cat9k_iosxe.<VERSION>.SPA.bin
install add file bootflash:cat9k_iosxe.<VERSION>.SPA.bin activate commit
install activate
install commit
install rollback to committed
install remove inactive
request platform software package install switch all file bootflash:<IMAGE> new auto-copy
issu loadversion
issu runversion
issu acceptversion
issu commitversion
show issu state detail



! Boot / config resilience
boot system flash bootflash:cat9k_iosxe.<VERSION>.SPA.bin
secure boot-image
secure boot-config
show secure bootset
show boot
show bootvar


Troubleshooting

27.1 Interfaces

show ip interface brief
show interfaces status err-disabled
show interfaces <INTERFACE>
show interfaces <INTERFACE> counters errors
show interfaces counters errors
show interfaces transceiver detail
show controllers ethernet-controller <INTERFACE> phy
show idprom interface <INTERFACE>
show logging | include LINK|LINEPROTO
clear counters <INTERFACE>
test cable-diagnostics tdr interface <INTERFACE>
show cable-diagnostics tdr interface <INTERFACE>
debug interface <INTERFACE>


27.2 VLANs

show vlan brief
show vlan id <VLAN_ID>
show interfaces <INTERFACE> switchport
show mac address-table vlan <VLAN_ID>
show vtp status
show spanning-tree vlan <VLAN_ID>
show running-config interface <INTERFACE>


27.3 Trunks

show interfaces trunk
show interfaces <INTERFACE> switchport
show dtp interface <INTERFACE>
show vlan brief
show cdp neighbors detail
debug sw-vlan packets


27.4 EtherChannel

show etherchannel summary
show etherchannel detail
show etherchannel port
show etherchannel load-balance
show lacp neighbor
show lacp internal
show lacp counters
show pagp neighbor
show interfaces <INTERFACE> etherchannel
show run interface <INTERFACE>
debug etherchannel
debug lacp all
clear lacp counters


27.5 STP

show spanning-tree summary
show spanning-tree root
show spanning-tree inconsistentports
show spanning-tree blockedports
show spanning-tree interface <INTERFACE> detail
show spanning-tree detail | include ieee|from|occurr
show spanning-tree mst configuration digest
show errdisable recovery
clear spanning-tree detected-protocols
debug spanning-tree events
debug spanning-tree bpdu


27.6 OSPF

show ip ospf neighbor
show ip ospf interface <INTERFACE>
show ip ospf interface brief
show ip protocols
show ip ospf database database-summary
show ip ospf events
show ip ospf statistics
show ip route ospf
clear ip ospf process
debug ip ospf adj
debug ip ospf hello
debug ip ospf packet


27.7 EIGRP

show ip eigrp neighbors
show ip eigrp interfaces detail
show ip eigrp topology all-links
show ip eigrp topology active
show ip protocols
show ip eigrp traffic
clear ip eigrp neighbors
debug eigrp packets hello
debug ip eigrp notifications


27.8 BGP

show ip bgp summary
show ip bgp neighbors <PEER_IP>
show ip bgp <NETWORK>/<PREFIX>
show ip bgp rib-failure
show bgp ipv4 unicast neighbors <PEER_IP> policy
show tcp brief all
clear ip bgp <PEER_IP> soft
debug ip bgp <PEER_IP> updates
debug ip bgp events


27.9 DHCP

show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip dhcp server statistics
show ip dhcp snooping binding
show ip interface <INTERFACE> | include Helper
debug ip dhcp server packet detail
debug ip dhcp server events
clear ip dhcp binding 


27.10 NAT

show ip nat translations verbose
show ip nat statistics
show ip nat pool
show ip interface <INTERFACE> | include NAT
clear ip nat translation 
debug ip nat detailed
debug ip packet <ACL_NUMBER> detail


27.11 HSRP / FHRP

show standby brief
show standby all
show standby <INTERFACE> <GROUP>
show vrrp brief
show glbp brief
show track
show logging | include HSRP|STANDBY|VRRP|GLBP
debug standby events
debug standby packets
debug vrrp state


27.12 ACLs

show access-lists
show ip access-lists <ACL_NAME>
show ipv6 access-list
show ip interface <INTERFACE> | include access list
show access-lists <ACL_NAME> | include matches
clear ip access-list counters <ACL_NAME>
show platform software access-list f0 summary
debug ip packet <ACL_NUMBER>


27.13 AAA

show aaa servers
show aaa sessions
show aaa method-lists all
show tacacs
show radius statistics
test aaa group RAD-GRP <USERNAME> <PASSWORD_PLACEHOLDER> new-code
debug aaa authentication
debug aaa authorization
debug aaa accounting
debug radius authentication
debug tacacs packet


27.14 Wireless

show wireless summary
show ap summary
show ap status
show wireless client summary
show wireless client mac-address <CLIENT_MAC> detail
show wlan summary
show wireless stats ap join summary
show wireless mobility summary
show aaa servers
debug wireless mac <CLIENT_MAC> monitor-time 600
show logging profile wireless filter mac <CLIENT_MAC>
show tech-support wireless


27.15 QoS

show policy-map interface <INTERFACE>
show policy-map interface <INTERFACE> output class CM-VOICE
show class-map
show table-map
show platform hardware qos interface <INTERFACE> output statistics
show ip nbar protocol-discovery interface <INTERFACE>
show interfaces <INTERFACE> | include drops|queue
clear counters <INTERFACE>


27.16 Multicast

show ip mroute
show ip mroute count
show ip mroute active
show ip pim neighbor
show ip pim rp mapping
show ip igmp groups
show ip igmp snooping groups
show ip rpf <SOURCE_IP>
mtrace <SOURCE_IP>
clear ip mroute 
debug ip pim
debug ip igmp
debug ip mpacket <MCAST_GROUP>


27.17 VRF

show vrf
show vrf detail
show ip route vrf <VRF_NAME>
show ip vrf interfaces
show ip bgp vpnv4 vrf <VRF_NAME>
ping vrf <VRFNAME> <IPADDRESS> source <INTERFACE>
traceroute vrf <VRFNAME> <IPADDRESS>
show ip cef vrf <VRFNAME> <IPADDRESS>


27.18 SD-WAN

show sdwan control connections
show sdwan control connection-history
show sdwan control local-properties
show sdwan omp peers
show sdwan omp routes
show sdwan bfd sessions
show sdwan ipsec outbound-connections
show sdwan policy from-vsmart
show sdwan tunnel statistics
show sdwan app-route stats
show sdwan certificate validity
clear sdwan control connections
request platform software sdwan admin-tech
debug platform software sdwan vdaemon all


27.19 SD-Access

show lisp session
show lisp instance-id <IID> ipv4 map-cache
show lisp instance-id <IID> ipv4 database
show lisp site detail
show device-tracking database
show access-session interface <INTERFACE> details
show cts role-based sgt-map all
show cts environment-data
show ip route vrf <VRF_NAME>
show nve peers
debug lisp control-plane all


27.20 Automation

show netconf-yang sessions detail
show netconf-yang statistics
show platform software yang-management process
show restconf
show telemetry ietf subscription all
show running-config | section restconf|netconf-yang
show ip http server status
show ip http server session-module
debug netconf-yang level debug
debug restconf


27.21 CPU / Memory

show processes cpu sorted
show processes cpu sorted 1min
show processes cpu history
show processes memory sorted
show memory statistics
show memory platform
show platform resources
show platform software status control-processor brief
show platform hardware qfp active datapath utilization
show buffers
show tcp statistics
monitor platform software process


27.22 Logs

show logging
show logging | include %
show logging last 100
show logging onboard uptime
show logging onboard temperature
show logging profile wireless
show archive log config all
show tech-support
show tech-support | redirect bootflash:tech.txt
clear logging


ENCOR Show Command Master List

28.1 System / Platform

show version
show inventory
show license summary
show license usage
show platform
show platform resources
show platform software status control-processor brief
show environment all
show processes cpu sorted
show processes memory sorted
show memory statistics
show redundancy
show switch detail
show stackwise-virtual
show boot
show install summary
show file systems
dir bootflash:
show tech-support


28.2 Layer 1 / Layer 2

show interfaces
show interfaces status
show interfaces description
show interfaces trunk
show interfaces counters errors
show interfaces transceiver detail
show vlan brief
show vtp status
show mac address-table
show cdp neighbors detail
show lldp neighbors detail
show etherchannel summary
show lacp neighbor
show pagp neighbor
show spanning-tree summary
show spanning-tree root
show spanning-tree inconsistentports
show spanning-tree mst configuration
show udld <INTERFACE>
show errdisable recovery


28.3 Layer 3 / Routing

show ip interface brief
show ipv6 interface brief
show ip route
show ipv6 route
show ip protocols
show ip cef
show ip arp
show ipv6 neighbors
show ip ospf neighbor
show ip ospf database
show ip ospf interface brief
show ipv6 ospf neighbor
show ip eigrp neighbors
show ip eigrp topology
show ip bgp summary
show ip bgp
show ip bgp neighbors <PEER_IP>
show route-map
show ip prefix-list
show ip sla statistics
show track
show vrf detail


28.4 Services

show ip nat translations
show ip nat statistics
show ip dhcp binding
show ip dhcp pool
show standby brief
show vrrp brief
show glbp brief
show ip mroute
show ip pim neighbor
show ip igmp snooping groups
show policy-map interface <INTERFACE>
show ip nbar protocol-discovery
show flow monitor <NAME> cache
show monitor session all


28.5 Security

show access-lists
show ip access-lists
show ipv6 access-list
show aaa servers
show aaa sessions
show authentication sessions
show access-session interface <INTERFACE> details
show dot1x all
show mab all
show port-security
show ip dhcp snooping binding
show ip arp inspection
show ip verify source
show cts role-based sgt-map all
show crypto ikev2 sa
show crypto ipsec sa
show crypto session
show policy-map control-plane
show ip ssh
show login failures


28.6 Wireless

show wireless summary
show wlan summary
show ap summary
show ap tag summary
show wireless client summary
show wireless client mac-address <CLIENT_MAC> detail
show wireless profile policy summary
show wireless mobility summary
show ap dot11 5ghz summary
show wireless stats client detail


28.7 Programmability / Telemetry

show netconf-yang sessions
show netconf-yang statistics
show restconf
show platform software yang-management process
show telemetry ietf subscription all
show telemetry connection all
show gnxi state detail
show event manager policy registered
show app-hosting list
show iox-service


28.8 SD-WAN / SD-Access

show sdwan control connections
show sdwan omp peers
show sdwan omp routes
show sdwan bfd sessions
show sdwan policy from-vsmart
show sdwan system status
show lisp session
show lisp site detail
show lisp instance-id <IID> ipv4 map-cache
show nve peers
show nve vni
show bgp l2vpn evpn summary


28.9 debug

debug ip ospf adj
debug ip ospf hello
debug eigrp packets
debug ip eigrp notifications
debug ip bgp updates
debug ip bgp events
debug ip nat detailed
debug ip dhcp server packet
debug standby events
debug vrrp state
debug glbp events
debug ip pim
debug ip igmp
debug ip packet <ACL_NUMBER> detail
debug spanning-tree events
debug etherchannel
debug lacp all
debug dot1x all
debug access-session all
debug aaa authentication
debug radius authentication
debug tacacs packet
debug crypto ikev2
debug crypto ipsec
debug ip policy
debug qos
debug netconf-yang level debug
debug telemetry level debug
debug wireless mac <CLIENT_MAC> monitor-time 600
debug capwap events
undebug all
no debug all
show debugging


28.10 clear

clear counters
clear counters <INTERFACE>
clear mac address-table dynamic
clear arp-cache
clear ip route 
clear ip ospf process
clear ip eigrp neighbors
clear ip bgp 
clear ip bgp <PEER_IP> soft in
clear ip nat translation 
clear ip dhcp binding 
clear ip dhcp conflict 
clear ip mroute 
clear ip igmp group
clear spanning-tree detected-protocols
clear lacp counters
clear logging
clear line vty 0
clear access-list counters <ACL_NAME>
clear crypto sa
clear access-session interface <INTERFACE>
clear flow monitor <NAME> cache
clear sdwan control connections


28.11 ping

ping <IP_ADDRESS>
ping <IP_ADDRESS> source <INTERFACE>
ping <IP_ADDRESS> repeat 100 size 1500 df-bit
ping <IP_ADDRESS> timeout 1 repeat 5
ping vrf <VRFNAME> <IPADDRESS>
ping ipv6 <IPV6_ADDRESS>
ping ipv6 <IPV6_ADDRESS> source <INTERFACE>
ping <MCAST_GROUP>
ping


28.12 traceroute

traceroute <IP_ADDRESS>
traceroute <IP_ADDRESS> source <INTERFACE>
traceroute <IP_ADDRESS> numeric probe 1 timeout 1
traceroute vrf <VRFNAME> <IPADDRESS>
traceroute ipv6 <IPV6_ADDRESS>
traceroute mac <SRCMAC> <DSTMAC>
traceroute mac ip <SRCIP> <DSTIP>
mtrace <SOURCEIP> <RECEIVERIP> <MCAST_GROUP>


28.13 test / misc exec

test aaa group RAD-GRP <USERNAME> <PASSWORD_PLACEHOLDER> new-code
test aaa group TAC-GRP <USERNAME> <PASSWORD_PLACEHOLDER> legacy
test cable-diagnostics tdr interface <INTERFACE>
test platform hardware fed switch active ...
terminal monitor
terminal length 0
terminal no monitor
send log "<MESSAGE>"
verify /md5 bootflash:<IMAGE>
copy running-config startup-config
copy running-config tftp://<IP_ADDRESS>/backup.cfg
archive config
show archive


Complete Configuration Examples

29.1 Enterprise Layer 2 Switch

hostname ACCESS-SW1
no ip domain lookup
ip domain name <DOMAIN_NAME>
enable secret <SECRET_PLACEHOLDER>
username <USERNAME> privilege 15 algorithm-type scrypt secret <SECRET_PLACEHOLDER>
service password-encryption
service timestamps log datetime msec localtime show-timezone
!
crypto key generate rsa modulus 2048
ip ssh version 2
!
vtp domain <VTP_DOMAIN>
vtp mode transparent
!
vlan 10
 name DATA
vlan 20
 name VOICE
vlan 30
 name MGMT
vlan 99
 name NATIVE
vlan 999
 name PARKING
!
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
spanning-tree loopguard default
udld enable
errdisable recovery cause bpduguard
errdisable recovery cause psecure-violation
errdisable recovery interval 300
!
ip dhcp snooping
ip dhcp snooping vlan 10,20
no ip dhcp snooping information option
ip arp inspection vlan 10,20
ip arp inspection validate src-mac dst-mac ip
!
interface range GigabitEthernet1/0/1 - 44
 description USER-ACCESS
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 switchport nonegotiate
 switchport port-security
 switchport port-security maximum 3
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 storm-control broadcast level pps 1k
 storm-control action trap
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
!
interface range GigabitEthernet1/0/45 - 46
 description UNUSED
 switchport mode access
 switchport access vlan 999
 shutdown
!
interface range TenGigabitEthernet1/1/1 - 2
 description UPLINK-TO-DIST
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 switchport nonegotiate
 channel-group 1 mode active
 ip dhcp snooping trust
 ip arp inspection trust
 udld port aggressive
 no shutdown
!
interface Port-channel1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 spanning-tree guard loop
!
interface Vlan30
 description MGMT-SVI
 ip address <IP_ADDRESS> <MASK>
 no shutdown
!
ip default-gateway <GATEWAY_IP>
!
ip access-list standard ACL-MGMT
 permit 10.30.0.0 0.0.255.255
 deny any log
line console 0
 exec-timeout 5 0
 logging synchronous
 login local
line vty 0 15
 exec-timeout 10 0
 access-class ACL-MGMT in
 transport input ssh
 login local
!
ntp server <IP_ADDRESS> prefer
logging host <SYSLOG_IP>
logging trap informational
end


29.2 Layer 3 Switch

hostname DIST-SW1
ip routing
ipv6 unicast-routing
!
vlan 10,20,30,99
!
spanning-tree mode rapid-pvst
spanning-tree vlan 1-999 root primary
!
interface Port-channel1
 description TO-ACCESS-SW1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
!
interface range TenGigabitEthernet1/0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
!
interface TenGigabitEthernet1/0/47
 description L3-TO-CORE
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ip ospf network point-to-point
 ip ospf 1 area 0
 no shutdown
!
interface Loopback0
 ip address <ROUTER_ID> 255.255.255.255
 ip ospf 1 area 0
!
interface Vlan10
 description DATA-GW
 ip address <IP_ADDRESS> <MASK>
 ip helper-address <DHCPSERVERIP>
 ipv6 address <IPV6_PREFIX>::1/64
 standby version 2
 standby 10 ip <VIRTUAL_IP>
 standby 10 priority 110
 standby 10 preempt delay minimum 60
 no shutdown
!
interface Vlan20
 description VOICE-GW
 ip address <IP_ADDRESS> <MASK>
 ip helper-address <DHCPSERVERIP>
 standby version 2
 standby 20 ip <VIRTUAL_IP>
 standby 20 priority 110
 standby 20 preempt
 no shutdown
!
interface Vlan30
 description MGMT-GW
 ip address <IP_ADDRESS> <MASK>
 no shutdown
!
router ospf 1
 router-id <ROUTER_ID>
 passive-interface default
 no passive-interface TenGigabitEthernet1/0/47
 auto-cost reference-bandwidth 100000
 network 10.0.0.0 0.255.255.255 area 0
end


29.3 OSPF Enterprise Network

! --- CORE / ABR (R1) ---
hostname R1-ABR
ip routing
!
interface Loopback0
 ip address 10.255.0.1 255.255.255.255
!
key chain OSPF-KC
 key 1
  key-string <KEY_PLACEHOLDER>
  cryptographic-algorithm hmac-sha-256
!
interface GigabitEthernet0/0/0
 description AREA0-CORE
 ip address <IP_ADDRESS> <MASK>
 ip ospf 1 area 0
 ip ospf network point-to-point
 ip ospf authentication key-chain OSPF-KC
 ip ospf cost 10
 no shutdown
!
interface GigabitEthernet0/0/1
 description AREA1-BRANCH
 ip address <IP_ADDRESS> <MASK>
 ip ospf 1 area 1
 ip ospf network point-to-point
 ip ospf authentication key-chain OSPF-KC
 ip ospf hello-interval 5
 ip ospf dead-interval 20
 no shutdown
!
router ospf 1
 router-id 10.255.0.1
 auto-cost reference-bandwidth 100000
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
 no passive-interface GigabitEthernet0/0/1
 area 1 stub no-summary
 area 1 range <IP_ADDRESS> <MASK>
 area 2 nssa default-information-originate
 default-information originate always
 maximum-paths 4
 log-adjacency-changes detail
 nsf ietf
!
! --- BRANCH (R2, totally stubby area 1) ---
hostname R2-BRANCH
interface Loopback0
 ip address 10.255.0.2 255.255.255.255
 ip ospf 1 area 1
router ospf 1
 router-id 10.255.0.2
 area 1 stub
 passive-interface default
 no passive-interface GigabitEthernet0/0/0
!
! --- OSPFv3 overlay on both ---
ipv6 unicast-routing
router ospfv3 1
 router-id 10.255.0.1
 address-family ipv6 unicast
  passive-interface default
  no passive-interface GigabitEthernet0/0/0
  exit-address-family
interface GigabitEthernet0/0/0
 ipv6 address <IPV6_PREFIX>::1/64
 ospfv3 1 ipv6 area 0
 ospfv3 network point-to-point
!
! Verify
! show ip ospf neighbor
! show ip ospf database
! show ip route ospf


29.4 EIGRP Enterprise Network

hostname R1-EIGRP
ip routing
!
interface Loopback0
 ip address 10.255.0.1 255.255.255.255
!
key chain EIGRP-KC
 key 1
  key-string <KEY_PLACEHOLDER>
  cryptographic-algorithm hmac-sha-256
!
router eigrp ENTERPRISE
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 10.255.0.1
  network 10.0.0.0 0.255.255.255
  af-interface default
   passive-interface
   exit-af-interface
  af-interface GigabitEthernet0/0/0
   no passive-interface
   hello-interval 5
   hold-time 15
   authentication mode hmac-sha-256 <KEY_PLACEHOLDER>
   exit-af-interface
  af-interface GigabitEthernet0/0/1
   no passive-interface
   authentication mode hmac-sha-256 <KEY_PLACEHOLDER>
   summary-address 10.10.0.0 255.255.0.0
   exit-af-interface
  topology base
   variance 4
   maximum-paths 6
   redistribute static
   exit-af-topology
  exit-address-family
 !
 address-family ipv6 unicast autonomous-system 100
  eigrp router-id 10.255.0.1
  af-interface default
   passive-interface
   exit-af-interface
  af-interface GigabitEthernet0/0/0
   no passive-interface
   exit-af-interface
  exit-address-family
!
interface GigabitEthernet0/0/0
 ip address <IP_ADDRESS> <MASK>
 bandwidth 100000
 delay 100
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 <NEXTHOPIP>
!
! --- Branch stub router ---
hostname R9-BRANCH
router eigrp ENTERPRISE
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 10.255.0.9
  eigrp stub connected summary
  network 10.0.0.0 0.255.255.255
  exit-address-family
!
! Verify
! show ip eigrp neighbors
! show ip eigrp topology
! show ip route eigrp


29.5 BGP Enterprise Network

! --- Edge router R1: eBGP to ISP + iBGP to R2 ---
hostname R1-EDGE
!
interface Loopback0
 ip address 10.255.0.1 255.255.255.255
interface GigabitEthernet0/0/0
 description TO-ISP-A
 ip address <PUBLIC_IP> <MASK>
 no shutdown
!
ip prefix-list PL-OUR-PREFIX seq 5 permit 203.0.113.0/24
ip prefix-list PL-ISP-IN seq 5 permit 0.0.0.0/0
ip prefix-list PL-ISP-IN seq 10 deny 0.0.0.0/0 le 32
!
ip as-path access-list 1 permit ^$
!
route-map RM-ISPA-IN permit 10
 match ip address prefix-list PL-ISP-IN
 set local-preference 200
!
route-map RM-ISPA-OUT permit 10
 match ip address prefix-list PL-OUR-PREFIX
 set community 65001:100 additive
route-map RM-ISPA-OUT deny 20
!
router bgp 65001
 bgp router-id 10.255.0.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 bgp deterministic-med
 bgp graceful-restart
 !
 neighbor <ISPPEERIP> remote-as 64512
 neighbor <ISPPEERIP> description eBGP-ISP-A
 neighbor <ISPPEERIP> password <BGPKEYPLACEHOLDER>
 neighbor <ISPPEERIP> fall-over bfd
 !
 neighbor IBGP peer-group
 neighbor IBGP remote-as 65001
 neighbor IBGP update-source Loopback0
 neighbor 10.255.0.2 peer-group IBGP
 !
 address-family ipv4 unicast
  network 203.0.113.0 mask 255.255.255.0
  neighbor <ISPPEERIP> activate
  neighbor <ISPPEERIP> soft-reconfiguration inbound
  neighbor <ISPPEERIP> route-map RM-ISPA-IN in
  neighbor <ISPPEERIP> route-map RM-ISPA-OUT out
  neighbor <ISPPEERIP> maximum-prefix 500000 85 restart 30
  neighbor <ISPPEERIP> send-community both
  neighbor IBGP activate
  neighbor IBGP next-hop-self all
  neighbor IBGP send-community both
  exit-address-family
!
! --- Edge router R2: backup ISP with AS prepend ---
hostname R2-EDGE
route-map RM-ISPB-OUT permit 10
 match ip address prefix-list PL-OUR-PREFIX
 set as-path prepend 65001 65001 65001
route-map RM-ISPB-IN permit 10
 set local-preference 100
!
router bgp 65001
 bgp router-id 10.255.0.2
 no bgp default ipv4-unicast
 neighbor <ISPBPEER_IP> remote-as 64513
 neighbor 10.255.0.1 remote-as 65001
 neighbor 10.255.0.1 update-source Loopback0
 address-family ipv4 unicast
  network 203.0.113.0 mask 255.255.255.0
  neighbor <ISPBPEER_IP> activate
  neighbor <ISPBPEER_IP> route-map RM-ISPB-IN in
  neighbor <ISPBPEER_IP> route-map RM-ISPB-OUT out
  neighbor 10.255.0.1 activate
  neighbor 10.255.0.1 next-hop-self
  exit-address-family
!
! Verify
! show ip bgp summary
! show ip bgp
! show ip bgp neighbors <ISPPEERIP> advertised-routes


29.6 HSRP Redundant Gateways

! --- DIST-SW1 (Active VLAN10 / Standby VLAN20) ---
hostname DIST-SW1
ip routing
track 1 interface TenGigabitEthernet1/0/47 line-protocol
track 2 ip route 0.0.0.0 0.0.0.0 reachability
!
interface Vlan10
 ip address 10.10.10.2 255.255.255.0
 ip helper-address <DHCPSERVERIP>
 standby version 2
 standby 10 ip 10.10.10.1
 standby 10 priority 110
 standby 10 preempt delay minimum 90
 standby 10 timers msec 250 msec 750
 standby 10 authentication md5 key-string <KEY_PLACEHOLDER>
 standby 10 name HSRP-V10
 standby 10 track 1 decrement 20
 standby 10 track 2 decrement 20
 no shutdown
!
interface Vlan20
 ip address 10.10.20.2 255.255.255.0
 standby version 2
 standby 20 ip 10.10.20.1
 standby 20 priority 90
 standby 20 preempt
 no shutdown
!
! --- DIST-SW2 (Standby VLAN10 / Active VLAN20) ---
hostname DIST-SW2
ip routing
track 1 interface TenGigabitEthernet1/0/47 line-protocol
!
interface Vlan10
 ip address 10.10.10.3 255.255.255.0
 ip helper-address <DHCPSERVERIP>
 standby version 2
 standby 10 ip 10.10.10.1
 standby 10 priority 90
 standby 10 preempt
 standby 10 authentication md5 key-string <KEY_PLACEHOLDER>
 no shutdown
!
interface Vlan20
 ip address 10.10.20.3 255.255.255.0
 standby version 2
 standby 20 ip 10.10.20.1
 standby 20 priority 110
 standby 20 preempt delay minimum 90
 standby 20 track 1 decrement 30
 no shutdown
!
! Verify
! show standby brief
! show track


29.7 EtherChannel

! --- Layer 2 LACP trunk bundle ---
interface range TenGigabitEthernet1/0/1 - 2
 description PO1-MEMBER
 switchport
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 switchport nonegotiate
 channel-protocol lacp
 channel-group 1 mode active
 lacp rate fast
 no shutdown
!
interface Port-channel1
 description UPLINK-BUNDLE
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 spanning-tree guard loop
!
port-channel load-balance src-dst-ip
lacp system-priority 100
!
! --- Layer 3 LACP bundle ---
interface range TenGigabitEthernet1/0/3 - 4
 no switchport
 channel-group 10 mode active
 no shutdown
interface Port-channel10
 description L3-CORE-BUNDLE
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ip ospf 1 area 0
 ip ospf network point-to-point
 no shutdown
!
! Verify
! show etherchannel summary
! show lacp neighbor
! show interfaces port-channel1 etherchannel


29.8 DHCP Server

hostname DHCP-RTR
!
ip dhcp excluded-address 10.10.10.1 10.10.10.20
ip dhcp excluded-address 10.10.20.1 10.10.20.20
ip dhcp excluded-address 10.10.30.1 10.10.30.10
!
ip dhcp pool VLAN10-DATA
 network 10.10.10.0 255.255.255.0
 default-router 10.10.10.1
 dns-server <DNSIP1> <DNSIP2>
 domain-name <DOMAIN_NAME>
 lease 7 0 0
!
ip dhcp pool VLAN20-VOICE
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server <DNSIP1>
 option 150 ip <TFTPSERVERIP>
 lease 0 12 0
!
ip dhcp pool AP-POOL
 network 10.10.30.0 255.255.255.0
 default-router 10.10.30.1
 option 43 hex f104.0a0a.1e05
 lease 8
!
ip dhcp pool PRINTER-RESERVED
 host 10.10.10.50 255.255.255.0
 hardware-address 0011.2233.4455
 default-router 10.10.10.1
 dns-server <DNSIP1>
!
! Relay on remote SVIs
interface Vlan40
 ip address 10.10.40.1 255.255.255.0
 ip helper-address 10.10.10.1
!
! Verify
! show ip dhcp binding
! show ip dhcp pool
! show ip dhcp server statistics


29.9 NAT / PAT

hostname EDGE-RTR
!
interface GigabitEthernet0/0/0
 description OUTSIDE-ISP
 ip address <PUBLIC_IP> <MASK>
 ip nat outside
 no shutdown
!
interface GigabitEthernet0/0/1
 description INSIDE-LAN
 ip address 10.10.10.1 255.255.255.0
 ip nat inside
 no shutdown
!
ip access-list standard ACL-NAT-INSIDE
 permit 10.10.0.0 0.0.255.255
!
ip nat pool PUBLIC-POOL <STARTIP> <ENDIP> netmask <MASK>
!
! PAT to interface for general users
ip nat inside source list ACL-NAT-INSIDE interface GigabitEthernet0/0/0 overload
!
! Dynamic NAT pool (alternate)
! ip nat inside source list ACL-NAT-INSIDE pool PUBLIC-POOL overload
!
! Static NAT for published servers
ip nat inside source static 10.10.10.50 <PUBLICSERVERIP>
ip nat inside source static tcp 10.10.10.60 443 <PUBLIC_IP> 443 extendable
ip nat inside source static tcp 10.10.10.61 25 <PUBLIC_IP> 25 extendable
!
ip nat translation timeout 3600
ip nat translation tcp-timeout 3600
ip nat translation udp-timeout 300
!
ip route 0.0.0.0 0.0.0.0 <ISPNEXTHOP>
!
! Verify
! show ip nat translations
! show ip nat statistics
! debug ip nat detailed


29.10 AAA with TACACS+

hostname CORE-RTR
!
username <BREAKGLASSUSER> privilege 15 algorithm-type scrypt secret <SECRETPLACEHOLDER>
enable secret <SECRET_PLACEHOLDER>
!
aaa new-model
aaa session-id common
!
tacacs server TAC-1
 address ipv4 <TACACSIP1>
 key <TACACSKEYPLACEHOLDER>
 timeout 5
 single-connection
tacacs server TAC-2
 address ipv4 <TACACSIP2>
 key <TACACSKEYPLACEHOLDER>
 timeout 5
!
aaa group server tacacs+ TAC-GRP
 server name TAC-1
 server name TAC-2
 ip tacacs source-interface Loopback0
!
aaa authentication login default group TAC-GRP local
aaa authentication login CONSOLE-AUTH local
aaa authentication enable default group TAC-GRP enable
aaa authorization config-commands
aaa authorization exec default group TAC-GRP local if-authenticated
aaa authorization commands 1 default group TAC-GRP local if-authenticated
aaa authorization commands 15 default group TAC-GRP local if-authenticated
aaa accounting exec default start-stop group TAC-GRP
aaa accounting commands 1 default start-stop group TAC-GRP
aaa accounting commands 15 default start-stop group TAC-GRP
aaa accounting system default start-stop group TAC-GRP
!
line console 0
 exec-timeout 5 0
 login authentication CONSOLE-AUTH
line vty 0 15
 exec-timeout 10 0
 login authentication default
 authorization exec default
 authorization commands 15 default
 accounting commands 15 default
 transport input ssh
!
login block-for 120 attempts 3 within 60
login on-failure log
login on-success log
!
! Verify
! show aaa servers
! show tacacs
! test aaa group TAC-GRP <USERNAME> <PASSWORD_PLACEHOLDER> legacy


29.11 Secure Cisco Router

hostname SEC-RTR
!
no ip domain lookup
ip domain name <DOMAIN_NAME>
enable secret <SECRET_PLACEHOLDER>
username <ADMINUSER> privilege 15 algorithm-type scrypt secret <SECRETPLACEHOLDER>
service password-encryption
service tcp-keepalives-in
service tcp-keepalives-out
service timestamps debug datetime msec localtime show-timezone
service timestamps log datetime msec localtime show-timezone
no service pad
no service config
!
no ip http server
ip http secure-server
ip http authentication aaa
no ip source-route
no ip bootp server
no cdp run
!
crypto key generate rsa modulus 4096
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 2
ip ssh dh min size 2048
ip ssh source-interface Loopback0
!
banner motd ^C Authorized access only. Activity is monitored. ^C
!
ip access-list standard ACL-MGMT
 permit 10.30.0.0 0.0.255.255
 deny any log
!
line console 0
 exec-timeout 5 0
 logging synchronous
 login local
line aux 0
 no exec
 transport input none
line vty 0 15
 exec-timeout 10 0
 access-class ACL-MGMT in
 transport input ssh
 transport output ssh
 login local
!
login block-for 300 attempts 3 within 60
login quiet-mode access-class ACL-MGMT
login on-failure log
login on-success log
!
archive
 log config
  logging enable
  hidekeys
  notify syslog contenttype plaintext
 path bootflash:archive/config
 maximum 10
 write-memory
!
! CoPP
ip access-list extended ACL-CoPP-MGMT
 permit tcp 10.30.0.0 0.0.255.255 any eq 22
 permit udp host <NMS_IP> any eq snmp
class-map match-all CM-CoPP-MGMT
 match access-group name ACL-CoPP-MGMT
class-map match-all CM-CoPP-ICMP
 match protocol icmp
policy-map PM-CoPP
 class CM-CoPP-MGMT
  police 256000 conform-action transmit exceed-action drop
 class CM-CoPP-ICMP
  police 32000 conform-action transmit exceed-action drop
 class class-default
  police 1000000 conform-action transmit exceed-action drop
control-plane
 service-policy input PM-CoPP
!
ntp authenticate
ntp authentication-key 1 md5 <NTPKEYPLACEHOLDER>
ntp trusted-key 1
ntp server <IP_ADDRESS> key 1 prefer
!
logging host <SYSLOG_IP>
logging trap informational
logging source-interface Loopback0
!
snmp-server view VIEW-ALL iso included
snmp-server group SNMP-GRP v3 priv read VIEW-ALL access ACL-MGMT
snmp-server user <SNMPUSER> SNMP-GRP v3 auth sha <AUTHKEYPLACEHOLDER> priv aes 256 <PRIVKEY_PLACEHOLDER>
end


29.12 QoS Policy

hostname WAN-RTR
!
ip nbar protocol-discovery
!
ip access-list extended ACL-BUSINESS-APP
 permit tcp any any eq 1521
 permit tcp any host <IP_ADDRESS> eq 443
!
class-map match-any CM-VOICE
 match dscp ef
 match protocol rtp audio
class-map match-any CM-VIDEO
 match dscp af41 af42 af43
class-map match-all CM-SIGNALING
 match dscp cs3
class-map match-any CM-TRANSACTIONAL
 match access-group name ACL-BUSINESS-APP
 match protocol attribute traffic-class transactional-data
class-map match-any CM-SCAVENGER
 match dscp cs1
 match protocol attribute business-relevance business-irrelevant
!
policy-map PM-INGRESS-MARK
 class CM-VOICE
  set dscp ef
 class CM-VIDEO
  set dscp af41
 class CM-SIGNALING
  set dscp cs3
 class CM-TRANSACTIONAL
  set dscp af21
 class CM-SCAVENGER
  set dscp cs1
 class class-default
  set dscp default
!
policy-map PM-WAN-CHILD
 class CM-VOICE
  priority level 1 percent 10
 class CM-VIDEO
  bandwidth remaining percent 25
  random-detect dscp-based
 class CM-SIGNALING
  bandwidth remaining percent 5
 class CM-TRANSACTIONAL
  bandwidth remaining percent 35
  fair-queue
  random-detect dscp-based
 class CM-SCAVENGER
  bandwidth remaining percent 1
  police cir 2000000 conform-action transmit exceed-action drop
 class class-default
  bandwidth remaining percent 34
  fair-queue
  random-detect
!
policy-map PM-WAN-PARENT
 class class-default
  shape average 50000000
  service-policy PM-WAN-CHILD
!
interface GigabitEthernet0/0/0
 description WAN-50Mbps
 bandwidth 50000
 service-policy input PM-INGRESS-MARK
 service-policy output PM-WAN-PARENT
!
! Verify
! show policy-map interface GigabitEthernet0/0/0
! show ip nbar protocol-discovery interface GigabitEthernet0/0/0 top-n 10


29.13 VRF

hostname VRF-RTR
ip routing
ipv6 unicast-routing
!
vrf definition CORP
 rd 65001:100
 address-family ipv4
  route-target export 65001:100
  route-target import 65001:100
  route-target import 65001:300
  exit-address-family
 address-family ipv6
  exit-address-family
!
vrf definition GUEST
 rd 65001:200
 address-family ipv4
  route-target export 65001:200
  route-target import 65001:200
  exit-address-family
!
vrf definition SHARED
 rd 65001:300
 address-family ipv4
  route-target export 65001:300
  route-target import 65001:100
  route-target import 65001:200
  exit-address-family
!
interface GigabitEthernet0/0/1.100
 encapsulation dot1Q 100
 vrf forwarding CORP
 ip address 10.100.0.1 255.255.255.0
 ipv6 address <IPV6_PREFIX>::1/64
!
interface GigabitEthernet0/0/1.200
 encapsulation dot1Q 200
 vrf forwarding GUEST
 ip address 10.200.0.1 255.255.255.0
!
interface GigabitEthernet0/0/1.300
 encapsulation dot1Q 300
 vrf forwarding SHARED
 ip address 10.30.0.1 255.255.255.0
!
router ospf 100 vrf CORP
 router-id 10.255.0.1
 network 10.100.0.0 0.0.255.255 area 0
!
router bgp 65001
 bgp router-id 10.255.0.1
 address-family ipv4 vrf CORP
  redistribute ospf 100
  redistribute connected
  exit-address-family
 address-family ipv4 vrf GUEST
  redistribute connected
  exit-address-family
 address-family ipv4 vrf SHARED
  redistribute connected
  exit-address-family
!
ip route vrf GUEST 0.0.0.0 0.0.0.0 <ISPNEXTHOP> global
ip route vrf CORP 10.30.0.0 255.255.255.0 GigabitEthernet0/0/1.300
!
! Verify
! show vrf detail
! show ip route vrf CORP
! ping vrf CORP 10.100.0.10 source GigabitEthernet0/0/1.100


29.14 Multicast

! --- RP router ---
hostname RP-RTR
ip multicast-routing distributed
!
interface Loopback0
 ip address 10.255.0.1 255.255.255.255
 ip pim sparse-mode
 ip ospf 1 area 0
!
ip access-list standard ACL-MCAST-GROUPS
 permit 239.0.0.0 0.255.255.255
!
ip pim rp-address 10.255.0.1 ACL-MCAST-GROUPS override
ip pim send-rp-announce Loopback0 scope 16 group-list ACL-MCAST-GROUPS
ip pim send-rp-discovery Loopback0 scope 16
ip pim autorp listener
no ip pim dm-fallback
!
interface GigabitEthernet0/0/0
 ip address <IP_ADDRESS> <MASK>
 ip pim sparse-mode
 ip ospf 1 area 0
 no shutdown
!
! --- Downstream L3 switch ---
hostname L3-SW
ip multicast-routing distributed
ip pim rp-address 10.255.0.1 ACL-MCAST-GROUPS override
ip pim autorp listener
!
interface Vlan10
 ip address 10.10.10.1 255.255.255.0
 ip pim sparse-mode
 ip igmp version 3
 no shutdown
!
interface TenGigabitEthernet1/0/47
 no switchport
 ip address <IP_ADDRESS> <MASK>
 ip pim sparse-mode
 no shutdown
!
ip igmp snooping
ip igmp snooping vlan 10
ip igmp snooping querier
!
! Verify
! show ip mroute
! show ip pim rp mapping
! show ip pim neighbor
! show ip igmp snooping groups
! show ip rpf <SOURCE_IP>


29.15 Network Automation using RESTCONF

! --- Device enablement ---
configure terminal
 aaa new-model
 aaa authentication login default local
 aaa authorization exec default local
 username <APIUSER> privilege 15 algorithm-type scrypt secret <SECRETPLACEHOLDER>
 ip domain name <DOMAIN_NAME>
 crypto key generate rsa modulus 2048
 ip http secure-server
 no ip http server
 ip http authentication local
 ip http max-connections 16
 restconf
 netconf-yang
 netconf-yang feature candidate-datastore
end
write memory
!
! Verify
show restconf
show netconf-yang sessions
show platform software yang-management process
show ip http server status


--- Read hostname (native model) ---
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-native:native/hostname" \
  -H "Accept: application/yang-data+json"

--- Create a loopback interface ---
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X POST "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces" \
  -H "Content-Type: application/yang-data+json" \
  -H "Accept: application/yang-data+json" \
  -d '{
    "ietf-interfaces:interface": {
      "name": "Loopback101",
      "description": "RESTCONF automated loopback",
      "type": "iana-if-type:softwareLoopback",
      "enabled": true,
      "ietf-ip:ipv4": {
        "address": [{ "ip": "10.255.101.1", "netmask": "255.255.255.255" }]
      },
      "ietf-ip:ipv6": {}
    }
  }'

--- Configure an OSPF process (native model) ---
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X PUT "https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-native:native/router/Cisco-IOS-XE-ospf:router-ospf/ospf/process-id=1" \
  -H "Content-Type: application/yang-data+json" \
  -d '{
    "Cisco-IOS-XE-ospf:process-id": {
      "id": 1,
      "router-id": "<ROUTER_ID>",
      "network": [
        { "ip": "10.0.0.0", "wildcard": "0.255.255.255", "area": 0 }
      ]
    }
  }'

--- Read operational interface statistics ---
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X GET "https://<IP_ADDRESS>/restconf/data/Cisco-IOS-XE-interfaces-oper:interfaces/interface=GigabitEthernet1/statistics" \
  -H "Accept: application/yang-data+json" | jq '.'

--- Delete the loopback ---
curl -k -u <APIUSER>:<PASSWORDPLACEHOLDER> \
  -X DELETE "https://<IP_ADDRESS>/restconf/data/ietf-interfaces:interfaces/interface=Loopback101"

restconf_loopback.py
import json
import requests
from requests.auth import HTTPBasicAuth
from urllib3 import disable_warnings
from urllib3.exceptions import InsecureRequestWarning

disable_warnings(InsecureRequestWarning)

HOST = "<IP_ADDRESS>"
USER = "<API_USER>"
PASS = "<PASSWORD_PLACEHOLDER>"
BASE = f"https://{HOST}/restconf/data"
HEADERS = {
    "Accept": "application/yang-data+json",
    "Content-Type": "application/yang-data+json",
}

payload = {
    "ietf-interfaces:interface": {
        "name": "Loopback102",
        "description": "Created by Python RESTCONF",
        "type": "iana-if-type:softwareLoopback",
        "enabled": True,
        "ietf-ip:ipv4": {
            "address": [{"ip": "10.255.102.1", "netmask": "255.255.255.255"}]
        },
    }
}

r = requests.put(
    f"{BASE}/ietf-interfaces:interfaces/interface=Loopback102",
    auth=HTTPBasicAuth(USER, PASS),
    headers=HEADERS,
    data=json.dumps(payload),
    verify=False,
)
print(r.status_code)

r = requests.get(
    f"{BASE}/ietf-interfaces:interfaces",
    auth=HTTPBasicAuth(USER, PASS),
    headers=HEADERS,
    verify=False,
)
print(json.dumps(r.json(), indent=2))

python3 restconf_loopback.py

restconf_playbook.yml
- name: Configure loopback via RESTCONF
  hosts: localhost
  gather_facts: false
  vars:
    devicehost: "<IPADDRESS>"
    deviceuser: "<APIUSER>"
    devicepass: "<PASSWORDPLACEHOLDER>"
  tasks:
    - name: Create Loopback103
      ansible.builtin.uri:
        url: "https://{{ device_host }}/restconf/data/ietf-interfaces:interfaces/interface=Loopback103"
        method: PUT
        user: "{{ device_user }}"
        password: "{{ device_pass }}"
        forcebasicauth: true
        validate_certs: false
        status_code: [201, 204]
        headers:
          Accept: application/yang-data+json
          Content-Type: application/yang-data+json
        body_format: json
        body:
          ietf-interfaces:interface:
            name: Loopback103
            type: iana-if-type:softwareLoopback
            enabled: true
            ietf-ip:ipv4:
              address:
                - ip: 10.255.103.1
                  netmask: 255.255.255.255

ansible-playbook restconf_playbook.yml


! Post-change verification on device
show ip interface brief | include Loopback
show running-config | section interface Loopback
show netconf-yang sessions detail
show logging | include RESTCONF|NETCONF