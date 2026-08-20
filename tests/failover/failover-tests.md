# Failover Tests

## Overview

Manual test procedures to verify high availability and failover mechanisms.

## Test Environment

- **HA Components:** Edge Router (planned VRRP), NAT failover
- **Scenarios:** Link failure, device failure, route convergence
- **Verification:** Ping, traceroute, show commands

## Test Matrix

### Edge Router Failover (Planned)

| # | Scenario | Test | Expected | Command |
|---|----------|------|----------|---------|
| 1 | Primary Edge down | Backup takes over | Traffic via backup | `ping 8.8.8.8` |
| 2 | Primary Edge recovers | Traffic returns to primary | Automatic | `show vrrp` |
| 3 | Both Edges down | No Internet | Timeout | `ping 8.8.8.8` |

### NAT Failover

| # | Scenario | Test | Expected | Command |
|---|----------|------|----------|---------|
| 4 | NAT Router reboot | Connectivity restored | Automatic | `ping 8.8.8.8` |
| 5 | NAT interface down | Other AZs unaffected | Isolation | `ping 8.8.8.8` from AZ2 |

### Core Router Failure

| # | Scenario | Test | Expected | Command |
|---|----------|------|----------|---------|
| 6 | Core Router reboot | Inter-AZ temporarily down | Manual recovery | `ping 10.0.17.10` |
| 7 | Core Router recovered | Inter-AZ restored | Automatic | `ping 10.0.17.10` |

### AZ Router Failure

| # | Scenario | Test | Expected | Command |
|---|----------|------|----------|---------|
| 8 | AZ1 Router down | AZ1 isolated | Isolation | `ping 10.0.1.10` |
| 9 | AZ1 Router recovered | AZ1 restored | Automatic | `ping 10.0.1.10` |
| 10 | AZ1 down, AZ2 up | AZ2 unaffected | Isolation | `ping 10.0.17.10` |

### Link Failure

| # | Scenario | Test | Expected | Command |
|---|----------|------|----------|---------|
| 11 | Core-AZ1 link down | AZ1 isolated | Isolation | `ping 10.0.1.10` |
| 12 | Core-AZ1 link restored | AZ1 restored | Automatic | `ping 10.0.1.10` |
| 13 | Edge-Core link down | No Internet | Isolation | `ping 8.8.8.8` |

## Test Procedure

### Step 1: Baseline Connectivity

```bash
# Verify all paths working
ping 8.8.8.8              # Internet
ping 10.0.17.10           # Inter-AZ
ping 10.0.2.10            # Intra-AZ
traceroute 10.0.17.10     # Verify path
```

### Step 2: Edge Router Failover

```bash
# Simulate failure (shutdown Edge Router)
# On Edge Router:
shutdown

# From AZ1 App Server:
ping 8.8.8.8              # Should fail

# Bring up backup Edge (if configured)
# Verify failover:
ping 8.8.8.8              # Should succeed via backup

# Restore primary:
no shutdown
# Verify return:
ping 8.8.8.8              # Should succeed via primary
```

### Step 3: Core Router Failure

```bash
# Simulate failure (shutdown Core Router)
# On Core Router:
shutdown

# From AZ1 App Server:
ping 10.0.17.10           # Should fail (inter-AZ)
ping 8.8.8.8              # Should fail (no path to Edge)

# Restore Core Router:
no shutdown

# Verify recovery:
ping 10.0.17.10           # Should succeed
ping 8.8.8.8              # Should succeed
```

### Step 4: AZ Router Failure

```bash
# Simulate AZ1 Router failure
# On AZ1 Router:
shutdown

# From AZ1 App Server:
ping 10.0.1.1             # Should fail (no gateway)
ping 8.8.8.8              # Should fail

# From AZ2 App Server:
ping 10.0.17.10           # Should succeed (AZ2 unaffected)

# Restore AZ1 Router:
no shutdown

# Verify AZ1 recovery:
ping 10.0.1.1             # Should succeed
ping 8.8.8.8              # Should succeed
```

### Step 5: Link Failure

```bash
# Simulate link failure (shutdown interface)
# On Core Router:
interface GigabitEthernet0/1
 shutdown

# From AZ1 App Server:
ping 10.0.17.10           # Should fail

# Restore link:
interface GigabitEthernet0/1
 no shutdown

# Verify recovery:
ping 10.0.17.10           # Should succeed
```

## Failover Verification Commands

```cisco
show vrrp                          ! VRRP status (planned)
show vrrp brief                    ! VRRP summary
show hsrp                         ! HSRP status (planned)
show ip route                      ! Verify routes
show ip ospf neighbor              ! OSPF neighbors (planned)
show ip eigrp neighbors            ! EIGRP neighbors (planned)
show interface status              ! Link status
show logging                       ! Failure logs
```

## Failover Timeline

| Event | Time | Action |
|-------|------|--------|
| T+0 | 0s | Failure occurs |
| T+1 | 0-30s | Detection (keepalives) |
| T+2 | 30-60s | Failover triggered |
| T+3 | 60-90s | Traffic rerouted |
| T+4 | 90-120s | Service restored |

## Failover Test Results Template

| Test # | Scenario | Result | Recovery Time | Notes |
|--------|----------|--------|---------------|-------|
| 1 | | | PASS/FAIL | |
| 2 | | | PASS/FAIL | |
| ... | | | | |

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Failover not triggered | Keepalive interval too high | Reduce keepalive timer |
| Asymmetric routing after failover | Route metrics unchanged | Verify route preferences |
| Flapping | Unstable link | Check cable/interface |
| Split-brain | Both nodes think they're primary | Verify VRRP/HSRP priority |

## VRRP Configuration (Planned)

```cisco
! Primary Edge Router
interface GigabitEthernet0/0
 vrrp 1 ip 10.0.0.1
 vrrp 1 priority 110
 vrrp 1 preempt

! Backup Edge Router
interface GigabitEthernet0/0
 vrrp 1 ip 10.0.0.1
 vrrp 1 priority 100
 vrrp 1 preempt
```

## AWS Comparison

| AWS Feature | Packet Tracer Equivalent |
|-------------|--------------------------|
| Multi-AZ | Multiple AZ Routers |
| Auto Recovery | Manual failover |
| Route53 Failover | DNS manual update |
| ELB Health Checks | Manual ping checks |

## TODO

- [ ] Implement VRRP for Edge Router
- [ ] Add automatic failover testing scripts
- [ ] Document RTO/RPO metrics
- [ ] Test cascading failures
