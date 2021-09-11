<h2>BGP Route Aggregation</h2>

<h3>1. Definition</h3>

- Route Aggregation (RA) also known as BGP Route Summarization is a method 
to minimize the size of the routing table, announcing the whole address block 
received from the Regional Internet Registry (RIR) to other ASes.

- RA is opposite to non-aggregation routing, where individual sub-prefixes of 
the address block are announced to BGP peers.

- RA reduces the size of the global routing table, decreases routers’ workload 
and saves network bandwidth.

<h3>2. Network topology</h3>

![image](https://user-images.githubusercontent.com/63696723/132642555-a0fc2a85-a22c-463d-a264-419f5af68b47.png)

<h3>3. Configuration</h3>

<h4>3.1. Initial configuration</h4>

Below is an initial configuration of all three routers.
- R1
```
conf t
router bgp 65100
 bgp router-id 1.1.1.1
 no bgp default ipv4
 neighbor 192.168.12.2 remote-as 65200
 address-family ipv4 unicast
   neighbor 192.168.12.2 activate
   network 192.168.0.0 mask 255.255.255.0
```
 
- R2
```
conf t
router bgp 65200
 bgp router-id 2.2.2.2
 no bgp default ipv4
 neighbor 192.168.12.1 remote-as 65100
 neighbor 192.168.23.3 remote-as 65300
 address-family ipv4 unicast
  neighbor 192.168.12.1 activate
  neighbor 192.168.23.3 activate
  
```

- R3
```
conf t
router bgp 65300
 bgp router-id 3.3.3.3
 no bgp default ipv4
 neighbor 192.168.23.2 remote-as 65200
 address-family ipv4 unicast
  neighbor 192.168.23.2 activate
```

<h4>3.2. BGP Route Aggregation with Static Discard Route</h4>

- R1: Create an aggregate address with a static discard route 192.168.0.0/20 pointing to a null interface.
- The discard static route 192.168.0.0/20 configured on a router R1 makes the router to discard any packet 
that matches the route. However, as long as there are more specific (longer prefix) working routes in a 
routing table of the router R1, packets matching these routes are not discarded. 
- The BGP tables of R2 and R3 routers are injected with the network command configured on R1 router, 
matching the static discard route.

- R1
```
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0 mask 255.255.240.0
  neighbor 192.168.12.2 activate
 exit-address-family

ip route 192.168.0.0 255.255.240.0 Null0
```

- The BGP table of the router R3 is shown in below

```
R3#show bgp ipv4 unicast 
BGP table version is 4, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0/20   192.168.23.2                           0 65200 65100 i
R3#
```

<h4>3.3. BGP Route Aggregation with Aggregate-address Command</h4>

- R1 advertise the aggregate prefix 192.168.0.0/20 to its BGP neighbor R2. 
- The aggregate address is advertised to a neighbor as long as it represents 
at least one part of the aggregate address in the BGP table of a router.
- The parts are called components or the contributing routes and represent 
more specific matches for the aggregated route.  
- We will inject a single route 192.168.0.0/24 into the BGP table of R1 with the network command.

```
R1#show running-config | section bgp
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0 mask 255.255.255.0
  aggregate-address 192.168.0.0 255.255.240.0
  neighbor 192.168.12.2 activate
 exit-address-family

R1# show bgp ipv4 uni
BGP table version is 7, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0      0.0.0.0                  0         32768 i
 *>  192.168.0.0/20   0.0.0.0                            32768 i
R1#
```



- The BGP table of the router R3
```
R3# show bgp ipv4 unicast 
BGP table version is 7, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0      192.168.23.2                           0 65200 65100 i
 *>  192.168.0.0/20   192.168.23.2                           0 65200 65100 i
R3#
```

The BGP table of R3 contains the aggregated prefix along with a more-specific route 192.168.0.0/24.

<h5>3.3.1. Option summary-only</h5>

- By default, all more-specific routes summarized by the aggregate route are advertised. 
- To advertise only the aggregate route and suppress the advertisement of all the component routes, 
the keyword **summary-only** can be used.

```
R1# show running-config | sec bgp
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0
  aggregate-address 192.168.0.0 255.255.240.0 summary-only
  neighbor 192.168.12.2 activate
 exit-address-family
```

- R1's bgp table is marking 192.168.0.0/24 suppressed s>
```
R1# show bgp ipv4 unicast        
BGP table version is 8, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 s>  192.168.0.0      0.0.0.0                  0         32768 i
 *>  192.168.0.0/20   0.0.0.0                            32768 i
 
R1#show bgp ipv4 uni 192.168.0.0
BGP routing table entry for 192.168.0.0/24, version 8
Paths: (1 available, best #1, table default, Advertisements suppressed by an aggregate.)
  Not advertised to any peer
  Refresh Epoch 1
  Local
    0.0.0.0 from 0.0.0.0 (1.1.1.1)
      Origin IGP, metric 0, localpref 100, weight 32768, valid, sourced, local, best
      rx pathid: 0, tx pathid: 0x0
R1#
```

- R3's bgp table now contains only the aggregated route 192.168.0.0/20.
```
R3#show bgp ipv4 unicast 
BGP table version is 8, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0/20   192.168.23.2                           0 65200 65100 i
R3#

R3#show bgp ipv4 unicast 192.168.0.0 255.255.240.0
BGP routing table entry for 192.168.0.0/20, version 7
Paths: (1 available, best #1, table default)
  Not advertised to any peer
  Refresh Epoch 1
  65200 65100, (aggregated by 65100 1.1.1.1)
    192.168.23.2 from 192.168.23.2 (2.2.2.2)
      Origin IGP, localpref 100, valid, external, atomic-aggregate, best
      rx pathid: 0, tx pathid: 0x0
R3#
```

<h5>3.3.2. Option suppress-map</h5>

- Suppress map defines components that should not be advertised.

```
ip prefix-list sup-list seq 10 permit 192.168.0.0/24
route-map sup-map permit 10
 match ip address prefix-list sup-list
 
R1# show run | sec bgp
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0
  network 192.168.1.0
  aggregate-address 192.168.0.0 255.255.240.0 suppress-map sup-map
  neighbor 192.168.12.2 activate
 exit-address-family
```

- Now 192.168.0.0/24 is marked as suppressed.
```
R1#show bgp ipv4 unicast 
BGP table version is 15, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 s>  192.168.0.0      0.0.0.0                  0         32768 i
 *>  192.168.0.0/20   0.0.0.0                            32768 i
 *>  192.168.1.0      0.0.0.0                  0         32768 i
R1#
```

- R3's bgp table
```
R3# show bgp ipv4 unicast 
BGP table version is 13, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0/20   192.168.23.2                           0 65200 65100 i
 *>  192.168.1.0      192.168.23.2                           0 65200 65100 i
R3#
```

<h5>3.3.3. Option unsuppress-map</h5>

- If a subset of the suppressed routes needs to be made available, we can unsuppress 
these routes on a **per neighbor** basis using the **neighbor unsuppress-map** command. 
```
R1#
ip prefix-list unsup-list seq 10 permit 192.168.0.0/24
route-map unsup-map permit 10
 match ip address prefix-list unsup-list
 
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0
  network 192.168.1.0
  aggregate-address 192.168.0.0 255.255.240.0 summary-only
  neighbor 192.168.12.2 activate
  neighbor 192.168.12.2 unsuppress-map unsup-map
 exit-address-family
```

The component routes 192.168.0.0/24 and 192.168.1.0/24 are suppressed by the aggregate route 192.168.0.0/20.
The route 192.168.0.0/24 is unsuppressed for neighbor 192.168.12.2.
```
R1# show bgp ipv4 unicast        
BGP table version is 6, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 s>  192.168.0.0      0.0.0.0                  0         32768 i
 *>  192.168.0.0/20   0.0.0.0                            32768 i
 s>  192.168.1.0      0.0.0.0                  0         32768 i
R1#
```

- R3's BGP table displays both the aggregate route 192.168.0.0/20 and a component route 192.168.0.0/24.
```
R3# show bgp ipv4 uni 
BGP table version is 19, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  192.168.0.0      192.168.23.2                           0 65200 65100 i
 *>  192.168.0.0/20   192.168.23.2                           0 65200 65100 i
R3#
```
 
<h5>3.3.4. Option attribute-map</h5>

- Attributes are inherited from component routes. If we need to remove attributes or set our own attributes 
to the aggregate route, we will use the **attribute-map**.
- The configuration below on R1 router advertises the aggregate route 192.168.0.0/20 with 
the community 65100:500 towards R2 router.
```
R1#show running-config | section bgp
router bgp 65100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor 192.168.12.2 remote-as 65200
 !
 address-family ipv4
  network 192.168.0.0
  network 192.168.1.0
  aggregate-address 192.168.0.0 255.255.240.0 as-set summary-only attribute-map set-attribute
  neighbor 192.168.12.2 activate
  neighbor 192.168.12.2 send-community
 exit-address-family
ip bgp-community new-format
```

- R2's bgp table
```
R2#show bgp ipv4 unicast  192.168.0.0
BGP routing table entry for 192.168.0.0/20, version 22
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     2         
  Refresh Epoch 4
  65100, (aggregated by 65100 1.1.1.1)
    192.168.12.1 from 192.168.12.1 (1.1.1.1)
      Origin IGP, metric 0, localpref 100, valid, external, best
      Community: 65100:500
      rx pathid: 0, tx pathid: 0x0
```



Full of this Lab just re-implement from below link:

> https://www.noction.com/knowledge-base/bgp-route-aggregation
