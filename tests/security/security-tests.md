# Security Tests

## Overview

Manual test procedures to verify firewall rules, ACLs, and security configurations.

## Test Environment

- **Firewalls:** Router ACLs, Cisco ASA (planned)
- **Security Zones:** Outside, DMZ, Inside
- **Test Tools:** Ping, Telnet, HTTP, Nmap (simulated)

## Test Matrix

### ACL Verification

| # | Rule | Source | Destination | Port | Expected | Command |
|---|------|--------|-------------|------|----------|---------|
| 1 | Inbound | Internet | Public Subnet | 80 | Allow | `telnet 203.0.113.10 80` |
| 2 | Inbound | Internet | Public Subnet | 443 | Allow | `telnet 203.0.113.10 443` |
| 3 | Inbound | Internet | Private Subnet | Any | Deny | `ping 10.0.1.10` |
| 4 | Inbound | Internet | DB Subnet | Any | Deny | `ping 10.0.2.10` |
| 5 | Outbound | Private | Internet | Any | Allow | `ping 8.8.8.8` |

### Subnet Isolation

| # | Source | Destination | Expected | ACL Rule |
|---|--------|-------------|----------|----------|
| 6 | AZ1-App (10.0.1.10) | AZ1-DB (10.0.2.10) | Allow | Same AZ |
| 7 | AZ1-App (10.0.1.10) | AZ2-DB (10.0.18.10) | Deny | Cross-AZ DB blocked |
| 8 | AZ1-App (10.0.1.10) | AZ2-App (10.0.17.10) | Allow | Cross-AZ App allowed |
| 9 | AZ1-DB (10.0.2.10) | AZ2-DB (10.0.18.10) | Deny | Cross-AZ DB blocked |

### NAT Security

| # | Test | Expected | Command |
|---|------|----------|---------|
| 10 | Private→Internet | Allow (PAT) | `ping 8.8.8.8` |
| 11 | Internet→Private | Deny | `telnet 10.0.1.10 80` |
| 12 | NAT Translation | Visible | `show ip nat translations` |

### Service Security

| # | Service | Source | Destination | Expected | Command |
|---|---------|--------|-------------|----------|---------|
| 13 | HTTP | Public | App Server | Allow | `curl http://10.0.0.10` |
| 14 | HTTPS | Public | App Server | Allow | `curl https://10.0.0.10` |
| 15 | SSH | Public | App Server | Deny | `ssh 10.0.1.10` |
| 16 | RDP | Public | App Server | Deny | `telnet 10.0.1.10 3389` |
| 17 | MySQL | App Server | DB Server | Allow | `telnet 10.0.2.10 3306` |
| 18 | MySQL | Public | DB Server | Deny | `telnet 10.0.2.10 3306` |

## Test Procedure

### Step 1: Verify ACLs

```bash
# On Router
show access-lists
show ip interface GigabitEthernet0/0

# Check hit counts
# Look for incrementing deny/permit counters
```

### Step 2: Test Inbound Security

```bash
# From Internet (or external device)
telnet 203.0.113.10 80     # Should succeed (public server)
telnet 203.0.113.10 22     # Should fail (SSH blocked)
ping 10.0.1.10              # Should fail (private IP blocked)
```

### Step 3: Test Subnet Isolation

```bash
# From AZ1 App Server
ping 10.0.2.10              # Should succeed (same AZ DB)
ping 10.0.18.10             # Should fail (cross-AZ DB blocked)
telnet 10.0.18.10 3306      # Should fail (cross-AZ DB blocked)

# From AZ1 App Server
ping 10.0.17.10             # Should succeed (cross-AZ App allowed)
```

### Step 4: Test NAT Security

```bash
# From Private Subnet
ping 8.8.8.8                # Should succeed (outbound NAT)
traceroute 8.8.8.8          # First hop should be NAT router

# Verify translation
show ip nat translations
```

### Step 5: Test Service Access

```bash
# From Public Subnet
telnet 10.0.1.10 80         # Should succeed (HTTP to App)
telnet 10.0.1.10 443        # Should succeed (HTTPS to App)
telnet 10.0.2.10 3306       # Should fail (DB not accessible from Public)
```

## Security Verification Commands

```cisco
show access-lists                       ! View ACL rules and hits
show ip nat translations                ! View NAT table
show ip interface brief                 ! Verify interfaces
show crypto isakmp sa                   ! VPN status (planned)
show firewall                           ! ASA firewall status
packet-tracer input outside tcp 8.8.8.8 1025 10.0.0.10 443  ! ASA trace
```

## Security Test Results Template

| Test # | Rule | Source | Destination | Result | Notes |
|--------|------|--------|-------------|--------|-------|
| 1 | | | | PASS/FAIL | |
| 2 | | | | PASS/FAIL | |
| ... | | | | | |

## Common Security Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Traffic passes when blocked | ACL order | Move deny before permit |
| Return traffic blocked | Stateful inspection | Ensure return path through firewall |
| NAT bypass | Missing `ip nat outside` | Add NAT to outside interface |
| ACL not applying | Wrong interface/direction | Verify `ip access-group` direction |

## Penetration Test Scenarios (Planned)

### Scenario 1: External Attack

```bash
# Simulate external scan
telnet 203.0.113.10 22     # SSH attempt
telnet 203.0.113.10 3389   # RDP attempt
telnet 203.0.113.10 3306   # MySQL attempt

# Expected: All blocked
```

### Scenario 2: Lateral Movement

```bash
# From compromised App Server
ping 10.0.2.10              # Attempt DB access
ping 10.0.18.10             # Attempt cross-AZ DB

# Expected: Same AZ allowed, cross-AZ blocked
```

### Scenario 3: Data Exfiltration

```bash
# From Private Subnet
telnet external-server.com 443  # Outbound HTTPS
ping external-server.com         # Outbound ICMP

# Expected: Allowed (outbound)
```

## TODO

- [ ] Document actual ACL rules from Packet Tracer
- [ ] Implement Cisco ASA configuration
- [ ] Add IDS/IPS testing
- [ ] Document security group equivalents
