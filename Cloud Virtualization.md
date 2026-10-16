<div align="center">

# IT Helpdesk Portfolio
### Virtualization & Cloud Infrastructure — Ticket Documentation

![Category](https://img.shields.io/badge/Category-Virtualization%20%26%20Cloud%20Infrastructure-0A66C2?style=for-the-badge)
![Tickets](https://img.shields.io/badge/Tickets-5-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-ServiceNow%20Style-informational?style=for-the-badge)
![Environment](https://img.shields.io/badge/Environment-VMware%20%7C%20Azure%20%7C%20Kubernetes-blueviolet?style=for-the-badge)

</div>

---

## About This Portfolio

This collection showcases five representative incident tickets spanning the most common **virtualization & cloud infrastructure** issues an enterprise Tier 2 support team encounters.

Each ticket follows a standard **ServiceNow-style incident format**:

> `Reported Symptom` → `Diagnostic Steps` → `Root Cause` → `Resolution` → `User Communication`

> [!NOTE]
> These tickets are **representative scenarios** documenting the type of Tier 2 incidents handled in enterprise support roles. Ticket numbers and environment details are presented in standard ITSM format for professional documentation; company names and other client-identifying details have been generalized or removed to preserve confidentiality.

---

## Ticket Index

| # | Ticket ID | Title | Priority | Time to Resolve |
|---|-----------|-------|:--------:|:----------------:|
| 1 | [`INC0462777`](#inc0462777--multiple-unrelated-vms-on-same-host-reporting-vague-performance-issues) | Multiple unrelated VMs on same host reporting vague performance issues | P2 | 2h 30m |
| 2 | [`INC0463330`](#inc0463330--critical-vm-backup-found-failing-for-over-a-week-during-compliance-audit) | Critical VM backup found failing for over a week during compliance audit | P1 | 3h 15m |
| 3 | [`INC0463884`](#inc0463884--multiple-systems-on-shared-san-reporting-elevated-storage-latency) | Multiple systems on shared SAN reporting elevated storage latency | P2 | 4h 05m |
| 4 | [`INC0464410`](#inc0464410--auto-scaling-misconfiguration-causes-outage-and-unexpected-cloud-cost-spike) | Auto-scaling misconfiguration causes outage and unexpected cloud cost spike | P1 | 3h 50m |
| 5 | [`INC0464955`](#inc0464955--internally-hosted-application-pods-stuck-in-crashloopbackoff-after-deployment) | Internally hosted application pods stuck in CrashLoopBackOff after deployment | P2 | 2h 05m |

---

<br>

## `INC0462777` — Multiple unrelated VMs on same host reporting vague performance issues

<table>
<tr><td><b>Environment</b></td><td>Multi-VM virtualization host, hypervisor cluster</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Virtualization & Cloud > VM Host Resource Contention</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 30m</td></tr>
</table>

**Reported Symptom**
Over the course of a week, complaints trickled in from several unrelated teams about sluggish applications and long-running jobs, with no obvious connection between them until someone noticed the affected VMs all lived on the same physical host.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Connected the seemingly unrelated complaints. Cross-referenced the affected VMs against the virtualization management console's inventory and confirmed every reported issue traced back to guests on the same physical host.
2. Pulled host-level resource metrics. Reviewed the host's CPU Ready Time, memory ballooning, and storage I/O latency and found CPU Ready Time elevated across nearly every VM on the host.
3. Ruled out a single runaway VM. Checked per-VM resource consumption individually and confirmed no single guest was monopolizing resources; the contention came from cumulative VM count relative to host capacity.
4. Reviewed the host's provisioning history. Checked when each VM had been deployed and confirmed a steady, incremental increase in VM count over months with no corresponding capacity review.
5. Rebalanced the immediate load. Used live migration to move a subset of lower-priority VMs to hosts in the cluster with confirmed available headroom, with no service interruption.
6. Fixed the underlying capacity planning gap. Established a standing resource utilization review as part of the VM provisioning process, requiring a host capacity check before any new VM deployment.

</details>

**Root Cause**
The host had been steadily overprovisioned over several months as new VMs were added without a corresponding capacity review. CPU Ready Time had climbed well past acceptable thresholds across nearly every guest on the host.

**Resolution**
Used live migration to move a subset of lower-priority VMs to hosts in the cluster with confirmed available headroom, with no service interruption. Established a standing resource utilization review as part of the VM provisioning process, requiring a host capacity check before any new VM deployment.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0463330` — Critical VM backup found failing for over a week during compliance audit

<table>
<tr><td><b>Environment</b></td><td>Critical production VM, backup platform</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Virtualization & Cloud > VM Backup / Snapshot Reliability</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 15m</td></tr>
</table>

**Reported Symptom**
A routine audit of backup job status, prompted by an unrelated compliance review, revealed that a critical production VM's nightly backup had been failing for over a week with no one having noticed the status change.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Confirmed the actual scope of the failure first. Reviewed the backup platform's job history and confirmed the exact date range of consecutive failures to establish the size of the recovery point gap.
2. Diagnosed the specific failure reason. Reviewed the detailed error logs and found a consistent "insufficient datastore space" error on every failed attempt.
3. Checked the datastore directly. Confirmed critically low free space and found an orphaned snapshot from an earlier failed attempt still consuming space.
4. Cleared the orphaned snapshot safely. Consolidated and removed the orphaned snapshot using the proper workflow, freeing space and removing the compounding factor.
5. Addressed the underlying capacity constraint. Coordinated with infrastructure to expand the datastore's available capacity for both the backup operation and normal VM growth.
6. Verified a successful backup and closed the alerting gap. Manually triggered the backup, confirmed end-to-end success, and configured active failure alerting so a future consecutive failure surfaces immediately.

</details>

**Root Cause**
The VM's datastore had run low on free space, and each nightly backup attempt failed partway through. A previous failed attempt's snapshot had never been cleaned up, consuming additional datastore space each night and compounding the problem.

**Resolution**
Coordinated with infrastructure to expand the datastore's available capacity for both the backup operation and normal VM growth. Manually triggered the backup, confirmed end-to-end success, and configured active failure alerting so a future consecutive failure surfaces immediately.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0463884` — Multiple systems on shared SAN reporting elevated storage latency

<table>
<tr><td><b>Environment</b></td><td>Shared SAN serving a database server, file server, and several VMs</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Virtualization & Cloud > Storage Array Performance</td></tr>
<tr><td><b>Time to Resolve</b></td><td>4h 05m</td></tr>
</table>

**Reported Symptom**
Several unrelated systems sharing the same storage array all started reporting slow disk response times within the same general timeframe, initially treated as separate performance investigations.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Recognized the shared dependency immediately. Noted that every affected system shared the same underlying storage array and treated the investigation as a single storage-layer problem.
2. Reviewed array-level performance metrics. Checked the array's performance dashboard for latency, IOPS, and queue depth and confirmed genuine elevated latency across the board.
3. Identified the disproportionate consumer. Broke down IOPS consumption by workload and found one VM's workload consuming a share far exceeding its typical usage pattern.
4. Confirmed the workload and its origin. Traced the workload to a recently deployed batch job and reviewed its I/O pattern, confirming an unoptimized, high-frequency small-write pattern.
5. Applied IOPS limits as an immediate mitigation. Configured a storage-level IOPS limit on the offending workload's LUN, immediately restoring performance headroom to every other consumer.
6. Worked with the workload's owner on a lasting fix. Coordinated optimization of the batch job's I/O pattern and evaluated whether the array's capacity planning needed to account for this workload class going forward.

</details>

**Root Cause**
A newly deployed VM running an unoptimized batch data-processing job was consuming a disproportionate share of the array's available IOPS, a noisy-neighbor condition starving every other consumer on the shared array.

**Resolution**
Configured a storage-level IOPS limit on the offending workload's LUN, immediately restoring performance headroom to every other consumer. Coordinated optimization of the batch job's I/O pattern and evaluated whether the array's capacity planning needed to account for this workload class going forward.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0464410` — Auto-scaling misconfiguration causes outage and unexpected cloud cost spike

<table>
<tr><td><b>Environment</b></td><td>Cloud-hosted production service (Azure), auto-scaling resource group</td></tr>
<tr><td><b>Priority</b></td><td>P1 — Critical</td></tr>
<tr><td><b>Category</b></td><td>Virtualization & Cloud > Azure Resource Governance</td></tr>
<tr><td><b>Time to Resolve</b></td><td>3h 50m</td></tr>
</table>

**Reported Symptom**
A cloud-hosted service went down during a traffic spike, and days later an unrelated finance review flagged an unusually high cloud bill for the same period — which turned out to be the same underlying incident.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Investigated the outage and cost anomaly as one incident. Recognized the timing overlap between the outage and the flagged billing spike and pulled both the scaling event history and the billing breakdown together.
2. Reviewed the auto-scaling event log. Found a rapid, repeated series of scale-out events during the traffic spike, far exceeding what the actual traffic increase would require.
3. Identified the specific misconfiguration. Found the scaling policy's trigger threshold was too low and no maximum instance count was configured to cap runaway growth.
4. Confirmed the quota collision as the outage's direct cause. Cross-referenced the point where new instance creation began failing against the account's resource quota and confirmed runaway scaling had consumed it entirely.
5. Corrected the scaling policy properly. Reconfigured the trigger to use a more stable metric with an appropriate threshold and set a sensible maximum instance count as a hard ceiling.
6. Closed the cost governance gap alongside the fix. Set up a billing anomaly alert on the resource group and requested a review of the account's resource quotas.

</details>

**Root Cause**
The auto-scaling configuration had a trigger threshold set low enough that normal traffic fluctuation caused repeated scale-out events with no maximum instance count configured, and the resulting runaway growth consumed the account's resource quota, which is what caused the outage.

**Resolution**
Reconfigured the trigger to use a more stable metric with an appropriate threshold and set a sensible maximum instance count as a hard ceiling. Set up a billing anomaly alert on the resource group and requested a review of the account's resource quotas.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket. Logged the preventive follow-up action identified during this incident for the team's process documentation.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## `INC0464955` — Internally hosted application pods stuck in CrashLoopBackOff after deployment

<table>
<tr><td><b>Environment</b></td><td>Internally hosted Kubernetes cluster</td></tr>
<tr><td><b>Priority</b></td><td>P2 — High</td></tr>
<tr><td><b>Category</b></td><td>Virtualization & Cloud > Kubernetes / Container Orchestration</td></tr>
<tr><td><b>Time to Resolve</b></td><td>2h 05m</td></tr>
</table>

**Reported Symptom**
Shortly after a routine deployment, an internally hosted application's pods entered a crash loop, starting and failing repeatedly without ever serving a request, despite the deployment pipeline reporting success.

<details>
<summary><b>Diagnostic Steps</b></summary>

1. Went straight to the pod's own diagnostic output. Ran kubectl describe pod against a crash-looping pod, showing CrashLoopBackOff status and confirming the container was failing immediately on startup.
2. Pulled the actual application logs from the failing container. Ran kubectl logs with the --previous flag to retrieve output from the most recent failed attempt, finding an explicit error referencing a missing required environment variable.
3. Compared the deployment's configuration against what the application needed. Reviewed the application's startup validation code alongside the current ConfigMap and confirmed the required variable had never been added.
4. Corrected the ConfigMap. Added the missing environment variable with its correct value, matching exactly what the application's startup validation expected.
5. Rolled out the corrected configuration. Applied the updated ConfigMap and triggered a rollout restart of the deployment so every pod picked up the correction immediately.
6. Verified stability before considering it resolved. Monitored the pods through several minutes of stable running status and confirmed the application was successfully serving requests before closing the ticket.

</details>

**Root Cause**
The new container image referenced an environment variable that a recently updated application configuration expected, but the corresponding Kubernetes ConfigMap had not been updated to include it, so the application failed its own startup validation and exited immediately.

**Resolution**
Applied the updated ConfigMap and triggered a rollout restart of the deployment so every pod picked up the correction immediately. Monitored the pods through several minutes of stable running status and confirmed the application was successfully serving requests before closing the ticket.

> **User Communication (Ticket Notes)**
> *Confirmed with the affected user/team that service was fully restored before closing the ticket.*

<div align="right"><a href="#ticket-index">back to top</a></div>

---

## Portfolio Snapshot

| Metric | Value |
|---|---|
| **Total Tickets Documented** | 5 |
| **Critical (P1) Incidents** | 2 |
| **High Priority (P2) Incidents** | 3 |
| **Average Time to Resolve** | ~3h 9m |
| **Systems Touched** | Virtualization Management Console · CPU Ready Time · Live Migration · Host Provisioning History · Capacity Planning Review Process · Backup Platform Job History & Detailed Error Logs · VM Snapshot Management · Datastore Capacity Management |

## Skills Demonstrated

`VM Host Resource Management` · `Backup / Snapshot Reliability` · `Storage Array Performance` · `Azure Resource Governance` · `Kubernetes Troubleshooting` · `Cost Anomaly Investigation`

---

<div align="center">

*Built to demonstrate structured, repeatable troubleshooting methodology across virtualization & cloud infrastructure — real-world Tier 2 scenarios, documented to enterprise standard.*

</div>