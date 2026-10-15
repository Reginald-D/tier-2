# Virtualization & Cloud Infrastructure

*A record of resolved Tier 2 incidents spanning VM host performance, backup reliability, storage, Azure/AWS governance, and Kubernetes.*

**Prepared by:** Ruler | Tier 2 Support / Cloud & Virtualization
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0462777 | Multiple unrelated VMs on same host reporting vague performance issues | P2 — High | Virtualization & Cloud > VM Host Resource Contention | 2h 30m |
| INC0463330 | Critical VM backup found failing for over a week during compliance audit | P1 — Critical | Virtualization & Cloud > VM Backup / Snapshot Reliability | 3h 15m |
| INC0463884 | Multiple systems on shared SAN reporting elevated storage latency | P2 — High | Virtualization & Cloud > Storage Array Performance | 4h 05m |
| INC0464410 | Auto-scaling misconfiguration causes outage and unexpected cloud cost spike | P1 — Critical | Virtualization & Cloud > Azure Resource Governance | 3h 50m |
| INC0464955 | Internally hosted application pods stuck in CrashLoopBackOff after deployment | P2 — High | Virtualization & Cloud > Kubernetes / Container Orchestration | 2h 05m |

---

## INC0462777 — Multiple unrelated VMs on same host reporting vague performance issues

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Multi-VM virtualization host, hypervisor cluster | P2 — High | Virtualization & Cloud > VM Host Resource Contention | 2h 30m | Resolved — Closed |

### Incident Summary

Over the course of a week, complaints trickled in from several unrelated teams about sluggish applications and long-running jobs, with no obvious connection between them until someone noticed the affected VMs all lived on the same physical host.

### Root Cause

The host had been steadily overprovisioned over several months as new VMs were added without a corresponding capacity review. CPU Ready Time had climbed well past acceptable thresholds across nearly every guest on the host.

### Resolution

1. **Connected the seemingly unrelated complaints.** Cross-referenced the affected VMs against the virtualization management console's inventory and confirmed every reported issue traced back to guests on the same physical host.
2. **Pulled host-level resource metrics.** Reviewed the host's CPU Ready Time, memory ballooning, and storage I/O latency and found CPU Ready Time elevated across nearly every VM on the host.
3. **Ruled out a single runaway VM.** Checked per-VM resource consumption individually and confirmed no single guest was monopolizing resources; the contention came from cumulative VM count relative to host capacity.
4. **Reviewed the host's provisioning history.** Checked when each VM had been deployed and confirmed a steady, incremental increase in VM count over months with no corresponding capacity review.
5. **Rebalanced the immediate load.** Used live migration to move a subset of lower-priority VMs to hosts in the cluster with confirmed available headroom, with no service interruption.
6. **Fixed the underlying capacity planning gap.** Established a standing resource utilization review as part of the VM provisioning process, requiring a host capacity check before any new VM deployment.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Virtualization Management Console (vCenter/Hyper-V Manager) | Correlated affected VMs to a shared physical host and reviewed resource metrics. |
| CPU Ready Time / Resource Contention Monitoring | Identified the specific contention metric explaining the performance complaints. |
| Live Migration (vMotion/equivalent) | Rebalanced VM load across the cluster without service interruption. |
| Host Provisioning History / Change Records | Traced the gradual, unchecked VM count growth. |
| Capacity Planning Review Process | Implemented as a standing prerequisite before future VM deployment. |

---

## INC0463330 — Critical VM backup found failing for over a week during compliance audit

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Critical production VM, backup platform | P1 — Critical | Virtualization & Cloud > VM Backup / Snapshot Reliability | 3h 15m | Resolved — Closed |

### Incident Summary

A routine audit of backup job status, prompted by an unrelated compliance review, revealed that a critical production VM's nightly backup had been failing for over a week with no one having noticed the status change.

### Root Cause

The VM's datastore had run low on free space, and each nightly backup attempt failed partway through. A previous failed attempt's snapshot had never been cleaned up, consuming additional datastore space each night and compounding the problem.

### Resolution

1. **Confirmed the actual scope of the failure first.** Reviewed the backup platform's job history and confirmed the exact date range of consecutive failures to establish the size of the recovery point gap.
2. **Diagnosed the specific failure reason.** Reviewed the detailed error logs and found a consistent "insufficient datastore space" error on every failed attempt.
3. **Checked the datastore directly.** Confirmed critically low free space and found an orphaned snapshot from an earlier failed attempt still consuming space.
4. **Cleared the orphaned snapshot safely.** Consolidated and removed the orphaned snapshot using the proper workflow, freeing space and removing the compounding factor.
5. **Addressed the underlying capacity constraint.** Coordinated with infrastructure to expand the datastore's available capacity for both the backup operation and normal VM growth.
6. **Verified a successful backup and closed the alerting gap.** Manually triggered the backup, confirmed end-to-end success, and configured active failure alerting so a future consecutive failure surfaces immediately.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Backup Platform Job History & Detailed Error Logs | Established the exact failure window and diagnosed the datastore space error. |
| Virtualization Management Console (Datastore Browser) | Confirmed critical capacity and identified the orphaned snapshot. |
| VM Snapshot Management (Consolidate/Delete) | Safely cleared the orphaned snapshot using the proper workflow. |
| Datastore Capacity Management | Coordinated expansion of available storage. |
| Backup Job Failure Alerting | Implemented so consecutive failures surface immediately. |

---

## INC0463884 — Multiple systems on shared SAN reporting elevated storage latency

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Shared SAN serving a database server, file server, and several VMs | P2 — High | Virtualization & Cloud > Storage Array Performance | 4h 05m | Resolved — Closed |

### Incident Summary

Several unrelated systems sharing the same storage array all started reporting slow disk response times within the same general timeframe, initially treated as separate performance investigations.

### Root Cause

A newly deployed VM running an unoptimized batch data-processing job was consuming a disproportionate share of the array's available IOPS, a noisy-neighbor condition starving every other consumer on the shared array.

### Resolution

1. **Recognized the shared dependency immediately.** Noted that every affected system shared the same underlying storage array and treated the investigation as a single storage-layer problem.
2. **Reviewed array-level performance metrics.** Checked the array's performance dashboard for latency, IOPS, and queue depth and confirmed genuine elevated latency across the board.
3. **Identified the disproportionate consumer.** Broke down IOPS consumption by workload and found one VM's workload consuming a share far exceeding its typical usage pattern.
4. **Confirmed the workload and its origin.** Traced the workload to a recently deployed batch job and reviewed its I/O pattern, confirming an unoptimized, high-frequency small-write pattern.
5. **Applied IOPS limits as an immediate mitigation.** Configured a storage-level IOPS limit on the offending workload's LUN, immediately restoring performance headroom to every other consumer.
6. **Worked with the workload's owner on a lasting fix.** Coordinated optimization of the batch job's I/O pattern and evaluated whether the array's capacity planning needed to account for this workload class going forward.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Storage Array Management Console | Reviewed array-level metrics and configured the IOPS limit. |
| Per-LUN / Per-Workload IOPS Breakdown | Identified the specific disproportionate consumer among shared workloads. |
| Storage QoS / IOPS Limiting Policy | Applied an immediate mitigation capping the noisy workload. |
| Application-Level I/O Pattern Review | Diagnosed the unoptimized batch job behavior. |
| Storage Capacity & Governance Policy | Implemented to require IOPS consideration for future workloads. |

---

## INC0464410 — Auto-scaling misconfiguration causes outage and unexpected cloud cost spike

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Cloud-hosted production service (Azure), auto-scaling resource group | P1 — Critical | Virtualization & Cloud > Azure Resource Governance | 3h 50m | Resolved — Closed |

### Incident Summary

A cloud-hosted service went down during a traffic spike, and days later an unrelated finance review flagged an unusually high cloud bill for the same period — which turned out to be the same underlying incident.

### Root Cause

The auto-scaling configuration had a trigger threshold set low enough that normal traffic fluctuation caused repeated scale-out events with no maximum instance count configured, and the resulting runaway growth consumed the account's resource quota, which is what caused the outage.

### Resolution

1. **Investigated the outage and cost anomaly as one incident.** Recognized the timing overlap between the outage and the flagged billing spike and pulled both the scaling event history and the billing breakdown together.
2. **Reviewed the auto-scaling event log.** Found a rapid, repeated series of scale-out events during the traffic spike, far exceeding what the actual traffic increase would require.
3. **Identified the specific misconfiguration.** Found the scaling policy's trigger threshold was too low and no maximum instance count was configured to cap runaway growth.
4. **Confirmed the quota collision as the outage's direct cause.** Cross-referenced the point where new instance creation began failing against the account's resource quota and confirmed runaway scaling had consumed it entirely.
5. **Corrected the scaling policy properly.** Reconfigured the trigger to use a more stable metric with an appropriate threshold and set a sensible maximum instance count as a hard ceiling.
6. **Closed the cost governance gap alongside the fix.** Set up a billing anomaly alert on the resource group and requested a review of the account's resource quotas.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Cloud Auto-Scaling Configuration (Azure) | Identified and corrected the misconfigured scaling trigger and missing instance cap. |
| Cloud Platform Scaling Event History | Confirmed the rapid, excessive scale-out pattern during the spike. |
| Cloud Billing / Cost Management Console | Correlated the cost anomaly against the scaling event timeline. |
| Resource Quota Management | Confirmed the quota collision as the direct mechanical cause of the outage. |
| Billing Anomaly Alerting | Implemented to catch future runaway resource creation via cost monitoring. |

---

## INC0464955 — Internally hosted application pods stuck in CrashLoopBackOff after deployment

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internally hosted Kubernetes cluster | P2 — High | Virtualization & Cloud > Kubernetes / Container Orchestration | 2h 05m | Resolved — Closed |

### Incident Summary

Shortly after a routine deployment, an internally hosted application's pods entered a crash loop, starting and failing repeatedly without ever serving a request, despite the deployment pipeline reporting success.

### Root Cause

The new container image referenced an environment variable that a recently updated application configuration expected, but the corresponding Kubernetes ConfigMap had not been updated to include it, so the application failed its own startup validation and exited immediately.

### Resolution

1. **Went straight to the pod's own diagnostic output.** Ran kubectl describe pod against a crash-looping pod, showing CrashLoopBackOff status and confirming the container was failing immediately on startup.
2. **Pulled the actual application logs from the failing container.** Ran kubectl logs with the --previous flag to retrieve output from the most recent failed attempt, finding an explicit error referencing a missing required environment variable.
3. **Compared the deployment's configuration against what the application needed.** Reviewed the application's startup validation code alongside the current ConfigMap and confirmed the required variable had never been added.
4. **Corrected the ConfigMap.** Added the missing environment variable with its correct value, matching exactly what the application's startup validation expected.
5. **Rolled out the corrected configuration.** Applied the updated ConfigMap and triggered a rollout restart of the deployment so every pod picked up the correction immediately.
6. **Verified stability before considering it resolved.** Monitored the pods through several minutes of stable running status and confirmed the application was successfully serving requests before closing the ticket.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| kubectl describe pod | Confirmed the CrashLoopBackOff status and exit behavior. |
| kubectl logs --previous | Retrieved the application's actual error output revealing the missing environment variable. |
| Kubernetes ConfigMap & Deployment Manifests | Compared current configuration against requirements and applied the correction. |
| kubectl rollout restart | Applied the corrected configuration to all pods immediately. |
| Post-Deployment Health Verification | Flagged as a pipeline improvement to catch pod-level failures before they reach users. |
