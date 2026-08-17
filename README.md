# AWS VPC Lab (Cisco Packet Tracer)

A hands-on portfolio project simulating a large-scale AWS-style VPC architecture — without using a real cloud provider — built entirely in **Cisco Packet Tracer**.

## Overview

This lab recreates the core networking concepts behind an AWS VPC: multiple Availability Zones, tiered subnets, routing, NAT, firewalling, and load balancing — implemented with Cisco routers, switches, and native Packet Tracer server devices.

The goal is to demonstrate practical understanding of cloud network design patterns (VPC, AZs, subnet tiers, high availability) using traditional networking tools, and to document the process the way a real infrastructure project would be documented.

## Architecture

- **1 main VPC**, split into **5 Availability Zones**
- Each AZ contains:
  - Public Subnet
  - Private Application Subnet
  - Private Database Subnet
- Edge router for external connectivity (single edge router to start — documented as a known SPOF, HA/VRRP planned later)
- Core router(s) interconnecting AZs
- Per-AZ routers handling intra-AZ routing and isolation
- Firewalling via ACLs / Cisco ASA
- NAT for outbound internet access from private subnets
- Load balancing (documented workaround, since Packet Tracer has no dedicated LB device)

See [`docs/architecture/network-architecture.md`](docs/architecture/network-architecture.md) and [`docs/architecture/addressing-plan.md`](docs/architecture/addressing-plan.md) for full details.

## Repo Structure

```
docs/                  Architecture docs, AWS comparison notes, diagrams, troubleshooting log
packet-tracer/         .pkt project file, exported device configs, per-AZ server configs
networking/            Subnet addressing, routing, NAT, firewall, load-balancer, DNS design docs
monitoring/            Notes on monitoring approach and Packet Tracer limitations
scripts/               Process docs for exporting configs
tests/                 Manual test procedures (connectivity, routing, security, failover)
.github/workflows/     Documentation linting
```

## Status

🚧 Work in progress — currently building out with 2 Availability Zones before scaling to 5.

## Tools

- Cisco Packet Tracer
- Cisco IOS (routing, ACLs)

## Concepts Covered

IP addressing & subnetting · routing (static & dynamic) · NAT · firewalling · network isolation · high availability · failover testing

## License

See [LICENSE](LICENSE).