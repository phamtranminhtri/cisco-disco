# Hospital network

|      Name     | VLAN |      Subnet     |                 |                 |
|:-------------:|:----:|:---------------:|:---------------:|:---------------:|
|               |      |       Main      |       DBP       |       BHTQ      |
| IT            | 10   | 192.168.10.0/24 | 192.168.11.0/24 | 192.168.12.0/24 |
| MEDICAL       | 20   | 10.20.0.0/20    | 10.21.0.0/20    | 10.22.0.0/20    |
| ADMIN         | 30   | 10.30.0.0/20    | 10.31.0.0/20    | 10.32.0.0/20    |
| CAMERA        | 40   | 10.40.0.0/20    | 10.41.0.0/24    | 10.42.0.0/24    |
| WLAN          | 50   | 10.50.0.0/20    | 10.51.0.0/20    | 10.52.0.0/20    |
| INSIDE-SERVER | 90   | 172.16.0.0/24   | 172.17.0.0/24   | 172.18.0.0/24   |
| DMZ           | None | 172.16.1.0/24   | None            | None            |

## 0. Network design and beautification (19:00)

* Network devices:
  * 2911 router x2: VNPT-ISP and FPT-ISP
  * 5506 firewall x2
  * 3650 multilayer switch x2: add AC power supply
  * 2960 switch x10 for building A & B
* Wires:
  * Top down, automatic
  * EtherChannel: 3x between 3650
* Host devices:
  * PC
  * Printer
  * Laptop: add wireless module
  * Smartphone
  * LAP-PT: add power adapter
* Copy host device 9 times for 10 floor (building A 5 floor, B 5 floor)
  * Wires: Connect the wired host
* Add 2960 switch (Inside Server (server farm) switch):
  * Server x4: DHCP, DNS, HIS, DATABASE
  * 2504 WLC: same VLAN as LAP
  * Connect wire in that order
* DMZ: only attach to 1 firewall
  * Add 2960 switch
  * 3 server: WEB, EMAIL, APP
  * Connect wire
* Cloud:
  * 2 PC (China, USA)
  * 1 switch
  * 1 router
  * Cluster Cloud (switch, router)
* Name devices, background color


## 1. Basic settings to all devices + SSH + Standard ACL for SSH (38:45)

* Basic setting for switches, not firewall, leave for last
* Hostname, console password, enable password, banner, password encryption, disable IP domain lookup
* Command: `en` and `conf t` first. Change hostname for each switch

```
hostname SW-A1

line console 0
password cisco
login
exec-timeout 3 0
logging synchronous
exit

enable password cisco
banner motd &HELLO&
no ip domain-lookup
service password-encryption

username cisco password cisco
ip domain-name cisco.com

crypto key generate rsa general-keys modulus 1024
ip ssh version 2  

line vty 0 15
login local
transport input ssh
exit

access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.11.0 0.0.0.255
access-list 1 permit 192.168.12.0 0.0.0.255
access-list 1 deny any

line vty 0 15
access-class 1 in
exit

do wr
```
* For 3650 multilayer switch, answer no first

## 2. VLAN assignment plus all access and trunk ports on l2 and l3 switches (1:00:15)

* Ensure access switch connect first 2 port to multilayer switch (gig0/1, gig0/2), to be trunk
* Bypass password on access switch: Go to config and click on any interface
* Ensure put consistence switch interface:
  * MEDICAL/ADMIN: fa0/1-16
  * CAMERA: fa0/17-20
  * LAP: fa0/21-24
* Floor 1 of each building (SW-A1, SW-B1): ADMIN is fa0/1-16, no MEDICAL, IT, INSIDE-SERVERS

```
int range gig0/1-2
switchport mode trunk
ex

vlan 10
name IT
vlan 20
name MEDICAL
vlan 30
name ADMIN
vlan 40
name CAMERA
vlan 50
name WLAN
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-16
switchport mode access
switchport access vlan 30
ex

int range fa0/17-20
switchport mode access
switchport access vlan 40
ex

int range fa0/21-24
switchport mode access
switchport access vlan 50
ex

do wr
```
* Floor 2-5 of each building (SW-A2, SW-B2,...): MEDICAL is fa0/1-16, no ADMIN, IT, INSIDE-SERVERS

```
int range gig0/1-2
switchport mode trunk
ex

vlan 10
name IT
vlan 20
name MEDICAL
vlan 30
name ADMIN
vlan 40
name CAMERA
vlan 50
name WLAN
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-16
switchport mode access
switchport access vlan 20
ex

int range fa0/17-20
switchport mode access
switchport access vlan 40
ex

int range fa0/21-24
switchport mode access
switchport access vlan 50
ex

do wr
```
* IT-ROOM in Data center (SW-IT): IT is fa0/1-16, no ADMIN, MEDICAL, INSIDE-SERVERS

```
int range gig0/1-2
switchport mode trunk
ex

vlan 10
name IT
vlan 20
name MEDICAL
vlan 30
name ADMIN
vlan 40
name CAMERA
vlan 50
name WLAN
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-16
switchport mode access
switchport access vlan 10
ex

int range fa0/17-20
switchport mode access
switchport access vlan 40
ex

int range fa0/21-24
switchport mode access
switchport access vlan 50
ex

do wr
```

* DMZ switch: No VLAN config
* Server switch: VLAN INSIDE SERVERS - 90, different VLAN from others
  * WLC in VLAN 50: fa0/1
  * Server: fa0/2-24

```
int range gig0/1-2
switchport mode trunk
ex

vlan 10
name IT
vlan 20
name MEDICAL
vlan 30
name ADMIN
vlan 40
name CAMERA
vlan 50
name WLAN
vlan 90
name INSIDE-SERVERS
ex

int range fa0/2-24
switchport mode access
switchport access vlan 90
ex

int fa0/1
switchport mode access
switchport access vlan 50
ex

do wr
```

### Multilayer switch (1:23:50)

* Trunk port:
  * CORE-SW1, CORE-SW2: gig1/0/3-13, gig1/0/17

```
int range gig1/0/3-13, gig1/0/17
switchport mode trunk
ex

vlan 10
name IT
vlan 20
name MEDICAL
vlan 30
name ADMIN
vlan 40
name CAMERA
vlan 50
name WLAN
vlan 90
name INSIDE-SERVERS
ex

do wr
```

### 2.1. STP Portfast and BPDUguard configs on all access ports (1:32:00)

* Portfast: Orange to green fast in switch
  * Access switch: 2 trunk (gig0/1-2)

```
int range fa0/1-24
spanning-tree portfast
spanning-tree bpduguard enable
ex

do wr
```

  * DMZ switch: no trunk

```
int range fa0/1-24
spanning-tree portfast
spanning-tree bpduguard enable
ex

do wr
```

## 3. EtherChannel LACP (1:38:00)

* 3 link between core switches (gig1/0/14-16)
* active-active, active-passive, but no passive-passive
* CORE-SW1:

```
int range gig1/0/14-16
channel-group 1 mode active
ex

interface Port-channel 1
switchport mode trunk
ex

do wr
```
* CORE-SW2:

```
int range gig1/0/14-16
channel-group 1 mode passive
ex

interface Port-channel 1
switchport mode trunk
ex

do wr
```

## 4. Subnetting and IP addressing (1:42:30)

* IP plan: see 1:43:00
* Network:
  * IT: 192.168.10.0/24
  * MEDICAL: 10.20.0.0/20
  * ADMIN: 10.30.0.0/20
  * CAMERA: 10.40.0.0/20
  * WLAN: 10.50.0.0/20
  * INSIDE-SERVERS: 172.16.0.0/24
  * DMZ: 172.16.1.0/24

* Between Cloud, ISP, Firewall, Router, L3 Switch:
  * Public:
    * VNPT: 105.100.50.0/30
    * FPT: 197.200.100.0/30
  * Cloud Area: 8.0.0.0/8
  * ISP1 - Internet: 20.20.20.0/30
  * ISP2 - Internet: 30.30.30.0/30
  * ISP1 - FWL1: 105.100.50.0/30
  * ISP1 - FWL2: 105.100.50.4/30
  * ISP2 - FWL1: 197.200.100.0/30
  * ISP2 - FWL2: 197.200.100.4/30
  * MLSW1 - FWL1: 10.2.2.0/30
  * MLSW1 - FWL2: 10.2.2.4/30
  * MLSW2 - FWL1: 10.2.2.8/30
  * MLSW2 - FWL2: 10.2.2.12/30
* Leave firewall for last, we config l3 switches, ISP router, cloud router
* Firewall always take second IP address (10.2.2.2, 10.2.2.6)
* L3 switch: Is both switch and router
* CORE-SW1: Config -> gig1/0/2
```
ip routing
```
* CORE-SW2: Config -> gig1/0/4
```
ip routing
```

* CORE-SW1:
```
int gi1/0/1
no switchport
no shut
ip add 10.2.2.1 255.255.255.252

int gi1/0/2
no switchport
no shut
ip add 10.2.2.5 255.255.255.252
ex

do wr
```

```
* CORE-SW2:
```
int gi1/0/1
no switchport
no shut
ip add 10.2.2.9 255.255.255.252

int gi1/0/2
no switchport
no shut
ip add 10.2.2.13 255.255.255.252
ex

do wr
```

* Uncluster cloud for 1 moment
* SEACOM router: GUI
  * Click top then minimize (optional)
  * gig0/0 connect to firewall 1
    * Config -> gig0/0
    * Click On
    * IP: 105.100.50.1
    * Subnet mask: 255.255.255.252
  * gig0/1 connect to firewall 2
    * Config -> gig0/1
    * Click On
    * IP: 105.100.50.5
    * Subnet mask: 255.255.255.252
  * gig0/2 connect to Cloud
    * Config -> gig0/2
    * Click On
    * IP: 20.20.20.1
    * Subnet mask: 255.255.255.252
* Similar for 2 other router
* SAFARICOM router:
  * gig0/0 connect to firewall 1
    * Config -> gig0/0
    * Click On
    * IP: 197.200.100.1
    * Subnet mask: 255.255.255.252
  * gig0/1 connect to firewall 2
    * Config -> gig0/1
    * Click On
    * IP: 197.200.100.5
    * Subnet mask: 255.255.255.252
  * gig0/2 connect to Cloud
    * Config -> gig0/2
    * Click On
    * IP: 30.30.30.1
    * Subnet mask: 255.255.255.252
* Cloud router:
  * gig0/0 connect to SEACOM
    * Config -> gig0/0
    * Click On
    * IP: 20.20.20.2
    * Subnet mask: 255.255.255.252
  * gig0/1 connect to SAFARICOM
    * Config -> gig0/1
    * Click On
    * IP: 30.30.30.2
    * Subnet mask: 255.255.255.252
  * gig0/2 connect to other switch
    * Config -> gig0/2
    * Click On
    * IP: 8.0.0.1
    * Subnet mask: 255.0.0.0

* Static IP for 2 distance computer:
  * PC1:
    * IP: 8.0.0.10
    * Subnet mask: 255.0.0.0
    * Default gateway: 8.0.0.1
    * DNS: 8.0.0.1
  * PC2:
    * IP: 8.0.0.20
    * Subnet mask: 255.0.0.0
    * Default gateway: 8.0.0.1
    * DNS: 8.0.0.1

## 5. HSRP and Inter-VLAN routing on the l3 switches plus IP DHCP helper addresses (1:56:30)

* Static IP for INSIDE server:
  * DHCP: 172.16.0.6
  * DNS: 172.16.0.7
  * HIS: 172.16.0.8
  * DATABASE: 172.16.0.9


* HSRP between 2 l3 switch: Redundancy, kinda like load balancer, standby <-> active
  * Interface with highest IP address become active router
* Not for voice, because voice IP managed by voice router
* CORE-SW1: active for IT (10) & MEDICAL (20) & ADMIN (30), standby for CAMERA (40) & WLAN (50) & INSDE SERVERS (90)
```
int vlan 10
no shut
ip add 192.168.10.3 255.255.255.0
standby 10 ip 192.168.10.1
ip helper-address 172.16.0.6
exit

int vlan 20
no shut
ip add 10.20.0.3 255.255.240.0
standby 20 ip 10.20.0.1
ip helper-address 172.16.0.6
exit

int vlan 30
no shut
ip add 10.30.0.3 255.255.240.0
standby 30 ip 10.30.0.1
ip helper-address 172.16.0.6
exit

int vlan 40
no shut
ip add 10.40.0.2 255.255.240.0
standby 40 ip 10.40.0.1
ip helper-address 172.16.0.6
exit

int vlan 50
no shut
ip add 10.50.0.2 255.255.240.0
standby 50 ip 10.50.0.1
ip helper-address 172.16.0.6
exit

int vlan 90
no shut
ip add 172.16.0.2 255.255.255.0
standby 90 ip 172.16.0.1
exit

do wr
```

* To show result:
```
do sh star
```
  * Copy result to NotePad (4 vlan)
  * Remove MAC address

* CORE-SW2: standby for IT (10) & MEDICAL (20) & ADMIN (30), active for CAMERA (40) & WLAN (50) & INSDE SERVERS (90)
```
int vlan 10
no shut
ip add 192.168.10.2 255.255.255.0
standby 10 ip 192.168.10.1
ip helper-address 172.16.0.6
exit

int vlan 20
no shut
ip add 10.20.0.2 255.255.240.0
standby 20 ip 10.20.0.1
ip helper-address 172.16.0.6
exit

int vlan 30
no shut
ip add 10.30.0.2 255.255.240.0
standby 30 ip 10.30.0.1
ip helper-address 172.16.0.6
exit

int vlan 40
no shut
ip add 10.40.0.3 255.255.240.0
standby 40 ip 10.40.0.1
ip helper-address 172.16.0.6
exit

int vlan 50
no shut
ip add 10.50.0.3 255.255.240.0
standby 50 ip 10.50.0.1
ip helper-address 172.16.0.6
exit

int vlan 90
no shut
ip add 172.16.0.3 255.255.255.0
standby 90 ip 172.16.0.1
exit

do wr
```

* Wait a bit for sync
```
show standby brief
```

## 6. Static IP address to DMZ/server farm devices (2:15:45)

* Desktop -> IP Configuration
* INSIDE SERVERS:
  * DHCP:
    * IP: 172.16.0.6
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.0.1 (standby IP of CORE-SW2)
    * DNS: 172.16.0.7
  * DNS:
    * IP: 172.16.0.7
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.0.1
    * DNS: 172.16.0.7
  * HIS:
    * IP: 172.16.0.8
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.0.1
    * DNS: 172.16.0.7
  * DATABASE:
    * IP: 172.16.0.9
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.0.1
    * DNS: 172.16.0.7
* DMZ SERVERS:
  * WEB:
    * IP: 172.16.1.6
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.1.1
    * DNS: 172.16.0.7
  * EMAIL:
    * IP: 172.16.1.7
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.1.1
    * DNS: 172.16.0.7
  * APP:
    * IP: 172.16.1.8
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.16.1.1
    * DNS: 172.16.0.7


## 7. DHCP server device configuration (2:19:25)

* Services -> DHCP
  * Turn everything to 0
  * Turn on and start edit
  * Only need DHCP for IT, MEDICAL, ADMIN, CAMERA, WLAN
  * IT:
    * Pool Name: IT-Pool
    * Default gateway: 192.168.10.1
    * DNS: 172.16.0.7
    * Start IP: 192.168.10.11
    * Subnet mask: 255.255.255.0
    * Max user: 200
    * Click Add
  * MEDICAL:
    * Pool Name: MEDICAL-Pool
    * Default gateway: 10.20.0.1
    * DNS: 172.16.0.7
    * Start IP: 10.20.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add
  * ADMIN:
    * Pool Name: ADMIN-Pool
    * Default gateway: 10.30.0.1
    * DNS: 172.16.0.7
    * Start IP: 10.30.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add
  * CAMERA:
    * Pool Name: CAMERA-Pool
    * Default gateway: 10.40.0.1
    * DNS: 172.16.0.7
    * Start IP: 10.40.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 200
    * Click Add
  * WLAN:
    * Pool Name: WLAN-Pool
    * Default gateway: 10.50.0.1
    * DNS: 172.16.0.7
    * Start IP: 10.50.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 4000
    * WLC address: 10.50.0.10 (assume)
    * Click Add


## 8. OSPF on the firewall, routers, and switches (2:24:00)

* Not touching firewall now, only router and l3 switch
* CORE-SW1: Advertise 8 network (10, 20, 30, 40, 50, 90, 2 firewall) (hover mouse over switch)

```
router ospf 35
router-id 1.1.1.1

network 10.2.2.0 0.0.0.3 area 0
network 10.2.2.4 0.0.0.3 area 0
network 192.168.10.0 0.0.0.255 area 0
network 10.20.0.0 0.0.15.255 area 0
network 10.30.0.0 0.0.15.255 area 0
network 10.40.0.0 0.0.15.255 area 0
network 10.50.0.0 0.0.15.255 area 0
network 172.16.0.0 0.0.0.255 area 0
ex

do wr
```

* Show network:

```
do sh star
```
* Router-id should be different
* CORE-SW2: Advertise 8 network (10, 20, 30, 40, 50, 90, 2 firewall)

```
router ospf 35
router-id 1.1.2.2

network 10.2.2.8 0.0.0.3 area 0
network 10.2.2.12 0.0.0.3 area 0
network 192.168.10.0 0.0.0.255 area 0
network 10.20.0.0 0.0.15.255 area 0
network 10.30.0.0 0.0.15.255 area 0
network 10.40.0.0 0.0.15.255 area 0
network 10.50.0.0 0.0.15.255 area 0
network 172.16.0.0 0.0.0.255 area 0
ex

do wr
```

* VNPT-ISP: Advertise 3 network (2 firewall, cloud)

```
router ospf 35
router-id 1.1.3.3

network 105.100.50.0 0.0.0.3 area 0
network 105.100.50.4 0.0.0.3 area 0
network 20.20.20.0 0.0.0.3 area 0
ex

do wr
```

* FPT-ISP: Advertise 3 network (2 firewall, cloud)

```
router ospf 35
router-id 1.1.4.4

network 197.200.100.0 0.0.0.3 area 0
network 197.200.100.4 0.0.0.3 area 0
network 30.30.30.0 0.0.0.3 area 0
ex

do wr
```

* Cloud router: Advertise 2 network (2 ISP)

```
router ospf 35
router-id 1.1.5.5

network 8.0.0.0 0.255.255.255 area 0
network 20.20.20.0 0.0.0.3 area 0
network 30.30.30.0 0.0.0.3 area 0
ex

do wr
```

* We can recluster the cloud (router + switch)

## 9. Firewall interface security zones and levels (2:33:20)

* Most important part in topology
* CORE-SW1: not show firewall because they are currently down
```
show ip ospf neighbor
```
* PERIMETER-FW1: 1 DMZ, 2 INSIDE ZONE, 2 OUTSIDE ZONE
  * From CLI, type `en` and enter (no password), then `conf t`
  * Connect to CORE-SW1 (gig1/3), CORE-SW2 (gig1/4), DMZ (gig1/5), VNPT (gig1/1), FPT (gig1/2) 
  * we trust everything inside, no trust outside, partial trust DMZ
```
hostname FWL1

int gig1/3
no shut
ip add 10.2.2.2 255.255.255.252
nameif INSIDE1
security-level 100
ex

int gig1/4
no shut
ip add 10.2.2.10 255.255.255.252
nameif INSIDE2
security-level 100
ex

int gig1/5
no shut
ip add 172.16.1.1 255.255.255.0
nameif DMZ
security-level 70
ex

int gig1/1
no shut
ip add 105.100.50.2 255.255.255.252
nameif OUTSIDE1
security-level 0
ex

int gig1/2
no shut
ip add 197.200.100.2 255.255.255.252
nameif OUTSIDE2
security-level 0
ex

wr mem
```
* PERIMETER-FW2: 2 INSIDE ZONE, 2 OUTSIDE ZONE
  * Connect to CORE-SW1 (gig1/3), CORE-SW2 (gig1/4), VNPT (gig1/1), FPT (gig1/2) 

```
hostname FWL2

int gig1/3
no shut
ip add 10.2.2.6 255.255.255.252
nameif INSIDE1
security-level 100
ex

int gig1/4
no shut
ip add 10.2.2.14 255.255.255.252
nameif INSIDE2
security-level 100
ex

int gig1/1
no shut
ip add 105.100.50.6 255.255.255.252
nameif OUTSIDE1
security-level 0
ex

int gig1/2
no shut
ip add 197.200.100.6 255.255.255.252
nameif OUTSIDE2
security-level 0
ex

wr mem
```

* By default block traffic from low security level to high (OUTSIDE to INSIDE)

### 9.1. Firewall routing - OSPF + Static routes (2:45:45)

* 2 static route: Default static route + Backup
* FW1: OUTSIDE1 is main, OUTSIDE2 is backup

```
route OUTSIDE1 0.0.0.0 0.0.0.0 105.100.50.1
route OUTSIDE2 0.0.0.0 0.0.0.0 197.200.100.1 70

router ospf 35
router-id 1.1.8.8
network 105.100.50.0 255.255.255.252 area 0
network 197.200.100.0 255.255.255.252 area 0

network 172.16.1.0 255.255.255.0 area 0

network 10.2.2.0 255.255.255.252 area 0
network 10.2.2.8 255.255.255.252 area 0
exit

wr mem
```
* FW2: OUTSIDE2 is main, OUTSIDE1 is backup

```
route OUTSIDE2 0.0.0.0 0.0.0.0 197.200.100.5
route OUTSIDE1 0.0.0.0 0.0.0.0 105.100.50.5 70

router ospf 35
router-id 1.1.9.9
network 197.200.100.4 255.255.255.252 area 0
network 105.100.50.4 255.255.255.252 area 0

network 10.2.2.4 255.255.255.252 area 0
network 10.2.2.12 255.255.255.252 area 0
exit

wr mem
```

## 10. Firewall inspection policy configuration (2:55:10)

* NAT for IT, MEDICAL, ADMIN, WLAN, DMZ, no need for CAMERA & INSIDE-SERVERS to access Internet
* Object network name unique but don't matter, what matter is NAT
  * FW1:
```
object network IT-INSIDE1-OUTSIDE1
subnet 192.168.10.0 255.255.255.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network IT-INSIDE2-OUTSIDE1
subnet 192.168.10.0 255.255.255.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network IT-INSIDE1-OUTSIDE2
subnet 192.168.10.0 255.255.255.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network IT-INSIDE2-OUTSIDE2
subnet 192.168.10.0 255.255.255.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network MEDICAL-INSIDE1-OUTSIDE1
subnet 10.20.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network MEDICAL-INSIDE2-OUTSIDE1
subnet 10.20.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network MEDICAL-INSIDE1-OUTSIDE2
subnet 10.20.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network MEDICAL-INSIDE2-OUTSIDE2
subnet 10.20.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network ADMIN-INSIDE1-OUTSIDE1
subnet 10.30.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network ADMIN-INSIDE2-OUTSIDE1
subnet 10.30.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network ADMIN-INSIDE1-OUTSIDE2
subnet 10.30.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network ADMIN-INSIDE2-OUTSIDE2
subnet 10.30.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network WLAN-INSIDE1-OUTSIDE1
subnet 10.50.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network WLAN-INSIDE2-OUTSIDE1
subnet 10.50.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network WLAN-INSIDE1-OUTSIDE2
subnet 10.50.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network WLAN-INSIDE2-OUTSIDE2
subnet 10.50.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network DMZ-OUTSIDE1
subnet 172.16.1.0 255.255.255.0
nat (DMZ,OUTSIDE1) dynamic interface

object network DMZ-OUTSIDE2
subnet 172.16.1.0 255.255.255.0
nat (DMZ,OUTSIDE2) dynamic interface

wr mem
ex
conf t
```

    * To show config
```
show start
```
  * FW2: No DMZ
```
object network IT-INSIDE1-OUTSIDE1
subnet 192.168.10.0 255.255.255.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network IT-INSIDE2-OUTSIDE1
subnet 192.168.10.0 255.255.255.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network IT-INSIDE1-OUTSIDE2
subnet 192.168.10.0 255.255.255.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network IT-INSIDE2-OUTSIDE2
subnet 192.168.10.0 255.255.255.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network MEDICAL-INSIDE1-OUTSIDE1
subnet 10.20.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network MEDICAL-INSIDE2-OUTSIDE1
subnet 10.20.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network MEDICAL-INSIDE1-OUTSIDE2
subnet 10.20.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network MEDICAL-INSIDE2-OUTSIDE2
subnet 10.20.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network ADMIN-INSIDE1-OUTSIDE1
subnet 10.30.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network ADMIN-INSIDE2-OUTSIDE1
subnet 10.30.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network ADMIN-INSIDE1-OUTSIDE2
subnet 10.30.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network ADMIN-INSIDE2-OUTSIDE2
subnet 10.30.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

ex
conf t

object network WLAN-INSIDE1-OUTSIDE1
subnet 10.50.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE1) dynamic interface

object network WLAN-INSIDE2-OUTSIDE1
subnet 10.50.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE1) dynamic interface

object network WLAN-INSIDE1-OUTSIDE2
subnet 10.50.0.0 255.255.240.0
nat (INSIDE1,OUTSIDE2) dynamic interface

object network WLAN-INSIDE2-OUTSIDE2
subnet 10.50.0.0 255.255.240.0
nat (INSIDE2,OUTSIDE2) dynamic interface

wr mem
ex
conf t
```

* Inspection Policy
  * FW1: 
```
access-list RES extended permit icmp any any
access-list RES extended permit tcp any any eq 80
access-list RES extended permit tcp any any eq 53
access-list RES extended permit udp any any eq 53

access-group RES in interface DMZ
access-group RES in interface OUTSIDE1
access-group RES in interface OUTSIDE2

wr mem
```
  * FW2: No DMZ
```
access-list RES extended permit icmp any any
access-list RES extended permit tcp any any eq 80
access-list RES extended permit tcp any any eq 53
access-list RES extended permit udp any any eq 53

access-group RES in interface OUTSIDE1
access-group RES in interface OUTSIDE2

wr mem
```

* Test DHCP and ping: 3:13:20

## 11. Wireless network configuration (3:15:30)

* WLC: Config -> Management 
  * IP: 10.50.0.10
  * Subnet mask: 255.255.240.0
  * Default gateway: 10.50.0.1
  * DNS: 172.16.0.7
* Connect 1 PC to WLC
* Desktop -> IP configuration
  * IP: 10.50.0.100
  * Subnet mask: 255.255.240.0
  * Default gateway: 10.50.0.1
  * DNS: 172.16.0.7
* Desktop -> Web browser -> Access 10.50.0.10
  * Register
    * Admin username: cisco
    * Password: i-love-HCMUT
  * Set up
    * System Name: CISCO-HOSPITAL
    * Management IP address: 10.50.0.10
    * Subnet mask: 255.255.240.0
    * Default gateway: 10.50.0.1
  * Wireless network screen
    * Network name: EMPLOYEE
    * Passphrase: i-love-HCMUT
  * Click apply and wait until popup, then close web browser
* Desktop -> Web browser -> Access https://10.50.0.10
  * Enter username & password
  * Wireless tab
    * See 11 LAP
  * Click WLANs tab
  * Click Create New Go
  * Guest: Profile name (GUEST WIFI), SSID (GUEST), ID (2), then click Apply
    * Click Enabled
    * Go to Security tab
    * Layer 2 Security: WPA+WPA2
    * Click WPA2 Policy box
    * Click PSK Enable
    * PSK format password: i-love-HCMUT
    * Click Apply and wait for popup

* Choose a laptop/smartphone/tablet
  * Config -> Wireless0
    * SSID: EMPLOYEE (or GUEST)
    * Authentication: WPA2-PSK, write password (i-love-HCMUT)

* If incorrect IP for devices: turn Packet Tracer off and on

