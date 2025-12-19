# WLC + VLAN

https://youtu.be/00gzQbIYJEw

## Set up (2:00)

* IP Plan
  * VLAN 10: 192.168.10.0/24 (IT)
  * VLAN 20: 192.168.20.0/24 (HR)
  * VLAN 30: 192.168.30.0/24 (FIN)
  * VLAN 50: 192.168.50.0/24 (MGT & NATIVE)
  * Server: 192.168.1.0/24 (DHCP)

* Devices:
  * 2960 Switch
  * Server
  * 2911 Router
  * 3504 WLC (new)
  * 3702i LAP (new) x4
  * Tablet x4

## Switch

* All 6 port of Switch is trunk: 

```
vlan 10
name IT
vlan 20
name HR
vlan 30
name FIN
vlan 50
name MGT

int range fa0/1-6
switchport mode trunk
switchport trunk native vlan 50
exit

do wr
```

## Router

* Router: Inter VLAN routing
* gig0/1 connect to switch, gig0/0 connect to Server
  * Turn them on in Config -> interface  
* Edit IP of router: gig0/0
  * IP: 192.168.1.1
  * Subnet mask: 255.255.255.0

```
interface GigabitEthernet0/1
interface GigabitEthernet0/1.10
encapsulation dot1Q 10
ip add 192.168.10.1 255.255.255.0
ip helper-address 192.168.1.5
exit

interface GigabitEthernet0/1
interface GigabitEthernet0/1.20
encapsulation dot1Q 20
ip add 192.168.20.1 255.255.255.0
ip helper-address 192.168.1.5
exit

interface GigabitEthernet0/1
interface GigabitEthernet0/1.30
encapsulation dot1Q 30
ip add 192.168.30.1 255.255.255.0
ip helper-address 192.168.1.5
exit

interface GigabitEthernet0/1
interface GigabitEthernet0/1.50
encapsulation dot1Q 50
ip add 192.168.50.1 255.255.255.0
ip helper-address 192.168.1.5
encapsulation dot1Q 50 native
exit

do wr
```

```
do show star
```

## DHCP server

* Desktop -> Configure IP:
  * IP: 192.168.1.5
  * Subnet mask: 255.255.255.0
  * Default gateway: 192.168.1.1
  * DNS: 192.168.1.5

* Service -> DHCP
  * Turn on, turn everything 0, save
  * ITPool:
    * Default gateway: 192.168.10.1
    * DNS: 192.168.1.5
    * Start IP: 192.168.10.11
    * Mask: 255.255.255.0
    * Max user: 200
    * WLC: 192.168.50.5 (under management vlan)
    * Click add
  * HRPool:
    * Default gateway: 192.168.20.1
    * DNS: 192.168.1.5
    * Start IP: 192.168.20.11
    * Mask: 255.255.255.0
    * Max user: 200
    * WLC: 192.168.50.5 (under management vlan)
    * Click add
  * FINPool:
    * Default gateway: 192.168.30.1
    * DNS: 192.168.1.5
    * Start IP: 192.168.30.11
    * Mask: 255.255.255.0
    * Max user: 200
    * WLC: 192.168.50.5 (under management vlan)
    * Click add
  * MGTPool:
    * Default gateway: 192.168.50.1
    * DNS: 192.168.1.5
    * Start IP: 192.168.50.11
    * Mask: 255.255.255.0
    * Max user: 200
    * WLC: 192.168.50.5 (under management vlan)
    * Click add

## WLC

* Connect 1 PC to WLC
* WLC -> Config -> Management:
  * IP: 192.168.50.5
  * Subnet mask: 255.255.255.0
  * Default gateway: 192.168.50.1
  * DNS: 192.168.1.5
* PC -> Desktop -> IP config
  * IP: 192.168.50.6
  * Subnet mask: 255.255.255.0
  * Default gateway: 192.168.50.1
  * DNS: 0.0.0.0
* Try to ping 192.168.50.5 from PC
* PC -> Web browser -> access 192.168.50.5
  * username (cisco), password (i-love-HCMUT)
  * System name: GTECH-WLC
  * IP: 192.168.50.5
  * Subnet mask: 255.255.255.0
  * Default gateway: 192.168.50.1
  * Management VLAN ID: 50 (Same as MGT VLAN)
  * Click Next

  * Network name: test@1234 (delete later)
  * Passphrase: test@1234
  * Click Next -> Apply -> OK -> Close window

* Reattach PC to Switch port fa0/7, need to be in same VLAN as WLC
  * Switch -> Config -> fa0/7 -> choose VLAN 50
  * Try to ping 192.168.50.5 from PC again
* PC -> Browser -> https://192.168.50.5
  * Login with username & password (admin, i-love-HCMUT)
  * Check WIRELESS tab, everything is blank
  * Go to every LAP, connect power
  * CONTROLLER tab, click Interfaces. We need to create for vlan 10,20,30,50
  * VLAN 10:
    * Click New
    * Interface name: VLAN10; VLAN id: 10
    * IMPORTANT: Port number is the port of WLC that connect to switch (Gig1) -> Port number 1
    * Hover over router to see IP
    * IP: 192.168.10.7 (not occupied)
    * Netmask: 255.255.255.0
    * Gateway: 192.168.10.1 (router)
    * DHCP: 192.168.1.5
    * Click Apply, mayber twice
  * VLAN 20:
    * Click New
    * Interface name: VLAN20; VLAN id: 20
    * Port number: 1
    * IP: 192.168.20.7 (not occupied)
    * Netmask: 255.255.255.0
    * Gateway: 192.168.20.1 (router)
    * DHCP: 192.168.1.5
    * Click Apply, mayber twice
  * VLAN 30:
    * Click New
    * Interface name: VLAN30; VLAN id: 30
    * Port number: 1
    * IP: 192.168.30.7 (not occupied)
    * Netmask: 255.255.255.0
    * Gateway: 192.168.30.1 (router)
    * DHCP: 192.168.1.5
    * Click Apply, mayber twice
  * VLAN 50 is already created (management)
  * Go to WLAN tab
  * Remove test WIFI
  * Create new GO for IT
    * Profile name: IT-WIFI
    * SSID: IT
    * Click Apply
    * Status: Enabled
    * Interface: VLAN10 (IMPORTANT)
    * Click TAB Security: WPA+WPA2
    * Click WPA2
    * Click PSK
    * Password: i-love-HCMUT
    * Click TAB Advanced
    * Scroll down, click Enable on FlexConnect Local Switching
    * Click enable on FlexConnect Local Auth
    * Click Apply (twice)


  * Create new GO for HR
    * Profile name: HR-WIFI
    * SSID: HR
    * Click Apply
    * Status: Enabled
    * Interface: VLAN20 (IMPORTANT)
    * Click TAB Security: WPA+WPA2
    * Click WPA2
    * Click PSK
    * Password: i-love-HCMUT
    * Click TAB Advanced
    * Scroll down, click Enable on FlexConnect Local Switching
    * Click enable on FlexConnect Local Auth
    * Click Apply (twice)

  * Create new GO for FIN
    * Profile name: FIN-WIFI
    * SSID: FIN
    * Click Apply
    * Status: Enabled
    * Interface: VLAN30 (IMPORTANT)
    * Click TAB Security: WPA+WPA2
    * Click WPA2
    * Click PSK
    * Password: i-love-HCMUT
    * Click TAB Advanced
    * Scroll down, click Enable on FlexConnect Local Switching
    * Click enable on FlexConnect Local Auth
    * Click Apply (twice)

  * Create new GO for MGT
    * Profile name: MGT-WIFI
    * SSID: MGT
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
  * DHCP: 192.168.1.5
  * Click Apply twice
  * Save Packet Tracer
  * Click Save Configuration (on top bar)

* Go to each LAP
  * Config -> Global -> DHCP


















