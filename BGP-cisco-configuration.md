<h1>Border Gateway Protocol (BGP)</h1>

<h3>1. Topology</h3>

![image](https://user-images.githubusercontent.com/63696723/130216847-f175f77c-8660-4e9a-8751-c211b3ecef32.png)

<h3>2. Configuraion each router</h3>

- AS132213

```
CNX-R1#show running-config | section bgp
router bgp 132213
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor Sabay peer-group
 neighbor Sabay remote-as 7712
 neighbor Sabay transport connection-mode passive
 neighbor ABC peer-group
 neighbor ABC remote-as 9999
 neighbor 103.7.144.9 peer-group Sabay
 neighbor 103.7.144.10 peer-group Sabay
 neighbor 103.7.144.20 peer-group ABC
 !
 address-family ipv4
  network 1.1.1.1 mask 255.255.255.255
  neighbor 103.7.144.9 activate
  neighbor 103.7.144.10 activate
  neighbor 103.7.144.20 activate
 exit-address-family
 ```
 
 - AS9999

```
router bgp 9999
 bgp router-id 7.7.7.7
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor CNX peer-group
 neighbor CNX remote-as 132213
 neighbor 103.7.144.1 peer-group CNX
 neighbor 103.7.144.2 peer-group CNX
 !
 address-family ipv4
  network 7.7.7.7 mask 255.255.255.255
  neighbor 103.7.144.1 activate
  neighbor 103.7.144.2 activate
 exit-address-family
 ```
 
 <h3>3. Verify</h3>
 
 ![image](https://user-images.githubusercontent.com/63696723/130220087-b66b0304-2441-4122-b66f-46a00d2892e5.png)
