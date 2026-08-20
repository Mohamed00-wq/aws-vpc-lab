# Connectivity Tests

## Overview

Manual test procedures to verify network connectivity in the AWS VPC Lab.

## Test Environment

- **Topology:** 5 AZs with Public/App/DB subnets
- **Tools:** Ping, Traceroute, Telnet, HTTP
- **Devices:** Routers, Servers, PCs

## Test Matrix

### Intra-AZ Connectivity

| # | Source | Destination | Expected | Command |
|---|--------|-------------|----------|---------|
| 1 | AZ1-App-1 (10.0.1.10) | AZ1-App-2 (10.0.1.11) | Pass | `ping 10.0.1.11` |
| 2 | AZ1-App-1 (10.0.1.10) | AZ1-DB (10.0.2.10) | Pass | `ping 10.0.2.10` |
| 3 | AZ1-App-1 (10.0.1.10) | AZ1-Router (10.0.1.1) | Pass | `ping 10.0.1.1` |
| 4 | AZ1-DB (10.0.2.10) | AZ1-Router (10.0.2.1) | Pass | `ping 10.0.2.1` |
| 5 | AZ1-App-1 (10.0.1.10) | AZ1-NAT (10.0.0.2) | Pass | `ping 10.0.0.2` |
| 6 | AZ1-App-1 (10.0.1.10) | AZ1-LB (10.0.0.10) | Pass | `ping 10.0.0.10` |

### Inter-AZ Connectivity

| # | Source | Destination | Expected | Command |
|---|--------|-------------|----------|---------|
| 7 | AZ1-App-1 (10.0.1.10) | AZ2-App-1 (10.0.17.10) | Pass | `ping 10.0.17.10` |
| 8 | AZ1-App-1 (10.0.1.10) | AZ3-DB (10.0.34.10) | Pass | `ping 10.0.34.10` |
| 9 | AZ2-App-1 (10.0.17.10) | AZ5-App-2 (10.0.65.11) | Pass | `ping 10.0.65.11` |
| 10 | AZ1-DB (10.0.2.10) | AZ2-DB (10.0.18.10) | Pass | `ping 10.0.18.10` |

### Internet Connectivity

| # | Source | Destination | Expected | Command |
|---|--------|-------------|----------|---------|
| 11 | AZ1-App-1 | 8.8.8.8 | Pass | `ping 8.8.8.8` |
| 12 | AZ1-App-1 | google.com | Pass | `ping google.com` |
| 13 | AZ2-App-1 | 8.8.8.8 | Pass | `ping 8.8.8.8` |
| 14 | AZ1-DB | 1.1.1.1 | Pass | `ping 1.1.1.1` |

### Service Connectivity

| # | Source | Destination | Port | Expected | Command |
|---|--------|-------------|------|----------|---------|
| 15 | AZ1-App-1 | AZ1-LB | 80 | Pass | `telnet 10.0.0.10 80` |
| 16 | AZ1-App-1 | AZ1-DB | 3306 | Pass | `telnet 10.0.2.10 3306` |
| 17 | AZ1-App-1 | DNS Server | 53 | Pass | `nslookup example.com` |
| 18 | Internet | AZ1-LB | 80 | Pass | `curl http://203.0.113.10` |

## Test Procedure

### Step 1: Basic Connectivity

```bash
# From AZ1 App Server
ping 10.0.1.11    # App-to-App same AZ
ping 10.0.2.10    # App-to-DB same AZ
ping 10.0.1.1     # App-to-Gateway

# Expected: 0% packet loss
```

### Step 2: Cross-AZ Connectivity

```bash
# From AZ1 App Server
ping 10.0.17.10   # AZ1-to-AZ2
ping 10.0.34.10   # AZ1-to-AZ3
traceroute 10.0.17.10  # Verify path through Core Router

# Expected: Pass, traceroute shows Core Router hop
```

### Step 3: Internet Access

```bash
# From AZ1 App Server (Private Subnet)
ping 8.8.8.8
ping google.com
nslookup www.google.com

# Expected: Pass via NAT
```

### Step 4: Service Verification

```bash
# From any device
telnet 10.0.0.10 80    # Web server
nslookup example.com   # DNS resolution
curl http://app.example.com  # HTTP response
```

## Troubleshooting

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Ping fails within same subnet | VLAN misconfiguration | Check switch VLANs |
| Ping fails to different subnet | Routing issue | Check router routes |
| Ping fails cross-AZ | Core Router ACL | Verify ACLs permit traffic |
| No Internet access | NAT misconfigured | Check NAT ACLs and interfaces |
| DNS fails | DNS server unreachable | Check DNS server IP and routes |

## Test Results Template

| Test # | Source | Destination | Result | Notes |
|--------|--------|-------------|--------|-------|
| 1 | | | PASS/FAIL | |
| 2 | | | PASS/FAIL | |
| ... | | | | |

## Automated Testing (Planned)

```bash
#!/bin/bash
# connectivity-test.sh

echo "Testing Intra-AZ connectivity..."
ping -c 3 10.0.1.11
if [ $? -eq 0 ]; then echo "PASS"; else echo "FAIL"; fi

echo "Testing Inter-AZ connectivity..."
ping -c 3 10.0.17.10
if [ $? -eq 0 ]; then echo "PASS"; else echo "FAIL"; fi

echo "Testing Internet connectivity..."
ping -c 3 8.8.8.8
if [ $? -eq 0 ]; then echo "PASS"; else echo "FAIL"; fi
```
