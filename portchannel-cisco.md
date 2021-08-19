Portchannel Cisco
==================

<h4>1. Topology</h4>

![image](https://user-images.githubusercontent.com/63696723/130017397-a4b41aa3-9cee-4786-8c7c-9ba615c540b6.png)

<h3>2. Reset Cisco switch port/interface to default
  
```
Switch# conf t
Switch(config)# default interface Gi1/0/1
Switch(config)# exit  
```
  <h4>3. Etherchannel between SW-core3 & SW-core4 (LACP)</h4>
  
 - SW-core3
  
```
int range e0/0-1
  shutdown
  switchport trunk encapsulation dot1q
  switchport mode trunk
  channel-group 34 mode active
  no shutdown
```
  
 - SW-core4
  
```
int range e0/0-1
  shutdown
  switchport trunk encapsulation dot1q
  switchport mode trunk
  channel-group 34 mode active
  no shutdown
```
  
  <h4>4. Check portchannel 34</h4>
  
  ![image](https://user-images.githubusercontent.com/63696723/130019660-9f399aeb-4c75-4777-a03e-6eb8bce22fd5.png)
  
- Allow vlan on portchannel 34
  
```
 interface Port-channel34
   switchport trunk allowed vlan 10
```
  
  <h4>5. Etherchannel port between Cisco switch & Mikrotik (static mode: ON)</h4>
  
   - SW-core3
  
```
int range e1/0-1
  shutdown
  switchport trunk encapsulation dot1q
  switchport mode trunk
  channel-group 1 mode on
  no shutdown
```
  
  - Mikrotik 1
  
```
/interface bonding
add name=Bonding-1 slaves=ether1,ether2
/interface vlan
add interface=Bonding-1 mtu=1496 name=MNGT vlan-id=10
/ip address
add address=172.30.11.17/24 interface=MNGT network=172.30.11.0
```

![image](https://user-images.githubusercontent.com/63696723/130021930-3e4512e8-7461-42fe-b284-70c2a6405b71.png)
  
![image](https://user-images.githubusercontent.com/63696723/130022130-0db2bbf8-a3ac-44a6-8460-d87caa557a3c.png)
  
![image](https://user-images.githubusercontent.com/63696723/130022433-ba381ce8-7631-4262-8132-23baaf170150.png)
