## DHCP & NAT


![image](https://user-images.githubusercontent.com/63696723/127296903-501d3293-9c9c-4f2b-a265-e096fe0bd6f1.png)

### Switch Local-12

```
interface Ethernet0/0
 switchport access vlan 215
 switchport mode access
!
interface Ethernet0/1
 switchport trunk allowed vlan 215,216
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 216
 switchport mode access
!
```

### Switch Local-13

```
interface Ethernet0/0
 switchport trunk allowed vlan 52
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 52
 switchport mode access
!
interface Ethernet0/2
 switchport trunk allowed vlan 52
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
```

### Router R7

```
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.52
 encapsulation dot1Q 52
 ip address x.x.x.1 255.255.255.248
!
```

### Router R1

```
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.52
 description NAT-out
 encapsulation dot1Q 52
 ip address x.x.x.2 255.255.255.248
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/1
 no ip address
!
interface Ethernet0/1.215
 encapsulation dot1Q 215
 ip address 10.12.0.1 255.255.248.0
 ip nat inside
 ip virtual-reassembly in
!
interface Ethernet0/1.216
 encapsulation dot1Q 216
 ip address 10.12.8.1 255.255.248.0
 ip nat inside
 ip virtual-reassembly in
!

ip nat inside source list WIFI1 interface Ethernet0/0.52 overload
ip nat inside source list WIFI2 interface Ethernet0/0.52 overload
!
ip access-list standard WIFI2
 permit 10.12.8.0 0.0.7.255
ip access-list standard WIFI1
 permit 10.12.0.0 0.0.7.255
!
!
route-map WIFI1 permit 10
 match ip address 30 WIFI1
 set ip next-hop x.x.x.1

```

### VPC4

![image](https://user-images.githubusercontent.com/63696723/127298613-7e3ce76a-72d2-4108-ab6e-310bd2f0c9ca.png)

### R1

![image](https://user-images.githubusercontent.com/63696723/127299109-39ed93b5-b7ad-45a7-9f6f-1a6b1f969537.png)
