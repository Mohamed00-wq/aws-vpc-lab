# Routing Tests

## Overview

Manual test procedures to verify routing configuration in the AWS VPC Lab.

## Test Environment

- **Topology:** 5 AZs with Core and Edge Routers
- **Routing:** Static routes (dynamic planned)
- **Verification:** show ip route, traceroute, ping

## Test Matrix

### Static Route Verification

| # | Device | Test | Expected | Command |
|---|--------|------|----------|---------|
| 1 | Core Router | Default route exists | 0.0.0.0/0 → Edge | `show ip route` |
| 2 | Core Router | AZ1 route exists | 10.0.0.0/20 → AZ1 | `show ip route` |
| 3 | Core Router | AZ2 route exists | 10.0.16.0/20 → AZ2 | `show ip route` |
| 4 | AZ1 Router | Default route exists | 0.0.0.0/0 → Core | `show ip route` |
| 5 | Edge Router | VPC route exists | 10.0.0.0/16 → Core | `show ip route` |

### Route Propagation

| # | Source | Destination | Expected Path | Command |
|---|--------|-------------|---------------|---------|
| 6 | AZ1-App | AZ2-App | AZ1→Core→AZ2 | `traceroute 10.0.17.10` |
| 7 | AZ1-App | Internet | AZ1→Core→Edge→ISP | `traceroute 8.8.8.8` |
| 8 | AZ2-App | AZ1-DB | AZ2→Core→AZ1 | `traceroute 10.0.2.10` |
| 9 | AZ5-App | AZ1-App | AZ5→Core→AZ1 | `traceroute 10.0.1.10` |

### Route Table Consistency

| # | Router | Route | Next Hop | Metric | Command |
|---|--------|-------|----------|--------|---------|
| 10 | Core | 10.0.0.0/20 | AZ1-Int | 1 | `show ip route 10.0.0.0` |
| 11 | Core | 10.0.16.0/20 | AZ2-Int | 1 | `show ip route 10.0.16.0` |
| 12 | AZ1 | 0.0.0.0/0 | Core-Int | 1 | `show ip route 0.0.0.0` |
| 13 | AZ2 | 0.0.0.0/0 | Core-Int | 1 | `show ip route 0.0.0.0` |

## Test Procedure

### Step 1: Verify Routing Tables

```bash
# On Core Router
show ip route
show ip route 10.0.0.0 255.255.240.0
show ip route 0.0.0.0

# On AZ1 Router
show ip route
show ip route 0.0.0.0

# On Edge Router
show ip route
show ip route 10.0.0.0 255.255.0.0
```

### Step 2: Verify Inter-AZ Routing

```bash
# From AZ1 App Server
traceroute 10.0.17.10   # Should go: AZ1→Core→AZ2
traceroute 10.0.34.10   # Should go: AZ1→Core→AZ3
traceroute 10.0.50.10   # Should go: AZ1→Core→AZ4
traceroute 10.0.66.10   # Should go: AZ1→Core→AZ5

# Expected: Each traceroute shows Core Router as second hop
```

### Step 3: Verify Internet Routing

```bash
# From AZ1 App Server
traceroute 8.8.8.8
traceroute google.com

# Expected path: AZ1→Core→Edge→ISP→Destination
```

### Step 4: Verify Return Routes

```bash
# From Internet (if possible)
traceroute 203.0.113.10   # Should reach Edge Router
# Then internal: Edge→Core→AZ1→Server
```

## Routing Verification Commands

```cisco
show ip route                          ! Full routing table
show ip route 10.0.0.0                ! Specific route
show ip route 0.0.0.0                 ! Default route
show ip protocols                     ! Routing protocols
show ip ospf neighbor                 ! OSPF neighbors (planned)
show ip eigrp neighbors               ! EIGRP neighbors (planned)
show ip interface brief                ! Interface status
debug ip routing                       ! Debug routing changes
```

## Troubleshooting

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| No route to network | Missing static route | Add route on appropriate router |
| Asymmetric routing | Multiple paths | Verify metrics, adjust if needed |
| Black hole | Missing return route | Ensure bidirectional routes |
| Slow convergence | Manual routes | Migrate to dynamic routing |
| Route flapping | Interface instability | Check cable/interface status |

## Route Filtering Tests

| # | Source | Destination | Expected | ACL Rule |
|---|--------|-------------|----------|----------|
| 14 | AZ1-App | AZ2-DB | Deny | Cross-AZ DB access blocked |
| 15 | AZ1-App | AZ2-App | Allow | Cross-AZ App access allowed |
| 16 | Internet | Private Subnet | Deny | Inbound blocked |
| 17 | Private | Internet | Allow | Outbound via NAT allowed |

## Test Results Template

| Test # | Device/Source | Test | Result | Notes |
|--------|---------------|------|--------|-------|
| 1 | | | PASS/FAIL | |
| 2 | | | PASS/FAIL | |
| ... | | | | |

## Dynamic Routing Tests (Planned)

```bash
# OSPF Tests
show ip ospf neighbor
show ip ospf database
ping 10.0.0.1   # OSPF neighbor

# EIGRP Tests
show ip eigrp neighbors
show ip eigrp topology
```
