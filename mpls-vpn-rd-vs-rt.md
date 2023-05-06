MPLS VPNs - RD vs RT
====================

## Definition

### RD - Route Distinguisher

  - 64-bit identifier
  - Append to an IP address (32-bit) to make a unique 96-bit VPNv4 address

### RT - Route Target
  - 64-bit BGP extended community
  - Defines VPN membership i.e. what routes are part of what VPNs
  - Applied to VRFs in order to import and export routes

## Configuration of RD/RT

### RD Configuration

```
ip vrf <vrf name>
rd 100:1
```

### RT configuration

```
ip vrf <vrf name>
rd 100:1
route-target import 100:100
route-target export 100:100
```

## Example

![mpls-vpn-rd-rt](https://user-images.githubusercontent.com/63696723/236618301-5dc51fd4-2ac7-4b7c-8071-7bae1764a156.png)


Thanks!
