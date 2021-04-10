<h2>Virtual Routing and Forwarding (VRF)</h2>

VRF (Virtual Routing and Forwarding) is a technology which allows to have more 
than one routing table on a single router. The concept of VRFs on routers is similar 
to VLANs on switches. VRFs are typically used in combination with MPLS VPNs. 
VRFs without MPLS is called VRF lite.

<h3> 1. Network Diagram</h3>

In this scenario, a service provider named ISP_XXX has two customers:

- Customer_A
- Customer_B

![image](https://user-images.githubusercontent.com/63696723/114263183-4901b380-9a0e-11eb-8e49-bea5d0b6c07f.png)

ISP_XXX uses a signle router named R1 and it is shared for both customers.
Interface e0/0 on R1 connected to a switch and the switch connected to 
each of the customer network.

The goal is to make Customer_A network able to access Loopback0 address and
Customer_B must be able to access Loopback1 address. However, for some reason both customers need to use the same 
network address but they refuse to expose their network to each other. Therefore, separate vlan is used.

<h3>2. Configuration roadmap</h3>

<h4>2.1 Basic configuration</h4>

- SW1

```
vlan 100
 name Customer_A
 exit
vlan 200
 name Customer_B
 exit
 
int range e0/0-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed VLan 100,200
 end
write
```

- SW2

```
vlan 100
 exit

int e0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed VLan 100
 exit

int vlan 100
 ip address 192.168.0.100 255.255.255.0
 end
 
write
```

- SW3

```
vlan 200
 exit

int e0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed VLan 200
 exit

int vlan 200
 ip address 192.168.0.200 255.255.255.0
 end
 
write
```

- R1

```
enable
conf terminal

hostname R1
int lo0
 no shutdown
 exit
int lo1
 no shutdown
 exit

 
int e0/0
 no shut
int e0/0.100
 encapsulation dot1q 100
 description Customer_A
int e0/0.200
 encapsulation dot1q 200
 description Customer_B
 exit
 ```
 
<h4>2.2 VRF configuration</h4>

- R1, Create vrf instance

```
ip vrf Customer_A
 exit
ip vrf Customer_B
 exit
```

To see list of VRF has been created:

```
R1#show ip vrf
  Name                             Default RD            Interfaces
  Customer_A                       <not set>
  Customer_B                       <not set>
R1#
```

On the output above, the interfaces column is blank because we haven’t assigned 
the VRF to any interface yet. This will be done in the next step.


- Assigning VRF to an interface

```
int e0/0.100
 ip vrf forwarding Customer_A
 ip address 192.168.0.1 255.255.255.0
 exit
int lo0 
 ip vrf forwarding Customer_A
 ip address 1.1.1.1 255.255.255.255
 exit
 
int e0/0.200
 ip vrf forwarding Customer_B
 ip address 192.168.0.1 255.255.255.0
 exit
int lo1
 ip vrf forwarding Customer_B
 ip address 2.2.2.2 255.255.255.255
 exit
```

- Show ip vrf again:

```
R1#show ip v
*Apr 10 09:00:02.242: %SYS-5-CONFIG_I: Configured from console by console
R1#show ip vrf
  Name                             Default RD            Interfaces
  Customer_A                       <not set>             Et0/0.100
                                                         Lo0
  Customer_B                       <not set>             Et0/0.200
                                                         Lo1
R1#
```

- Check routing tabel (global routing table, vrf):

```
R1#show ip route  | begin Gateway
Gateway of last resort is not set

R1#
R1#show ip route vrf Customer_A | begin Gateway
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
      192.168.0.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.0.0/24 is directly connected, Ethernet0/0.100
L        192.168.0.1/32 is directly connected, Ethernet0/0.100
R1#
R1#
R1#show ip route vrf Customer_B | begin Gateway
Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback1
      192.168.0.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.0.0/24 is directly connected, Ethernet0/0.200
L        192.168.0.1/32 is directly connected, Ethernet0/0.200
R1#
```

From SW2, SW3 need add route to reach loopback(0,1)

- SW2

```
ip route 0.0.0.0 0.0.0.0 192.168.0.1
```

- SW3

```
ip route 0.0.0.0 0.0.0.0 192.168.0.1
```

Now SW1 can ping Loopback0

```
SW2#ping 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
SW2#

R1#ping 192.168.0.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.100, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
R1#
R1#ping vrf Customer_A 192.168.0.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R1#

```

- Notice that, Customer_A and Customer_B also has the same network, but cannot 
access each other because their network is completely separate.

```
SW2#ping 192.168.0.200
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.200, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
SW2#
```

-------------------- **DONE** ----------------------------
