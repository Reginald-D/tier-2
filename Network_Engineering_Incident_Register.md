# Network Engineering

*A record of resolved Tier 2 incidents spanning VLANs, site-to-site VPN, firewall policy, QoS, switching, and load balancing.*

**Prepared by:** Ruler | Tier 2 Support / Network Engineering
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0448055 | Two departments lose cross-network access after switch replacement | P2 — High | Network Engineering > VLAN Segmentation | 1h 40m |
| INC0448602 | Recurring site-to-site VPN tunnel drops between HQ and branch office | P2 — High | Network Engineering > Site-to-Site VPN | 3h 20m |
| INC0449140 | New firewall rule unexpectedly blocks unrelated business application | P2 — High | Network Engineering > Firewall Policy | 4h 05m |
| INC0449688 | VoIP call quality degrades during afternoon peak traffic hours | P3 — Medium | Network Engineering > QoS / Voice Traffic | 6h 50m |
| INC0450210 | Single workstation experiencing intermittent wired connectivity drops | P4 — Low | Network Engineering > Switch Port Diagnostics | 55m |
| INC0450777 | Load balancer pulls healthy backend servers out of rotation | P2 — High | Network Engineering > Load Balancing | 5h 15m |

---

## INC0448055 — Two departments lose cross-network access after switch replacement

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Multi-department access layer, wiring closet switch refresh | P2 — High | Network Engineering > VLAN Segmentation | 1h 40m | Resolved — Closed |

### Incident Summary

Within an hour of a routine switch replacement in a wiring closet, two departments that had no business communicating with each other's resources were suddenly unable to reach anything at all — not each other, not the file server, not the internet.

### Root Cause

The replacement switch's trunk port to the core switch had its native VLAN left at the factory default (VLAN 1) instead of the environment's dedicated native VLAN, and its allowed VLAN list on the trunk excluded two of the three VLANs actually in use on that closet's access ports.

### Resolution

1. **Confirmed the blast radius before touching configuration.** Verified which departments and which specific access ports were affected, ruling out a single bad port in favor of a trunk-level problem.
2. **Compared the new switch's configuration against a known-good sibling.** Pulled the running configuration from the new switch and compared it side-by-side against a working switch, immediately spotting the mismatched native VLAN and a truncated allowed-VLAN list on the uplink trunk.
3. **Verified the trunk logically, not just visually.** Ran show interface trunk on both the new switch and its core switch uplink partner to confirm exactly which VLANs were actually being allowed and tagged across the link.
4. **Corrected the trunk configuration.** Set the native VLAN to match the environment standard and expanded the allowed VLAN list to include every VLAN actually present on the closet's access ports.
5. **Validated connectivity per department.** Tested connectivity from a representative device on each affected VLAN, confirming it could reach its intended resources while still being unable to reach the other department's VLAN, preserving the original segmentation.
6. **Closed the process gap for future swaps.** Documented the standard trunk configuration as a template for future switch replacements to prevent the same oversight.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Switch CLI (show running-config, show interface trunk) | Compared the misconfigured switch against a known-good sibling and confirmed actual trunk VLAN state. |
| Switch Management Console | Applied the corrected native VLAN and allowed-VLAN list. |
| Network Topology & VLAN Documentation | Verified which VLANs were required on the affected access ports. |
| ping / traceroute | Validated per-department connectivity and confirmed segmentation remained intact. |
| Configuration Templates / Change Process | Implemented post-incident to standardize future switch replacements. |

---

## INC0448602 — Recurring site-to-site VPN tunnel drops between HQ and branch office

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| HQ ↔ branch office IPsec tunnel, two perimeter firewalls | P2 — High | Network Engineering > Site-to-Site VPN | 3h 20m | Resolved — Closed |

### Incident Summary

The site-to-site VPN tunnel connecting HQ to a branch office had developed a pattern of dropping every few hours, severing every application and file share dependent on it for thirty to sixty seconds before reconnecting on its own.

### Root Cause

The two VPN endpoints had mismatched Phase 2 (IPsec Security Association) lifetime values. Each time the shorter lifetime expired, that endpoint initiated a rekey negotiation the other side wasn't expecting, and the mismatch was enough to tear the tunnel down before it re-established moments later.

### Resolution

1. **Established the pattern before diagnosing a cause.** Correlated VPN log timestamps across several drop events and found they clustered at a consistent interval, pointing toward a rekey-related cause rather than a connectivity fluke.
2. **Pulled negotiation logs from both endpoints.** Reviewed the IKE/IPsec negotiation logs on both the HQ and branch firewalls around a drop event and found a Phase 2 renegotiation attempt immediately preceding each teardown.
3. **Compared configuration side-by-side.** Pulled the full IPsec policy from both endpoints and laid the Phase 1 and Phase 2 parameters next to each other, isolating the mismatched Phase 2 SA lifetime as the discrepancy.
4. **Aligned the negotiation parameters.** Set both endpoints' Phase 2 lifetimes to the identical value and double-checked encryption and hashing algorithm parity to rule out any secondary mismatch.
5. **Monitored through several full rekey cycles.** Watched the tunnel through multiple expected rekey intervals to confirm renegotiation now completed silently, with zero tunnel teardown.
6. **Documented the standard for future tunnels.** Recorded the matched Phase 1/Phase 2 parameter set as the required baseline for any new site-to-site tunnel built going forward.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Firewall VPN Logs (IKE/IPsec Negotiation) | Correlated drop timestamps and identified the Phase 2 renegotiation event preceding each teardown. |
| Firewall Management Console (both endpoints) | Compared Phase 1/Phase 2 parameters side-by-side to isolate the mismatch. |
| Site-to-Site VPN Configuration Documentation | Recorded the corrected, matched parameter baseline for future builds. |
| Continuous Ping / Tunnel Monitoring | Validated tunnel stability across multiple rekey cycles post-fix. |

---

## INC0449140 — New firewall rule unexpectedly blocks unrelated business application

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Perimeter firewall, multi-rule policy set | P2 — High | Network Engineering > Firewall Policy | 4h 05m | Resolved — Closed |

### Incident Summary

A firewall change intended to tighten outbound access for a specific subnet went in cleanly, and an hour later a completely unrelated internal business application stopped being able to reach its cloud-hosted API.

### Root Cause

The new rule was scoped more broadly than intended and was inserted above an existing allow rule the business application's traffic depended on. Firewall rules are evaluated top-down, first match wins, and the new rule's IP range overlapped enough to shadow the older, more specific allow rule.

### Resolution

1. **Confirmed the timing correlation first.** Checked the firewall's change log and confirmed the application's failure began within minutes of the new rule's deployment.
2. **Traced the actual traffic, not just rule intent.** Used the firewall's packet trace / policy lookup tool against the application's specific source and destination to see exactly which rule was matching and denying the traffic.
3. **Identified the shadowing.** Confirmed the new deny rule sat above the application's existing allow rule in processing order and that its IP range overlapped enough to catch the application's traffic too.
4. **Narrowed the new rule's scope.** Tightened the new rule's source/destination definitions to precisely match only the subnet it was meant to restrict, eliminating the unintended overlap.
5. **Validated both outcomes.** Confirmed the originally intended restriction still worked as designed and separately confirmed the business application's API traffic was flowing again.
6. **Added a pre-deployment safeguard.** Instituted a policy lookup / traffic simulation check against known critical application traffic patterns as a required step before any new firewall rule reaches production.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Firewall Change / Audit Log | Established the timing correlation between the new rule and the application failure. |
| Firewall Packet Trace / Policy Lookup Tool | Identified the exact rule matching and denying the application's real traffic. |
| Firewall Management Console | Reviewed rule processing order and narrowed the new rule's scope. |
| Application-Level Connectivity Testing | Validated the business application's API connectivity was restored. |
| Pre-Deployment Traffic Simulation / Change Review | Implemented post-incident to catch rule-overlap conflicts before production. |

---

## INC0449688 — VoIP call quality degrades during afternoon peak traffic hours

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Converged voice/data network, enterprise VoIP system | P3 — Medium | Network Engineering > QoS / Voice Traffic | 6h 50m | Resolved — Closed |

### Incident Summary

Calls placed through the office VoIP system sounded fine in the morning and fell apart by mid-afternoon — choppy, robotic audio, occasionally a call cutting off entirely — tracking closely with the busiest hours of the business day.

### Root Cause

Voice traffic had no prioritization over general data traffic sharing the same network path. During peak hours, bandwidth-heavy activity competed directly with voice packets for the same queue, and because voice is highly sensitive to latency and jitter even when raw bandwidth is available, it consistently lost that competition.

### Resolution

1. **Ruled out a simple bandwidth shortage first.** Confirmed available bandwidth during a reported bad call and found ample headroom, redirecting the investigation toward how that bandwidth was being shared rather than its total amount.
2. **Measured what actually matters for voice.** Used the VoIP system's call quality diagnostics and a network jitter/latency test during peak hours, which showed jitter and packet loss spiking past acceptable voice thresholds despite moderate overall bandwidth utilization.
3. **Checked for existing prioritization.** Reviewed the switch and router QoS configuration and confirmed no DSCP tagging or prioritization policy existed for voice traffic.
4. **Implemented DSCP-based classification and prioritization.** Configured the VoIP system and network edge devices to tag voice traffic with the Expedited Forwarding (EF) DSCP value and configured QoS queuing on the switches and router to give that traffic priority.
5. **Verified prioritization was actually being honored end to end.** Confirmed DSCP markings were preserved through the full network path and re-ran jitter/latency testing during peak hours to confirm voice traffic now stayed within acceptable thresholds under load.
6. **Validated with real users during the worst hours.** Coordinated test calls during the previously worst-affected afternoon window and confirmed with users that call quality now held steady.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| VoIP System Call Quality Diagnostics | Measured jitter, latency, and packet loss tied directly to voice call quality. |
| Network Jitter / Latency Testing Tools | Quantified voice-specific performance during peak hours versus baseline. |
| Switch / Router QoS Configuration (DSCP Tagging) | Implemented voice traffic classification and priority queuing. |
| Packet Capture / DSCP Marking Verification | Confirmed voice traffic tags were preserved end-to-end. |
| Live Call Testing | Validated real-world quality improvement during the previously worst-affected window. |

---

## INC0450210 — Single workstation experiencing intermittent wired connectivity drops

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Access-layer switching, single affected workstation | P4 — Low | Network Engineering > Switch Port Diagnostics | 55m | Resolved — Closed |

### Incident Summary

One workstation kept losing its wired network connection for a few seconds at a time, several times an hour, with no obvious trigger. A cable swap and a machine reboot had already been tried with no improvement.

### Root Cause

The switch port was flapping due to a duplex mismatch between the switch port (fixed at 100 Mbps/full-duplex through a legacy manual configuration) and the workstation's network adapter (set to auto-negotiate).

### Resolution

1. **Went straight to the switch, not the workstation.** Since a cable swap and reboot had already ruled out the most common user-side causes, pulled the switch port's interface logs rather than continuing to troubleshoot the workstation.
2. **Confirmed the flapping pattern.** Reviewed the switch's interface status history and found repeated link up/down transitions matching the timing of the user's reported drops.
3. **Checked duplex and speed settings on both sides.** Compared the switch port's configured speed/duplex against the workstation NIC's settings and found the switch port manually fixed to 100/Full while the NIC was set to Auto-Negotiate.
4. **Corrected the mismatch.** Set the switch port to Auto-Negotiate to match the workstation, the more resilient long-term configuration for a standard access port.
5. **Monitored for recurrence.** Watched the port's interface status over the following hours of normal use and confirmed zero further flap events.
6. **Swept for the same risk elsewhere.** Queried the switch's full port configuration for any other ports still manually fixed to a speed/duplex setting and corrected two additional ports with the same latent risk.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Switch CLI (show interface status / show logging) | Identified the link flapping pattern and correlated it with reported drop timing. |
| Switch Port Configuration Review | Compared speed/duplex settings and identified the mismatch. |
| Switch Management Console | Corrected the port to auto-negotiate and monitored stability. |
| Network Inventory / Port Configuration Audit | Proactively identified additional ports with the same risk. |

---

## INC0450777 — Load balancer pulls healthy backend servers out of rotation

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Web application server pool behind a load balancer | P2 — High | Network Engineering > Load Balancing | 5h 15m | Resolved — Closed |

### Incident Summary

The load balancer in front of an internal web application began marking backend servers as unhealthy and removing them from rotation, despite the servers being fully reachable and serving the application correctly when tested directly.

### Root Cause

The health check was configured with a timeout that hadn't been adjusted since the application was lighter-weight. A recent application update added heavier per-request initialization to the specific health endpoint, pushing its response time past the configured timeout under any concurrent load.

### Resolution

1. **Verified the servers really were healthy.** Manually hit each flagged backend directly, bypassing the load balancer, and confirmed the application responded correctly and quickly, ruling out an actual backend failure.
2. **Reviewed the health check configuration.** Checked the exact endpoint, expected response, interval, and timeout values configured for the health check.
3. **Measured real response time under load.** Timed the health check endpoint's response time on a backend under typical concurrent request load and found it consistently landing just past the configured timeout.
4. **Correlated against the recent application change.** Cross-referenced the issue's onset against the application's deployment history and confirmed it began shortly after a release that added initialization work to the health endpoint.
5. **Adjusted the health check parameters appropriately.** Increased the timeout to accommodate the endpoint's new real-world response time and required two consecutive failures rather than one before pulling a backend.
6. **Restored the full pool and monitored a full peak cycle.** Returned the incorrectly flagged backends to rotation and confirmed no further false-positive removals across a complete peak-traffic cycle.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Load Balancer Management Console | Reviewed and adjusted health check timeout and failure threshold configuration. |
| Direct Backend Testing (bypassing the load balancer) | Confirmed backend servers were genuinely healthy. |
| Response Time Monitoring / APM Tooling | Measured the health endpoint's real-world response time under load. |
| Application Deployment / Release History | Correlated the issue's onset with the release that changed the endpoint's behavior. |
| Load Balancer Health & Traffic Monitoring | Validated pool stability across a complete peak-traffic cycle. |
