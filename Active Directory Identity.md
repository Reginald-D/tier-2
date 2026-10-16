<div align="center">

# IT Helpdesk Portfolio
### Active Directory & Identity Infrastructure — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Active%20Directory%20%26%20Identity%20Infrastructure-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-6-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-Entra%20ID%20%7C%20AD%20%7C%20Hybrid%20Identity-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases six representative incident tickets spanning the most common **active directory & identity infrastructure** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0441287`](#inc0441287--endpoint-hardening-gpo-not-applying-to-finance-ou-workstations) | Endpoint hardening GPO not applying to Finance OU workstations | P3 | 5h 40m |
| 2 | [`INC0441953`](#inc0441953--security-group-inconsistent-between-primary-and-secondary-site-domain-controllers) | Security group inconsistent between primary and secondary site domain controllers | P2 | 2h 10m |
| 3 | [`INC0442510`](#inc0442510--entra-id-connect-delta-sync-halted--conditional-access-pilot-group-not-updating-in-cloud) | Entra ID Connect delta sync halted — Conditional Access pilot group not updating in cloud | P2 | 3h 05m |
| 4 | [`INC0443018`](#inc0443018--conditional-access-policy-locking-out-entire-pilot-user-group) | Conditional Access policy locking out entire pilot user group | P1 | 48m |
| 5 | [`INC0443722`](#inc0443722--intermittent-authentication-failures-on-ad-integrated-application-server) | Intermittent authentication failures on AD-integrated application server | P2 | 6h 15m |
| 6 | [`INC0444206`](#inc0444206--saml-sso-failing-for-all-users-of-internally-integrated-application) | SAML SSO failing for all users of internally integrated application | P1 | 2h 30m |

---

<br>

## `INC0441287` — Endpoint hardening GPO not applying to Finance OU workstations

<table>
<tr><td><b>Environment</b></td><td>Multi-site AD forest — Windows Server 2019 domain controllers, ~1,200 managed endpoints</td></tr>
<tr><td><b>Priority</b></td><td>P3 — Medium</td></tr>
<tr><td><b>Category</b></td><td>Active Directory > Group Policy</td></tr>
<tr><td><b>Time to Resolve</b></td><td>5h 40m</td></tr>
</table>

**Reported Symptom**
A new GPO enforcing endpoint hardening (Defender for Endpoint onboarding, local firewall profile, Credential Guard) applied correctly to two of three targeted OUs. Workstations in the Finance OU continued to show the pre-hardening baseline with no error surfaced to end users or in the GPO status console.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reviewed the GPO's Delegation tab in GPMC and confirmed the applied security group did not match the intended group.
2. Cross-referenced intended scope using Get-GPPermission and Get-ADGroupMember in PowerShell.
3. Re-scoped security filtering to the correct hardening group and removed the stale entry.
4. Identified and reconciled the conflicting Enforced GPO higher in the Finance OU inheritance chain.
5. Forced replication with repadmin /syncall and validated with a second gpresult /h and gpupdate /force on a test machine.
6. Confirmed Defender onboarding, firewall profile, and Credential Guard applied correctly across all Finance OU endpoints.

</details>

**Root Cause**
The GPO had been cloned from an earlier test policy and its security filtering was still scoped to the default "Authenticated Users" group rather than the intended hardening group. A separate legacy GPO higher in the Finance OU's inheritance chain also had "Enforced" set, overriding the settings that did attempt to apply. Confirmed via gpresult /h on an affected workstation, which showed the new GPO as "Denied (Security Filtering)."

**Resolution**
Forced replication with repadmin /syncall and validated with a second gpresult /h and gpupdate /force on a test machine. Confirmed Defender onboarding, firewall profile, and Credential Guard applied correctly across all Finance OU endpoints.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0441953` — Security group inconsistent between primary and secondary site domain controllers

<table>
<tr><td><b>Environment</b></td><td>Hub-and-spoke multi-site AD topology — two physical sites, dedicated DCs per site</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Active Directory > Replication</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 10m</td></tr>
</table>

**Reported Symptom**
A newly created security group used to scope file-share and Conditional Access permissions was present and functional at the primary site but missing entirely at the secondary site. Users authenticating against the secondary site's domain controller were denied access that users at the primary site received without issue.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Ran repadmin /replsummary from the secondary DC to isolate the failing replication partner.
2. Ran dcdiag /v /test:dns and dcdiag /test:replications for detailed diagnostic output.
3. Verified DNS client configuration against documented standard and identified it was pointed at the wrong DNS server.
4. Corrected DNS client settings, flushed the DNS cache, and confirmed SRV record resolution via nslookup.
5. Forced full replication with repadmin /syncall and confirmed the security group populated correctly at the secondary site.
6. Added a post-maintenance DNS validation step to the network change checklist to prevent recurrence.

</details>

**Root Cause**
The secondary site's domain controller had lost the ability to resolve its replication partner due to a DNS client misconfiguration introduced during an unrelated network maintenance window. repadmin /replsummary identified the DC as the outlier, and dcdiag /test:dns confirmed a SRV record resolution failure specific to that DC.

**Resolution**
Forced full replication with repadmin /syncall and confirmed the security group populated correctly at the secondary site. Added a post-maintenance DNS validation step to the network change checklist to prevent recurrence.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0442510` — Entra ID Connect delta sync halted — Conditional Access pilot group not updating in cloud

<table>
<tr><td><b>Environment</b></td><td>Hybrid identity — on-premises AD synchronized to Entra ID via Entra ID Connect</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Identity > Directory Synchronization</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 05m</td></tr>
</table>

**Reported Symptom**
A group membership change made in on-premises AD, intended to scope a Conditional Access pilot, was not reflected in Entra ID after the standard 30-minute delta sync window. Azure AD Connect Health reported the sync engine as running, but the last successful export was several hours old.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reviewed Azure AD Connect Health and identified the export failure and associated error code.
2. Opened Synchronization Service Manager and reviewed the Operations log for the failed export run to isolate the specific object and attribute.
3. Queried the object directly with Get-ADGroup -Properties proxyAddresses to confirm the duplicate value.
4. Corrected the duplicate attribute on the source object in on-premises AD.
5. Triggered a manual delta sync with Start-ADSyncSyncCycle -PolicyType Delta and monitored for clean completion.
6. Confirmed the pilot group's updated membership was present in the Entra admin center before closing the ticket.

</details>

**Root Cause**
A mailbox-enabled security group carried a duplicate proxyAddresses attribute value that violated Entra ID's uniqueness constraint, halting the export step for the full sync batch rather than only the affected object. Isolated via the Operations log in Synchronization Service Manager (miisclient.exe).

**Resolution**
Triggered a manual delta sync with Start-ADSyncSyncCycle -PolicyType Delta and monitored for clean completion. Confirmed the pilot group's updated membership was present in the Entra admin center before closing the ticket.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0443018` — Conditional Access policy locking out entire pilot user group

<table>
<tr><td><b>Environment</b></td><td>Entra ID Conditional Access, Intune-managed device fleet</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Identity > Conditional Access</td></tr>
<tr><td><b>Time to Resolve</b></td><td>48m</td></tr>
</table>

**Reported Symptom**
Shortly after enabling a new Conditional Access policy requiring device compliance for access to core SaaS applications, the entire pilot group — not just non-compliant devices — was unable to sign in, generating an active, business-impacting lockout.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Used the Conditional Access "What If" tool against an affected user to confirm which policy and grant control was denying sign-in.
2. Set the policy to Report-Only mode to restore access immediately while root cause was confirmed.
3. Reviewed Entra ID sign-in logs to confirm the denial reason was consistent across all affected users.
4. Checked device compliance status for the pilot group in Intune and confirmed zero completed enrollments.
5. Rebuilt the policy to require MFA first, with device compliance staged as a second phase pending Intune enrollment completion.
6. Validated the corrected policy with the What If tool and a live test sign-in before re-enabling for the pilot group.

</details>

**Root Cause**
The policy's grant control required device compliance as the sole condition with no alternate (e.g., MFA) path, and none of the pilot group's devices had completed Intune compliance enrollment — a prerequisite that had not been sequenced before the policy went live. Confirmed with the Conditional Access "What If" tool and Entra ID sign-in logs.

**Resolution**
Rebuilt the policy to require MFA first, with device compliance staged as a second phase pending Intune enrollment completion. Validated the corrected policy with the What If tool and a live test sign-in before re-enabling for the pilot group.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0443722` — Intermittent authentication failures on AD-integrated application server

<table>
<tr><td><b>Environment</b></td><td>Windows application server with Windows Integrated Authentication against on-prem AD</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Active Directory > Authentication (Kerberos)</td></tr>
<tr><td><b>Time to Resolve</b></td><td>6h 15m</td></tr>
</table>

**Reported Symptom**
Users accessing an internal web application configured for Windows Integrated Authentication were intermittently prompted for credentials repeatedly and ultimately denied access, despite entering correct credentials. Issue was isolated to a single application server.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reviewed Kerberos-related Event Viewer entries and identified the KRB_AP_ERR_SKEW signature.
2. Quantified time drift with w32tm /stripchart against the PDC emulator.
3. Corrected time source configuration with w32tm /config /syncfromflags:domhier /update and forced resync.
4. Re-tested and found failures persisting intermittently; checked for SPN conflicts with setspn -X.
5. Identified and removed the duplicate SPN registration from the decommissioned service account with setspn -D.
6. Purged cached tickets on a test client with klist purge and confirmed clean re-authentication with no further prompts.

</details>

**Root Cause**
Two compounding faults: the application server's system clock had drifted approximately six minutes from the domain time hierarchy, exceeding Kerberos's five-minute tolerance (Event ID 4 / KRB_AP_ERR_SKEW); and a duplicate Service Principal Name (SPN) existed on a decommissioned service account, causing residual authentication ambiguity even after the clock was corrected.

**Resolution**
Identified and removed the duplicate SPN registration from the decommissioned service account with setspn -D. Purged cached tickets on a test client with klist purge and confirmed clean re-authentication with no further prompts.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0444206` — SAML SSO failing for all users of internally integrated application

<table>
<tr><td><b>Environment</b></td><td>Entra ID enterprise application (SAML federation) integrated with a line-of-business SaaS application</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Identity > Federation / SSO</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 30m</td></tr>
</table>

**Reported Symptom**
Users attempting single sign-on into a SAML-integrated application began receiving a generic "SSO failed" error for all sign-in attempts. Direct sign-in to Entra ID itself was unaffected, isolating the fault to the federation trust rather than core identity services.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Reviewed the Entra ID Enterprise Application's Single Sign-On configuration, which flagged the signing certificate as expired.
2. Cross-checked the downstream application's SAML configuration and confirmed it still referenced the expired certificate's thumbprint.
3. Reviewed Entra ID sign-in logs and confirmed failures correlated with SAML response signature validation errors.
4. Generated a new SAML signing certificate in the Entra ID enterprise application configuration.
5. Downloaded the updated federation metadata and applied it to the downstream application's SAML configuration.
6. Performed a full end-to-end SSO test and confirmed successful authentication before closing the ticket.
7. Implemented certificate expiration monitoring on the enterprise application registration to prevent recurrence.

</details>

**Root Cause**
The SAML signing certificate on the Entra ID enterprise application registration had expired. No certificate rotation automation or expiration alerting existed, so the trust relationship broke the moment the certificate lapsed with no preceding change event on either system.

**Resolution**
Performed a full end-to-end SSO test and confirmed successful authentication before closing the ticket. Implemented certificate expiration monitoring on the enterprise application registration to prevent recurrence.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 6 |
| **Critical (P1) Incidents** | 2 |
| **High Priority (P2) Incidents** | 3 |
| **Medium Priority (P3) Incidents** | 1 |
| **Average Time to Resolve** | ~3h 25m |
| **Systems Touched** | Active Directory · Group Policy · Entra ID · DNS · Kerberos · SAML / SSO |

## Skills Demonstrated

`Group Policy Management` · `Active Directory Replication` · `Entra ID Connect / Hybrid Sync` · `Conditional Access` · `Kerberos Authentication` · `SAML Federation / SSO` · `Root Cause Analysis` · `ServiceNow-Style Incident Documentation`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across active directory & identity infrastructure — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>