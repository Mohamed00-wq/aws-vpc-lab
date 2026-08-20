# DNS Design

## Overview

DNS resolves domain names to IP addresses within the VPC and provides Internet name resolution.

## DNS Architecture

```
                    ┌─────────────────┐
                    │   Internet DNS  │
                    │  (8.8.8.8 etc)  │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │   Edge Router   │
                    │  (DNS Forward)  │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │   Core Router   │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
│  AZ1 DNS      │    │  AZ2 DNS      │    │  AZ3 DNS      │
│  10.0.0.5     │    │  10.0.16.5    │    │  10.0.32.5    │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
   ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
   │ Public  │          │ Public  │          │ Public  │
   │ App     │          │ App     │          │ App     │
   │ DB      │          │ DB      │          │ DB      │
   └─────────┘          └─────────┘          └─────────┘
```

## DNS Zones

### Internal Zone

```
vpc.local
├── az1
│   ├── router.vpc.local      → 10.0.0.1
│   ├── nat.vpc.local         → 10.0.0.2
│   ├── lb.vpc.local          → 10.0.0.10
│   ├── app1.vpc.local        → 10.0.1.10
│   ├── app2.vpc.local        → 10.0.1.11
│   └── db.vpc.local          → 10.0.2.10
├── az2
│   ├── router.vpc.local      → 10.0.16.1
│   ├── nat.vpc.local         → 10.0.16.2
│   ├── lb.vpc.local          → 10.0.16.10
│   ├── app1.vpc.local        → 10.0.17.10
│   ├── app2.vpc.local        → 10.0.17.11
│   └── db.vpc.local          → 10.0.18.10
└── core
    ├── router.vpc.local      → <core-router-ip>
    └── edge.vpc.local        → <edge-router-ip>
```

### External Zone (Public)

```
example.com
├── www.example.com           → 203.0.113.10 (Edge Router)
├── app.example.com           → 203.0.113.10 (Load Balanced)
└── mail.example.com          → 203.0.113.20 (Planned)
```

## DNS Server Configuration

### Packet Tracer DNS Server

```bash
# Service: DNS
# Zone: vpc.local
# Records:

Name                  Type    Address
router.az1.vpc.local  A       10.0.0.1
nat.az1.vpc.local     A       10.0.0.2
lb.az1.vpc.local      A       10.0.0.10
app1.az1.vpc.local    A       10.0.1.10
app2.az1.vpc.local    A       10.0.1.11
db.az1.vpc.local      A       10.0.2.10

# Repeat for AZ2-AZ5...
```

### Router DNS Configuration

```cisco
! On AZ Router
ip name-server 10.0.0.5
ip domain-lookup

! DNS forwarding to Core/Edge
ip dns server
ip dns primary vpc.local SOA ns1.vpc.local admin.vpc.local
```

## DNS Resolution Flow

```
1. Client (10.0.1.10) queries: app.example.com
2. Forwarded to AZ DNS (10.0.0.5)
3. AZ DNS checks cache → miss
4. Forwarded to Edge Router
5. Edge Router forwards to ISP DNS (8.8.8.8)
6. Response: 203.0.113.10
7. Cached at each level
8. Returned to client
```

## DNS Records Summary

| Record Type | Purpose | Example |
|-------------|---------|---------|
| A | IPv4 address | `app.az1.vpc.local → 10.0.1.10` |
| AAAA | IPv6 address | Not implemented |
| CNAME | Alias | `www → app.example.com` |
| MX | Mail exchange | Planned |
| PTR | Reverse DNS | Planned |
| SOA | Start of Authority | `ns1.vpc.local` |
| NS | Name Server | `ns1.vpc.local` |

## DNS Verification

```bash
# On PC/Server
nslookup app.example.com
nslookup 10.0.1.10

# On Router
show hosts
ping app.az1.vpc.local
```

## DNS Security

```cisco
! DNS ACLs
ip access-list standard DNS-ACL
 permit 10.0.0.0 0.0.255.255
 permit host 8.8.8.8
 permit host 8.8.4.4
 deny any

! Apply to DNS interface
interface GigabitEthernet0/0
 ip access-group DNS-ACL in
```

## AWS Comparison

| AWS Service | Packet Tracer Equivalent |
|-------------|--------------------------|
| Route 53 | Packet Tracer DNS Server |
| Private Hosted Zone | Internal DNS zone |
| Public Hosted Zone | External DNS zone |
| DNS Failover | Manual DNS update |

## TODO

- [ ] Configure DNS servers for each AZ
- [ ] Add reverse DNS zones
- [ ] Document DNS failover strategy
- [ ] Implement DNSSEC (if supported)
