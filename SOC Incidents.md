<div align="center">

# IT Helpdesk Portfolio
### Security Operations — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Security%20Operations-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-5-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-Sentinel%20%7C%20Defender%20%7C%20DLP%20%7C%20PAM-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases five representative incident tickets spanning the most common **security operations** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0457330`](#inc0457330--impossible-travel-sign-in-alert-requiring-cross-system-correlation) | Impossible-travel sign-in alert requiring cross-system correlation | P1 | 2h 20m |
| 2 | [`INC0457884`](#inc0457884--edr-generating-excessive-alerts-for-a-legitimate-internal-deployment-tool) | EDR generating excessive alerts for a legitimate internal deployment tool | P3 | 3h 30m |
| 3 | [`INC0458410`](#inc0458410--privileged-credential-checkout-failure-during-active-infrastructure-incident) | Privileged credential checkout failure during active infrastructure incident | P1 | 2h 45m |
| 4 | [`INC0458955`](#inc0458955--dlp-policy-blocking-finance-teams-approved-monthly-vendor-payment-email) | DLP policy blocking finance team's approved monthly vendor payment email | P2 | 2h 10m |
| 5 | [`INC0459501`](#inc0459501--suspected-lateral-movement-investigation-across-multiple-hosts) | Suspected lateral movement investigation across multiple hosts | P1 | 5h 50m |

---

<br>

## `INC0457330` — Impossible-travel sign-in alert requiring cross-system correlation

<table>
<tr><td><b>Environment</b></td><td>Microsoft Sentinel workspace, hybrid identity environment</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Security Operations > SIEM Alert Triage</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 20m</td></tr>
</table>

**Reported Symptom**
A Sentinel alert fired for an impossible-travel sign-in: the same account authenticating from two distant locations within a window too short for legitimate travel — a pattern common enough to be a routine false positive from VPN exit nodes.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Refused to close the alert on pattern-matching alone. Pulled the full sign-in log entry to check the token type and authentication method, which showed session token reuse rather than a fresh interactive login.
2. Correlated against endpoint telemetry. Queried Microsoft Defender for Endpoint for unusual process activity on the user's known device around the same timestamp and found none, confirming the second location's activity wasn't from the user's actual machine.
3. Pulled mail audit logs for the account. Ran a KQL query against the Office 365 mail audit log and found a newly created inbox rule, added minutes after the sign-in, silently forwarding incoming email to an external address.
4. Checked cloud app activity for lateral reach. Reviewed the account's activity across connected SaaS applications and found a brief attempt to access a file-sharing application the user didn't normally use.
5. Built the full correlated timeline before acting. Assembled the sign-in event, absent endpoint activity, mail rule creation, and cloud app probe into a single timeline, making the true-positive determination unambiguous.
6. Contained and eradicated based on the full picture. Disabled the account, revoked all active sessions and tokens, removed the malicious inbox rule, forced a credential reset with MFA re-enrollment, and reviewed the file-sharing application for unauthorized access.

</details>

**Root Cause**
The account had been compromised through a phishing kit that harvested both the password and an active session token, allowing the attacker to bypass MFA by replaying a valid session rather than authenticating fresh.

**Resolution**
Assembled the sign-in event, absent endpoint activity, mail rule creation, and cloud app probe into a single timeline, making the true-positive determination unambiguous. Disabled the account, revoked all active sessions and tokens, removed the malicious inbox rule, forced a credential reset with MFA re-enrollment, and reviewed the file-sharing application for unauthorized access.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0457884` — EDR generating excessive alerts for a legitimate internal deployment tool

<table>
<tr><td><b>Environment</b></td><td>Microsoft Defender for Endpoint, enterprise device fleet</td></tr>
<tr><td><b>Priority</b></td><td>P3 — Medium</td></tr>
<tr><td><b>Category</b></td><td>Security Operations > EDR Tuning</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 30m</td></tr>
</table>

**Reported Symptom**
A legitimate internal deployment tool used daily by IT was triggering the same EDR detection dozens of times a day, and analysts had started reflexively dismissing the alert type without reading it closely.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Validated the tool's legitimacy before touching the rule. Confirmed the deployment tool's publisher, code-signing certificate, and file hash against the internal software inventory rather than trusting the process name alone.
2. Quantified exactly how much noise it was generating. Queried the EDR's alert history for the detection over the past thirty days and confirmed the deployment tool accounted for the large majority of that alert type.
3. Checked what else triggered the same detection. Reviewed the remaining alerts not attributable to the tool to confirm none were themselves suspicious before building an exclusion.
4. Built a narrow, hash-and-path-scoped exclusion. Created the exclusion scoped specifically to the verified file hash and installation path, deliberately avoiding a broad process-name exclusion an attacker could exploit.
5. Preserved detection for a spoofed version of the tool. Verified a file with the same process name but a different hash or path would still trigger the original detection normally.
6. Monitored the alert volume post-tuning. Tracked the detection's daily alert count for two weeks, confirming a dramatic drop while spot-checking that any new instances were genuinely worth investigation.

</details>

**Root Cause**
The deployment tool's normal operation — using a scripting engine to install software with elevated rights — matched a broad behavioral detection rule almost exactly, and no exclusion had been created for it.

**Resolution**
Verified a file with the same process name but a different hash or path would still trigger the original detection normally. Tracked the detection's daily alert count for two weeks, confirming a dramatic drop while spot-checking that any new instances were genuinely worth investigation.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0458410` — Privileged credential checkout failure during active infrastructure incident

<table>
<tr><td><b>Environment</b></td><td>Privileged Access Management (PAM) platform, emergency credential vaulting</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Security Operations > PAM / Credential Vaulting</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 45m</td></tr>
</table>

**Reported Symptom**
During an active infrastructure incident requiring urgent administrative access, the on-call engineer attempted to check out a privileged credential from the PAM vault and received a hard failure.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Did not let the vault failure block the active incident. Immediately used the documented break-glass emergency access procedure to restore administrative access and keep the primary incident response moving.
2. Investigated the vault failure in parallel. Pulled the PAM platform's rotation job logs and found the most recent scheduled rotation had failed with a policy rejection error from the target system.
3. Confirmed the account's actual live state. Checked the target system's local account status directly and confirmed it had been left in a partially changed state by the failed rotation.
4. Manually reset the account to a known-good state. Used a separate administrative path to set the account's password to a value meeting its current complexity policy.
5. Re-synchronized the vault to match reality. Updated the vault's stored credential record to match the manually set password and verified a successful check-out/check-in cycle.
6. Fixed the rotation job so it couldn't fail silently again. Corrected the rotation job's password generation policy to align with current complexity requirements and added alerting so a future failure generates an immediate notification.

</details>

**Root Cause**
The vaulted credential's automated password rotation job had failed silently on its most recent run because the target system had rejected the new password due to a recently tightened complexity policy, leaving the account in a broken, inconsistent state.

**Resolution**
Updated the vault's stored credential record to match the manually set password and verified a successful check-out/check-in cycle. Corrected the rotation job's password generation policy to align with current complexity requirements and added alerting so a future failure generates an immediate notification.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0458955` — DLP policy blocking finance team's approved monthly vendor payment email

<table>
<tr><td><b>Environment</b></td><td>Microsoft Purview DLP, finance distribution group</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Security Operations > DLP Policy Tuning</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 10m</td></tr>
</table>

**Reported Symptom**
The finance team's routine monthly process of emailing verified vendor payment details to an approved payment processor was blocked outright by a DLP policy designed to prevent exactly that kind of data from leaving uncontrolled.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the workflow's legitimacy before changing anything. Verified with the finance team and their manager that the blocked email was the expected, pre-approved monthly transmission to a known, contracted payment processor.
2. Reviewed exactly what the policy matched on. Pulled the DLP incident report and confirmed it was correctly detecting genuine bank account/routing patterns, ruling out a pattern-matching fix.
3. Evaluated the recipient and workflow specifically. Confirmed the payment processor's domain was a known, contracted business partner already documented in the approved vendor list.
4. Built a precisely scoped exception. Created a DLP exception limited specifically to mail from the finance distribution group to the payment processor's verified domain, leaving the policy fully intact for every other scenario.
5. Validated the exception's precision. Tested that the same finance user sending similar data to an unapproved external address was still correctly blocked.
6. Documented the exception's business justification. Recorded the approval chain and exact scope of the exception for future audit and periodic review.

</details>

**Root Cause**
The DLP policy's sensitive information pattern was matching correctly — genuine bank account and routing number formats — but no exception existed for this specific, pre-approved business workflow.

**Resolution**
Tested that the same finance user sending similar data to an unapproved external address was still correctly blocked. Recorded the approval chain and exact scope of the exception for future audit and periodic review.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0459501` — Suspected lateral movement investigation across multiple hosts

<table>
<tr><td><b>Environment</b></td><td>Multi-host enterprise network, EDR-monitored endpoints</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Security Operations > Lateral Movement Investigation</td></tr>
<tr><td><b>Time to Resolve</b></td><td>5h 50m</td></tr>
</table>

**Reported Symptom**
An EDR alert flagged unusual use of a legitimate remote administration protocol from one workstation to a small number of other hosts, originating from a user account with no administrative responsibilities.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Established a scoped hypothesis before chasing every host. Started from the single alerting workstation and the small number of hosts it had connected to, keeping the initial investigation focused.
2. Confirmed the initial foothold. Reviewed the originating workstation's EDR process timeline and found a malicious document opened shortly before the first unusual outbound connection.
3. Traced authentication activity across every touched host. Pulled Windows security event logs from each connected host and confirmed successful authentication using harvested local account credentials rather than the user's own domain credentials.
4. Expanded the scope only as evidence justified it. For each newly confirmed host, checked its own outbound activity for further lateral movement before expanding the investigation boundary.
5. Isolated every confirmed host simultaneously. Once the full set of genuinely compromised hosts was confirmed, isolated all of them at the same time to prevent the attacker from reacting to a partial response.
6. Eradicated and validated clean before reconnecting. Removed persistence mechanisms, reset the harvested local account credentials environment-wide, and validated each host was clean before returning it to the network.

</details>

**Root Cause**
The originating workstation had been compromised through a malicious document that established a foothold and used harvested local credentials to authenticate to other hosts via the same remote administration protocol IT staff use legitimately.

**Resolution**
Once the full set of genuinely compromised hosts was confirmed, isolated all of them at the same time to prevent the attacker from reacting to a partial response. Removed persistence mechanisms, reset the harvested local account credentials environment-wide, and validated each host was clean before returning it to the network.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 5 |
| **Critical (P1) Incidents** | 3 |
| **High Priority (P2) Incidents** | 1 |
| **Medium Priority (P3) Incidents** | 1 |
| **Average Time to Resolve** | ~3h 19m |
| **Systems Touched** | Microsoft Sentinel · Microsoft Entra ID Sign-In Logs · Microsoft Defender for Endpoint · Office 365 Unified Audit Log · Sentinel Analytics Rules · Defender for Endpoint Indicator & Exclusion Management · Code Signing Certificate · Controlled Test Execution |

## Skills Demonstrated

`SIEM Alert Triage (KQL)` · `EDR Tuning` · `Privileged Access Management` · `DLP Policy Tuning` · `Lateral Movement Investigation` · `Threat Correlation`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across security operations — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>