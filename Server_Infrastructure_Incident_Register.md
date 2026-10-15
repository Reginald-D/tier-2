# Server & Infrastructure

*A record of resolved Tier 2 incidents spanning domain controllers, file services, DHCP, DNS, print services, and internal PKI.*

**Prepared by:** Ruler | Tier 2 Support / Server & Infrastructure
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0444805 | Domain controller services (NTDS, DNS, Netlogon) failed to start after patch reboot | P1 — Critical | Server Infrastructure > Domain Controller | 4h 50m |
| INC0445312 | Unintended access to confidential HR subfolder found during routine access audit | P2 — High | Server Infrastructure > File Services | 3h 10m |
| INC0445890 | New devices at branch office failing to obtain an IP address | P2 — High | Server Infrastructure > DHCP | 2h 25m |
| INC0446410 | Intermittent resolution failures for internal application traced to stale DNS record | P2 — High | Server Infrastructure > DNS | 3h 45m |
| INC0446977 | Print Spooler service crash loop taking down printing for entire site | P2 — High | Server Infrastructure > Print Services | 1h 55m |
| INC0447502 | Expired internal CA intermediate certificate breaking multiple downstream services | P1 — Critical | Server Infrastructure > PKI / Certificate Authority | 5h 30m |

---

## INC0444805 — Domain controller services (NTDS, DNS, Netlogon) failed to start after patch reboot

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Secondary-site domain controller, Windows Server 2019, ~400 site users | P1 — Critical | Server Infrastructure > Domain Controller | 4h 50m | Resolved — Closed |

### Incident Summary

A scheduled patch reboot on a secondary-site domain controller did not return cleanly. Active Directory Domain Services, DNS Server, and Netlogon all failed to start, taking down authentication, file share access, and printer mapping for the entire site. Standard service restarts had no effect.

### Root Cause

Netlogon was failing because it depends on a fully initialized SYSVOL share, and SYSVOL was stuck behind a DFS Replication (DFSR) journal wrap — a state where DFSR loses confidence in its own change history and halts replication to avoid propagating bad data. NTDS and DNS were reporting degraded for the same underlying reason: both were waiting on the same stalled dependency chain.

### Resolution

1. **Triaged the failure sequence before touching any service.** Reviewed Services.msc and the System event log to confirm the order of failure. Netlogon's dependency failure was the common thread behind NTDS and DNS reporting degraded, which ruled out three independent faults and pointed to one shared root cause.
2. **Identified the DFSR journal wrap.** Reviewed the DFS Replication event log and found Event ID 2213, the definitive signature of a journal wrap, confirming DFSR had halted SYSVOL replication rather than risk corrupting it further.
3. **Confirmed SYSVOL replication health before choosing a recovery path.** Ran dfsrdiag replicationstate and dfsrdiag backlog against the affected DC to verify SYSVOL was in fact not replicating, and checked whether a second domain controller in the forest still held a fully replicated, healthy copy.
4. **Selected the lower-risk recovery method.** Because a healthy source DC existed, performed a non-authoritative SYSVOL restore rather than an authoritative restore, which carries a materially higher risk of data loss and should only be used when no healthy source remains.
5. **Executed the restore and monitored the resync.** Set the DFSR SYSVOL replication state to non-authoritative through the appropriate AD attribute, restarted the DFSR service, and monitored the resulting full resync from the healthy DC to completion.
6. **Validated full recovery before returning the DC to production.** Confirmed Netlogon, NTDS, and DNS all started cleanly once SYSVOL finished syncing, ran dcdiag /v against the recovered DC, and had site users retest authentication, file access, and printing before closing the ticket.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Services.msc / System Event Log | Triaged the failure sequence and identified Netlogon's dependency failure as the root trigger. |
| DFS Replication Event Log (Event ID 2213) | Confirmed the journal wrap condition halting SYSVOL replication. |
| dfsrdiag | Verified SYSVOL replication state and backlog, and confirmed a healthy source DC existed. |
| ADSI Edit / Attribute Editor | Set the DFSR SYSVOL replication state to trigger the non-authoritative restore. |
| dcdiag / repadmin | Performed full post-recovery health validation before returning the DC to production. |

---

## INC0445312 — Unintended access to confidential HR subfolder found during routine access audit

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Windows file server, departmental shares (~15 TB, 40 nested shares) | P2 — High | Server Infrastructure > File Services | 3h 10m | Resolved — Closed |

### Incident Summary

A routine access audit found a small group of users with read/write access to a subfolder containing confidential HR documents, despite never being granted access to the parent HR share. The permission structure on the parent share itself appeared entirely correct.

### Root Cause

Inheritance had been broken on the specific subfolder at some point during a prior reorganization, and explicit permissions were applied directly to it as a one-off fix for a since-resolved access request. Because explicit NTFS permissions always take precedence over inherited ones, the leftover explicit entry silently overrode the parent share's intended restriction.

### Resolution

1. **Diagnosed the access source directly, not just the folder's listed permissions.** Used the Advanced Security / Effective Access tab for one of the flagged users on the subfolder, which showed the access was being granted by an explicit ACL entry rather than inheritance from the parent.
2. **Confirmed inheritance had been disabled.** Checked the subfolder's Advanced Security Settings and confirmed "Include inheritable permissions from this object's parent" was unchecked, isolating it from the parent share's ACL entirely.
3. **Audited every explicit entry on the subfolder.** Ran icacls against the subfolder to produce a complete, scriptable list of every explicit ACL entry, confirming exactly which accounts had been added outside of inheritance.
4. **Validated each entry against the documented access policy.** Cross-referenced the explicit entries against the HR share's documented access policy and confirmed none of the flagged users' access was currently authorized or justified.
5. **Removed the stale entries and re-enabled inheritance.** Cleared the explicit ACEs and re-enabled inheritance on the subfolder so its permissions once again derived correctly from the parent HR share.
6. **Verified the fix and documented the finding.** Re-ran the Effective Access check for the previously flagged users to confirm access was now correctly denied, and documented the finding for the HR data owner and the next scheduled access review.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| NTFS Advanced Security / Effective Access | Diagnosed whether access was coming from inheritance or an explicit ACL entry. |
| icacls | Produced a complete, auditable list of every explicit entry on the affected subfolder. |
| Active Directory Users and Computers (ADUC) | Cross-referenced flagged accounts against their intended group-based access model. |
| PowerShell (Get-Acl / Set-Acl) | Scripted validation of the corrected permission state after remediation. |
| File Server Resource Manager (FSRM) | Supported the broader access audit that originally flagged the discrepancy. |

---

## INC0445890 — New devices at branch office failing to obtain an IP address

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Branch office subnet (/24), dedicated DHCP server | P2 — High | Server Infrastructure > DHCP | 2h 25m | Resolved — Closed |

### Incident Summary

A branch office rolling out a batch of new laptops found that none of the new devices could obtain a valid IP address. Existing devices on the network were unaffected; new devices connecting to the office network received an Automatic Private IP Address (169.254.x.x) instead.

### Root Cause

The subnet's DHCP scope had genuinely run out of available addresses. The scope had been sized appropriately when the office was first provisioned, but growth in laptops, phones, and IoT-connected office equipment had quietly consumed the pool, and the default 8-day lease duration meant stale leases from decommissioned devices were not expiring fast enough to free up space.

### Resolution

1. **Confirmed the symptom pointed to DHCP specifically.** Verified new devices were receiving an APIPA address, the standard signature of a client failing to receive any DHCP offer at all, ruling out a switch port or VLAN issue.
2. **Checked scope utilization directly.** Opened the DHCP console and reviewed the Address Pool and Leases for the branch subnet, confirming the scope was at 100% utilization with zero addresses available.
3. **Reclaimed stale leases to restore immediate capacity.** Identified leases with hostnames matching decommissioned equipment and manually cleared them, freeing enough addresses to onboard the new laptop batch the same day.
4. **Reviewed historical lease data to confirm the pattern was systemic.** Pulled lease history over several months and confirmed this was a sustained growth trend rather than a one-time spike, ruling out a transient cause.
5. **Expanded the scope with confirmed available address space.** Coordinated with network engineering to confirm adjacent IP space was free and extended the scope's address range to provide adequate headroom for current and near-term device counts.
6. **Shortened the lease duration to reduce future churn risk.** Reduced the lease time from the default 8 days to a value better suited to the office's device turnover, so stale leases expire and recycle faster going forward.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| DHCP Management Console | Diagnosed scope utilization and identified reclaimable stale leases. |
| PowerShell (DHCP Server module) | Scripted lease auditing and scope configuration changes. |
| ipconfig / APIPA diagnostics | Confirmed the client-side symptom pattern consistent with DHCP exhaustion. |
| Network IPAM / Subnet Documentation | Verified available address space before expanding the scope. |
| DHCP Audit Logging | Reviewed historical lease trends to confirm a genuine growth pattern. |

---

## INC0446410 — Intermittent resolution failures for internal application traced to stale DNS record

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Company-wide internal DNS zone, ~3,000 user base | P2 — High | Server Infrastructure > DNS | 3h 45m | Resolved — Closed |

### Incident Summary

An internal application had generated a slow trickle of complaints for several weeks: some users could reach it by name without issue, others got a connection failure, and reloading a few seconds later often resolved it. The pattern was inconsistent enough to initially be dismissed as an application issue.

### Root Cause

The DNS zone held a stale A record pointing to a server decommissioned months earlier during an infrastructure consolidation, sitting alongside the correct, current record for the same hostname. With two records for one name, round-robin resolution was intermittently handing clients the dead IP. DNS scavenging, which should have aged out the orphaned record automatically, had never been enabled on the zone.

### Resolution

1. **Reframed the issue as DNS rather than application behavior.** Correlated user reports against direct nslookup queries run during reported failures and found the resolved IP was inconsistent across identical queries run seconds apart, ruling out the application and pointing at name resolution.
2. **Audited the zone directly.** Opened DNS Manager and reviewed the records for the affected hostname, finding two A records where there should have been exactly one.
3. **Identified which record was stale.** Cross-referenced both IPs against the current infrastructure inventory and confirmed one belonged to a server decommissioned during a prior consolidation project.
4. **Removed the stale record.** Deleted the orphaned A record, leaving only the current, correct entry, and verified resolution consistency with repeated nslookup queries.
5. **Closed the systemic gap by enabling scavenging.** Enabled DNS scavenging on the zone with appropriate no-refresh and refresh intervals, so records left behind by future decommissioning work age out automatically instead of lingering indefinitely.
6. **Flushed client caches and validated end to end.** Had affected users run ipconfig /flushdns and confirmed consistent, correct application access across multiple test machines before closing the ticket.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| DNS Manager | Audited the zone and identified the duplicate, conflicting A records. |
| nslookup | Diagnosed inconsistent resolution behavior in real time and validated the fix. |
| DNS Scavenging & Aging Configuration | Implemented to automatically age out orphaned records going forward. |
| Infrastructure Asset Inventory | Cross-referenced conflicting IPs against current versus decommissioned servers. |
| ipconfig /flushdns | Cleared client-side caches to confirm the fix in real user experience. |

---

## INC0446977 — Print Spooler service crash loop taking down printing for entire site

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Site print server, ~25 networked printers | P2 — High | Server Infrastructure > Print Services | 1h 55m | Resolved — Closed |

### Incident Summary

Within the span of an hour, print failures were reported from across the entire site. The Print Spooler service was crashing and restarting repeatedly, taking every queued job and connected printer down with each crash.

### Root Cause

A single corrupted print job, submitted from a workstation using an outdated, mismatched printer driver, was stuck at the front of the spool queue in a state the spooler could not process or discard cleanly. Every time the spooler restarted, it immediately tried to reprocess the same corrupted job and crashed again.

### Resolution

1. **Reviewed the crash signature before taking any action.** Checked the Application event log and found a consistent crash signature tied to the same faulting module on every occurrence, pointing to a specific stuck job rather than a general server fault.
2. **Stopped the crash cycle.** Stopped the Print Spooler service entirely rather than letting it keep auto-restarting into the same failure, creating room to investigate without the queue compounding further.
3. **Located the corrupted spool files.** Navigated to the spooler's queue directory and identified the specific corrupted .SPL and .SHD file pair matching the timestamp of the first crash.
4. **Cleared the offending job directly from the file system.** Deleted the corrupted spool file pair from the queue directory, since the print management UI was not accessible while the spooler was down.
5. **Restarted and monitored under normal load.** Restarted the Print Spooler service and monitored it as queued jobs from the rest of the site began processing normally without a repeat crash.
6. **Traced the issue back to its source and corrected it.** Identified the originating workstation and driver version from the corrupted job's metadata, updated it to the current supported driver, and confirmed no further malformed jobs were generated.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Windows Event Viewer (Application Log) | Identified the consistent crash signature pointing to a specific stuck job. |
| Services.msc | Stopped and restarted the Print Spooler service in a controlled manner. |
| Print Spooler Queue Directory (File System) | Located and removed the corrupted spool file pair directly. |
| Print Management Console | Verified queue and printer status site-wide once the spooler was stable. |
| Windows Driver Management | Identified and corrected the outdated driver at the originating workstation. |

---

## INC0447502 — Expired internal CA intermediate certificate breaking multiple downstream services

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internal Certificate Authority supporting web application, VPN gateway, and LOB tool | P1 — Critical | Server Infrastructure > PKI / Certificate Authority | 5h 30m | Resolved — Closed |

### Incident Summary

Overnight, an internal web application began throwing certificate trust errors, a VPN client began rejecting connections, and a line-of-business tool stopped authenticating cleanly — three apparently unrelated failures within the same short window.

### Root Cause

An intermediate certificate issued by the internal Certificate Authority had expired at midnight, and every affected service depended on a certificate chain that ultimately validated against it. No automated renewal process or expiration monitoring existed for CA-issued certificates, so the lapse went undetected until it took down every downstream service simultaneously.

### Resolution

1. **Recognized the shared pattern before chasing three separate tickets.** Noticed the near-identical failure timestamps across otherwise unrelated services and treated it as a single shared-dependency incident rather than three separate application problems.
2. **Traced the shared dependency.** Examined the certificate chain on one of the affected services and found the trust failure resolved to an expired intermediate certificate rather than any individual service's own certificate.
3. **Confirmed the scope in the CA console directly.** Opened the Certificate Authority console and confirmed the intermediate certificate's expiration timestamp matched the exact moment the outages began.
4. **Issued a replacement certificate.** Generated and issued a new intermediate certificate with an appropriately extended validity period, following the documented PKI hierarchy.
5. **Redeployed the corrected chain to every affected service individually.** Updated the certificate chain on the web application, VPN gateway, and line-of-business tool one at a time, since each needed the new intermediate certificate installed and trusted before its own service certificate would validate again.
6. **Validated each service independently before closing the ticket.** Tested a full authentication or connection cycle on each of the three affected services individually, rather than assuming a fix on one implied the others were resolved too.
7. **Closed the systemic gap with monitoring.** Implemented certificate-expiration monitoring across the full internal PKI hierarchy with tiered advance warnings, so the next expiration is caught weeks ahead of time rather than discovered through an outage.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Certificate Authority Console (certsrv.msc) | Identified the expired intermediate certificate and issued its replacement. |
| Certificate Chain Viewer (MMC Certificates Snap-in) | Traced each affected service's trust failure back to the shared expired intermediate. |
| PKI Hierarchy Documentation | Confirmed the correct reissuance process and validity parameters. |
| IIS Manager / Service-Specific Certificate Bindings | Redeployed the corrected certificate chain to each affected service. |
| Certificate Expiration Monitoring & Alerting | Implemented post-incident to prevent a silent, multi-service repeat. |
