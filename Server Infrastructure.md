<div align="center">

# IT Helpdesk Portfolio
### Server & Infrastructure — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Server%20%26%20Infrastructure-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-6-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-Windows%20Server%20%7C%20DNS%20%7C%20DHCP%20%7C%20PKI-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases six representative incident tickets spanning the most common **server & infrastructure** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0444805`](#inc0444805--domain-controller-services-ntds-dns-netlogon-failed-to-start-after-patch-reboot) | Domain controller services (NTDS, DNS, Netlogon) failed to start after patch reboot | P1 | 4h 50m |
| 2 | [`INC0445312`](#inc0445312--unintended-access-to-confidential-hr-subfolder-found-during-routine-access-audit) | Unintended access to confidential HR subfolder found during routine access audit | P2 | 3h 10m |
| 3 | [`INC0445890`](#inc0445890--new-devices-at-branch-office-failing-to-obtain-an-ip-address) | New devices at branch office failing to obtain an IP address | P2 | 2h 25m |
| 4 | [`INC0446410`](#inc0446410--intermittent-resolution-failures-for-internal-application-traced-to-stale-dns-record) | Intermittent resolution failures for internal application traced to stale DNS record | P2 | 3h 45m |
| 5 | [`INC0446977`](#inc0446977--print-spooler-service-crash-loop-taking-down-printing-for-entire-site) | Print Spooler service crash loop taking down printing for entire site | P2 | 1h 55m |
| 6 | [`INC0447502`](#inc0447502--expired-internal-ca-intermediate-certificate-breaking-multiple-downstream-services) | Expired internal CA intermediate certificate breaking multiple downstream services | P1 | 5h 30m |

---

<br>

## `INC0444805` — Domain controller services (NTDS, DNS, Netlogon) failed to start after patch reboot

<table>
<tr><td><b>Environment</b></td><td>Secondary-site domain controller, Windows Server 2019, ~400 site users</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > Domain Controller</td></tr>
<tr><td><b>Time to Resolve</b></td><td>4h 50m</td></tr>
</table>

**Reported Symptom**
A scheduled patch reboot on a secondary-site domain controller did not return cleanly. Active Directory Domain Services, DNS Server, and Netlogon all failed to start, taking down authentication, file share access, and printer mapping for the entire site. Standard service restarts had no effect.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Triaged the failure sequence before touching any service. Reviewed Services.msc and the System event log to confirm the order of failure. Netlogon's dependency failure was the common thread behind NTDS and DNS reporting degraded, which ruled out three independent faults and pointed to one shared root cause.
2. Identified the DFSR journal wrap. Reviewed the DFS Replication event log and found Event ID 2213, the definitive signature of a journal wrap, confirming DFSR had halted SYSVOL replication rather than risk corrupting it further.
3. Confirmed SYSVOL replication health before choosing a recovery path. Ran dfsrdiag replicationstate and dfsrdiag backlog against the affected DC to verify SYSVOL was in fact not replicating, and checked whether a second domain controller in the forest still held a fully replicated, healthy copy.
4. Selected the lower-risk recovery method. Because a healthy source DC existed, performed a non-authoritative SYSVOL restore rather than an authoritative restore, which carries a materially higher risk of data loss and should only be used when no healthy source remains.
5. Executed the restore and monitored the resync. Set the DFSR SYSVOL replication state to non-authoritative through the appropriate AD attribute, restarted the DFSR service, and monitored the resulting full resync from the healthy DC to completion.
6. Validated full recovery before returning the DC to production. Confirmed Netlogon, NTDS, and DNS all started cleanly once SYSVOL finished syncing, ran dcdiag /v against the recovered DC, and had site users retest authentication, file access, and printing before closing the ticket.

</details>

**Root Cause**
Netlogon was failing because it depends on a fully initialized SYSVOL share, and SYSVOL was stuck behind a DFS Replication (DFSR) journal wrap — a state where DFSR loses confidence in its own change history and halts replication to avoid propagating bad data. NTDS and DNS were reporting degraded for the same underlying reason: both were waiting on the same stalled dependency chain.

**Resolution**
Set the DFSR SYSVOL replication state to non-authoritative through the appropriate AD attribute, restarted the DFSR service, and monitored the resulting full resync from the healthy DC to completion. Confirmed Netlogon, NTDS, and DNS all started cleanly once SYSVOL finished syncing, ran dcdiag /v against the recovered DC, and had site users retest authentication, file access, and printing before closing the ticket.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0445312` — Unintended access to confidential HR subfolder found during routine access audit

<table>
<tr><td><b>Environment</b></td><td>Windows file server, departmental shares (~15 TB, 40 nested shares)</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > File Services</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 10m</td></tr>
</table>

**Reported Symptom**
A routine access audit found a small group of users with read/write access to a subfolder containing confidential HR documents, despite never being granted access to the parent HR share. The permission structure on the parent share itself appeared entirely correct.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Diagnosed the access source directly, not just the folder's listed permissions. Used the Advanced Security / Effective Access tab for one of the flagged users on the subfolder, which showed the access was being granted by an explicit ACL entry rather than inheritance from the parent.
2. Confirmed inheritance had been disabled. Checked the subfolder's Advanced Security Settings and confirmed "Include inheritable permissions from this object's parent" was unchecked, isolating it from the parent share's ACL entirely.
3. Audited every explicit entry on the subfolder. Ran icacls against the subfolder to produce a complete, scriptable list of every explicit ACL entry, confirming exactly which accounts had been added outside of inheritance.
4. Validated each entry against the documented access policy. Cross-referenced the explicit entries against the HR share's documented access policy and confirmed none of the flagged users' access was currently authorized or justified.
5. Removed the stale entries and re-enabled inheritance. Cleared the explicit ACEs and re-enabled inheritance on the subfolder so its permissions once again derived correctly from the parent HR share.
6. Verified the fix and documented the finding. Re-ran the Effective Access check for the previously flagged users to confirm access was now correctly denied, and documented the finding for the HR data owner and the next scheduled access review.

</details>

**Root Cause**
Inheritance had been broken on the specific subfolder at some point during a prior reorganization, and explicit permissions were applied directly to it as a one-off fix for a since-resolved access request. Because explicit NTFS permissions always take precedence over inherited ones, the leftover explicit entry silently overrode the parent share's intended restriction.

**Resolution**
Cleared the explicit ACEs and re-enabled inheritance on the subfolder so its permissions once again derived correctly from the parent HR share. Re-ran the Effective Access check for the previously flagged users to confirm access was now correctly denied, and documented the finding for the HR data owner and the next scheduled access review.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0445890` — New devices at branch office failing to obtain an IP address

<table>
<tr><td><b>Environment</b></td><td>Branch office subnet (/24), dedicated DHCP server</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > DHCP</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 25m</td></tr>
</table>

**Reported Symptom**
A branch office rolling out a batch of new laptops found that none of the new devices could obtain a valid IP address. Existing devices on the network were unaffected; new devices connecting to the office network received an Automatic Private IP Address (169.254.x.x) instead.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the symptom pointed to DHCP specifically. Verified new devices were receiving an APIPA address, the standard signature of a client failing to receive any DHCP offer at all, ruling out a switch port or VLAN issue.
2. Checked scope utilization directly. Opened the DHCP console and reviewed the Address Pool and Leases for the branch subnet, confirming the scope was at 100% utilization with zero addresses available.
3. Reclaimed stale leases to restore immediate capacity. Identified leases with hostnames matching decommissioned equipment and manually cleared them, freeing enough addresses to onboard the new laptop batch the same day.
4. Reviewed historical lease data to confirm the pattern was systemic. Pulled lease history over several months and confirmed this was a sustained growth trend rather than a one-time spike, ruling out a transient cause.
5. Expanded the scope with confirmed available address space. Coordinated with network engineering to confirm adjacent IP space was free and extended the scope's address range to provide adequate headroom for current and near-term device counts.
6. Shortened the lease duration to reduce future churn risk. Reduced the lease time from the default 8 days to a value better suited to the office's device turnover, so stale leases expire and recycle faster going forward.

</details>

**Root Cause**
The subnet's DHCP scope had genuinely run out of available addresses. The scope had been sized appropriately when the office was first provisioned, but growth in laptops, phones, and IoT-connected office equipment had quietly consumed the pool, and the default 8-day lease duration meant stale leases from decommissioned devices were not expiring fast enough to free up space.

**Resolution**
Coordinated with network engineering to confirm adjacent IP space was free and extended the scope's address range to provide adequate headroom for current and near-term device counts. Reduced the lease time from the default 8 days to a value better suited to the office's device turnover, so stale leases expire and recycle faster going forward.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0446410` — Intermittent resolution failures for internal application traced to stale DNS record

<table>
<tr><td><b>Environment</b></td><td>Company-wide internal DNS zone, ~3,000 user base</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > DNS</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 45m</td></tr>
</table>

**Reported Symptom**
An internal application had generated a slow trickle of complaints for several weeks: some users could reach it by name without issue, others got a connection failure, and reloading a few seconds later often resolved it. The pattern was inconsistent enough to initially be dismissed as an application issue.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reframed the issue as DNS rather than application behavior. Correlated user reports against direct nslookup queries run during reported failures and found the resolved IP was inconsistent across identical queries run seconds apart, ruling out the application and pointing at name resolution.
2. Audited the zone directly. Opened DNS Manager and reviewed the records for the affected hostname, finding two A records where there should have been exactly one.
3. Identified which record was stale. Cross-referenced both IPs against the current infrastructure inventory and confirmed one belonged to a server decommissioned during a prior consolidation project.
4. Removed the stale record. Deleted the orphaned A record, leaving only the current, correct entry, and verified resolution consistency with repeated nslookup queries.
5. Closed the systemic gap by enabling scavenging. Enabled DNS scavenging on the zone with appropriate no-refresh and refresh intervals, so records left behind by future decommissioning work age out automatically instead of lingering indefinitely.
6. Flushed client caches and validated end to end. Had affected users run ipconfig /flushdns and confirmed consistent, correct application access across multiple test machines before closing the ticket.

</details>

**Root Cause**
The DNS zone held a stale A record pointing to a server decommissioned months earlier during an infrastructure consolidation, sitting alongside the correct, current record for the same hostname. With two records for one name, round-robin resolution was intermittently handing clients the dead IP. DNS scavenging, which should have aged out the orphaned record automatically, had never been enabled on the zone.

**Resolution**
Enabled DNS scavenging on the zone with appropriate no-refresh and refresh intervals, so records left behind by future decommissioning work age out automatically instead of lingering indefinitely. Had affected users run ipconfig /flushdns and confirmed consistent, correct application access across multiple test machines before closing the ticket.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0446977` — Print Spooler service crash loop taking down printing for entire site

<table>
<tr><td><b>Environment</b></td><td>Site print server, ~25 networked printers</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > Print Services</td></tr>
<tr><td><b>Time to Resolve</b></td><td>1h 55m</td></tr>
</table>

**Reported Symptom**
Within the span of an hour, print failures were reported from across the entire site. The Print Spooler service was crashing and restarting repeatedly, taking every queued job and connected printer down with each crash.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reviewed the crash signature before taking any action. Checked the Application event log and found a consistent crash signature tied to the same faulting module on every occurrence, pointing to a specific stuck job rather than a general server fault.
2. Stopped the crash cycle. Stopped the Print Spooler service entirely rather than letting it keep auto-restarting into the same failure, creating room to investigate without the queue compounding further.
3. Located the corrupted spool files. Navigated to the spooler's queue directory and identified the specific corrupted .SPL and .SHD file pair matching the timestamp of the first crash.
4. Cleared the offending job directly from the file system. Deleted the corrupted spool file pair from the queue directory, since the print management UI was not accessible while the spooler was down.
5. Restarted and monitored under normal load. Restarted the Print Spooler service and monitored it as queued jobs from the rest of the site began processing normally without a repeat crash.
6. Traced the issue back to its source and corrected it. Identified the originating workstation and driver version from the corrupted job's metadata, updated it to the current supported driver, and confirmed no further malformed jobs were generated.

</details>

**Root Cause**
A single corrupted print job, submitted from a workstation using an outdated, mismatched printer driver, was stuck at the front of the spool queue in a state the spooler could not process or discard cleanly. Every time the spooler restarted, it immediately tried to reprocess the same corrupted job and crashed again.

**Resolution**
Restarted the Print Spooler service and monitored it as queued jobs from the rest of the site began processing normally without a repeat crash. Identified the originating workstation and driver version from the corrupted job's metadata, updated it to the current supported driver, and confirmed no further malformed jobs were generated.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0447502` — Expired internal CA intermediate certificate breaking multiple downstream services

<table>
<tr><td><b>Environment</b></td><td>Internal Certificate Authority supporting web application, VPN gateway, and LOB tool</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Server Infrastructure > PKI / Certificate Authority</td></tr>
<tr><td><b>Time to Resolve</b></td><td>5h 30m</td></tr>
</table>

**Reported Symptom**
Overnight, an internal web application began throwing certificate trust errors, a VPN client began rejecting connections, and a line-of-business tool stopped authenticating cleanly — three apparently unrelated failures within the same short window.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Recognized the shared pattern before chasing three separate tickets. Noticed the near-identical failure timestamps across otherwise unrelated services and treated it as a single shared-dependency incident rather than three separate application problems.
2. Traced the shared dependency. Examined the certificate chain on one of the affected services and found the trust failure resolved to an expired intermediate certificate rather than any individual service's own certificate.
3. Confirmed the scope in the CA console directly. Opened the Certificate Authority console and confirmed the intermediate certificate's expiration timestamp matched the exact moment the outages began.
4. Issued a replacement certificate. Generated and issued a new intermediate certificate with an appropriately extended validity period, following the documented PKI hierarchy.
5. Redeployed the corrected chain to every affected service individually. Updated the certificate chain on the web application, VPN gateway, and line-of-business tool one at a time, since each needed the new intermediate certificate installed and trusted before its own service certificate would validate again.
6. Validated each service independently before closing the ticket. Tested a full authentication or connection cycle on each of the three affected services individually, rather than assuming a fix on one implied the others were resolved too.
7. Closed the systemic gap with monitoring. Implemented certificate-expiration monitoring across the full internal PKI hierarchy with tiered advance warnings, so the next expiration is caught weeks ahead of time rather than discovered through an outage.

</details>

**Root Cause**
An intermediate certificate issued by the internal Certificate Authority had expired at midnight, and every affected service depended on a certificate chain that ultimately validated against it. No automated renewal process or expiration monitoring existed for CA-issued certificates, so the lapse went undetected until it took down every downstream service simultaneously.

**Resolution**
Tested a full authentication or connection cycle on each of the three affected services individually, rather than assuming a fix on one implied the others were resolved too. Implemented certificate-expiration monitoring across the full internal PKI hierarchy with tiered advance warnings, so the next expiration is caught weeks ahead of time rather than discovered through an outage.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 6 |
| **Critical (P1) Incidents** | 2 |
| **High Priority (P2) Incidents** | 4 |
| **Average Time to Resolve** | ~3h 36m |
| **Systems Touched** | Services.msc · DFS Replication Event Log · dfsrdiag · ADSI Edit · dcdiag · NTFS Advanced Security · icacls · Active Directory Users and Computers |

## Skills Demonstrated

`Domain Controller Recovery` · `DFS Replication` · `NTFS / File Server Permissions` · `DHCP Management` · `DNS Administration` · `Print Services` · `PKI / Certificate Authority` · `Root Cause Analysis`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across server & infrastructure — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>