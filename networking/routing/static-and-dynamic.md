# Routing Design

## Overview

This document describes the routing strategy for the AWS VPC Lab, including static and dynamic routing approaches.

## Routing Architecture

```
                    ┌─────────────────┐
                    │   Internet      │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │   Edge Router   │
                    │  (Default GW)   │
                    └────────┬────────┘
                             │ Static: 0.0.0.0/0 → Edge
                    ┌────────┴────────┐
                    │   Core Router   │
                    │ (Inter-AZ Hub)  │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
│    AZ1        │    │    AZ2        │    │    AZ3...     │
│   Router      │    │   Router      │    │   Router      │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
   ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
   │ Public  │          │ Public  │          │ Public  │
   │ App     │          │ App     │          │ App     │
   │ DB      │          │ DB      │          │ DB      │
   └─────────┘          └─────────┘          └─────────┘
```

## Static Routing (Current Implementation)

### Edge Router

```cisco
! Default route to ISP
ip route 0.0.0.0 0.0.0.0 <ISP-Next-Hop>

! Route to VPC internal networks via Core Router
ip route 10.0.0.0 255.255.0.0 <Core-Router-Inside-IP>
```

### Core Router

```cisco
! Default route to Edge Router for Internet
ip route 0.0.0.0 0.0.0.0 <Edge-Router-Inside-IP>

! Static routes to each AZ
ip route 10.0.0.0 255.255.240.0 <AZ1-Router-Interface>
ip route 10.0.16.0 255.255.240.0 <AZ2-Router-Interface>
ip route 10.0.32.0 255.255.240.0 <AZ3-Router-Interface>
ip route 10.0.48.0 255.255.240.0 <AZ4-Router-Interface>
ip route 10.0.64.0 255.255.240.0 <AZ5-Router-Interface>
```

### AZ Routers

```cisco
! Default route to Core Router
ip route 0.0.0.0 0.0.0.0 <Core-Router-AZ-Interface>

! Connected subnets (automatic)
! 10.0.X.0/24 - Public
! 10.0.X+1.0/24 - Private App
! 10.0.X+2.0/24 - Private DB
```

## Dynamic Routing (Planned)

### OSPF Configuration

```cisco
! Enable OSPF
router ospf 1
 network 10.0.0.0 0.0.255.255 area 0
 passive-interface default
 no passive-interface <uplink-interfaces>
```

### EIGRP Alternative

```cisco
! Enable EIGRP
router eigrp 100
 network 10.0.0.0
 no auto-summary
```

## Routing Table Summary

| Destination | Next Hop | Protocol | Metric |
|-------------|----------|----------|--------|
| `0.0.0.0/0` | Edge Router | Static | 1 |
| `10.0.0.0/16` | Core Router | Static | 1 |
| `10.0.0.0/20` | AZ1 Router | Static | 2 |
| `10.0.16.0/20` | AZ2 Router | Static | 2 |
| `10.0.32.0/20` | AZ3 Router | Static | 2 |
| `10.0.48.0/20` | AZ4 Router | Static | 2 |
| `10.0.64.0/20` | AZ5 Router | Static | 2 |

## Inter-AZ Routing

Traffic between AZs flows through the Core Router:

1. AZ1 App Server → AZ1 Router → Core Router → AZ2 Router → AZ2 App Server
2. All inter-AZ traffic is inspected at Core Router (ACLs applied)

## Intra-AZ Routing

Within each AZ, the AZ Router handles:

- **Public ↔ App:** Direct routing (same AZ)
- **Public ↔ DB:** Direct routing (same AZ)
- **App ↔ DB:** Direct routing (same AZ)
- **App/DB → Internet:** Via NAT Router

## Route Filtering

ACLs on Core Router control inter-AZ traffic:

```cisco
! Allow inter-AZ communication
access-list 101 permit ip 10.0.0.0 0.0.15.255 10.0.16.0 0.0.15.255
access-list 101 permit ip 10.0.16.0 0.0.15.255 10.0.0.0 0.0.15.255

! Deny direct App-to-DB cross-AZ (force through Core)
access-list 101 deny ip 10.0.1.0 0.0.0.255 10.0.18.0 0.0.0.255
access-list 101 permit ip 10.0.0.0 0.0.255.255 any
```

## Verification Commands

```cisco
show ip route                    ! View routing table
show ip route 10.0.0.0          ! Check specific route
show ip ospf neighbor           ! OSPF neighbors (when implemented)
show ip eigrp neighbors         ! EIGRP neighbors (when implemented)
ping <destination-ip>           ! Test connectivity
traceroute <destination-ip>     ! Trace path
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No route to AZ | Missing static route | Add route on Core Router |
| Asymmetric routing | Multiple equal-cost paths | Verify route metrics |
| Black hole | Missing return route | Ensure bidirectional routes |
| Slow convergence | Static routes | Migrate to OSPF/EIGRP |

## TODO

- [ ] Document actual IPs from Packet Tracer file
- [ ] Implement OSPF for dynamic routing
- [ ] Add route redistribution between static/dynamic
- [ ] Document VRRP/HSRP for Edge Router HA
