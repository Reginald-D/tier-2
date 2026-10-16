<div align="center">

# IT Helpdesk Portfolio
### Network Engineering — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Network%20Engineering-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-6-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-VLANs%20%7C%20VPN%20%7C%20Firewall%20%7C%20QoS-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases six representative incident tickets spanning the most common **network engineering** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0448055`](#inc0448055--two-departments-lose-cross-network-access-after-switch-replacement) | Two departments lose cross-network access after switch replacement | P2 | 1h 40m |
| 2 | [`INC0448602`](#inc0448602--recurring-site-to-site-vpn-tunnel-drops-between-hq-and-branch-office) | Recurring site-to-site VPN tunnel drops between HQ and branch office | P2 | 3h 20m |
| 3 | [`INC0449140`](#inc0449140--new-firewall-rule-unexpectedly-blocks-unrelated-business-application) | New firewall rule unexpectedly blocks unrelated business application | P2 | 4h 05m |
| 4 | [`INC0449688`](#inc0449688--voip-call-quality-degrades-during-afternoon-peak-traffic-hours) | VoIP call quality degrades during afternoon peak traffic hours | P3 | 6h 50m |
| 5 | [`INC0450210`](#inc0450210--single-workstation-experiencing-intermittent-wired-connectivity-drops) | Single workstation experiencing intermittent wired connectivity drops | P4 | 55m |
| 6 | [`INC0450777`](#inc0450777--load-balancer-pulls-healthy-backend-servers-out-of-rotation) | Load balancer pulls healthy backend servers out of rotation | P2 | 5h 15m |

---

<br>

## `INC0448055` — Two departments lose cross-network access after switch replacement

<table>
<tr><td><b>Environment</b></td><td>Multi-department access layer, wiring closet switch refresh</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > VLAN Segmentation</td></tr>
<tr><td><b>Time to Resolve</b></td><td>1h 40m</td></tr>
</table>

**Reported Symptom**
Within an hour of a routine switch replacement in a wiring closet, two departments that had no business communicating with each other's resources were suddenly unable to reach anything at all — not each other, not the file server, not the internet.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the blast radius before touching configuration. Verified which departments and which specific access ports were affected, ruling out a single bad port in favor of a trunk-level problem.
2. Compared the new switch's configuration against a known-good sibling. Pulled the running configuration from the new switch and compared it side-by-side against a working switch, immediately spotting the mismatched native VLAN and a truncated allowed-VLAN list on the uplink trunk.
3. Verified the trunk logically, not just visually. Ran show interface trunk on both the new switch and its core switch uplink partner to confirm exactly which VLANs were actually being allowed and tagged across the link.
4. Corrected the trunk configuration. Set the native VLAN to match the environment standard and expanded the allowed VLAN list to include every VLAN actually present on the closet's access ports.
5. Validated connectivity per department. Tested connectivity from a representative device on each affected VLAN, confirming it could reach its intended resources while still being unable to reach the other department's VLAN, preserving the original segmentation.
6. Closed the process gap for future swaps. Documented the standard trunk configuration as a template for future switch replacements to prevent the same oversight.

</details>

**Root Cause**
The replacement switch's trunk port to the core switch had its native VLAN left at the factory default (VLAN 1) instead of the environment's dedicated native VLAN, and its allowed VLAN list on the trunk excluded two of the three VLANs actually in use on that closet's access ports.

**Resolution**
Tested connectivity from a representative device on each affected VLAN, confirming it could reach its intended resources while still being unable to reach the other department's VLAN, preserving the original segmentation. Documented the standard trunk configuration as a template for future switch replacements to prevent the same oversight.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0448602` — Recurring site-to-site VPN tunnel drops between HQ and branch office

<table>
<tr><td><b>Environment</b></td><td>HQ ↔ branch office IPsec tunnel, two perimeter firewalls</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > Site-to-Site VPN</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 20m</td></tr>
</table>

**Reported Symptom**
The site-to-site VPN tunnel connecting HQ to a branch office had developed a pattern of dropping every few hours, severing every application and file share dependent on it for thirty to sixty seconds before reconnecting on its own.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Established the pattern before diagnosing a cause. Correlated VPN log timestamps across several drop events and found they clustered at a consistent interval, pointing toward a rekey-related cause rather than a connectivity fluke.
2. Pulled negotiation logs from both endpoints. Reviewed the IKE/IPsec negotiation logs on both the HQ and branch firewalls around a drop event and found a Phase 2 renegotiation attempt immediately preceding each teardown.
3. Compared configuration side-by-side. Pulled the full IPsec policy from both endpoints and laid the Phase 1 and Phase 2 parameters next to each other, isolating the mismatched Phase 2 SA lifetime as the discrepancy.
4. Aligned the negotiation parameters. Set both endpoints' Phase 2 lifetimes to the identical value and double-checked encryption and hashing algorithm parity to rule out any secondary mismatch.
5. Monitored through several full rekey cycles. Watched the tunnel through multiple expected rekey intervals to confirm renegotiation now completed silently, with zero tunnel teardown.
6. Documented the standard for future tunnels. Recorded the matched Phase 1/Phase 2 parameter set as the required baseline for any new site-to-site tunnel built going forward.

</details>

**Root Cause**
The two VPN endpoints had mismatched Phase 2 (IPsec Security Association) lifetime values. Each time the shorter lifetime expired, that endpoint initiated a rekey negotiation the other side wasn't expecting, and the mismatch was enough to tear the tunnel down before it re-established moments later.

**Resolution**
Watched the tunnel through multiple expected rekey intervals to confirm renegotiation now completed silently, with zero tunnel teardown. Recorded the matched Phase 1/Phase 2 parameter set as the required baseline for any new site-to-site tunnel built going forward.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0449140` — New firewall rule unexpectedly blocks unrelated business application

<table>
<tr><td><b>Environment</b></td><td>Perimeter firewall, multi-rule policy set</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > Firewall Policy</td></tr>
<tr><td><b>Time to Resolve</b></td><td>4h 05m</td></tr>
</table>

**Reported Symptom**
A firewall change intended to tighten outbound access for a specific subnet went in cleanly, and an hour later a completely unrelated internal business application stopped being able to reach its cloud-hosted API.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the timing correlation first. Checked the firewall's change log and confirmed the application's failure began within minutes of the new rule's deployment.
2. Traced the actual traffic, not just rule intent. Used the firewall's packet trace / policy lookup tool against the application's specific source and destination to see exactly which rule was matching and denying the traffic.
3. Identified the shadowing. Confirmed the new deny rule sat above the application's existing allow rule in processing order and that its IP range overlapped enough to catch the application's traffic too.
4. Narrowed the new rule's scope. Tightened the new rule's source/destination definitions to precisely match only the subnet it was meant to restrict, eliminating the unintended overlap.
5. Validated both outcomes. Confirmed the originally intended restriction still worked as designed and separately confirmed the business application's API traffic was flowing again.
6. Added a pre-deployment safeguard. Instituted a policy lookup / traffic simulation check against known critical application traffic patterns as a required step before any new firewall rule reaches production.

</details>

**Root Cause**
The new rule was scoped more broadly than intended and was inserted above an existing allow rule the business application's traffic depended on. Firewall rules are evaluated top-down, first match wins, and the new rule's IP range overlapped enough to shadow the older, more specific allow rule.

**Resolution**
Confirmed the originally intended restriction still worked as designed and separately confirmed the business application's API traffic was flowing again. Instituted a policy lookup / traffic simulation check against known critical application traffic patterns as a required step before any new firewall rule reaches production.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0449688` — VoIP call quality degrades during afternoon peak traffic hours

<table>
<tr><td><b>Environment</b></td><td>Converged voice/data network, enterprise VoIP system</td></tr>
<tr><td><b>Priority</b></td><td>P3 — Medium</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > QoS / Voice Traffic</td></tr>
<tr><td><b>Time to Resolve</b></td><td>6h 50m</td></tr>
</table>

**Reported Symptom**
Calls placed through the office VoIP system sounded fine in the morning and fell apart by mid-afternoon — choppy, robotic audio, occasionally a call cutting off entirely — tracking closely with the busiest hours of the business day.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Ruled out a simple bandwidth shortage first. Confirmed available bandwidth during a reported bad call and found ample headroom, redirecting the investigation toward how that bandwidth was being shared rather than its total amount.
2. Measured what actually matters for voice. Used the VoIP system's call quality diagnostics and a network jitter/latency test during peak hours, which showed jitter and packet loss spiking past acceptable voice thresholds despite moderate overall bandwidth utilization.
3. Checked for existing prioritization. Reviewed the switch and router QoS configuration and confirmed no DSCP tagging or prioritization policy existed for voice traffic.
4. Implemented DSCP-based classification and prioritization. Configured the VoIP system and network edge devices to tag voice traffic with the Expedited Forwarding (EF) DSCP value and configured QoS queuing on the switches and router to give that traffic priority.
5. Verified prioritization was actually being honored end to end. Confirmed DSCP markings were preserved through the full network path and re-ran jitter/latency testing during peak hours to confirm voice traffic now stayed within acceptable thresholds under load.
6. Validated with real users during the worst hours. Coordinated test calls during the previously worst-affected afternoon window and confirmed with users that call quality now held steady.

</details>

**Root Cause**
Voice traffic had no prioritization over general data traffic sharing the same network path. During peak hours, bandwidth-heavy activity competed directly with voice packets for the same queue, and because voice is highly sensitive to latency and jitter even when raw bandwidth is available, it consistently lost that competition.

**Resolution**
Confirmed DSCP markings were preserved through the full network path and re-ran jitter/latency testing during peak hours to confirm voice traffic now stayed within acceptable thresholds under load. Coordinated test calls during the previously worst-affected afternoon window and confirmed with users that call quality now held steady.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0450210` — Single workstation experiencing intermittent wired connectivity drops

<table>
<tr><td><b>Environment</b></td><td>Access-layer switching, single affected workstation</td></tr>
<tr><td><b>Priority</b></td><td>P4 — Low</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > Switch Port Diagnostics</td></tr>
<tr><td><b>Time to Resolve</b></td><td>55m</td></tr>
</table>

**Reported Symptom**
One workstation kept losing its wired network connection for a few seconds at a time, several times an hour, with no obvious trigger. A cable swap and a machine reboot had already been tried with no improvement.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Went straight to the switch, not the workstation. Since a cable swap and reboot had already ruled out the most common user-side causes, pulled the switch port's interface logs rather than continuing to troubleshoot the workstation.
2. Confirmed the flapping pattern. Reviewed the switch's interface status history and found repeated link up/down transitions matching the timing of the user's reported drops.
3. Checked duplex and speed settings on both sides. Compared the switch port's configured speed/duplex against the workstation NIC's settings and found the switch port manually fixed to 100/Full while the NIC was set to Auto-Negotiate.
4. Corrected the mismatch. Set the switch port to Auto-Negotiate to match the workstation, the more resilient long-term configuration for a standard access port.
5. Monitored for recurrence. Watched the port's interface status over the following hours of normal use and confirmed zero further flap events.
6. Swept for the same risk elsewhere. Queried the switch's full port configuration for any other ports still manually fixed to a speed/duplex setting and corrected two additional ports with the same latent risk.

</details>

**Root Cause**
The switch port was flapping due to a duplex mismatch between the switch port (fixed at 100 Mbps/full-duplex through a legacy manual configuration) and the workstation's network adapter (set to auto-negotiate).

**Resolution**
Watched the port's interface status over the following hours of normal use and confirmed zero further flap events. Queried the switch's full port configuration for any other ports still manually fixed to a speed/duplex setting and corrected two additional ports with the same latent risk.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0450777` — Load balancer pulls healthy backend servers out of rotation

<table>
<tr><td><b>Environment</b></td><td>Web application server pool behind a load balancer</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Network Engineering > Load Balancing</td></tr>
<tr><td><b>Time to Resolve</b></td><td>5h 15m</td></tr>
</table>

**Reported Symptom**
The load balancer in front of an internal web application began marking backend servers as unhealthy and removing them from rotation, despite the servers being fully reachable and serving the application correctly when tested directly.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Verified the servers really were healthy. Manually hit each flagged backend directly, bypassing the load balancer, and confirmed the application responded correctly and quickly, ruling out an actual backend failure.
2. Reviewed the health check configuration. Checked the exact endpoint, expected response, interval, and timeout values configured for the health check.
3. Measured real response time under load. Timed the health check endpoint's response time on a backend under typical concurrent request load and found it consistently landing just past the configured timeout.
4. Correlated against the recent application change. Cross-referenced the issue's onset against the application's deployment history and confirmed it began shortly after a release that added initialization work to the health endpoint.
5. Adjusted the health check parameters appropriately. Increased the timeout to accommodate the endpoint's new real-world response time and required two consecutive failures rather than one before pulling a backend.
6. Restored the full pool and monitored a full peak cycle. Returned the incorrectly flagged backends to rotation and confirmed no further false-positive removals across a complete peak-traffic cycle.

</details>

**Root Cause**
The health check was configured with a timeout that hadn't been adjusted since the application was lighter-weight. A recent application update added heavier per-request initialization to the specific health endpoint, pushing its response time past the configured timeout under any concurrent load.

**Resolution**
Increased the timeout to accommodate the endpoint's new real-world response time and required two consecutive failures rather than one before pulling a backend. Returned the incorrectly flagged backends to rotation and confirmed no further false-positive removals across a complete peak-traffic cycle.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 6 |
| **High Priority (P2) Incidents** | 4 |
| **Medium Priority (P3) Incidents** | 1 |
| **Low Priority (P4) Incidents** | 1 |
| **Average Time to Resolve** | ~3h 41m |
| **Systems Touched** | Switch CLI · Switch Management Console · Network Topology & VLAN Documentation · ping · Configuration Templates · Firewall VPN Logs · Firewall Management Console · Site-to-Site VPN Configuration Documentation |

## Skills Demonstrated

`VLAN Segmentation` · `Site-to-Site VPN (IPsec)` · `Firewall Policy Management` · `QoS / Voice Traffic Prioritization` · `Switch Port Diagnostics` · `Load Balancer Health Checks` · `Packet-Level Troubleshooting`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across network engineering — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>