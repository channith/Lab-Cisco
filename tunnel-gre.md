<h2>Generic Routing Encapsulation - GRE</h2>


GRE stands for Generic Routing Encapsulation, which is a very simple form of tunneling. 
With GRE we can easily create a virtual link between routers and allow them to be directly connected, even if they physically aren’t.


<h3>1. Network diagram</h3>

![tunnel-gre](https://user-images.githubusercontent.com/63696723/111909057-4a366500-8a8e-11eb-86d2-eb8682724a74.png)

<h3>2. Configuration Roadmap</h3>

<h4>2.1. Basic Configuration</h4>

- R1

```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 ip address 192.168.1.1 255.255.255.0
!
interface Ethernet0/1
 ip address 172.30.12.1 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
router ospf 1
 router-id 1.1.1.1
!
```

- R2

```
interface Ethernet0/0
 ip address 172.30.12.2 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 172.30.23.2 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
router ospf 1
 router-id 2.2.2.2
!
```

- R3

```
interface Ethernet0/0
 ip address 172.30.23.3 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 172.30.34.3 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
router ospf 1
 router-id 3.3.3.3
!
```

- R4

```
interface Loopback0
 ip address 4.4.4.4 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/0
 ip address 192.168.2.1 255.255.255.0
 shutdown
!
interface Ethernet0/1
 ip address 172.30.34.4 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown
!
router ospf 1
 router-id 4.4.4.4
!
```

<h4>2.2 GRE tunnel configuration</h4>

- R1

```
interface Tunnel100
 ip address 10.100.100.1 255.255.255.252
 tunnel source 1.1.1.1
 tunnel destination 4.4.4.4
!
ip route 192.168.2.0 255.255.255.0 10.100.100.2
```

- R2

```
interface Tunnel100
 ip address 10.100.100.2 255.255.255.252
 tunnel source 4.4.4.4
 tunnel destination 1.1.1.1
!
ip route 192.168.1.0 255.255.255.0 10.100.100.1
```

Now let try ping between 2 VPC:

![image](https://user-images.githubusercontent.com/63696723/111909966-eca41780-8a91-11eb-96f6-d44c1ac36753.png)

<h3>Read more here</h3>

- https://www.9tut.com/gre-tunnel-tutorial
- https://www.9tut.com/gre-tunnel-lab
