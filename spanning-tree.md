<h2>Spanning Tree</h2>

<h3> 1. STP port stats</h3>

![image](https://user-images.githubusercontent.com/63696723/143674117-09d4e243-8f15-440f-8dfe-1887eb881185.png)

 <h3>2. STP operation</h3>

- Root Bridge: Lowest Bridge ID
  - BID: Priority + MAC address

- Root ports: (non-root, 1 switch has 1 root port)
  - Lowest Path cost to root bridge (from the port)
  - Lowest Sender BID
  - Lowest Port Priority

- Designated ports & Non-Designated ports
  - Lowest path cost to the bridge (from the switch)
  - Lowest BID


<h3>3. Lab</h3>

![image](https://user-images.githubusercontent.com/63696723/143675873-817c78ac-0d34-4768-ab37-85bb9a82c873.png)

<h4>3.1. Default</h4>

- SW1
```
SW1#show spanning-tree  

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Desg FWD 100       128.2    Shr 
Et0/2               Desg FWD 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 


SW1#
```

- SW2
```
SW2#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Desg FWD 100       128.2    Shr 
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 


SW2#
```

- SW3
```
SW3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Altn BLK 100       128.3    Shr 
Et0/3               Root FWD 100       128.4    Shr 


SW3#
```
<h4>3.2. Make a switch to be Root Bridge</h4>

- Has 2 methods: 
  - Decrease priority that switch (recommend)
  - Command root primary (when all switches are default priority)

```
SW3(config)#spanning-tree vlan 1 priority ?
  <0-61440>  bridge priority in increments of 4096

SW3(config)#spanning-tree vlan 1 priority 1 ?
  <cr>

SW3(config)#spanning-tree vlan 1 priority 1 
% Bridge Priority must be in increments of 4096.
% Allowed values are: 
  0     4096  8192  12288 16384 20480 24576 28672
  32768 36864 40960 45056 49152 53248 57344 61440
SW3(config)#spanning-tree vlan 1 priority 12288 
SW3(config)#
SW3(config)#
SW3(config)#
SW3(config)#
SW3(config)#do show span
SW3(config)#do show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    12289
             Address     aabb.cc00.3000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    12289  (priority 12288 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Desg LIS 100       128.2    Shr 
Et0/2               Desg LIS 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 


SW3(config)#
```

Second method (priority will be 32678 - 8192 = 24576):
```
SW3(config)#spanning-tree vlan 1 root ?
  primary    Configure this switch as primary root for this spanning tree
  secondary  Configure switch as secondary root

SW3(config)#spanning-tree vlan 1 root pri
SW3(config)#spanning-tree vlan 1 root primary ?
  diameter  Network diameter of this spanning tree
  <cr>

SW3(config)#spanning-tree vlan 1 root primary 
SW3(config)#
SW3(config)#
SW3(config)#do show spanning-tree             

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    24577
             Address     aabb.cc00.3000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24577  (priority 24576 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Desg FWD 100       128.2    Shr 
Et0/2               Desg FWD 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 


SW3(config)#
```



<h4>3.3. STP path manipulation</h4>

Supose that we will modify SW3 port e0/3 from **root port** to **block port**

- Default:
```
SW3# show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Altn BLK 100       128.3    Shr 
Et0/3               Root FWD 100       128.4    Shr 


SW3#
```

- Modify cost of interface
```
SW3(config-if)#spanning-tree cost 500

SW3#show spanning-tree vlan 1

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        200
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Root FWD 100       128.2    Shr 
Et0/2               Altn BLK 100       128.3    Shr 
Et0/3               Altn BLK 500       128.4    Shr 


SW3#
```

So now SW3, port e0/3 is blocked, e0/1 is root, e0/2 is blocked.

- Now we want modify port e0/2 to *root port*, => modify sender port cost (port e0/3 of switch SW2) from 128.4 to 64.

```
SW2(config)#int e0/3
SW2(config-if)#spanning-tree port-priority 64
SW2# show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Desg FWD 100       128.2    Shr 
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Desg FWD 100        64.4    Shr 


SW2#
```

Here result on SW3:
```
SW3# show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        200
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Altn BLK 500       128.4    Shr 


SW3#
```




