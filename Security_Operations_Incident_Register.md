# Security Operations

*A record of resolved Tier 2 incidents spanning SIEM correlation, EDR tuning, PAM, DLP, and lateral movement investigation.*

**Prepared by:** Ruler | Tier 2 Support / Security Operations
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0457330 | Impossible-travel sign-in alert requiring cross-system correlation | P1 — Critical | Security Operations > SIEM Alert Triage | 2h 20m |
| INC0457884 | EDR generating excessive alerts for a legitimate internal deployment tool | P3 — Medium | Security Operations > EDR Tuning | 3h 30m |
| INC0458410 | Privileged credential checkout failure during active infrastructure incident | P1 — Critical | Security Operations > PAM / Credential Vaulting | 2h 45m |
| INC0458955 | DLP policy blocking finance team's approved monthly vendor payment email | P2 — High | Security Operations > DLP Policy Tuning | 2h 10m |
| INC0459501 | Suspected lateral movement investigation across multiple hosts | P1 — Critical | Security Operations > Lateral Movement Investigation | 5h 50m |

---

## INC0457330 — Impossible-travel sign-in alert requiring cross-system correlation

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Microsoft Sentinel workspace, hybrid identity environment | P1 — Critical | Security Operations > SIEM Alert Triage | 2h 20m | Resolved — Closed (True Positive) |

### Incident Summary

A Sentinel alert fired for an impossible-travel sign-in: the same account authenticating from two distant locations within a window too short for legitimate travel — a pattern common enough to be a routine false positive from VPN exit nodes.

### Root Cause

The account had been compromised through a phishing kit that harvested both the password and an active session token, allowing the attacker to bypass MFA by replaying a valid session rather than authenticating fresh.

### Resolution

1. **Refused to close the alert on pattern-matching alone.** Pulled the full sign-in log entry to check the token type and authentication method, which showed session token reuse rather than a fresh interactive login.
2. **Correlated against endpoint telemetry.** Queried Microsoft Defender for Endpoint for unusual process activity on the user's known device around the same timestamp and found none, confirming the second location's activity wasn't from the user's actual machine.
3. **Pulled mail audit logs for the account.** Ran a KQL query against the Office 365 mail audit log and found a newly created inbox rule, added minutes after the sign-in, silently forwarding incoming email to an external address.
4. **Checked cloud app activity for lateral reach.** Reviewed the account's activity across connected SaaS applications and found a brief attempt to access a file-sharing application the user didn't normally use.
5. **Built the full correlated timeline before acting.** Assembled the sign-in event, absent endpoint activity, mail rule creation, and cloud app probe into a single timeline, making the true-positive determination unambiguous.
6. **Contained and eradicated based on the full picture.** Disabled the account, revoked all active sessions and tokens, removed the malicious inbox rule, forced a credential reset with MFA re-enrollment, and reviewed the file-sharing application for unauthorized access.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Sentinel (KQL Queries) | Correlated identity, endpoint, mail, and cloud app logs into a single investigative timeline. |
| Microsoft Entra ID Sign-In Logs | Identified the session token reuse pattern distinguishing this from a routine false positive. |
| Microsoft Defender for Endpoint | Confirmed the absence of corresponding activity on the user's actual device. |
| Office 365 Unified Audit Log | Discovered the malicious inbox forwarding rule used for persistence. |
| Sentinel Analytics Rules (Custom Correlation) | Codified the confirmed attack pattern into an automated detection for future occurrences. |

---

## INC0457884 — EDR generating excessive alerts for a legitimate internal deployment tool

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Microsoft Defender for Endpoint, enterprise device fleet | P3 — Medium | Security Operations > EDR Tuning | 3h 30m | Resolved — Closed |

### Incident Summary

A legitimate internal deployment tool used daily by IT was triggering the same EDR detection dozens of times a day, and analysts had started reflexively dismissing the alert type without reading it closely.

### Root Cause

The deployment tool's normal operation — using a scripting engine to install software with elevated rights — matched a broad behavioral detection rule almost exactly, and no exclusion had been created for it.

### Resolution

1. **Validated the tool's legitimacy before touching the rule.** Confirmed the deployment tool's publisher, code-signing certificate, and file hash against the internal software inventory rather than trusting the process name alone.
2. **Quantified exactly how much noise it was generating.** Queried the EDR's alert history for the detection over the past thirty days and confirmed the deployment tool accounted for the large majority of that alert type.
3. **Checked what else triggered the same detection.** Reviewed the remaining alerts not attributable to the tool to confirm none were themselves suspicious before building an exclusion.
4. **Built a narrow, hash-and-path-scoped exclusion.** Created the exclusion scoped specifically to the verified file hash and installation path, deliberately avoiding a broad process-name exclusion an attacker could exploit.
5. **Preserved detection for a spoofed version of the tool.** Verified a file with the same process name but a different hash or path would still trigger the original detection normally.
6. **Monitored the alert volume post-tuning.** Tracked the detection's daily alert count for two weeks, confirming a dramatic drop while spot-checking that any new instances were genuinely worth investigation.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Defender for Endpoint (Advanced Hunting / KQL) | Quantified alert volume and isolated the deployment tool as the dominant source. |
| Defender for Endpoint Indicator & Exclusion Management | Built and applied the hash-and-path-scoped exclusion. |
| Code Signing Certificate / File Hash Verification | Validated the tool's legitimacy against its official publisher. |
| Controlled Test Execution (Isolated Environment) | Confirmed a spoofed version of the tool would still trigger detection. |
| Detection Alert Volume Monitoring / Reporting | Validated the tuning's real-world impact over a two-week period. |

---

## INC0458410 — Privileged credential checkout failure during active infrastructure incident

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Privileged Access Management (PAM) platform, emergency credential vaulting | P1 — Critical | Security Operations > PAM / Credential Vaulting | 2h 45m | Resolved — Closed |

### Incident Summary

During an active infrastructure incident requiring urgent administrative access, the on-call engineer attempted to check out a privileged credential from the PAM vault and received a hard failure.

### Root Cause

The vaulted credential's automated password rotation job had failed silently on its most recent run because the target system had rejected the new password due to a recently tightened complexity policy, leaving the account in a broken, inconsistent state.

### Resolution

1. **Did not let the vault failure block the active incident.** Immediately used the documented break-glass emergency access procedure to restore administrative access and keep the primary incident response moving.
2. **Investigated the vault failure in parallel.** Pulled the PAM platform's rotation job logs and found the most recent scheduled rotation had failed with a policy rejection error from the target system.
3. **Confirmed the account's actual live state.** Checked the target system's local account status directly and confirmed it had been left in a partially changed state by the failed rotation.
4. **Manually reset the account to a known-good state.** Used a separate administrative path to set the account's password to a value meeting its current complexity policy.
5. **Re-synchronized the vault to match reality.** Updated the vault's stored credential record to match the manually set password and verified a successful check-out/check-in cycle.
6. **Fixed the rotation job so it couldn't fail silently again.** Corrected the rotation job's password generation policy to align with current complexity requirements and added alerting so a future failure generates an immediate notification.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| PAM Platform (Credential Vaulting & Rotation) | Diagnosed the failed rotation job and re-synchronized the vault. |
| Break-Glass Emergency Access Procedure | Provided immediate administrative access without waiting on the failed vault checkout. |
| PAM Rotation Job Logs | Identified the exact policy rejection error causing the silent failure. |
| Target System Local Account Administration | Manually restored the account to a known, consistent password state. |
| PAM Alerting & Monitoring Configuration | Implemented so future rotation failures generate immediate notification. |

---

## INC0458955 — DLP policy blocking finance team's approved monthly vendor payment email

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Microsoft Purview DLP, finance distribution group | P2 — High | Security Operations > DLP Policy Tuning | 2h 10m | Resolved — Closed |

### Incident Summary

The finance team's routine monthly process of emailing verified vendor payment details to an approved payment processor was blocked outright by a DLP policy designed to prevent exactly that kind of data from leaving uncontrolled.

### Root Cause

The DLP policy's sensitive information pattern was matching correctly — genuine bank account and routing number formats — but no exception existed for this specific, pre-approved business workflow.

### Resolution

1. **Confirmed the workflow's legitimacy before changing anything.** Verified with the finance team and their manager that the blocked email was the expected, pre-approved monthly transmission to a known, contracted payment processor.
2. **Reviewed exactly what the policy matched on.** Pulled the DLP incident report and confirmed it was correctly detecting genuine bank account/routing patterns, ruling out a pattern-matching fix.
3. **Evaluated the recipient and workflow specifically.** Confirmed the payment processor's domain was a known, contracted business partner already documented in the approved vendor list.
4. **Built a precisely scoped exception.** Created a DLP exception limited specifically to mail from the finance distribution group to the payment processor's verified domain, leaving the policy fully intact for every other scenario.
5. **Validated the exception's precision.** Tested that the same finance user sending similar data to an unapproved external address was still correctly blocked.
6. **Documented the exception's business justification.** Recorded the approval chain and exact scope of the exception for future audit and periodic review.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Purview DLP (Policy Management) | Reviewed the matched pattern and built the precisely scoped exception. |
| DLP Incident Reports | Confirmed the policy was correctly detecting genuine sensitive data. |
| Approved Third-Party Vendor Documentation | Established the legitimate business basis for the exception. |
| Exchange Online Mail Flow Testing | Validated the exception applied only to the intended sender/recipient pair. |
| Policy Exception Documentation & Change Records | Recorded the business justification for future audit. |

---

## INC0459501 — Suspected lateral movement investigation across multiple hosts

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Multi-host enterprise network, EDR-monitored endpoints | P1 — Critical | Security Operations > Lateral Movement Investigation | 5h 50m | Resolved — Closed (True Positive, Contained) |

### Incident Summary

An EDR alert flagged unusual use of a legitimate remote administration protocol from one workstation to a small number of other hosts, originating from a user account with no administrative responsibilities.

### Root Cause

The originating workstation had been compromised through a malicious document that established a foothold and used harvested local credentials to authenticate to other hosts via the same remote administration protocol IT staff use legitimately.

### Resolution

1. **Established a scoped hypothesis before chasing every host.** Started from the single alerting workstation and the small number of hosts it had connected to, keeping the initial investigation focused.
2. **Confirmed the initial foothold.** Reviewed the originating workstation's EDR process timeline and found a malicious document opened shortly before the first unusual outbound connection.
3. **Traced authentication activity across every touched host.** Pulled Windows security event logs from each connected host and confirmed successful authentication using harvested local account credentials rather than the user's own domain credentials.
4. **Expanded the scope only as evidence justified it.** For each newly confirmed host, checked its own outbound activity for further lateral movement before expanding the investigation boundary.
5. **Isolated every confirmed host simultaneously.** Once the full set of genuinely compromised hosts was confirmed, isolated all of them at the same time to prevent the attacker from reacting to a partial response.
6. **Eradicated and validated clean before reconnecting.** Removed persistence mechanisms, reset the harvested local account credentials environment-wide, and validated each host was clean before returning it to the network.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Defender for Endpoint (Process Timeline) | Confirmed the initial malicious document execution and credential harvesting activity. |
| Windows Security Event Logs (Logon Events) | Traced authentication activity across every touched host. |
| Microsoft Sentinel (Cross-Host KQL Correlation) | Correlated connection and authentication activity to scope the full compromise. |
| EDR Network Isolation | Simultaneously isolated every confirmed compromised host. |
| Sentinel Analytics Rules (Behavioral Detection) | Codified the confirmed evasion technique into a standing detection. |
