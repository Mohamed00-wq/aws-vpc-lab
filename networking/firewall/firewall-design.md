# Firewall Design

## Overview

This document describes the firewall architecture using Cisco ACLs and ASA to protect the VPC infrastructure.

## Firewall Architecture

```
                    ┌─────────────────┐
                    │   Edge Router   │
                    │  (First Line)   │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │  Core Firewall  │
                    │   (Cisco ASA)   │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
│  AZ1 Firewall │    │  AZ2 Firewall │    │  AZ3 Firewall │
│  (Router ACL) │    │  (Router ACL) │    │  (Router ACL) │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
   ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
   │ Public  │          │ Public  │          │ Public  │
   │ App     │          │ App     │          │ App     │
   │ DB      │          │ DB      │          │ DB      │
   └─────────┘          └─────────┘          └─────────┘
```

## Security Zones

| Zone | Interface | Security Level | Description |
|------|-----------|----------------|-------------|
| Outside | Edge Outside | 0 | Internet |
| DMZ | Edge Inside | 50 | Public-facing services |
| Inside | Core Inside | 100 | Internal networks |

## ACL Rules

### Edge Router ACL (Inbound from Internet)

```cisco
! Block private IPs from Internet
access-list 100 deny ip 10.0.0.0 0.255.255.255 any
access-list 100 deny ip 172.16.0.0 0.15.255.255 any
access-list 100 deny ip 192.168.0.0 0.0.255.255 any

! Allow established connections
access-list 100 permit tcp any any established

! Allow HTTP/HTTPS to public servers
access-list 100 permit tcp any 10.0.0.0 0.0.0.255 eq 80
access-list 100 permit tcp any 10.0.0.0 0.0.0.255 eq 443

! Allow DNS
access-list 100 permit udp any 10.0.0.0 0.0.0.255 eq 53

! Deny all other inbound
access-list 100 deny ip any any

interface GigabitEthernet0/0
 ip access-group 100 in
```

### Core Router ACL (Inter-AZ)

```cisco
! Allow inter-AZ communication
access-list 101 permit ip 10.0.0.0 0.0.15.255 10.0.16.0 0.0.15.255
access-list 101 permit ip 10.0.16.0 0.0.15.255 10.0.0.0 0.0.15.255

! Allow App-to-DB within same AZ
access-list 101 permit tcp 10.0.1.0 0.0.0.255 10.0.2.0 0.0.0.255 eq 3306
access-list 101 permit tcp 10.0.17.0 0.0.0.255 10.0.18.0 0.0.0.255 eq 3306

! Deny direct App-to-DB cross-AZ
access-list 101 deny ip 10.0.1.0 0.0.0.255 10.0.18.0 0.0.0.255
access-list 101 deny ip 10.0.17.0 0.0.0.255 10.0.2.0 0.0.0.255

! Allow all other internal traffic
access-list 101 permit ip 10.0.0.0 0.0.255.255 any
```

### AZ Router ACL (Subnet Isolation)

```cisco
! Block cross-subnet traffic except through gateway
access-list 102 deny ip 10.0.1.0 0.0.0.255 10.0.2.0 0.0.0.255
access-list 102 permit ip 10.0.0.0 0.0.0.255 any
access-list 102 permit ip 10.0.1.0 0.0.0.255 10.0.0.0 0.0.0.255
access-list 102 permit ip 10.0.2.0 0.0.0.255 10.0.0.0 0.0.0.255

interface GigabitEthernet0/1
 ip access-group 102 in
```

## Cisco ASA Configuration (Planned)

```cisco
! Define security zones
interface GigabitEthernet0/0
 nameif outside
 security-level 0

interface GigabitEthernet0/1
 nameif inside
 security-level 100

! Security policy
access-list OUTSIDE-IN extended permit tcp any host 203.0.113.10 eq 443
access-list OUTSIDE-IN extended deny ip any any

access-group OUTSIDE-IN in interface outside

! NAT (if ASA handles NAT)
object network OBJ-AZ1-APP
 subnet 10.0.1.0 255.255.255.0
 nat (inside,outside) dynamic interface
```

## Security Rules Summary

| Source | Destination | Port | Action | Reason |
|--------|-------------|------|--------|--------|
| Internet | Public Subnet | 80,443 | Allow | Web servers |
| Internet | Any Private | Any | Deny | Internal only |
| App Server | DB Server | 3306 | Allow | Database access |
| App Server | App Server (cross-AZ) | Any | Allow | App replication |
| DB Server | DB Server (cross-AZ) | 3306 | Allow | DB replication |
| Private | Internet | Any | Allow | Outbound via NAT |
| Public | Private App | Any | Deny | Isolation |

## Stateful Inspection

The ASA provides stateful inspection:

1. **Outbound connections:** Automatically allowed return traffic
2. **Inbound connections:** Only allowed if explicitly permitted
3. **Connection tracking:** Maintains state table for TCP/UDP

## Verification Commands

```cisco
show access-lists                 ! View ACL hits
show interface | include protocol ! Check ACL applied
show conn all                     ! ASA: View connections
show asp drop                     ! ASA: View dropped packets
packet-tracer input outside tcp 8.8.8.8 1025 10.0.0.10 443  ! ASA: Trace packet
```

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Traffic blocked unexpectedly | ACL order | Check ACL hit counts, reorder |
| No return traffic | Stateful inspection | Verify return path through firewall |
| High CPU | Too many ACL hits | Optimize ACLs, remove redundant entries |
| Asymmetric routing | Bypasses firewall | Ensure traffic flows through firewall |

## TODO

- [ ] Implement Cisco ASA configuration
- [ ] Add IDS/IPS rules
- [ ] Document security group equivalents
- [ ] Add logging and monitoring
