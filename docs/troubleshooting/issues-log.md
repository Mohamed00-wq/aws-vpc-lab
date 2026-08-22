## Issue: LB-Server (AZ4) ping to local gateway timed out
**Date:** 2026-08-22
**Symptom:** `ping 10.0.48.1` from LB4-Server → 100% packet loss
**Root cause:** IP address was never assigned on the end device itself (only the router interface had one)
**Fix:** Assigned correct IP/subnet/gateway via Desktop > IP Configuration on the server
**Lesson:** Router interface up/up doesn't guarantee end-device connectivity — always verify `ipconfig` on the host too

---

## Issue: Cannot assign IP address on CORE-Router transit interfaces
**Date:** 2026-08-22
**Symptom:** `ip address` command rejected on NIM-ES2-4 module ports; `no switchport` returns "Incomplete command"
**Root cause:** NIM-ES2-4 is a Layer2-only EtherSwitch module — Packet Tracer doesn't support converting its ports to routed (Layer3) mode
**Fix:** Used onboard GigabitEthernet0/0/x ports (native Layer3) instead of the NIM-ES2-4 module for transit links
**Lesson:** In Packet Tracer, EtherSwitch/NIM modules (names containing "ES" or "SW") stay Layer2-only — use onboard router ports or WIC/Serial modules for routed links

---

## Issue: Scaled down from 5 AZs to 3 AZs
**Date:** 2026-08-22
**Decision:** Dropped AZ4/AZ5 due to lack of available Layer3 interfaces on CORE-Router (module limitation above)
**Rationale:** 3 AZs is sufficient to demonstrate multi-AZ HA design; matches real-world AWS regional patterns; avoids further Packet Tracer module workarounds

---

## Issue: Firewall (ASA5505) misconfigured — inside/outside on same VLAN
**Date:** 2026-08-22
**Symptom:** No isolation between Edge and NAT segments after initial firewall config
**Root cause:** Both Ethernet0/0 and Ethernet0/1 were assigned to the same VLAN (vlan 2), collapsing outside/inside into one broadcast domain
**Fix:** Moved Ethernet0/1 back to default VLAN 1; confirmed Vlan1=inside (security-level 100), Vlan2=outside (security-level 0)

---

## Issue: Firewall inside interface IP wouldn't apply
**Date:** 2026-08-22
**Symptom:** `ip address 203.0.113.6 ...` on Vlan1 silently failed to register (`show running-config` showed `no ip address`)
**Root cause:** Conflicting DHCP server pool still active on the inside interface
**Fix:** Disabled DHCP on inside (`no dhcpd enable inside`) before reapplying the static IP
**Lesson:** On ASA, an active DHCP pool can block static IP reassignment on the same interface — disable DHCP first

---

## Issue: Direct Edge–NAT cable bypassing the firewall
**Date:** 2026-08-22
**Symptom:** Topology had a direct link Edge-Router ↔ NAT-Router in addition to the ASA links, making the firewall non-inline (traffic could skip it entirely)
**Fix:** Removed the direct cable; enforced proper chain: Edge-Router → ASA (outside/inside) → NAT-Router

---

## Issue: Edge↔NAT ping across firewall failing (0% success) despite correct config
**Date:** 2026-08-22
**Symptom:** `ping 203.0.113.6` from Edge and `ping 203.0.113.1` from NAT both failed, even though: routing tables were correct on both routers, ASA could ping both routers directly, ACL hit-counters incremented correctly on both directions (5→10) confirming full round-trip packet flow, ARP tables were fully resolved
**Root cause:** Default Cisco `ping` timeout (2 sec) was too short for Packet Tracer's ASA inspection/processing delay — not an actual connectivity or config issue
**Fix:** `ping <ip> timeout 5 repeat 10` succeeded at 100%
**Lesson:** When all diagnostic evidence (routes, ACL hit counts, ARP, direct pings) points to a working path but the actual end-to-end ping still fails, suspect a Packet Tracer simulation timing artifact before re-auditing the config