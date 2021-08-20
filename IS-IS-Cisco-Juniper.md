<h1>IS-IS configuration</h1>

<h2>1. Introduction to IS-IS protocol</h2>

[IS-IS](https://networklessons.com/cisco/ccie-routing-switching-written/introduction-to-is-is/)
is an IGP, link-state routing protocol, similar to OSPF. It forms neighbor adjacencies, has areas, 
exchanges link-state packets, builds a link-state database and runs the Dijkstra SPF algorithm 
to find the best path to each destination, which is installed in the routing table.

<h2>2. Configuration Itegrated guide</h2>

Please check [this](https://cdn.ttgtmedia.com/searchNetworking/downloads/CCNP_BSCI_Portable_Command_Guide1587201895_CH04.pdf)
for more detail.

<h2>3. IS-IS configuration</h3>

<h3>3.1. Network diagram</h3>

![image](https://user-images.githubusercontent.com/63696723/130204855-f722d084-946e-486c-b76e-c80cb8746712.png)

<h3>3.2. Cisco router (R4)</h3>

- IP addresses assigan on interfaces

![image](https://user-images.githubusercontent.com/63696723/130205268-0eaf624c-6139-4065-b5e5-4d7d8887dd9c.png)

- IS-IS configration

```
conf t
  router is-is 0.0.0.1
    net 49.0040.0400.4004.00
    is-type level-1
    metric-style wide
    exit
    
  interface Loopback0
    ip address 4.4.4.4 255.255.255.255
    ip router isis 0.0.0.1
    exit
  interface Ethernet0/1
    ip address 172.16.0.4 255.255.255.0
    ip router isis 0.0.0.1
    end
 write
```

<h3>3.2. Juniper router (R5)</h3>

```

set interfaces ge-0/0/0 unit 0 family inet address 172.16.0.5/24
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 103.7.144.10/25
set interfaces lo0 unit 0 family inet address 5.5.5.5/32
set interfaces lo0 unit 0 family iso address 49.0050.0500.5005.00
set protocols isis interface ge-0/0/0.0
set protocols isis interface lo0.0 passive
```

<h2>4. Verify</h2>

<h3>4.1 Cisco</h3>

![image](https://user-images.githubusercontent.com/63696723/130208539-dfa4611f-39f4-4285-bfb0-b0f218e76655.png)

<h3>4.2. Juniper</h3>

![image](https://user-images.githubusercontent.com/63696723/130209214-e3e39e74-6d37-4d15-9ecd-0c386b4637dc.png)

<h3>4.3. Ping, routing table</h3>

![image](https://user-images.githubusercontent.com/63696723/130209561-c0f9e832-d6d4-4d20-a808-4d71cbd72b2a.png)

Thanks!!
