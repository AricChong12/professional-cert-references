# CCIE Enterprise Infrastructure — Blueprint Command & Code Reference

Companion reference mapped to the CCIE Enterprise Infrastructure v1.1 blueprint. Complements the general Cisco commands file with topics not covered there: switch administration internals, VRF route-leaking, protocol optimization/scale features, SD-Access/SD-WAN, MPLS/DMVPN, security hardening, system management, QoS (MQC), network services, telemetry/automation, and programmability.

---

## Table of Contents
1. [1.1 Switch Administration](#11-switch-administration)
2. [1.2 Routing Concepts (VRF-Lite, PBR, Redistribution, BFD)](#12-routing-concepts)
3. [1.3 EIGRP Advanced](#13-eigrp-advanced)
4. [1.4 OSPF Advanced](#14-ospf-advanced)
5. [1.5 BGP Advanced](#15-bgp-advanced)
6. [1.6 Multicast Advanced](#16-multicast-advanced)
7. [2.1 Cisco SD-Access](#21-cisco-sd-access)
8. [2.2 Cisco SD-WAN](#22-cisco-sd-wan)
9. [3. GRE / MPLS L3VPN / DMVPN](#3-gre--mpls-l3vpn--dmvpn)
10. [4.1–4.2 Device & Network Security](#41-42-device--network-security)
11. [4.3 System Management](#43-system-management)
12. [4.4 QoS (MQC)](#44-qos-mqc)
13. [4.5 Network Services](#45-network-services)
14. [4.6–4.7 Optimization & Operations](#46-47-optimization--operations)
15. [5. Automation & Programmability](#5-automation--programmability)

---

## 1.1 Switch Administration

**MAC address table:**
```
show mac address-table
show mac address-table dynamic
show mac address-table interface Gi1/0/1
mac address-table static aaaa.bbbb.cccc vlan 10 interface Gi1/0/1
mac address-table aging-time 300 vlan 10
clear mac address-table dynamic
```

**Errdisable recovery:**
```
errdisable recovery cause bpduguard
errdisable recovery cause udld
errdisable recovery cause security-violation
errdisable recovery interval 300
show errdisable recovery
show interfaces status err-disabled
```

**L2 MTU / Jumbo frames:**
```
system mtu 9198
system mtu jumbo 9198
interface Gi1/0/1
 mtu 9000
show system mtu
```

**CDP / LLDP:**
```
cdp run
interface Gi1/0/1
 cdp enable
show cdp neighbors detail

lldp run
interface Gi1/0/1
 lldp transmit
 lldp receive
show lldp neighbors detail
```

**UDLD:**
```
udld enable
udld aggressive
interface Gi1/0/1
 udld port aggressive
show udld neighbors
```

**VLAN pruning / extended VLANs:**
```
interface Gi1/0/1
 switchport trunk pruning vlan 10,20,30
vlan 2000
 name EXTENDED-VLAN
vtp version 3           ! required for extended-range VLAN propagation
```

**EtherChannel misconfig guard & multichassis:**
```
spanning-tree etherchannel guard misconfig
show etherchannel summary
show platform etherchannel
! Multichassis EtherChannel (StackWise Virtual / VSS) allows a single
! Port-channel to span two physical chassis for dual-homed redundancy.
```

---

## 1.2 Routing Concepts

**Administrative distance:**
```
ip route 10.10.10.0 255.255.255.0 10.1.1.2 100    ! custom AD
show ip route 10.10.10.0
```

**Policy-Based Routing (PBR):**
```
route-map PBR-VOICE permit 10
 match ip address VOICE-ACL
 set ip next-hop 10.2.2.2
 set interface Tunnel0

interface Gi0/0
 ip policy route-map PBR-VOICE
show route-map PBR-VOICE
show ip policy
```

**VRF-Lite:**
```
ip vrf CUSTOMER_A
 rd 65001:1
interface Gi0/1
 ip vrf forwarding CUSTOMER_A
 ip address 10.10.1.1 255.255.255.0
show ip vrf interfaces
show ip route vrf CUSTOMER_A
```

**VRF-aware routing protocols:**
```
router eigrp CCIE
 address-family ipv4 vrf CUSTOMER_A autonomous-system 100
  network 10.10.1.0 0.0.0.255
 exit-address-family

router ospf 10 vrf CUSTOMER_A
 network 10.10.1.0 0.0.0.255 area 0

router bgp 65001
 address-family ipv4 vrf CUSTOMER_A
  neighbor 10.10.1.2 remote-as 65010
  neighbor 10.10.1.2 activate
 exit-address-family
```

**Route leaking between VRFs (route-map + import/export, or VASI):**
```
ip vrf CUSTOMER_A
 route-target export 65001:1
 route-target import 65001:2

! Leaking via static route referencing another VRF
ip route vrf CUSTOMER_A 10.20.1.0 255.255.255.0 10.10.1.254 global
ip route vrf CUSTOMER_A 10.20.1.0 255.255.255.0 vrf CUSTOMER_B 10.20.1.1

! VASI (VRF-Aware Software Infrastructure) - virtual interface pair
interface vasileft1
 vrf forwarding CUSTOMER_A
 ip unnumbered Loopback0
interface vasiright1
 vrf forwarding CUSTOMER_B
 ip unnumbered Loopback0
```

**Route filtering:**
```
router bgp 65001
 neighbor 10.1.1.2 prefix-list PL-OUT out

router eigrp CCIE
 address-family ipv4 autonomous-system 100
  topology base
   distribute-list route-map RM-FILTER in
```

**Redistribution (mutual, with route-maps to avoid loops):**
```
route-map CONN-TO-OSPF permit 10
 match tag 50

router ospf 1
 redistribute connected subnets route-map CONN-TO-OSPF
 redistribute eigrp 100 subnets metric 100

router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500 route-map OSPF-TO-EIGRP
```

**Routing protocol authentication (recap):**
```
! OSPF
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 cisco123
! EIGRP named mode
key chain EIGRP-KEYS
 key 1
  key-string cisco123
router eigrp CCIE
 address-family ipv4 autonomous-system 100
  af-interface Gi0/0
   authentication mode md5
   authentication key-chain EIGRP-KEYS
! BGP
neighbor 10.1.1.2 password cisco123
```

**BFD:**
```
interface Gi0/0
 bfd interval 150 min_rx 150 multiplier 3

router ospf 1
 bfd all-interfaces

router eigrp CCIE
 address-family ipv4 autonomous-system 100
  af-interface Gi0/0
   bfd

router bgp 65001
 neighbor 10.1.1.2 fall-over bfd

show bfd neighbors detail
```

**L3 MTU:**
```
interface Gi0/0
 ip mtu 1400
show ip interface Gi0/0 | include MTU
```

---

## 1.3 EIGRP Advanced

**Feasibility condition & terms:**
- Feasible Distance (FD) = best metric to reach a destination.
- Reported Distance (RD) = neighbor's metric to the destination.
- Feasibility Condition: RD < FD (that neighbor becomes a Feasible Successor).

**EIGRP named mode (full example):**
```
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 100
  eigrp router-id 1.1.1.1
  metric weights 0 1 0 1 0 0     ! enable wide/classic metric compatibility
  network 10.0.0.0
  topology base
   variance 2
   maximum-paths 4
  af-interface Gi0/0
   hello-interval 5
   hold-time 15
 exit-address-family
```

**Wide metrics (K-values, bandwidth in Gbps era):**
```
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 100
  metric rib-scale 128
  metric version 64bit           ! wide metric support for high-bandwidth links
```

**Stuck-In-Active (SIA) / graceful shutdown:**
```
router eigrp CCIE
 address-family ipv4 unicast autonomous-system 100
  timers active-time 1 30        ! SIA timer 1 min 30 sec
eigrp graceful-restart              ! (classic mode) enable NSF-style restart
```

**Query boundaries — summarization stops queries:**
```
interface Gi0/0
 ip summary-address eigrp 100 10.1.0.0 255.255.0.0
```

**Leak-map with summary routes:**
```
route-map LEAK-SPECIFIC permit 10
 match ip address LOOPBACK-ACL

interface Gi0/0
 ip summary-address eigrp 100 10.1.0.0 255.255.0.0 leak-map LEAK-SPECIFIC
```

**EIGRP stub with leak-map:**
```
router eigrp 100
 eigrp stub connected summary leak-map LEAK-SPECIFIC
```

Verification:
```
show ip eigrp topology all-links
show ip eigrp topology active
show ip eigrp accounting
```

---

## 1.4 OSPF Advanced

**OSPFv3 with address families (IPv4 + IPv6 under one process):**
```
router ospfv3 1
 router-id 1.1.1.1
 !
 address-family ipv4 unicast
 exit-address-family
 !
 address-family ipv6 unicast
 exit-address-family

interface Gi0/0
 ospfv3 1 ipv4 area 0
 ospfv3 1 ipv6 area 0
```

**Network types:**
```
interface Gi0/0
 ip ospf network point-to-point
 ip ospf network broadcast
 ip ospf network non-broadcast
 ip ospf network point-to-multipoint
```

**GTSM (Generic TTL Security Mechanism):**
```
router ospf 1
 ttl-security all-interfaces hops 1
```

**Graceful shutdown / NSF:**
```
router ospf 1
 nsf cisco
 nsf ietf
```

**LSA throttling & SPF tuning:**
```
router ospf 1
 timers throttle spf 200 400 5000
 timers throttle lsa 200 400 5000
 timers lsa arrival 100
```

**Stub router (max-metric):**
```
router ospf 1
 max-metric router-lsa on-startup 300
 max-metric router-lsa summary-lsa
```

**Prefix suppression:**
```
router ospf 1
 prefix-suppression
interface Gi0/0
 ip ospf prefix-suppression disable
```

Verification:
```
show ip ospf
show ipv6 ospf neighbor
show ip ospf timers throttle spf
```

---

## 1.5 BGP Advanced

**Peer groups & templates:**
```
router bgp 65001
 neighbor IBGP-PEERS peer-group
 neighbor IBGP-PEERS remote-as 65001
 neighbor IBGP-PEERS update-source Loopback0
 neighbor 10.1.1.2 peer-group IBGP-PEERS

! Templates (session + policy)
template peer-session SESSION-TMPL
 remote-as 65001
 update-source Loopback0
template peer-policy POLICY-TMPL
 route-reflector-client
router bgp 65001
 neighbor 10.1.1.3 inherit peer-session SESSION-TMPL
 neighbor 10.1.1.3 inherit peer-policy POLICY-TMPL
```

**Passive peering & dynamic neighbors:**
```
router bgp 65001
 neighbor 10.1.1.0 remote-as 65002
 neighbor 10.1.1.0 transport connection-mode passive
 bgp listen range 10.1.1.0/24 peer-group DYNAMIC-PG
 neighbor DYNAMIC-PG peer-group
 neighbor DYNAMIC-PG remote-as 65002
```

**4-byte ASN & private ASN handling:**
```
router bgp 650001
 neighbor 10.1.1.2 remote-as 65002
 neighbor 10.1.1.2 local-as 65001 no-prepend replace-as
 neighbor 10.1.1.2 remove-private-as
```

**Timers:**
```
neighbor 10.1.1.2 timers 10 30
```

**Path selection / attribute manipulation:**
```
route-map SET-LOCALPREF permit 10
 set local-preference 200

route-map SET-MED permit 10
 set metric 50

router bgp 65001
 neighbor 10.1.1.2 route-map SET-LOCALPREF in
 neighbor 10.1.1.2 weight 500
```

**Conditional advertisement:**
```
router bgp 65001
 neighbor 10.1.1.2 advertise-map ADVERTISE-THIS non-exist-map BACKUP-DOWN
```

**Outbound route filtering (ORF):**
```
neighbor 10.1.1.2 capability orf prefix-list send
neighbor 10.1.1.2 prefix-list PL-ORF in
```

**Communities:**
```
route-map SET-COMM permit 10
 set community 65001:100 additive

router bgp 65001
 neighbor 10.1.1.2 send-community both
 neighbor 10.1.1.2 route-map SET-COMM out
ip community-list standard COMM-100 permit 65001:100
```

**AS-path manipulation:**
```
route-map PREPEND permit 10
 set as-path prepend 65001 65001 65001

ip as-path access-list 1 permit ^65002_
router bgp 65001
 neighbor 10.1.1.2 filter-list 1 in
```

**Route reflectors & aggregation:**
```
router bgp 65001
 neighbor 10.1.1.3 route-reflector-client
 bgp cluster-id 1.1.1.1

 aggregate-address 10.0.0.0 255.0.0.0 summary-only as-set
```

**Soft reconfiguration / route refresh:**
```
neighbor 10.1.1.2 soft-reconfiguration inbound
clear ip bgp 10.1.1.2 soft in
clear ip bgp 10.1.1.2 soft out
clear ip bgp * soft
```

Verification:
```
show ip bgp neighbors 10.1.1.2 advertised-routes
show ip bgp neighbors 10.1.1.2 received-routes
show ip bgp regexp ^65002_
show bgp ipv4 unicast summary
```

---

## 1.6 Multicast Advanced

**IGMP versions, snooping, querier, filter:**
```
interface Gi0/0
 ip igmp version 3
 ip igmp limit 50

ip igmp snooping
ip igmp snooping querier
ip igmp snooping vlan 10 querier address 10.1.10.1

interface Gi0/0
 ip igmp access-group IGMP-FILTER
```

**MLD (IPv6):**
```
interface Gi0/0
 ipv6 mld version 2
ipv6 mld snooping
```

**RPF check:**
```
show ip rpf 10.1.1.1
```

**PIM Sparse Mode & RP options:**
```
ip multicast-routing
interface Gi0/0
 ip pim sparse-mode

! Static RP
ip pim rp-address 10.1.1.1 1

! Auto-RP
ip pim send-rp-announce Loopback0 scope 16 group-list 1
ip pim send-rp-discovery Loopback0 scope 16

! BSR
ip pim bsr-candidate Loopback0 0
ip pim rp-candidate Loopback0 group-list 1
```

**SSM:**
```
ip pim ssm default
! or custom range
ip pim ssm range SSM-RANGE-ACL
```

**Multicast boundary & RP filtering:**
```
interface Gi0/1
 ip multicast boundary 10 filter-autorp
ip access-list standard RP-ANNOUNCE-FILTER
 permit 10.1.1.1
```

**PIMv6 anycast RP:**
```
ipv6 pim rp-address 2001:DB8::1
ipv6 pim anycast-RP 2001:DB8::1 2001:DB8::2
```

**IPv4 anycast RP using MSDP:**
```
ip msdp peer 10.2.2.2 connect-source Loopback0
ip msdp originator-id Loopback0
show ip msdp peer
show ip msdp sa-cache
```

**Multicast multipath:**
```
ip multicast multipath
```

Verification:
```
show ip mroute
show ip pim rp mapping
show ip pim interface
show ip igmp groups detail
```

---

## 2.1 Cisco SD-Access

SD-Access is largely orchestrated via DNA Center; CLI is mostly used for underlay/verification.

**Underlay (manual, if not using LAN Automation/PnP):**
```
router isis
 net 49.0001.0000.0000.0001.00
 is-type level-2-only
 metric-style wide
interface Gi0/0
 ip router isis
```

**Fabric data-plane concepts (VXLAN/LISP) — verification only, config is DNAC-driven:**
```
show lisp site
show lisp instance-id 4097 ipv4 map-cache
show vxlan vni
show fabric border-node
show segment-routing mpls connected-prefix-sid-map ipv4
```

**TrustSec (SGT/SGACL) manual CLI:**
```
cts role-based enforcement
cts role-based sgt-map 10.1.1.0/24 sgt 10
cts role-based permissions from 10 to 20 SGACL-NAME
show cts role-based sgt-map all
show cts role-based permissions
```

**Fabric in a Box / extended node basics:**
```
device-role extended-node
show fabric extended-node summary
```

> Note: Full SD-Access fabric provisioning (host onboarding, authentication templates, border handoff, multisite) is performed through Cisco DNA Center's GUI/API rather than direct CLI, per the exam blueprint's intent-based design.

---

## 2.2 Cisco SD-WAN

SD-WAN device config uses **vManage CLI templates** or the **VPN-based configuration model** (Viptela OS). Example device-side (CLI) constructs:

```
system
 system-ip 10.0.0.1
 site-id 100
 organization-name "CCIE-LAB"
 vbond 203.0.113.1

vpn 0
 interface GigabitEthernet0/0
  ip address 203.0.113.5/24
  tunnel-interface
   encapsulation ipsec
   color biz-internet
   allow-service all
  no shutdown

vpn 512
 interface GigabitEthernet0
  ip address 192.168.1.1/24
  no shutdown
```

**OMP (Overlay Management Protocol):**
```
omp
 no shutdown
 graceful-restart
 advertise connected
 advertise static
show omp peers
show omp routes
```

**Centralized policy (vSmart CLI-equivalent, applied via vManage):**
```
policy
 lists
  site-list SITE-100
   site-id 100
 control-policy CONTROL-POLICY-1
  sequence 10
   match route
    site-list SITE-100
   action accept
 lists
  data-prefix-list DATA-PREFIX-1
   ip-prefix 10.1.1.0/24
 data-policy DATA-POLICY-1
  vpn-list VPN-LIST-1
   sequence 10
    match
     source-data-prefix-list DATA-PREFIX-1
    action accept
```

**Localized policy (device-side ACL/route-policy):**
```
policy
 access-list ACL-BLOCK-TELNET
  sequence 10
   match
    destination-port 23
   action drop
```

Verification:
```
show control connections
show control local-properties
show bfd sessions
show app-route stats
```

> Note: These SD-WAN constructs reflect the vEdge/cEdge XML/CLI configuration model; production deployments are managed centrally through vManage templates and APIs (see Section 5).

---

## 3. GRE / MPLS L3VPN / DMVPN

**Static point-to-point GRE:**
```
interface Tunnel0
 ip address 172.16.1.1 255.255.255.252
 tunnel source Gi0/0
 tunnel destination 203.0.113.2
 tunnel mode gre ip
```

**MPLS basics & LDP:**
```
mpls ip
mpls label protocol ldp
mpls ldp router-id Loopback0 force
interface Gi0/0
 mpls ip

show mpls ldp neighbor
show mpls ldp bindings
show mpls interfaces
mpls ping ipv4 destination 10.1.1.1/32
mpls traceroute ipv4 destination 10.1.1.1/32
```

**MPLS L3VPN (PE-CE with BGP, MP-BGP VPNv4):**
```
! PE router
ip vrf CUST_A
 rd 65000:1
 route-target export 65000:1
 route-target import 65000:1

interface Gi0/1
 ip vrf forwarding CUST_A
 ip address 10.10.1.1 255.255.255.0

router bgp 65000
 neighbor 10.10.1.2 remote-as 65010            ! PE-CE eBGP
 address-family ipv4 vrf CUST_A
  neighbor 10.10.1.2 activate
 exit-address-family

 neighbor 10.0.0.2 remote-as 65000              ! PE-PE iBGP (loopback)
 neighbor 10.0.0.2 update-source Loopback0
 address-family vpnv4
  neighbor 10.0.0.2 activate
  neighbor 10.0.0.2 send-community extended
 exit-address-family
```

Verification:
```
show ip bgp vpnv4 vrf CUST_A summary
show ip route vrf CUST_A
show mpls forwarding-table
```

**DMVPN Phase 3 (dual hub, NHRP + IPsec/IKEv2 PSK):**
```
crypto ikev2 keyring KEYRING
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key cisco123

crypto ikev2 profile IKEV2-PROFILE
 match identity remote address 0.0.0.0
 authentication local pre-share
 authentication remote pre-share
 keyring local KEYRING

crypto ipsec transform-set TSET esp-aes esp-sha-hmac
 mode transport

crypto ipsec profile IPSEC-PROFILE
 set transform-set TSET
 set ikev2-profile IKEV2-PROFILE

interface Tunnel0
 ip address 172.16.0.1 255.255.255.0
 ip nhrp network-id 100
 ip nhrp map multicast dynamic
 ip nhrp redirect                     ! hub, Phase 3
 tunnel source Gi0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile IPSEC-PROFILE

! Spoke
interface Tunnel0
 ip address 172.16.0.11 255.255.255.0
 ip nhrp network-id 100
 ip nhrp nhs 172.16.0.1 nbma 203.0.113.1 multicast
 ip nhrp shortcut                     ! spoke, Phase 3
 tunnel source Gi0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile IPSEC-PROFILE
```

Verification:
```
show dmvpn detail
show ip nhrp
show crypto ikev2 sa
show crypto ipsec sa
```

---

## 4.1–4.2 Device & Network Security

**Control Plane Policing (CoPP):**
```
class-map match-all COPP-CRITICAL
 match access-group name CRITICAL-ACL

policy-map COPP-POLICY
 class COPP-CRITICAL
  police 512000 8000 conform-action transmit exceed-action drop
 class class-default
  police 32000 8000 conform-action transmit exceed-action drop

control-plane
 service-policy input COPP-POLICY
```

**AAA (TACACS+/RADIUS):**
```
aaa new-model
tacacs server TAC01
 address ipv4 10.1.1.100
 key cisco123

aaa group server tacacs+ TACGROUP
 server name TAC01

aaa authentication login default group TACGROUP local
aaa authorization exec default group TACGROUP local
aaa accounting exec default start-stop group TACGROUP
```

**VACL / PACL:**
```
! PACL (port ACL, applied to L2 access port)
interface Gi1/0/1
 ip access-group PACL-IN in

! VACL
vlan access-map VACL-MAP 10
 match ip address VACL-ACL
 action drop
vlan access-map VACL-MAP 20
 action forward
vlan filter VACL-MAP vlan-list 10
```

**Storm control:**
```
interface Gi1/0/1
 storm-control broadcast level 10.00
 storm-control multicast level 20.00
 storm-control action shutdown
```

**DHCP snooping (+ option 82):**
```
ip dhcp snooping
ip dhcp snooping vlan 10
interface Gi1/0/1
 ip dhcp snooping trust
interface Gi1/0/2
 ip dhcp snooping limit rate 15
ip dhcp snooping information option
show ip dhcp snooping binding
```

**IP Source Guard:**
```
interface Gi1/0/2
 ip verify source
 ip verify source port-security
```

**Dynamic ARP Inspection:**
```
ip arp inspection vlan 10
interface Gi1/0/1
 ip arp inspection trust
show ip arp inspection
```

**Port security (recap):**
```
interface Gi1/0/2
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

**IPv4 ACLs / uRPF:**
```
interface Gi0/0
 ip verify unicast source reachable-via rx
 ip verify unicast source reachable-via any allow-default
```

**IPv6 traffic filters:**
```
ipv6 access-list IPV6-FILTER
 permit tcp any any eq 22
 deny ipv6 any any log
interface Gi0/0
 ipv6 traffic-filter IPV6-FILTER in
```

**IPv6 first-hop security:**
```
ipv6 nd raguard policy RA-GUARD-POLICY
 device-role host
interface Gi1/0/1
 ipv6 nd raguard attach-policy RA-GUARD-POLICY

ipv6 dhcp guard policy DHCP-GUARD-POLICY
 device-role server
interface Gi1/0/1
 ipv6 dhcp guard attach-policy DHCP-GUARD-POLICY

ipv6 snooping policy ND-POLICY
 device-role host
interface Gi1/0/1
 ipv6 snooping attach-policy ND-POLICY

show ipv6 neighbor-tracking
show ipv6 source-guard binding
```

---

## 4.3 System Management

**SSH/SCP:**
```
ip domain-name lab.local
crypto key generate rsa modulus 2048
ip ssh version 2
ip ssh time-out 60
ip scp server enable
```

**NETCONF/RESTCONF:**
```
netconf-yang
restconf
ip http secure-server
show netconf-yang sessions
```

RESTCONF example (curl):
```bash
curl -k -u admin:cisco123 \
  -H "Accept: application/yang-data+json" \
  https://10.1.1.1/restconf/data/ietf-interfaces:interfaces
```

**SNMP:**
```
snmp-server community public RO
snmp-server group SNMPV3GRP v3 priv
snmp-server user snmpadmin SNMPV3GRP v3 auth sha AuthPass123 priv aes 128 PrivPass123
snmp-server host 10.1.1.50 version 3 priv snmpadmin
snmp-server enable traps
```

**Logging:**
```
logging host 10.1.1.100
logging trap informational
logging buffered 16384
service timestamps log datetime msec localtime show-timezone
archive
 log config
  logging enable
  notify syslog
show archive log config all
```

**Conditional / targeted debugs:**
```
debug ip packet detail 100
access-list 100 permit ip host 10.1.1.1 host 10.1.1.2
debug platform condition interface Gi0/0 both
debug platform condition start
```

---

## 4.4 QoS (MQC)

**Classification with NBAR:**
```
class-map match-any BUSINESS-APPS
 match protocol nbar webex-meeting
 match protocol nbar ssl

ip nbar protocol-discovery
show ip nbar protocol-discovery interface Gi0/0
```

**Trust boundary / marking:**
```
interface Gi1/0/1
 mls qos trust dscp
 auto qos voip cisco-phone

class-map match-all VOICE
 match dscp ef
policy-map MARK-IN
 class VOICE
  set dscp ef
interface Gi0/0
 service-policy input MARK-IN
```

**Policing / shaping:**
```
policy-map POLICE-POLICY
 class class-default
  police cir 10000000 bc 250000 conform-action transmit exceed-action drop

policy-map SHAPE-POLICY
 class class-default
  shape average 5000000
```

**Congestion management/avoidance (LLQ + WRED):**
```
policy-map WAN-QOS
 class VOICE
  priority level 1 percent 10
 class VIDEO
  bandwidth percent 30
  random-detect dscp-based
 class class-default
  fair-queue
  random-detect
```

**HQoS (hierarchical, parent/child):**
```
policy-map CHILD-POLICY
 class VOICE
  priority percent 10
 class class-default
  fair-queue

policy-map PARENT-SHAPE
 class class-default
  shape average 10000000
  service-policy CHILD-POLICY

interface Gi0/0
 service-policy output PARENT-SHAPE
```

Verification:
```
show policy-map interface Gi0/0
show mls qos interface Gi1/0/1
show class-map
```

---

## 4.5 Network Services

**FHRP recap (HSRP/VRRP) — see other file; IPv6 RA-based redundancy:**
```
interface Gi0/0
 ipv6 nd ra suppress   ! disable on device acting purely as host
 ipv6 nd prefix default no-advertise
```

**NTP:**
```
ntp server 10.1.1.1 prefer
ntp authenticate
ntp authentication-key 1 md5 cisco123
ntp trusted-key 1
show ntp status
show ntp associations
```

**PTP (design note):**
```
ptp mode boundary
ptp domain 0
interface Gi0/0
 ptp enable
show ptp clock
```

**DHCP client/server/relay + options:**
```
! Server
ip dhcp pool LAN-POOL
 network 10.1.1.0 255.255.255.0
 default-router 10.1.1.1
 dns-server 8.8.8.8
 option 150 ip 10.1.1.5          ! e.g., TFTP for VoIP

! Relay
interface Gi0/1
 ip helper-address 10.1.1.10

! Client
interface Gi0/0
 ip address dhcp
```

**DHCPv6 (stateful/stateless, PD):**
```
ipv6 dhcp pool DHCPV6-POOL
 address prefix 2001:DB8:1::/64
 dns-server 2001:4860:4860::8888
interface Gi0/0
 ipv6 dhcp server DHCPV6-POOL
 ipv6 nd other-config-flag       ! stateless
 ipv6 nd managed-config-flag     ! stateful

! Prefix delegation
ipv6 dhcp pool PD-POOL
 prefix-delegation pool PD-CLIENT-POOL lifetime 3600 1800
ipv6 local pool PD-CLIENT-POOL 2001:DB8:100::/48 64
```

**NAT (recap + VRF-aware / VASI NAT):**
```
ip nat inside source list NAT-ACL pool NAT-POOL vrf CUSTOMER_A overload
ip nat inside source static 10.1.1.5 203.0.113.5 vrf CUSTOMER_A match-in-vrf

! VASI NAT bridges NAT between two VRFs across a vasi pair
interface vasileft1
 ip nat inside
interface vasiright1
 ip nat outside
```

---

## 4.6–4.7 Optimization & Operations

**IP SLA (ICMP/UDP/TCP probes):**
```
ip sla 1
 icmp-echo 10.1.1.2
 frequency 10
ip sla schedule 1 life forever start-time now

ip sla 2
 udp-jitter 10.1.1.2 5000
ip sla schedule 2 life forever start-time now

show ip sla statistics
```

**Object tracking:**
```
track 1 ip sla 1 reachability
track 2 interface Gi0/1 line-protocol
track 10 list boolean and
 object 1
 object 2

interface Gi0/0
 standby 1 track 10 decrement 20
```

**Flexible NetFlow:**
```
flow record MY-RECORD
 match ipv4 source address
 match ipv4 destination address
 collect counter bytes

flow exporter MY-EXPORTER
 destination 10.1.1.100
 transport udp 2055

flow monitor MY-MONITOR
 record MY-RECORD
 exporter MY-EXPORTER

interface Gi0/0
 ip flow monitor MY-MONITOR input

show flow monitor MY-MONITOR cache
```

**SPAN / RSPAN / ERSPAN:**
```
monitor session 1 source interface Gi0/1
monitor session 1 destination interface Gi0/2

! RSPAN
vlan 999
 remote-span
monitor session 1 source interface Gi0/1
monitor session 1 destination remote vlan 999

! ERSPAN (source side)
monitor session 1 type erspan-source
 source interface Gi0/1
 destination
  ip address 10.1.1.100
  erspan-id 1
  origin ip address 10.1.1.1
```

**Embedded Packet Capture (EPC):**
```
monitor capture CAP1 interface Gi0/0 both
monitor capture CAP1 match any
monitor capture CAP1 buffer size 10
monitor capture CAP1 start
show monitor capture CAP1 buffer
monitor capture CAP1 export flash:cap1.pcap
```

**Data path packet trace:**
```
debug platform packet-trace enable
debug platform packet-trace packet 10
show platform packet-trace summary
show platform packet-trace packet 0
```

---

## 5. Automation & Programmability

**Data encoding formats:**

JSON:
```json
{
  "hostname": "R1",
  "interfaces": [
    {"name": "GigabitEthernet0/0", "ip": "10.1.1.1", "mask": "255.255.255.0"}
  ]
}
```

YAML:
```yaml
hostname: R1
interfaces:
  - name: GigabitEthernet0/0
    ip: 10.1.1.1
    mask: 255.255.255.0
```

Jinja2 template (used with NAPALM/Ansible/Netmiko for config generation):
```jinja
hostname {{ hostname }}
{% for intf in interfaces %}
interface {{ intf.name }}
 ip address {{ intf.ip }} {{ intf.mask }}
{% endfor %}
```

**EEM applet:**
```
event manager applet INTERFACE-DOWN
 event syslog pattern "Interface Gi0/0, changed state to down"
 action 1.0 syslog msg "ALERT: Gi0/0 went down"
 action 2.0 cli command "enable"
 action 3.0 cli command "show interfaces Gi0/0"
```

**EEM with Python (Guest Shell CLI Python module):**
```
guestshell enable
guestshell run python3
```

```python
# Inside Guest Shell — using the cli Python module
from cli import cli, configure

output = cli("show ip interface brief")
print(output)

configure(["interface Loopback99", "ip address 1.1.1.1 255.255.255.255"])
```

**EEM Python module (event_manager applet calling a python script):**
```
event manager applet PY-CHECK
 event timer watchdog time 60
 action 1.0 cli command "guestshell run python3 /bootflash/scripts/check_health.py"
```

**vManage API (Python requests):**
```python
import requests

base_url = "https://vmanage.lab.local"
session = requests.Session()
session.post(f"{base_url}/j_security_check",
             data={"j_username": "admin", "j_password": "cisco123"},
             verify=False)

token = session.get(f"{base_url}/dataservice/client/token", verify=False).text
session.headers.update({"X-XSRF-TOKEN": token})

devices = session.get(f"{base_url}/dataservice/device", verify=False).json()
print(devices)
```

**Cisco DNA Center API (Python requests):**
```python
import requests
from requests.auth import HTTPBasicAuth

dnac = "https://dnac.lab.local"
auth = HTTPBasicAuth("admin", "cisco123")

resp = requests.post(f"{dnac}/dna/system/api/v1/auth/token",
                      auth=auth, verify=False)
token = resp.json()["Token"]

headers = {"X-Auth-Token": token}
devices = requests.get(f"{dnac}/dna/intent/api/v1/network-device",
                        headers=headers, verify=False).json()
print(devices)
```

**Model-driven telemetry (gRPC dial-out, on-change subscription):**
```
telemetry ietf subscription 100
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface
 stream yang-push
 update-policy on-change
 receiver ip address 10.1.1.100 port 57500 protocol grpc-tcp

show telemetry ietf subscription all
show telemetry ietf subscription 100 detail
```

---

### Notes
- SD-Access and SD-WAN are intent/controller-driven architectures; the CLI shown here is for underlay verification and local overrides — day-to-day operation is via DNA Center / vManage GUI and APIs.
- This is a study aid aligned to the CCIE Enterprise Infrastructure blueprint structure, not a replacement for Cisco's official documentation.
