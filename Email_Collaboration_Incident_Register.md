# Email & Collaboration Platform

*A record of resolved Tier 2 incidents spanning Exchange Online, on-prem Exchange, SharePoint/Teams, compliance holds, and hybrid mail flow.*

**Prepared by:** Ruler | Tier 2 Support / Messaging & Collaboration
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0454602 | Legitimate vendor invoices silently blocked by overly broad mail flow rule | P2 — High | Email & Collaboration > Exchange Online Mail Flow | 3h 15m |
| INC0455140 | Primary mailbox database fails to mount after unexpected server crash | P1 — Critical | Email & Collaboration > On-Prem Exchange Database Recovery | 4h 30m |
| INC0455688 | SharePoint document library permissions broken after site migration | P2 — High | Email & Collaboration > SharePoint / Teams Permissions | 3h 50m |
| INC0456210 | Litigation hold found not covering a custodian's project shared mailbox | P1 — Critical | Email & Collaboration > Compliance Hold / eDiscovery | 6h 00m |
| INC0456777 | Hybrid mail flow experiencing inconsistent delivery delays | P2 — High | Email & Collaboration > Hybrid Mail Flow | 4h 55m |

---

## INC0454602 — Legitimate vendor invoices silently blocked by overly broad mail flow rule

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Exchange Online / Microsoft 365 tenant | P2 — High | Email & Collaboration > Exchange Online Mail Flow | 3h 15m | Resolved — Closed |

### Incident Summary

A finance team reported vendor invoices had stopped arriving with no bounce and no spam folder hit. A separate team mentioned a client complained about a missing proposal around the same time.

### Root Cause

A transport rule created weeks earlier to block a phishing pattern was scoped too broadly, matching legitimate business terms in ordinary invoices and proposals, and its action silently deleted matching messages rather than quarantining them.

### Resolution

1. **Started with message trace, not the rule list.** Ran a message trace for the specific missing invoice, searching by sender and approximate date, and found the message had been received but never delivered, with a transport rule listed as the reason.
2. **Identified the exact rule responsible.** Opened the referenced transport rule and reviewed its conditions and actions, confirming it matched on broad keyword patterns and was set to silently delete matching messages.
3. **Quantified the blast radius.** Ran a broader message trace across the affected window, filtered for messages matching the rule's action, and found a meaningful number of legitimate emails had been silently caught.
4. **Narrowed the rule instead of removing it.** Rewrote the rule's conditions to match the actual phishing indicators from the original campaign rather than broad keyword matches that could catch legitimate language.
5. **Changed the action from silent delete to quarantine.** Updated the rule so any future match would route to quarantine for review, ensuring a future false positive would be recoverable rather than invisible.
6. **Recovered what could be recovered.** Used the message trace data to identify and notify senders of messages caught during the affected window so lost communications could be reissued.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Exchange Admin Center (Message Trace) | Traced the missing messages and identified the responsible transport rule. |
| Exchange Admin Center (Mail Flow Rules) | Reviewed, corrected, and re-tested the overly broad rule. |
| Microsoft 365 Defender (Threat Explorer) | Cross-referenced the original phishing campaign to correctly narrow the rule's scope. |
| PowerShell (Exchange Online Management module) | Ran bulk message trace queries to quantify the scope of affected messages. |
| Quarantine Management (Microsoft 365 Defender) | Configured as the corrected action so future matches are recoverable. |

---

## INC0455140 — Primary mailbox database fails to mount after unexpected server crash

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| On-premises Exchange server, single database holding ~600 mailboxes | P1 — Critical | Email & Collaboration > On-Prem Exchange Database Recovery | 4h 30m | Resolved — Closed |

### Incident Summary

An unexpected power event took an Exchange server down hard. When it came back online, its primary mailbox database refused to mount, making mail, calendar, and contacts completely inaccessible for every user on that database.

### Root Cause

The database had been left in a Dirty Shutdown state, meaning its transaction logs had not been fully committed before the crash. Exchange refuses to mount a database in this state without either replaying its full log set or an explicit repair.

### Resolution

1. **Confirmed the exact database state before attempting anything.** Ran eseutil /mh against the database file to check its header state directly, confirming Dirty Shutdown rather than assuming based on the mount failure alone.
2. **Located and verified the available transaction logs.** Checked the log file directory and confirmed a contiguous, unbroken sequence of transaction logs was present covering the period up to the crash.
3. **Chose soft recovery over repair.** Since the full log sequence was intact, opted for soft recovery rather than eseutil /p (hard repair), because soft recovery carries no risk of data loss while repair can discard corrupted pages.
4. **Executed the soft recovery.** Ran eseutil /r with the correct log file prefix and path, replaying every available transaction log into the database.
5. **Verified consistency before attempting to mount.** Re-ran eseutil /mh after recovery completed and confirmed the database header now reported Clean Shutdown.
6. **Mounted, validated, and backed up immediately.** Mounted the database, ran Test-ServiceHealth and spot-checked several mailboxes for data integrity, and triggered an immediate full backup before anything else touched the database.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| eseutil (Exchange Database Utilities) | Diagnosed the exact database state, performed soft recovery, and confirmed consistency before mounting. |
| Exchange Management Shell | Mounted the recovered database and ran service and mailbox health validation. |
| Exchange Transaction Log Analysis | Verified the log sequence was complete before selecting a recovery method. |
| Exchange Server Backup Tooling | Performed an immediate full backup once recovery was confirmed successful. |

---

## INC0455688 — SharePoint document library permissions broken after site migration

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Post-migration SharePoint Online site, departmental document libraries | P2 — High | Email & Collaboration > SharePoint / Teams Permissions | 3h 50m | Resolved — Closed |

### Incident Summary

A department's SharePoint site was migrated as part of a broader reorganization. Most users noticed nothing different, but a smaller group discovered they had lost access to specific document libraries they used daily.

### Root Cause

The affected libraries had unique permissions on the original site that went beyond the site's general membership, and the migration tooling only carried over baseline inherited permissions, not the unique, library-level overrides.

### Resolution

1. **Compared old and new permission structures directly.** Pulled a pre-migration permissions export and compared it against the current permissions on the corresponding libraries, isolating exactly which libraries had unique permissions that didn't carry over.
2. **Confirmed the scope precisely.** Used the SharePoint admin center and PnP PowerShell to enumerate every user and group with unique access on each affected library in the old site.
3. **Rebuilt unique permissions on the new libraries.** Broke inheritance where necessary and reapplied the specific user and group permissions that had existed in the original structure.
4. **Validated access from the user's perspective.** Had a representative user from each previously affected group confirm they could access exactly the libraries they were supposed to, with the correct permission level.
5. **Extended the audit beyond the reported issue.** Ran the same comparison across every other library in the migrated site and caught two additional libraries with the same silent gap.
6. **Documented the migration gap for future projects.** Flagged unique-permission verification as a required explicit step in the site migration checklist.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| SharePoint Admin Center | Compared and rebuilt permission structures on the affected libraries. |
| PnP PowerShell | Enumerated unique permissions at scale to build a complete restoration list. |
| Pre-Migration Permissions Documentation/Export | Served as the source of truth for exactly what access existed before migration. |
| SharePoint Site Permissions UI | Manually validated and applied corrected access grants. |
| Migration Checklist / Change Documentation | Updated to require explicit unique-permission verification on future migrations. |

---

## INC0456210 — Litigation hold found not covering a custodian's project shared mailbox

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Microsoft 365 Compliance / Purview, active legal matter | P1 — Critical | Email & Collaboration > Compliance Hold / eDiscovery | 6h 00m | Resolved — Closed |

### Incident Summary

Legal counsel requested a complete export of a custodian's communications for an active matter. The eDiscovery search returned recent messages, but a stretch from several months back — during a period the custodian used a project shared mailbox — was entirely missing.

### Root Cause

The litigation hold had been correctly applied to the custodian's primary mailbox, but a project-specific shared mailbox the custodian had delegated access to and actively used during the relevant period had never been added to the same hold.

### Resolution

1. **Confirmed the hold's actual scope, not its intended scope.** Reviewed the litigation hold configuration in Microsoft Purview and listed every mailbox it was explicitly applied to.
2. **Mapped every mailbox associated with the custodian.** Worked with the custodian's manager and IT access records to build a complete list of every mailbox the custodian had used, including shared mailboxes.
3. **Identified the specific gap.** Cross-referenced the complete mailbox list against the hold's actual scope and confirmed the project shared mailbox was the sole gap.
4. **Applied the hold retroactively.** Added the shared mailbox to the existing litigation hold immediately and confirmed retention policies would govern its content going forward.
5. **Checked for recoverable historical content.** Searched Recoverable Items and available backup/archive sources for the shared mailbox to determine whether the gap period's content could still be recovered.
6. **Re-ran eDiscovery and reported transparently to legal.** Re-executed the content search with the corrected mailbox scope and provided legal counsel with a documented account of exactly what was and wasn't covered and why.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Purview Compliance Portal | Reviewed the hold's actual scope and applied the correction. |
| Content Search / eDiscovery (Microsoft Purview) | Ran the original and corrected discovery searches. |
| Exchange Online Delegate & Mailbox Access Records | Mapped every mailbox the custodian had actually used. |
| Exchange Recoverable Items / Mailbox Backup Sources | Investigated whether historical content from the gap period was recoverable. |

---

## INC0456777 — Hybrid mail flow experiencing inconsistent delivery delays

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Hybrid Exchange — on-premises Exchange and Exchange Online, mid-migration | P2 — High | Email & Collaboration > Hybrid Mail Flow | 4h 55m | Resolved — Closed |

### Incident Summary

In a hybrid environment mid-migration, mail between a cloud mailbox and an on-prem mailbox sometimes delivered instantly, sometimes sat for hours, undermining confidence in the migration for users still waiting their turn.

### Root Cause

The on-prem send connector was referencing a certificate that had been renewed on the Exchange Online side but not correspondingly updated on-premises, causing intermittent TLS negotiation failures that sent affected messages into retry queues.

### Resolution

1. **Established the pattern from message tracking, not assumptions.** Pulled message tracking logs from both environments for several delayed messages and found a consistent multi-retry pattern before eventual delivery, rather than an outright failure.
2. **Reviewed the hybrid connector configuration on both sides.** Compared the Send Connector configuration on-premises against the Exchange Online hybrid connector, checking certificate references and TLS requirements.
3. **Found the certificate mismatch.** Confirmed the on-prem send connector referenced a certificate thumbprint that no longer matched the currently valid certificate bound to the connector.
4. **Corrected the certificate binding.** Updated the on-prem send connector to reference the current, valid certificate thumbprint and cross-verified the cloud-side configuration matched.
5. **Validated with monitored test messages.** Sent test messages in both directions and monitored message tracking in real time to confirm delivery now completed without retry delay.
6. **Extended validation across a full business day.** Continued monitoring live hybrid mail flow across normal business volume for a full day and found zero recurrence of the delay pattern.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Exchange Message Tracking Logs (on-prem & Exchange Online) | Identified the retry-and-delay pattern pointing to a negotiation issue. |
| Exchange Management Shell (Send Connector configuration) | Reviewed and corrected the on-prem send connector's certificate reference. |
| Exchange Online Hybrid Configuration Wizard / Admin Center | Cross-verified the cloud-side hybrid connector configuration. |
| Certificate Management (on-prem CA / third-party cert) | Confirmed the valid certificate thumbprint and its correct binding. |
| Queue Viewer & Live Mail Flow Monitoring | Validated consistent, delay-free delivery across a full business day. |
