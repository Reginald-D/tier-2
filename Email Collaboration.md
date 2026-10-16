<div align="center">

# IT Helpdesk Portfolio
### Email & Collaboration Platform — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Email%20%26%20Collaboration%20Platform-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-5-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-Exchange%20Online%20%7C%20SharePoint%20%7C%20Teams-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases five representative incident tickets spanning the most common **email & collaboration platform** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0454602`](#inc0454602--legitimate-vendor-invoices-silently-blocked-by-overly-broad-mail-flow-rule) | Legitimate vendor invoices silently blocked by overly broad mail flow rule | P2 | 3h 15m |
| 2 | [`INC0455140`](#inc0455140--primary-mailbox-database-fails-to-mount-after-unexpected-server-crash) | Primary mailbox database fails to mount after unexpected server crash | P1 | 4h 30m |
| 3 | [`INC0455688`](#inc0455688--sharepoint-document-library-permissions-broken-after-site-migration) | SharePoint document library permissions broken after site migration | P2 | 3h 50m |
| 4 | [`INC0456210`](#inc0456210--litigation-hold-found-not-covering-a-custodians-project-shared-mailbox) | Litigation hold found not covering a custodian's project shared mailbox | P1 | 6h 00m |
| 5 | [`INC0456777`](#inc0456777--hybrid-mail-flow-experiencing-inconsistent-delivery-delays) | Hybrid mail flow experiencing inconsistent delivery delays | P2 | 4h 55m |

---

<br>

## `INC0454602` — Legitimate vendor invoices silently blocked by overly broad mail flow rule

<table>
<tr><td><b>Environment</b></td><td>Exchange Online / Microsoft 365 tenant</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Email & Collaboration > Exchange Online Mail Flow</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 15m</td></tr>
</table>

**Reported Symptom**
A finance team reported vendor invoices had stopped arriving with no bounce and no spam folder hit. A separate team mentioned a client complained about a missing proposal around the same time.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Started with message trace, not the rule list. Ran a message trace for the specific missing invoice, searching by sender and approximate date, and found the message had been received but never delivered, with a transport rule listed as the reason.
2. Identified the exact rule responsible. Opened the referenced transport rule and reviewed its conditions and actions, confirming it matched on broad keyword patterns and was set to silently delete matching messages.
3. Quantified the blast radius. Ran a broader message trace across the affected window, filtered for messages matching the rule's action, and found a meaningful number of legitimate emails had been silently caught.
4. Narrowed the rule instead of removing it. Rewrote the rule's conditions to match the actual phishing indicators from the original campaign rather than broad keyword matches that could catch legitimate language.
5. Changed the action from silent delete to quarantine. Updated the rule so any future match would route to quarantine for review, ensuring a future false positive would be recoverable rather than invisible.
6. Recovered what could be recovered. Used the message trace data to identify and notify senders of messages caught during the affected window so lost communications could be reissued.

</details>

**Root Cause**
A transport rule created weeks earlier to block a phishing pattern was scoped too broadly, matching legitimate business terms in ordinary invoices and proposals, and its action silently deleted matching messages rather than quarantining them.

**Resolution**
Updated the rule so any future match would route to quarantine for review, ensuring a future false positive would be recoverable rather than invisible. Used the message trace data to identify and notify senders of messages caught during the affected window so lost communications could be reissued.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0455140` — Primary mailbox database fails to mount after unexpected server crash

<table>
<tr><td><b>Environment</b></td><td>On-premises Exchange server, single database holding ~600 mailboxes</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Email & Collaboration > On-Prem Exchange Database Recovery</td></tr>
<tr><td><b>Time to Resolve</b></td><td>4h 30m</td></tr>
</table>

**Reported Symptom**
An unexpected power event took an Exchange server down hard. When it came back online, its primary mailbox database refused to mount, making mail, calendar, and contacts completely inaccessible for every user on that database.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the exact database state before attempting anything. Ran eseutil /mh against the database file to check its header state directly, confirming Dirty Shutdown rather than assuming based on the mount failure alone.
2. Located and verified the available transaction logs. Checked the log file directory and confirmed a contiguous, unbroken sequence of transaction logs was present covering the period up to the crash.
3. Chose soft recovery over repair. Since the full log sequence was intact, opted for soft recovery rather than eseutil /p (hard repair), because soft recovery carries no risk of data loss while repair can discard corrupted pages.
4. Executed the soft recovery. Ran eseutil /r with the correct log file prefix and path, replaying every available transaction log into the database.
5. Verified consistency before attempting to mount. Re-ran eseutil /mh after recovery completed and confirmed the database header now reported Clean Shutdown.
6. Mounted, validated, and backed up immediately. Mounted the database, ran Test-ServiceHealth and spot-checked several mailboxes for data integrity, and triggered an immediate full backup before anything else touched the database.

</details>

**Root Cause**
The database had been left in a Dirty Shutdown state, meaning its transaction logs had not been fully committed before the crash. Exchange refuses to mount a database in this state without either replaying its full log set or an explicit repair.

**Resolution**
Re-ran eseutil /mh after recovery completed and confirmed the database header now reported Clean Shutdown. Mounted the database, ran Test-ServiceHealth and spot-checked several mailboxes for data integrity, and triggered an immediate full backup before anything else touched the database.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0455688` — SharePoint document library permissions broken after site migration

<table>
<tr><td><b>Environment</b></td><td>Post-migration SharePoint Online site, departmental document libraries</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Email & Collaboration > SharePoint / Teams Permissions</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 50m</td></tr>
</table>

**Reported Symptom**
A department's SharePoint site was migrated as part of a broader reorganization. Most users noticed nothing different, but a smaller group discovered they had lost access to specific document libraries they used daily.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Compared old and new permission structures directly. Pulled a pre-migration permissions export and compared it against the current permissions on the corresponding libraries, isolating exactly which libraries had unique permissions that didn't carry over.
2. Confirmed the scope precisely. Used the SharePoint admin center and PnP PowerShell to enumerate every user and group with unique access on each affected library in the old site.
3. Rebuilt unique permissions on the new libraries. Broke inheritance where necessary and reapplied the specific user and group permissions that had existed in the original structure.
4. Validated access from the user's perspective. Had a representative user from each previously affected group confirm they could access exactly the libraries they were supposed to, with the correct permission level.
5. Extended the audit beyond the reported issue. Ran the same comparison across every other library in the migrated site and caught two additional libraries with the same silent gap.
6. Documented the migration gap for future projects. Flagged unique-permission verification as a required explicit step in the site migration checklist.

</details>

**Root Cause**
The affected libraries had unique permissions on the original site that went beyond the site's general membership, and the migration tooling only carried over baseline inherited permissions, not the unique, library-level overrides.

**Resolution**
Ran the same comparison across every other library in the migrated site and caught two additional libraries with the same silent gap. Flagged unique-permission verification as a required explicit step in the site migration checklist.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0456210` — Litigation hold found not covering a custodian's project shared mailbox

<table>
<tr><td><b>Environment</b></td><td>Microsoft 365 Compliance / Purview, active legal matter</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Email & Collaboration > Compliance Hold / eDiscovery</td></tr>
<tr><td><b>Time to Resolve</b></td><td>6h 00m</td></tr>
</table>

**Reported Symptom**
Legal counsel requested a complete export of a custodian's communications for an active matter. The eDiscovery search returned recent messages, but a stretch from several months back — during a period the custodian used a project shared mailbox — was entirely missing.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the hold's actual scope, not its intended scope. Reviewed the litigation hold configuration in Microsoft Purview and listed every mailbox it was explicitly applied to.
2. Mapped every mailbox associated with the custodian. Worked with the custodian's manager and IT access records to build a complete list of every mailbox the custodian had used, including shared mailboxes.
3. Identified the specific gap. Cross-referenced the complete mailbox list against the hold's actual scope and confirmed the project shared mailbox was the sole gap.
4. Applied the hold retroactively. Added the shared mailbox to the existing litigation hold immediately and confirmed retention policies would govern its content going forward.
5. Checked for recoverable historical content. Searched Recoverable Items and available backup/archive sources for the shared mailbox to determine whether the gap period's content could still be recovered.
6. Re-ran eDiscovery and reported transparently to legal. Re-executed the content search with the corrected mailbox scope and provided legal counsel with a documented account of exactly what was and wasn't covered and why.

</details>

**Root Cause**
The litigation hold had been correctly applied to the custodian's primary mailbox, but a project-specific shared mailbox the custodian had delegated access to and actively used during the relevant period had never been added to the same hold.

**Resolution**
Searched Recoverable Items and available backup/archive sources for the shared mailbox to determine whether the gap period's content could still be recovered. Re-executed the content search with the corrected mailbox scope and provided legal counsel with a documented account of exactly what was and wasn't covered and why.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0456777` — Hybrid mail flow experiencing inconsistent delivery delays

<table>
<tr><td><b>Environment</b></td><td>Hybrid Exchange — on-premises Exchange and Exchange Online, mid-migration</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Email & Collaboration > Hybrid Mail Flow</td></tr>
<tr><td><b>Time to Resolve</b></td><td>4h 55m</td></tr>
</table>

**Reported Symptom**
In a hybrid environment mid-migration, mail between a cloud mailbox and an on-prem mailbox sometimes delivered instantly, sometimes sat for hours, undermining confidence in the migration for users still waiting their turn.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Established the pattern from message tracking, not assumptions. Pulled message tracking logs from both environments for several delayed messages and found a consistent multi-retry pattern before eventual delivery, rather than an outright failure.
2. Reviewed the hybrid connector configuration on both sides. Compared the Send Connector configuration on-premises against the Exchange Online hybrid connector, checking certificate references and TLS requirements.
3. Found the certificate mismatch. Confirmed the on-prem send connector referenced a certificate thumbprint that no longer matched the currently valid certificate bound to the connector.
4. Corrected the certificate binding. Updated the on-prem send connector to reference the current, valid certificate thumbprint and cross-verified the cloud-side configuration matched.
5. Validated with monitored test messages. Sent test messages in both directions and monitored message tracking in real time to confirm delivery now completed without retry delay.
6. Extended validation across a full business day. Continued monitoring live hybrid mail flow across normal business volume for a full day and found zero recurrence of the delay pattern.

</details>

**Root Cause**
The on-prem send connector was referencing a certificate that had been renewed on the Exchange Online side but not correspondingly updated on-premises, causing intermittent TLS negotiation failures that sent affected messages into retry queues.

**Resolution**
Sent test messages in both directions and monitored message tracking in real time to confirm delivery now completed without retry delay. Continued monitoring live hybrid mail flow across normal business volume for a full day and found zero recurrence of the delay pattern.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 5 |
| **Critical (P1) Incidents** | 2 |
| **High Priority (P2) Incidents** | 3 |
| **Average Time to Resolve** | ~4h 30m |
| **Systems Touched** | Exchange Admin Center · Microsoft 365 Defender · PowerShell · Quarantine Management · eseutil · Exchange Management Shell · Exchange Transaction Log Analysis · Exchange Server Backup Tooling |

## Skills Demonstrated

`Exchange Online Mail Flow` · `On-Prem Exchange Database Recovery` · `SharePoint / Teams Permissions` · `Compliance Hold / eDiscovery` · `Hybrid Mail Flow` · `Root Cause Analysis`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across email & collaboration platform — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>