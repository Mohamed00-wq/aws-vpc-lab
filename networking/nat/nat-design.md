# NAT Design

## Overview

Network Address Translation (NAT) enables private subnets to access the Internet while maintaining internal IP addressing. This lab uses NAT overload (PAT) on per-AZ NAT routers.

## NAT Architecture

```
                        ┌─────────────────┐
                        │    Internet     │
                        └────────┬────────┘
                                 │
                        ┌────────┴────────┐
                        │   Edge Router   │
                        └────────┬────────┘
                                 │
                        ┌────────┴────────┐
                        │   Core Router   │
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
       ┌──────┴──────┐   ┌──────┴──────┐   ┌──────┴──────┐
       │   AZ1 NAT   │   │   AZ2 NAT   │   │   AZ3 NAT   │
       │  10.0.0.2   │   │  10.0.16.2  │   │  10.0.32.2  │
       └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
              │                  │                  │
       ┌──────┴──────┐   ┌──────┴──────┐   ┌──────┴──────┐
       │  AZ1 Private│   │  AZ2 Private│   │  AZ3 Private│
       │  10.0.1.0/24│   │  10.0.17.0/24│  │  10.0.33.0/24│
       │  10.0.2.0/24│   │  10.0.18.0/24│  │  10.0.34.0/24│
       └─────────────┘   └─────────────┘   └─────────────┘
```

## NAT Router Configuration

Based on `NAT-router.TXT`:

```cisco
hostname NAT-Router

interface GigabitEthernet0/0
 description Inside - Public Subnet - 10.0.0.0/24
 ip address 10.0.0.2 255.255.255.0
 ip nat inside
 no shutdown

interface GigabitEthernet0/1
 description Outside - Uplink to Core/Edge router
 ip address <outside-ip> 255.255.255.0
 ip nat outside
 no shutdown

! NAT overload rule
ip nat inside source list NAT-ACL interface GigabitEthernet0/1 overload

! ACL defining which internal addresses are NAT'd
ip access-list standard NAT-ACL
 permit 10.0.0.0 0.0.15.255
```

## NAT Translation Table

| Inside Local | Inside Global | Outside Local | Outside Global |
|--------------|---------------|---------------|----------------|
| `10.0.1.10:1025` | `10.0.0.2:1025` | `8.8.8.8:53` | `8.8.8.8:53` |
| `10.0.1.11:1026` | `10.0.0.2:1026` | `8.8.8.8:53` | `8.8.8.8:53` |
| `10.0.2.10:1027` | `10.0.0.2:1027` | `1.1.1.1:53` | `1.1.1.1:53` |

## NAT Types Implemented

### PAT (Port Address Translation) - Current

- Multiple internal IPs share one public IP
- Distinguished by port numbers
- Sufficient for lab environment

### Static NAT (Planned)

- One-to-one mapping for servers needing inbound access
- Example: `10.0.0.100` → `203.0.113.100`

### Dynamic NAT (Planned)

- Pool of public IPs for larger deployments
- Automatic assignment from pool

## NAT Verification

```cisco
show ip nat translations          ! View active translations
show ip nat statistics            ! NAT statistics
clear ip nat translation *        ! Clear all translations
debug ip nat                      ! Debug NAT events
```

## NAT for Each AZ

| AZ | NAT IP (Inside) | NAT ACL Range | Purpose |
|----|-----------------|---------------|---------|
| AZ1 | `10.0.0.2` | `10.0.0.0/20` | App/DB Internet access |
| AZ2 | `10.0.16.2` | `10.0.16.0/20` | App/DB Internet access |
| AZ3 | `10.0.32.2` | `10.0.32.0/20` | App/DB Internet access |
| AZ4 | `10.0.48.2` | `10.0.48.0/20` | App/DB Internet access |
| AZ5 | `10.0.64.2` | `10.0.64.0/20` | App/DB Internet access |

## AWS Comparison

| AWS Service | Packet Tracer Equivalent |
|-------------|--------------------------|
| NAT Gateway | NAT Router with PAT |
| EIP (Elastic IP) | Static NAT mapping |
| VPC Endpoint | Not implemented (planned) |

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No Internet from private subnet | NAT not configured | Verify `ip nat inside` on inside interface |
| One-way traffic | Missing `ip nat outside` | Add `ip nat outside` on outside interface |
| NAT table full | Too many concurrent connections | IncreasePAT range or add NAT routers |
| Asymmetric NAT | Return traffic bypasses NAT | Ensure return path goes through NAT router |

## TODO

- [ ] Configure NAT for AZ2–AZ5
- [ ] Add static NAT mappings for inbound access
- [ ] Implement VPC endpoints for AWS services
- [ ] Document NAT gateway failover
