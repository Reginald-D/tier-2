# Database & Application-Level

*A record of resolved Tier 2 incidents spanning connection pooling, scheduled jobs, API integration, SQL performance, and application stability.*

**Prepared by:** Ruler | Tier 2 Support / Database & Application Systems
**Certifications:** SC-300 (Identity & Access Administrator) • CompTIA Security+

---

## About This Document

This register documents resolved Tier 2 incidents reflecting the type of issues handled in enterprise support roles. Ticket numbers, environment descriptions, and timestamps are presented in standard ITSM format for consistent, professional documentation; company names, domain names, and other client-identifying details have been generalized or removed to preserve confidentiality. Priority levels and resolution times reflect realistic incident handling consistent with ITIL-aligned SLA practices.

## Incident Index

| INC # | Short Description | Priority | Category | TTR |
|---|---|---|---|---|
| INC0460055 | Application server throwing intermittent HTTP 500 errors under afternoon load | P2 — High | Database & Application > Connection Pooling | 4h 10m |
| INC0460602 | Nightly sales data import job silently stopped running for several days | P2 — High | Database & Application > Scheduled Job Reliability | 3h 35m |
| INC0461140 | Third-party billing platform integration silently stops syncing data | P2 — High | Database & Application > API Integration | 3h 55m |
| INC0461688 | Daily report load time degrades from under a second to nearly a minute | P3 — Medium | Database & Application > SQL Query / Index Performance | 2h 50m |
| INC0462210 | Internal service restarting unexpectedly several times per week | P3 — Medium | Database & Application > Application Stability | 5h 20m |

---

## INC0460055 — Application server throwing intermittent HTTP 500 errors under afternoon load

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internal web application server, backend SQL database | P2 — High | Database & Application > Connection Pooling | 4h 10m | Resolved — Closed |

### Incident Summary

An internal application ran fine each morning and began throwing intermittent HTTP 500 errors every afternoon once usage picked up, with the error rate climbing in step with traffic through the day.

### Root Cause

A connection leak in a report-generation code path: a database connection was opened at the start of the method but only explicitly closed in the success path, so a specific error condition caused an early return that skipped cleanup, and the connection pool exhausted under afternoon load.

### Resolution

1. **Confirmed the pattern before touching code.** Correlated 500 error timestamps against request volume and found the error rate climbing with traffic rather than occurring randomly, pointing to resource exhaustion.
2. **Checked the database connection pool directly.** Reviewed the application server's connection pool metrics and confirmed active connections climbing steadily through the day without returning to idle.
3. **Correlated pool growth against application logs.** Cross-referenced the timestamps of pool growth against request logs and found a consistent correlation with a specific report-generation endpoint.
4. **Reproduced the leak in a controlled test.** Ran the report-generation feature repeatedly against a known problematic data condition in a test environment and confirmed each execution left a connection permanently checked out.
5. **Found the exact code defect.** Reviewed the code path and found the connection was opened at the start of the method but only closed in the success path, skipped entirely on a specific error return.
6. **Fixed the leak and validated under real load.** Corrected the code to guarantee connection release in a finally block regardless of exit path, deployed the fix, and monitored the pool through a full afternoon of real traffic with zero recurrence.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Application Server Connection Pool Metrics | Identified the steadily climbing active connection count pointing to a leak. |
| Application Request/Error Logs | Correlated error timing against traffic volume and isolated the responsible endpoint. |
| Isolated Test Environment | Reliably reproduced the connection leak before touching production code. |
| Source Code Review / IDE Debugging | Located the exact code path where connection cleanup was being skipped. |
| Production Load Monitoring (Post-Deployment) | Validated the fix held stable through a full real-world peak traffic cycle. |

---

## INC0460602 — Nightly sales data import job silently stopped running for several days

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internal line-of-business application, scheduled reporting job | P2 — High | Database & Application > Scheduled Job Reliability | 3h 35m | Resolved — Closed |

### Incident Summary

A nightly scheduled job responsible for pulling the previous day's sales data into the reporting system had quietly stopped running for several days, discovered only when someone preparing a weekly report noticed missing figures.

### Root Cause

The job ran under a dedicated service account whose password had expired under the organization's standard rotation policy, which had not accounted for service accounts running unattended scheduled tasks, and no failure alerting existed on the job itself.

### Resolution

1. **Confirmed the job was actually failing.** Checked the reporting system for the missing date range to confirm data genuinely hadn't loaded before investigating the job itself.
2. **Went straight to the task scheduler history.** Reviewed the Windows Task Scheduler run history for the job and found several consecutive nights of failures with the same authentication error code.
3. **Diagnosed the authentication failure specifically.** Checked the service account's status in Active Directory and confirmed its password had expired under the standard rotation policy.
4. **Restored the job immediately.** Reset the service account's password, updated the stored credential in the Task Scheduler job, and manually triggered a run to confirm success.
5. **Backfilled the missing data.** Identified the exact affected date range and manually re-ran the data pull for each missing day, validating the reporting system reflected complete figures.
6. **Closed the systemic gap.** Configured the service account for a non-expiring password under the approved service account exception policy and added failure alerting to the job itself.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Windows Task Scheduler (Run History) | Identified the consecutive authentication failures and their error signature. |
| Active Directory Users and Computers (ADUC) | Diagnosed the expired service account password. |
| Line-of-Business Application / Reporting System | Confirmed the scope of missing data and validated the backfill. |
| Service Account Password Policy Exception | Applied a non-expiring configuration appropriate for unattended automation. |
| Task Scheduler Failure Notification / Alerting | Implemented so future failures surface immediately. |

---

## INC0461140 — Third-party billing platform integration silently stops syncing data

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internal CRM integrated with third-party billing SaaS platform | P2 — High | Database & Application > API Integration | 3h 55m | Resolved — Closed |

### Incident Summary

The integration syncing customer billing data between the internal CRM and a third-party billing platform stopped updating, discovered only when a customer's updated payment status failed to reflect during a support call.

### Root Cause

The billing platform had rotated its API authentication credentials and notified the organization by email, but the notification went to a distribution list no longer actively monitored after a recent team reorganization, leaving the integration's stored API key invalid.

### Resolution

1. **Verified the CRM side was healthy first.** Confirmed the CRM was functioning normally with whatever data it currently held, ruling out an application bug before investigating whether new data was arriving.
2. **Checked the integration's own error logs.** Reviewed the integration middleware logs and found a consistent authentication failure on every sync attempt going back several days.
3. **Confirmed the credential was the actual problem.** Manually tested the stored API key directly against the billing platform's API and received the same authentication failure, confirming the key itself was invalid.
4. **Traced why the credential had gone stale.** Contacted the vendor's support team and learned of the recent credential rotation, then found the original notification had gone to an inactive distribution list.
5. **Obtained and applied the new credential.** Retrieved the current valid API key from the vendor's admin portal, updated the integration's stored credential securely, and confirmed data began flowing again.
6. **Backfilled the gap and closed the communication loop.** Pulled the full range of records that should have synced during the outage window, reconciled them against the CRM, and updated the vendor notification contact to an actively monitored list.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Integration Middleware Logs | Identified the consistent authentication failure pointing directly at the credential. |
| API Request Testing Tool (Postman/equivalent) | Manually validated the stored API key against the vendor's API. |
| Vendor SaaS Admin Portal | Retrieved the current, valid API credential following rotation. |
| Secure Credential Storage (Integration Configuration) | Updated the integration's stored API key securely. |
| Vendor API (Historical Record Retrieval) | Backfilled the outage window's missing records. |

---

## INC0461688 — Daily report load time degrades from under a second to nearly a minute

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Internal application database | P3 — Medium | Database & Application > SQL Query / Index Performance | 2h 50m | Resolved — Closed |

### Incident Summary

A report inside an internal application that used to return in under a second had gradually slowed to the point of taking nearly a minute to load, with nothing about the report itself having changed in months.

### Root Cause

The report's query filtered and sorted on a table column with no index. The table was small enough early on that a full table scan was fast, but as the table grew substantially over time, the same full scan grew linearly slower.

### Resolution

1. **Confirmed it was a database issue, not an application one.** Timed the underlying query directly against the database, isolated from the application layer, and confirmed the slowness originated at the database level.
2. **Pulled the query's execution plan.** Ran the exact query through the execution plan analyzer and found a full table scan on the primary filtering column rather than an index seek.
3. **Confirmed the missing index was the actual gap.** Checked the table's existing index definitions and confirmed no index existed on the filtered and sorted column.
4. **Sized the fix against real data volume.** Checked current row count against the table's size when the report was originally built, confirming data growth fully explained the slowdown.
5. **Built and tested the index in a safe environment first.** Created the missing index in a non-production test environment against a copy of production data and confirmed execution time dropped dramatically.
6. **Deployed during a low-impact window and validated live.** Applied the index during an off-peak maintenance window, confirmed the live report returned in well under a second, and monitored over following days to confirm the improvement held.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Database Query Execution Plan Analyzer | Identified the full table scan as the specific structural cause. |
| SQL Server Management Studio / Database Admin Console | Reviewed existing index definitions and built the missing index. |
| Database Performance / Query Timing Tools | Isolated the slowness to the database layer and measured timing before/after. |
| Non-Production Test Environment (Data Volume Parity) | Validated the index's performance impact before touching production. |
| Production Monitoring (Post-Deployment) | Confirmed the improvement held steady over multiple days. |

---

## INC0462210 — Internal service restarting unexpectedly several times per week

| Environment | Priority | Category | Time to Resolve | Status |
|---|---|---|---|---|
| Long-running internal application service | P3 — Medium | Database & Application > Application Stability | 5h 20m | Resolved — Closed |

### Incident Summary

A long-running internal service had begun restarting unexpectedly several times a week, each restart causing a brief but disruptive interruption to the application it supported, with no obvious trigger in the application logs.

### Root Cause

A gradual memory leak in the service's request-handling code caused memory consumption to climb steadily until it hit the host's configured memory ceiling, at which point the operating system or container orchestrator terminated and restarted the process.

### Resolution

1. **Confirmed the pattern was memory-related before inspecting code.** Reviewed host-level memory utilization metrics over the days leading up to each restart and found a consistent, steady climb to the configured ceiling immediately before each event.
2. **Correlated restarts against the memory ceiling precisely.** Overlaid the exact restart timestamps against the memory utilization graph and confirmed every restart coincided with memory hitting its configured limit.
3. **Captured a memory profile from the running service.** Attached a memory profiling tool to a running instance and captured heap snapshots at intervals to identify which objects were accumulating rather than being released.
4. **Identified the specific leaking code path.** Traced the accumulating objects back to a caching structure in the request-handling code that was never evicting old entries, growing unbounded over the service's uptime.
5. **Corrected the caching logic.** Implemented a bounded cache with proper eviction and validated in a test environment that memory usage now plateaued rather than climbing indefinitely under sustained load.
6. **Deployed and monitored across a full uptime cycle.** Rolled out the fix and monitored memory utilization over the following two weeks, confirming it remained stable with zero unexpected restarts.

### Tools & Systems Used

| Tool / System | Role in This Incident |
|---|---|
| Host / Container Memory Utilization Monitoring | Correlated the restart timing precisely against memory consumption. |
| Memory Profiling Tool (Heap Snapshot Analysis) | Identified which objects were accumulating rather than being released. |
| Source Code Review / IDE Debugging | Traced the leak to an unbounded caching structure in the request-handling code. |
| Non-Production Load Testing | Validated the corrected caching logic plateaued memory usage under sustained load. |
| Production Monitoring (Extended Window) | Confirmed stability across a full two-week uptime cycle post-fix. |
