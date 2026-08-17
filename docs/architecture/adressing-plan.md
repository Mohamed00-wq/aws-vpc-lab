# Addressing Plan

## VPC

| Name | CIDR |
|---|---|
| VPC | 10.0.0.0/16 |

## Availability Zone 1 (AZ1)

| Name | CIDR |
|---|---|
| AZ1 | 10.0.0.0/20 |

### Subnets

| Tier | Subnet | Usable Range | Gateway |
|---|---|---|---|
| Public | 10.0.0.0/24 | 10.0.0.1 – 10.0.0.254 | 10.0.0.1 |
| Private (App) | 10.0.1.0/24 | 10.0.1.1 – 10.0.1.254 | 10.0.1.1 |
| DB | 10.0.2.0/24 | 10.0.2.1 – 10.0.2.254 | 10.0.2.1 |

### Device IP Assignments — AZ1

**Public Subnet**
| Device | IP |
|---|---|
| Router / Gateway | 10.0.0.1 |
| NAT | 10.0.0.2 |
| Load Balancer | 10.0.0.10 |

**Private (App) Subnet**
| Device | IP |
|---|---|
| Router / Gateway | 10.0.1.1 |
| App Server 1 | 10.0.1.10 |
| App Server 2 | 10.0.1.11 |

**DB Subnet**
| Device | IP |
|---|---|
| Gateway | 10.0.2.1 |
| DB Server | 10.0.2.10 |

---

## AZ2–AZ5 (planned)

Each additional AZ will follow the same /20 block pattern within the 10.0.0.0/16 VPC, e.g.:

| AZ | Block |
|---|---|
| AZ1 | 10.0.0.0/20 |
| AZ2 | 10.0.16.0/20 |
| AZ3 | 10.0.32.0/20 |
| AZ4 | 10.0.48.0/20 |
| AZ5 | 10.0.64.0/20 |

Each AZ block will be further split into Public / Private (App) / DB /24 subnets, matching the AZ1 pattern above, with IP assignments to be added as each AZ is built out.