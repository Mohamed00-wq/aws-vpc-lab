# Load Balancer Design

## Overview

Packet Tracer has no dedicated load balancer device. This document describes workarounds to simulate load balancing across application servers.

## Load Balancing Approaches

### Approach 1: DNS Round Robin (Recommended)

Use DNS to distribute traffic across multiple servers:

```
Client → DNS Server → Returns multiple IPs → Client picks one
```

**Configuration:**

```cisco
! On DNS Server (Packet Tracer Server Device)
! Add multiple A records for same domain
app.example.com → 10.0.1.10
app.example.com → 10.0.1.11
app.example.com → 10.0.17.10
app.example.com → 10.0.17.11
```

### Approach 2: Router-Based Load Balancing

Use router static routes with equal-cost paths:

```cisco
! On AZ Router
ip route 10.0.1.10 255.255.255.255 <next-hop-1>
ip route 10.0.1.11 255.255.255.255 <next-hop-2>
```

### Approach 3: HTTP Redirect (Manual)

Use a dedicated server to redirect requests:

```
Client → LB Server (10.0.0.10) → Redirects to 10.0.1.10 or 10.0.1.11
```

## Load Balancer Architecture

```
                    ┌─────────────────┐
                    │     Clients     │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │   DNS Server    │
                    │  (Round Robin)  │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
│  AZ1-LB       │    │  AZ2-LB       │    │  AZ3-LB       │
│  10.0.0.10    │    │  10.0.16.10   │    │  10.0.32.10   │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
   ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
   │ App-1   │          │ App-1   │          │ App-1   │
   │ App-2   │          │ App-2   │          │ App-2   │
   └─────────┘          └─────────┘          └─────────┘
```

## Server Configuration

### App Server 1 (AZ1)

```bash
# IP: 10.0.1.10
# Gateway: 10.0.1.1
# Services: HTTP, HTTPS
# Response: "Hello from AZ1-App-Server-1"
```

### App Server 2 (AZ1)

```bash
# IP: 10.0.1.11
# Gateway: 10.0.1.1
# Services: HTTP, HTTPS
# Response: "Hello from AZ1-App-Server-2"
```

## Health Checks (Manual)

Since Packet Tracer lacks automated health checks:

```cisco
! Manual verification
ping 10.0.1.10
ping 10.0.1.11
telnet 10.0.1.10 80
telnet 10.0.1.11 80
```

## Load Balancing Algorithms

| Algorithm | Implementation | Complexity |
|-----------|----------------|------------|
| Round Robin | DNS multiple A records | Low |
| Least Connections | Not possible in PT | N/A |
| IP Hash | Router static routes | Medium |
| Weighted | DNS TTL manipulation | Low |

## AWS Comparison

| AWS Service | Packet Tracer Equivalent |
|-------------|--------------------------|
| ALB (Application LB) | DNS Round Robin + Servers |
| NLB (Network LB) | Router ECMP (limited) |
| ELB Health Checks | Manual ping/telnet |
| Target Groups | DNS A record sets |

## Testing Load Balancing

```bash
# Test from client
nslookup app.example.com    ! Should return multiple IPs
curl http://app.example.com ! Run multiple times, note different responses

# Verify distribution
# Run 10 requests, should see ~5 on each server
for i in {1..10}; do curl -s http://app.example.com; done
```

## Limitations

1. No automatic failover (manual DNS update required)
2. No health checks (manual monitoring)
3. No SSL termination (handled by servers)
4. No session persistence (sticky sessions)

## TODO

- [ ] Implement DNS round robin configuration
- [ ] Document server configurations
- [ ] Add manual health check scripts
- [ ] Test failover scenarios
