<h1>Dynamic Host Configuration Protocol (DHCP)</h1>


<h3>1. Definition</h3>

- DHCP is a service.
- It allows devices to acquire their IP configuration dynamically. 
- It is defined in RFC 2131 and 2939. It works in the server/client model. 
- The server offers and delivers IP configurations. Clients request and acquire their IP configurations.

<h3>2. Operation</h3>

![dhcp-operation](https://user-images.githubusercontent.com/63696723/111034975-a83dc980-844a-11eb-8ac7-f69a33757b2b.png)


<h3>3. DHCP Address Allocation Methods</h3>

<h4>3.1. Static allocation</h4>

Static DHCP (aka DHCP reservation) is a useful feature which makes the DHCP server on your router always assign the 
same IP address to a specific computer on your LAN.

To be more specific, the DHCP server assigns this static IP to a unique MAC address assigned to each NIC on your LAN. 
Your computer boots and requests its IP from the router's DHCP server. The DHCP server recognizes the MAC address of 
your device's NIC and assigns the static IP address to it.

In this method, the administrator configures an allocation table on the DHCP server. In this table, 
the administrator fills the MAC addresses of all clients and assigns an IP configuration to each client.

<h4>3.2. Dynamic allocation</h4>

The DHCP server assigns a reusable IP address from IP Pools of addresses to a client for a maximum period of time, 
known as a **lease**. This method of address allocation is useful when the customer has a limited number of IP 
addresses; they can be assigned to clients who need only temporary access to the network.

<h4>3.3. Automatic allocation</h4>

The DHCP server assigns a permanent IP address to a client from its IP Pools. On the firewall, 
a Lease specified as Unlimited means the allocation is permanent.

<h3>4. Lab</h3>

<h4>4.1 Network diagram</h4>

![image](https://user-images.githubusercontent.com/63696723/111035878-02d92480-844f-11eb-9b47-801928412ace.png)

<h4>4.2. Configuration</h4>

- ISP

```
interface Loopback0
 ip address 8.8.8.8 255.255.255.255
!
interface Ethernet0/0
 ip address 192.168.1.2 255.255.255.252
!
ip route 172.16.0.0 255.255.255.0 192.168.1.1
!
```

- R1 (DHCP server for pool-172)

```
interface Ethernet0/0
 ip address 192.168.1.1 255.255.255.252
!
interface Ethernet0/1
 ip address 172.16.0.1 255.255.255.0
!
ip route 0.0.0.0 0.0.0.0 192.168.1.2
!
! #provide IP dhcp start from 0.50 to 0.254
ip dhcp excluded-address 172.16.0.1 172.16.0.49
!
ip dhcp pool pool-172
 network 172.16.0.0 255.255.255.0
 default-router 172.16.0.1
 dns-server 1.1.1.1 1.1.1.2
!
```

- VPC4

![image](https://user-images.githubusercontent.com/63696723/111036229-c60e2d00-8450-11eb-84a5-f2be27d79d2a.png)

Success!!

- Verify on R1

![image](https://user-images.githubusercontent.com/63696723/111036600-8c3e2600-8452-11eb-84c5-dbbb525e9f94.png)

Default, lease time is 1 day. By the way, you can set **lease day minute**.


- How ever, if we want to DHCP static allocation?

Example, VPC4 (MAC 00:50:79:66:68:26) we want to assign IP with 172.16.0.51.

> R1#debug ip dhcp server packet

To see client ID of VPC4.

![image](https://user-images.githubusercontent.com/63696723/111038162-1b9b0780-845a-11eb-86b6-193d947abfdb.png)

So client-identifier of VPC4 is **0100.5079.6668.26**.

Configure pool-static-vpc4 for VPC4:
```
ip dhcp pool pool-static-vpc4
 host 172.16.0.51 255.255.255.0
 client-identifier 0100.5079.6668.26
 end
!
```

Try **ip dhcp** serveral times, but still got IP the same. Bingoo!!!


- IP dhcp relay

Supose, ISP router is a DHCP server:

```
Router#show runn | sec dhcp
ip dhcp pool ISP
 network 172.16.0.0 255.255.255.0
 dns-server 9.9.9.9
 domain-name test.com
 default-router 172.16.0.1
```

R1:

```
R1#sh run int e0/1
!
interface Ethernet0/1
 description TO-VPC
 ip address 172.16.0.1 255.255.255.0
 ip helper-address 192.168.1.2
end

R1#
```

This is the command which enables hosts in R1 to dynamically obtain IP addresses from the DHCP server 
configured on Router ISP( a server located in a different broadcast domain).


Done~~.
