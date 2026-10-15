# Endpoint Management & Automation

*A record of resolved Tier 2 incidents spanning Intune/Autopilot, provisioning automation, co-management, BitLocker, and patch deployment.*

**Prepared by:** Ruler | Tier 2 Support / Endpoint Management & Automation
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0451330 | Intune application deployment stalls at 60% across device collection | P2 — High | Endpoint Management > Intune Deployment | 3h 40m |
| INC0451884 | New-hire provisioning script silently failing for a subset of usernames | P3 — Medium | Endpoint Management > Automated Provisioning | 4h 20m |
| INC0452410 | Screen lock timeout setting flip-flopping on co-managed laptops | P3 — Medium | Endpoint Management > GPO / Intune Policy Conflict | 5h 05m |
| INC0452955 | Autopilot enrollment failing on roughly one in seven devices during mass rollout | P2 — High | Endpoint Management > Autopilot / Imaging Pipeline | 6h 30m |
| INC0453501 | BitLocker recovery key missing from AD and Entra ID during active recovery request | P2 — High | Endpoint Management > BitLocker Key Escrow | 7h 10m |
| INC0454028 | Monthly security patch causes line-of-business application to crash on launch | P1 — Critical | Endpoint Management > Patch Management | 4h 45m |

---

## INC0451330 — Intune application deployment stalls at 60% across device collection

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Company-wide Intune-managed device collection (~800 devices) | P2 — High | Endpoint Management > Intune Deployment | 3h 40m | Resolved — Closed |

### Incident Summary

A required application deployment pushed through Intune climbed steadily to roughly sixty percent installed, then stopped moving entirely. Devices already showing "Pending" stayed pending with no error surfaced in the console.

### Root Cause

The stalled devices were running an outdated build of the Intune Management Extension with a known check-in reliability issue, so they had effectively stopped syncing with Intune and never received the deployment policy at all.

### Resolution

1. **Isolated a representative stalled device.** Picked one machine stuck at Pending and forced a manual sync from the Company Portal app to see whether it would respond to a direct push.
2. **Reviewed the extension logs directly.** Checked IntuneManagementExtension.log on the stalled device and found repeated check-in failures matching a known issue in an older extension build.
3. **Confirmed the scope of the problem.** Queried the device collection's Intune Management Extension version distribution and found every stalled device shared the same outdated build.
4. **Forced the extension update at scale.** Pushed the current Intune Management Extension version to the affected devices as a required update ahead of the stalled deployment.
5. **Verified renewed check-in and deployment progress.** Confirmed via the extension log and Intune console that updated devices resumed checking in on schedule and the stalled deployment began progressing again.
6. **Closed the gap for future deployments.** Added an Intune Management Extension version compliance policy so any future device falling behind gets flagged and remediated automatically.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Intune Admin Center | Tracked deployment progress and identified the plateau at a specific device subset. |
| IntuneManagementExtension.log | Diagnosed the specific check-in failure pattern on stalled devices. |
| Company Portal App | Forced manual sync attempts during initial triage. |
| Intune Device Compliance Policies | Implemented post-incident to catch future extension version drift automatically. |
| PowerShell (Intune/Graph reporting scripts) | Queried the device collection at scale to confirm the shared extension version. |

---

## INC0451884 — New-hire provisioning script silently failing for a subset of usernames

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| New-hire onboarding automation (PowerShell), HR-to-IT provisioning pipeline | P3 — Medium | Endpoint Management > Automated Provisioning | 4h 20m | Resolved — Closed |

### Incident Summary

The automated PowerShell workflow responsible for creating new-hire home directories had run without complaint for months until a new hire's home directory simply didn't exist on their first day, with no failure logged anywhere.

### Root Cause

The script had no real error handling and assumed every username would produce a valid file path. A small number of edge cases — usernames with special characters, or paths that exceeded the legacy Windows MAX_PATH limit — caused the directory creation command to fail silently, and the script simply moved on.

### Resolution

1. **Reproduced the failure deliberately.** Built test cases using known problematic patterns — special characters in usernames and a deliberately long nested path — and ran the script against them in a test environment to force the failure predictably.
2. **Confirmed the silent failure mode.** Watched the script run to completion without any error output on the crafted test cases, confirming the directory creation command was failing silently.
3. **Added structured error handling.** Wrapped the core directory and permission-setting logic in try/catch blocks, logging the specific username, path, and error to a persistent log file.
4. **Fixed the two root causes directly.** Added input sanitization for special characters and enabled Windows long path support at the system level to remove the MAX_PATH ceiling.
5. **Re-ran the full test suite.** Re-executed every previously failing test case and confirmed each now completed successfully with clean logging.
6. **Backfilled the real-world gap.** Cross-referenced HR's new-hire list against provisioning records to identify every employee who had silently fallen through the bug and manually created their missing directories.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| PowerShell ISE / VS Code (PowerShell extension) | Rebuilt the script with structured error handling and diagnostic logging. |
| Windows Long Path Support (Group Policy / Registry) | Removed the legacy MAX_PATH limitation. |
| Test-Driven Script Validation | Reliably reproduced the silent failure before and validated the fix after. |
| Active Directory PowerShell Module | Cross-referenced provisioning records against HR data. |
| Centralized Script Logging | Implemented persistent logging so future failures surface immediately. |

---

## INC0452410 — Screen lock timeout setting flip-flopping on co-managed laptops

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Hybrid domain-joined and Intune-enrolled laptop fleet (co-management) | P3 — Medium | Endpoint Management > GPO / Intune Policy Conflict | 5h 05m | Resolved — Closed |

### Incident Summary

A screen lock timeout setting kept flip-flopping on a subset of laptops, sometimes enforcing the Group Policy value and sometimes the Intune value, with no clear pattern to which won on a given day.

### Root Cause

The affected laptops were co-managed without a clearly defined workload authority for device configuration settings. Both GPO and Intune were actively trying to enforce their own version of the setting on every refresh cycle, and because they ran on different sync schedules, whichever processed most recently temporarily won.

### Resolution

1. **Confirmed the co-management state.** Verified the affected devices were genuinely domain-joined and Intune-enrolled simultaneously rather than assuming a single management channel was misbehaving.
2. **Reviewed the co-management workload configuration.** Checked the tenant's co-management settings and found the Device Configuration workload had never been explicitly assigned to either channel.
3. **Traced the actual conflict in real time.** Used gpresult /h alongside the Intune device configuration profile view on one affected laptop to watch the setting change value across a full refresh cycle from each tool.
4. **Made an explicit authority decision.** Set the co-management workload slider to give Intune sole authority over device configuration settings, consistent with the organization's long-term platform direction.
5. **Removed the competing GPO setting.** Removed the conflicting screen lock timeout setting from the Group Policy object so it could no longer attempt to apply at all.
6. **Validated a stable, single outcome.** Monitored the affected laptops across multiple full refresh cycles from both tools and confirmed the setting now held a single, consistent value.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Microsoft Intune Admin Center (Co-Management Workloads) | Identified the undefined workload authority and assigned device configuration ownership. |
| gpresult / RSoP Reporting | Traced the Group Policy side of the conflicting setting in real time. |
| Intune Device Configuration Profiles | Reviewed the competing policy assignment pushing the conflicting value. |
| Group Policy Management Console (GPMC) | Removed the competing setting once workload authority was assigned. |
| Device Monitoring Across Refresh Cycles | Validated a single, stable policy outcome post-remediation. |

---

## INC0452955 — Autopilot enrollment failing on roughly one in seven devices during mass rollout

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Mass new-device rollout (Windows Autopilot, ~140 devices in batch) | P2 — High | Endpoint Management > Autopilot / Imaging Pipeline | 6h 30m | Resolved — Closed |

### Incident Summary

A large batch of new laptops being provisioned through Autopilot mostly completed without issue, but a consistent slice of devices failed at the exact same step every time, leaving a growing pile of half-provisioned machines.

### Root Cause

Two distinct causes behind the same failure rate: a subset of devices had been powered on and begun enrollment before their hardware hash finished propagating through Autopilot registration, and a separate subset hit a network timeout during enrollment because a newly imaged site's firewall rule had not yet been updated to allow a required Autopilot/Intune endpoint.

### Resolution

1. **Grouped the failures instead of treating them individually.** Pulled the Autopilot deployment failure logs for the batch and sorted failures by specific error code, revealing two distinct failure clusters instead of one problem.
2. **Diagnosed the larger cluster.** Cross-referenced failing devices' serial numbers against Autopilot registration timestamps and found they had all been powered on within a narrow window right after their hash was uploaded, before backend sync completed.
3. **Diagnosed the smaller cluster separately.** Reviewed Enrollment Status Page failure details on the remaining devices and found a consistent network timeout reaching a specific Autopilot service endpoint.
4. **Fixed the network-side cluster.** Confirmed the required endpoint was missing from the site's firewall allow-list, added it, and re-attempted enrollment, which then completed successfully.
5. **Resolved the sync-timing cluster with a process change.** Held the affected devices for a short buffer period to allow backend registration sync to complete, then re-attempted enrollment, which succeeded without further intervention.
6. **Adjusted the rollout process to prevent recurrence.** Built a minimum wait-time buffer between hash upload and device shipment into the standard rollout procedure, and added the required endpoints to the site network checklist.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Windows Autopilot Deployment Logs | Grouped failures by error code to reveal two distinct failure clusters. |
| Intune Admin Center (Autopilot Devices) | Cross-referenced device serial numbers against hash registration timestamps. |
| Enrollment Status Page (ESP) Diagnostics | Identified the specific network timeout affecting the second failure cluster. |
| Firewall Management Console | Added the missing required Autopilot/Intune endpoint to the site allow-list. |
| Mass Deployment Process Documentation | Implemented a hash-to-shipment buffer and endpoint checklist to prevent recurrence. |

---

## INC0453501 — BitLocker recovery key missing from AD and Entra ID during active recovery request

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Encrypted endpoint fleet, hybrid AD / Entra ID key escrow | P2 — High | Endpoint Management > BitLocker Key Escrow | 7h 10m | Resolved — Closed |

### Incident Summary

A user's laptop triggered BitLocker recovery mode after a firmware update, and the recovery key that should have been escrowed centrally was not found under any known identifier for the device.

### Root Cause

The device had encryption enabled locally before the policy responsible for escrowing the recovery key to AD/Entra ID had actually been applied and processed, so the key generated at encryption time was never captured centrally.

### Resolution

1. **Searched every plausible escrow location first.** Checked both on-premises Active Directory and Entra ID for the recovery key using every known identifier for the device, since escrow location can vary by enrollment method.
2. **Confirmed the key was genuinely missing.** Ran a targeted query against both directories' BitLocker recovery information objects and confirmed no key existed under any identifier.
3. **Resolved the immediate user need.** Worked with the user through Microsoft's manual BitLocker recovery options and organizational identity verification to regain access without the escrowed key.
4. **Investigated why escrow had failed.** Reviewed the device's enrollment and encryption timeline and confirmed BitLocker had been enabled before the escrow-enforcing policy was applied, a sequencing gap rather than a misconfiguration.
5. **Audited the fleet for the same exposure.** Queried the device fleet for devices showing BitLocker enabled locally but no corresponding key in AD or Entra ID, identifying every other device carrying the same risk.
6. **Closed the gap at the policy level.** Corrected the deployment sequencing so encryption can only complete after the escrow policy is confirmed applied, and forced a key rotation on every affected device to generate a correctly escrowed key.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Active Directory & Entra ID (BitLocker Recovery Objects) | Searched for and confirmed the absence of the recovery key under every known identifier. |
| Microsoft BitLocker Manual Recovery Process | Resolved the immediate access need through identity-verified recovery. |
| Intune / Group Policy Deployment Timeline Review | Identified the sequencing gap between encryption and escrow policy application. |
| PowerShell (BitLocker & Graph API queries) | Audited the full device fleet for the same unescrowed risk. |
| BitLocker Key Rotation | Forced regeneration and correct escrow of keys for every affected device. |

---

## INC0454028 — Monthly security patch causes line-of-business application to crash on launch

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Critical line-of-business application fleet, ~90 affected machines | P1 — Critical | Endpoint Management > Patch Management | 4h 45m | Resolved — Closed |

### Incident Summary

The morning after a routine monthly security patch deployment, a critical line-of-business application began crashing on launch across a wide swath of machines, bringing several departments to a standstill.

### Root Cause

The patch updated a shared system component the application depended on for a specific integration feature, introducing a compatibility regression the vendor had not yet identified. The patch had gone to the full fleet without first passing through a pilot ring of machines running the actual application.

### Resolution

1. **Contained the business impact first.** Identified the specific patch deployed the prior night and uninstalled it on affected machines to restore functionality immediately.
2. **Confirmed the correlation definitively.** Cross-referenced the patch deployment timestamp against application crash timestamps across multiple machines, confirming failures began precisely after the patch installed.
3. **Reproduced the failure in a controlled environment.** Installed the same patch on an isolated test machine running the application and reliably reproduced the crash, confirming the patch as the direct cause.
4. **Escalated with concrete evidence.** Reported the compatibility regression to the application vendor and Microsoft with the reproducible test case, crash logs, and exact patch KB identifier.
5. **Held the patch fleet-wide pending a fix.** Placed the patch on hold across the deployment management tool for any remaining unpatched machines, preventing further exposure while a vendor fix was pending.
6. **Implemented a structural safeguard.** Established a mandatory pilot-ring deployment stage that any future patch must pass through and be validated against before fleet-wide deployment.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Endpoint Management Console (Intune/SCCM Patch History) | Identified the specific patch and correlated its timing against application failures. |
| Windows Update / Patch Uninstall Tools | Removed the offending patch from affected machines to restore functionality. |
| Isolated Test Environment | Reliably reproduced the compatibility regression outside of production. |
| Vendor & Microsoft Support Escalation | Reported the reproducible issue with concrete evidence to accelerate a fix. |
| Pilot Ring Deployment Rings (Intune/SCCM) | Implemented as a mandatory validation stage before future fleet-wide patch deployment. |
