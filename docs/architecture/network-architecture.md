# Network Architecture

## Overview

This lab simulates an AWS VPC architecture using Cisco Packet Tracer. The design follows AWS best practices for network segmentation, high availability, and security.

## VPC Design

- **VPC CIDR:** `10.0.0.0/16`
- **Availability Zones:** 5 (AZ1–AZ5)
- **Subnet Tiers per AZ:** Public, Private (Application), Private (Database)

## Topology

```
                         ┌─────────────┐
                         │ Edge Router │
                         │ (Internet)  │
                         └──────┬──────┘
                                │
                         ┌──────┴──────┐
                         │ Core Router │
                         └──────┬──────┘
                                │
          ┌─────────────┬───────┴───────┬─────────────┬─────────────┐
          │             │               │             │             │
     ┌────┴────┐   ┌────┴────┐    ┌────┴────┐  ┌────┴────┐  ┌────┴────┐
     │  AZ1    │   │  AZ2    │    │  AZ3    │  │  AZ4    │  │  AZ5    │
     │ Router  │   │ Router  │    │ Router  │  │ Router  │  │ Router  │
     └────┬────┘   └────┬────┘    └────┬────┘  └────┬────┘  └────┬────┘
          │             │              │            │             │
    ┌─────┴─────┐ ┌─────┴─────┐  ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐
    │ Public    │ │ Public    │  │ Public    │ │ Public    │ │ Public    │
    │ App       │ │ App       │  │ App       │ │ App       │ │ App       │
    │ DB        │ │ DB        │  │ DB        │ │ DB        │ │ DB        │
    └───────────┘ └───────────┘  └───────────┘ └───────────┘ └───────────┘
```

## Device Roles

| Device | Role | Location |
|--------|------|----------|
| Edge Router | External connectivity, Internet gateway | Perimeter |
| Core Router | Inter-AZ routing, default route to Edge | Core |
| AZ Router | Intra-AZ routing, subnet isolation | Per AZ |
| NAT Router | Outbound PAT for private subnets | Per AZ (Public) |
| Firewall | ACL enforcement, security boundaries | Per AZ or Core |
| Load Balancer | Traffic distribution to app servers | Per AZ (Public) |

## Addressing Scheme

Each AZ uses a `/20` block within the VPC, further divided into three `/24` subnets:

| AZ | Block | Public | App | DB |
|----|-------|--------|-----|----|
| AZ1 | `10.0.0.0/20` | `10.0.0.0/24` | `10.0.1.0/24` | `10.0.2.0/24` |
| AZ2 | `10.0.16.0/20` | `10.0.16.0/24` | `10.0.17.0/24` | `10.0.18.0/24` |
| AZ3 | `10.0.32.0/20` | `10.0.32.0/24` | `10.0.33.0/24` | `10.0.34.0/24` |
| AZ4 | `10.0.48.0/20` | `10.0.48.0/24` | `10.0.49.0/24` | `10.0.50.0/24` |
| AZ5 | `10.0.64.0/20` | `10.0.64.0/24` | `10.0.65.0/24` | `10.0.66.0/24` |

See [Addressing Plan](addressing-plan.md) for detailed IP assignments.

## Routing

- **Static routes** for inter-AZ communication via Core Router
- **Dynamic routing** (OSPF/EIGRP) planned for scalability
- **Default route** on Core Router pointing to Edge Router for Internet access
- **Per-AZ routes** for subnet isolation

See [Routing Design](../../networking/routing/static-and-dynamic.md) for details.

## NAT

- NAT overload (PAT) on per-AZ NAT routers
- Private and DB subnets access Internet through NAT
- Public subnets have direct Internet access

See [NAT Design](../../networking/nat/nat-design.md) for details.

## Security

- ACLs on routers for subnet isolation
- Firewall (Cisco ASA) for advanced filtering
- Security groups simulated via ACLs

See [Firewall Design](../../networking/firewall/firewall-design.md) for details.

## High Availability

- **Current:** Single Edge Router (known SPOF)
- **Planned:** VRRP/HSRP for Edge Router redundancy
- **Current:** Active/Passive failover testing
- **Planned:** Active/Active load balancing

## Known Limitations

1. Packet Tracer has no dedicated load balancer device (workaround documented)
2. Single Edge Router is a single point of failure
3. No VPN connectivity (planned)
4. Limited QoS capabilities in Packet Tracer

## AWS Comparison

| AWS Concept | Packet Tracer Equivalent |
|-------------|--------------------------|
| VPC | Entire topology |
| Availability Zone | AZ subnet block + router |
| Public Subnet | Public /24 with IGW route |
| Private Subnet | App/DB /24 with NAT route |
| Internet Gateway | Edge Router |
| NAT Gateway | NAT Router with PAT |
| Security Groups | ACLs on router interfaces |
| Route Table | Router routing table |
| Load Balancer | Workaround with DNS/redirect |
