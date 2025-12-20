# Hospital network

|      Name     | VLAN |      Subnet     |                 |                 |
|:-------------:|:----:|:---------------:|:---------------:|:---------------:|
|               |      |       Main      |       DBP       |       BHTQ      |
| IT            | 10   | 192.168.10.0/24 | 192.168.11.0/24 | 192.168.12.0/24 |
| MEDICAL       | 20   | 10.20.0.0/20    | 10.21.0.0/20    | 10.22.0.0/20    |
| ADMIN         | 30   | 10.30.0.0/20    | 10.31.0.0/20    | 10.32.0.0/20    |
| CAMERA        | 40   | 10.40.0.0/20    | 10.41.0.0/20    | 10.42.0.0/20    |
| WLAN          | 50   | 10.50.0.0/20    | 10.51.0.0/20    | 10.52.0.0/20    |
| WLAN-GUEST    | 60   | 10.60.0.0/20    | 10.61.0.0/20    | 10.62.0.0/20    |
| MED-DEVICE    | 70   | 10.70.0.0/20    | 10.71.0.0/20    | 10.72.0.0/20    |
| INSIDE-SERVER | 90   | 172.16.0.0/24   | 172.17.0.0/24   | 172.18.0.0/24   |
| DMZ           | None | 172.16.1.0/24   | None            | None            |

## 0. Network design and beautification (19:00)

* Network devices:
  * 2911 router x1: BHTQ-ROUTER (BHTQ), add HWIC-2T (for DCE serial)
  * 3650 multilayer switch x2: add AC power supply
  * 2960 switch x4 for 1 for INSIDE SERVERS, 1 for IT room, 1 ADMIN room, 1 MEDICAL room
* Wires:
  * Top down, automatic
  * Access switch to multilayer switch: Use gig0/1-2 instead of fa0/1-24
  * EtherChannel: 2x between 3650
* Host devices:
  * PC
  * Printer (mocking medical device)
  * Laptop: add wireless module
  * Smartphone (mocking guest device)
  * 3702i LAP: add power adapter
  * Camera: config -> wireless0 -> Turn port status Off
* Inside Server (server farm) switch:
  * Server x2: DHCP, DNS
  * 3504 WLC: same VLAN as LAP
  * Connect wire in that order

* WAN:
  * Serial DCE: Connect WAN-ROUTER to BHTQ-ROUTER
* Name devices, background color


## 1. Basic settings to all devices + SSH + Standard ACL for SSH (38:45)

* Basic setting for switches, not firewall, leave for last
* Hostname, console password, enable password, banner, password encryption, disable IP domain lookup
* Command: `en` and `conf t` first. Change hostname for each switch
  * Access switch: SW-BHTQ1, SW-BHTQ2, SW-BHTQ3, SW-BHTQ4 
  * Core switch (l3/multilayer switch): BHTQ-CORE1, BHTQ-CORE2
```
hostname SW-BHTQ1

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
* Inside Server (SW-BHTQ1):
  * Server: fa0/1-20
  * WLC in VLAN 50: fa0/21-24
* IT room (SW-BHTQ2):
  * IT: fa0/1-16
  * CAMERA: fa0/17-20
  * LAP: fa0/21-24
* ADMIN room (SW-BHTQ3):
  * ADMIN: fa0/1-8
  * MED-DEVICE: fa0/9-16
  * CAMERA: fa0/17-20
  * LAP: fa0/21-24
* MEDICAL room (SW-BHTQ4):
  * MEDICAL: fa0/1-8
  * MED-DEVICE: fa0/9-16
  * CAMERA: fa0/17-20
  * LAP: fa0/21-24
* ADMIN room (SW-BHTQ3):

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
vlan 60
name WLAN-GUEST
vlan 70
name MED-DEVICE
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-8
switchport mode access
switchport access vlan 30
ex

int range fa0/9-16
switchport mode access
switchport access vlan 70
ex

int range fa0/17-20
switchport mode access
switchport access vlan 40
ex

int range fa0/21-24
switchport mode trunk
switchport trunk native vlan 50
ex

do wr
```
* MEDICAL room (SW-BHTQ4):

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
vlan 60
name WLAN-GUEST
vlan 70
name MED-DEVICE
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-8
switchport mode access
switchport access vlan 20
ex

int range fa0/9-16
switchport mode access
switchport access vlan 70
ex

int range fa0/17-20
switchport mode access
switchport access vlan 40
ex

int range fa0/21-24
switchport mode trunk
switchport trunk native vlan 50
ex

do wr
```
* IT room (SW-BHTQ2):

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
vlan 60
name WLAN-GUEST
vlan 70
name MED-DEVICE
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
switchport mode trunk
switchport trunk native vlan 50
ex

do wr
```

* Inside Server (SW-BHTQ1):
  * Server: fa0/1-20
  * WLC in VLAN 50: fa0/21-24

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
vlan 60
name WLAN-GUEST
vlan 70
name MED-DEVICE
vlan 90
name INSIDE-SERVERS
ex

int range fa0/1-20
switchport mode access
switchport access vlan 90
ex

int range fa0/21-24
switchport mode trunk
switchport trunk native vlan 50
ex

do wr
```

### Multilayer switch (1:23:50)

* Connection:
  * gig1/0/1: BHTQ-ROUTER
  * gig1/0/2-5: Access switch
  * gig1/0/6-7: core-core (redundant)
* Trunk port:
  * CORE-SW1, CORE-SW2: gig1/0/2-5

```
int range gig1/0/2-5
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
vlan 60
name WLAN-GUEST
vlan 70
name MED-DEVICE
vlan 90
name INSIDE-SERVERS
ex

do wr
```

### 2.1. STP Portfast and BPDUguard configs on all access ports (1:32:00)

* Portfast: Orange to green fast in switch
  * Access switch: 6 trunk (gig0/1-2, fa0/21-24)

```
int range fa0/1-20
spanning-tree portfast
spanning-tree bpduguard enable
ex

do wr
```

## 3. EtherChannel LACP (1:38:00)

* 2 link between core switches (gig1/0/6-7)
* active-active, active-passive, but no passive-passive
* BHTQ-CORE1:

```
int range gig1/0/6-7
channel-group 1 mode active
ex

interface Port-channel 1
switchport mode trunk
ex

do wr
```
* BHTQ-CORE2:

```
int range gig1/0/6-7
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
  * IT: 192.168.12.0/24
  * MEDICAL: 10.22.0.0/20
  * ADMIN: 10.32.0.0/20
  * CAMERA: 10.42.0.0/20
  * WLAN: 10.52.0.0/20
  * WLAN-GUEST: 10.62.0.0/20
  * MED-DEVICE: 10.72.0.0/20
  * INSIDE-SERVERS: 172.18.0.0/24

* Between Cloud, ISP, Firewall, Router, L3 Switch:
  * CORE-SW1 - WAN-ROUTER: 10.2.2.44/30
  * CORE-SW2 - WAN-ROUTER: 10.2.2.16/30
  * WAN-ROUTER - DBP-ROUTER: 10.2.2.20/30
  * DBP-CORE1 - DBP-ROUTER: 10.2.2.24/30
  * DBP-CORE2 - DBP-ROUTER: 10.2.2.28/30
  * WAN-ROUTER - BHTQ-ROUTER: 10.2.2.32/30
  * BHTQ-CORE1 - BHTQ-ROUTER: 10.2.2.36/30
  * BHTQ-CORE2 - BHTQ-ROUTER: 10.2.2.40/30
* Leave firewall for last, we config l3 switches, ISP router, cloud router
* Firewall always take second IP address (10.2.2.18)
* L3 switch: Is both switch and router
* BHTQ-CORE1:
```
ip routing

int gi1/0/1
no switchport
no shut
ip add 10.2.2.37 255.255.255.252
ex
do wr
```


* BHTQ-CORE2:
```
ip routing

int gi1/0/1
no switchport
no shut
ip add 10.2.2.41 255.255.255.252
ex

do wr
```

* BHTQ-ROUTER: GUI + CLI
  * gig0/0 connect to BHTQ-CORE1
    * Config -> gig0/0
    * Click On
    * IP: 10.2.2.38
    * Subnet mask: 255.255.255.252
  * gig0/1 connect to BHTQ-CORE2
    * Config -> gig0/1
    * Click On
    * IP: 10.2.2.42
    * Subnet mask: 255.255.255.252
  * se0/3/0 connect to WAN-ROUTER (take second IP, let WAN-ROUTER has first IP)
```
int se0/3/0
no shut
ip add 10.2.2.34 255.255.255.252
```

* WAN-ROUTER (main):
  * se0/3/1 connect to BHTQ-ROUTER
```
int se0/3/1
no shut
ip add 10.2.2.33 255.255.255.252
clock rate 64000
```

## 5. HSRP and Inter-VLAN routing on the l3 switches plus IP DHCP helper addresses (1:56:30)

* Static IP for INSIDE server:
  * DHCP: 172.18.0.6
  * DNS: 172.18.0.7


* HSRP between 2 l3 switch: Redundancy, kinda like load balancer, standby <-> active
  * Interface with highest IP address become active router
* BHTQ-CORE1: active for (10,20,30,40), standby (50,60,70,90)
```
int vlan 10
no shut
ip add 192.168.12.3 255.255.255.0
standby 10 ip 192.168.12.1
ip helper-address 172.18.0.6
exit

int vlan 20
no shut
ip add 10.22.0.3 255.255.240.0
standby 20 ip 10.22.0.1
ip helper-address 172.18.0.6
exit

int vlan 30
no shut
ip add 10.32.0.3 255.255.240.0
standby 30 ip 10.32.0.1
ip helper-address 172.18.0.6
exit

int vlan 40
no shut
ip add 10.42.0.3 255.255.240.0
standby 40 ip 10.42.0.1
ip helper-address 172.18.0.6
exit

int vlan 50
no shut
ip add 10.52.0.2 255.255.240.0
standby 50 ip 10.52.0.1
ip helper-address 172.18.0.6
exit

int vlan 60
no shut
ip add 10.62.0.2 255.255.240.0
standby 60 ip 10.62.0.1
ip helper-address 172.18.0.6
exit

int vlan 70
no shut
ip add 10.72.0.2 255.255.240.0
standby 70 ip 10.72.0.1
ip helper-address 172.18.0.6
exit

int vlan 90
no shut
ip add 172.18.0.2 255.255.255.0
standby 90 ip 172.18.0.1
exit

do wr
```

* To show result:
```
do sh star
```
  * Copy result to NotePad (4 vlan)
  * Remove MAC address

* BHTQ-CORE2: standby for (10,20,30,40), active for (50,60,70,90)
```
int vlan 10
no shut
ip add 192.168.12.2 255.255.255.0
standby 10 ip 192.168.12.1
ip helper-address 172.18.0.6
exit

int vlan 20
no shut
ip add 10.22.0.2 255.255.240.0
standby 20 ip 10.22.0.1
ip helper-address 172.18.0.6
exit

int vlan 30
no shut
ip add 10.32.0.2 255.255.240.0
standby 30 ip 10.32.0.1
ip helper-address 172.18.0.6
exit

int vlan 40
no shut
ip add 10.42.0.2 255.255.240.0
standby 40 ip 10.42.0.1
ip helper-address 172.18.0.6
exit

int vlan 50
no shut
ip add 10.52.0.3 255.255.240.0
standby 50 ip 10.52.0.1
ip helper-address 172.18.0.6
exit

int vlan 60
no shut
ip add 10.62.0.3 255.255.240.0
standby 60 ip 10.62.0.1
ip helper-address 172.18.0.6
exit

int vlan 70
no shut
ip add 10.72.0.3 255.255.240.0
standby 70 ip 10.72.0.1
ip helper-address 172.18.0.6
exit

int vlan 90
no shut
ip add 172.18.0.3 255.255.255.0
standby 90 ip 172.18.0.1
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
    * IP: 172.18.0.6
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.18.0.1 (standby IP of CORE-SW2)
    * DNS: 172.18.0.7
  * DNS:
    * IP: 172.18.0.7
    * Subnet mask: 255.255.255.0
    * Default gateway: 172.18.0.1
    * DNS: 172.18.0.7



## 7. DHCP server device configuration (2:19:25)

* Services -> DHCP
  * Turn everything to 0
  * Turn on and start edit
  * No need DHCP for INSIDE-SERVER
  * IT:
    * Pool Name: IT-Pool
    * Default gateway: 192.168.12.1
    * DNS: 172.18.0.7
    * Start IP: 192.168.12.11
    * Subnet mask: 255.255.255.0
    * Max user: 200
    * Click Add
  * MEDICAL:
    * Pool Name: MEDICAL-Pool
    * Default gateway: 10.22.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.22.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add
  * ADMIN:
    * Pool Name: ADMIN-Pool
    * Default gateway: 10.32.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.32.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add
  * CAMERA:
    * Pool Name: CAMERA-Pool
    * Default gateway: 10.42.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.42.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add
  * WLAN:
    * Pool Name: WLAN-Pool
    * Default gateway: 10.52.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.52.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * WLC address: 10.52.0.10 (assume)
    * Click Add
  * WLAN-GUEST:
    * Pool Name: WLAN-GUEST-Pool
    * Default gateway: 10.62.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.62.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * WLC address: 10.52.0.10 (assume)
    * Click Add
  * MED-DEVICE:
    * Pool Name: MED-DEVICE-Pool
    * Default gateway: 10.72.0.1
    * DNS: 172.18.0.7
    * Start IP: 10.72.0.11
    * Subnet mask: 255.255.240.0
    * Max user: 1000
    * Click Add


## 8. OSPF on the firewall, routers, and switches (2:24:00)

* Not touching firewall now, only router and l3 switch
* BHTQ-CORE1: Advertise 9 network (10, 20, 30, 40, 50, 60, 70, 90, BHTQ-ROUTER) (hover mouse over switch)

```
router ospf 35
router-id 1.2.1.1

network 10.2.2.36 0.0.0.3 area 0
network 192.168.12.0 0.0.0.255 area 0
network 10.22.0.0 0.0.15.255 area 0
network 10.32.0.0 0.0.15.255 area 0
network 10.42.0.0 0.0.15.255 area 0
network 10.52.0.0 0.0.15.255 area 0
network 10.62.0.0 0.0.15.255 area 0
network 10.72.0.0 0.0.15.255 area 0
network 172.18.0.0 0.0.0.255 area 0
ex

do wr
```

* Show network:

```
do sh star
```
* Router-id should be different
* BHTQ-CORE2: Advertise 9 network (10, 20, 30, 40, 50, 60, 70, 90, BHTQ-ROUTER)

```
router ospf 35
router-id 1.2.2.2

network 10.2.2.40 0.0.0.3 area 0
network 192.168.12.0 0.0.0.255 area 0
network 10.22.0.0 0.0.15.255 area 0
network 10.32.0.0 0.0.15.255 area 0
network 10.42.0.0 0.0.15.255 area 0
network 10.52.0.0 0.0.15.255 area 0
network 10.62.0.0 0.0.15.255 area 0
network 10.72.0.0 0.0.15.255 area 0
network 172.18.0.0 0.0.0.255 area 0
ex

do wr
```

* BHTQ-ROUTER: Advertise 3 network (2 CORE Switch, WAN-ROUTER)

```
router ospf 35
router-id 1.2.3.3

network 10.2.2.32 0.0.0.3 area 0
network 10.2.2.36 0.0.0.3 area 0
network 10.2.2.40 0.0.0.3 area 0
ex

do wr
```

* WAN-ROUTER: Advertise 4 network (2 site router, CORE-SW1 & CORE-SW2), BUT  now we only need to advertise 1 site left (BHTQ)

```
router ospf 35
router-id 1.2.4.4

network 10.2.2.32 0.0.0.3 area 0
ex

do wr
```




* Test DHCP and ping: 3:13:20
  * IMPORTANT: If can't ping Camera, turn off Wireless0 and restart

## 11. Wireless network configuration (3:15:30)

* Connect 1 PC to WLC
* WLC -> Config -> Management:
  * IP: 10.52.0.10
  * Subnet mask: 255.255.240.0
  * Default gateway: 10.52.0.1
  * DNS: 172.18.0.7
* PC -> Desktop -> IP config
  * IP: 10.52.0.9
  * Subnet mask: 255.255.240.0
  * Default gateway: 10.52.0.1
  * DNS: 172.18.0.7
* Try to ping 10.52.0.10 from PC
* PC -> Web browser -> access 10.52.0.10
  * username (cisco), password (i-love-HCMUT)
  * System name: CISCO-HOSPITAL
  * IP: 10.52.0.10
  * Subnet mask: 255.255.240.0
  * Default gateway: 10.52.0.1
  * Management VLAN ID: 50 (Same as WLAN VLAN)
  * Click Next

  * Network name: test@1234 (delete later)
  * Passphrase: test@1234
  * Click Next -> Apply -> OK -> Close window

* Reattach PC to Switch port fa0/22, need to be in same VLAN as WLC
  * Switch -> Config -> fa0/22 -> choose VLAN 50
  * Try to ping 10.52.0.10 from PC again
* PC -> Browser -> https://10.52.0.10
  * Login with username & password (cisco, i-love-HCMUT)
  * Check WIRELESS tab, everything is blank -> Go to every LAP, connect power
  * CONTROLLER tab, click Interfaces. We need to create for vlan 50,60
  * VLAN 60:
    * Click New
    * Interface name: VLAN60; VLAN id: 60
    * IMPORTANT: Port number is the port of WLC that connect to switch (Gig1) -> Port number 1
    * Hover over router to see IP
    * IP: 10.62.0.9 (not occupied)
    * Netmask: 255.255.240.0
    * Gateway: 10.62.0.1 (router)
    * DHCP: 172.18.0.6
    * Click Apply, mayber twice

  * VLAN 50 is already created (management)
  * Go to WLAN tab
  * Remove test WIFI
  * Create new GO for GUEST
    * Profile name: GUEST-WIFI
    * SSID: GUEST-BHTQ
    * Click Apply
    * Status: Enabled
    * Interface: VLAN60 (IMPORTANT)
    * Click TAB Security: WPA+WPA2
    * Click WPA2
    * Click PSK
    * Password: i-love-HCMUT
    * Click TAB Advanced
    * Scroll down, click Enable on FlexConnect Local Switching
    * Click enable on FlexConnect Local Auth
    * Click Apply (twice)

  * Create new GO for EMPLOYEE
    * Profile name: EMPLOYEE-WIFI
    * SSID: EMPLOYEE-BHTQ
    * Click Apply
    * Status: Enabled
    * Interface: management (IMPORTANT)
    * Click TAB Security: WPA+WPA2
    * Click WPA2
    * Click PSK
    * Password: i-love-HCMUT
    * Click TAB Advanced
    * Scroll down, click Enable on FlexConnect Local Switching
    * Click enable on FlexConnect Local Auth
    * Click Apply (twice)

* Go to CONTROLLER -> Interfaces -> management
  * DHCP: 172.18.0.6
  * Click Apply twice
  * Save Packet Tracer
  * Click Save Configuration (on top bar)

* Go to each LAP
  * Config -> Global -> DHCP

### 11.1. Update ACL for WLAN-GUEST

* For CORE-SW1 and CORE-SW2, and BHTQ-CORE1, BHTQ-CORE2:

```
no ip access-list extended GUEST-RESTRICT

ip access-list extended GUEST-RESTRICT

permit icmp any host 172.16.0.7
permit udp any host 172.16.0.7 eq domain
permit tcp any host 172.16.0.7 eq domain
permit icmp any host 172.16.0.6
permit udp any host 172.16.0.6 eq bootps

permit icmp any host 172.18.0.7
permit udp any host 172.18.0.7 eq domain
permit tcp any host 172.18.0.7 eq domain
permit icmp any host 172.18.0.6
permit udp any host 172.18.0.6 eq bootps

deny ip any 192.168.10.0 0.0.0.255
deny ip any 10.20.0.0 0.0.15.255
deny ip any 10.30.0.0 0.0.15.255
deny ip any 10.40.0.0 0.0.15.255
deny ip any 10.50.0.0 0.0.15.255
deny ip any 10.70.0.0 0.0.15.255
deny ip any 172.16.0.0 0.0.0.255
deny ip any 172.16.1.0 0.0.0.255

deny ip any 192.168.12.0 0.0.0.255
deny ip any 10.22.0.0 0.0.15.255
deny ip any 10.32.0.0 0.0.15.255
deny ip any 10.42.0.0 0.0.15.255
deny ip any 10.52.0.0 0.0.15.255
deny ip any 10.72.0.0 0.0.15.255
deny ip any 172.18.0.0 0.0.0.255
deny ip any 172.18.1.0 0.0.0.255

permit ip any any

ex

int vlan 60
ip access-group GUEST-RESTRICT in
ex

do wr
```


```
show access-lists GUEST-RESTRICT
```