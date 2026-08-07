# Business Central Telemetry — KQL Query Library

A collection of production-tested Kusto Query Language (KQL) queries for diagnosing and monitoring Microsoft Dynamics 365 Business Central environments through Azure Application Insights.

These queries were developed and refined through real-world BC SaaS support and implementation work — troubleshooting ISV conflicts, performance degradation, job queue failures, deadlocks, and integration issues across multi-tenant environments. They are organized by diagnostic category and are designed to be dropped into Log Analytics with minimal modification.

---

## Who this is for

- BC functional consultants and support engineers diagnosing tenant issues
- Partners and ISVs monitoring extension behavior in SaaS environments
- Admins and IT teams triaging performance or error spikes
- Anyone building out a BC telemetry practice from scratch

---

## Prerequisites

- Access to an Azure subscription with an Application Insights resource connected to a BC environment
- Reader permissions (or higher) on the Application Insights resource
- Access to the Azure Portal or VS Code with the KQL extension
- Basic familiarity with KQL is helpful but not required — queries include inline comments explaining what each section does

---

## Query catalog

### Root-level queries

| File | What it answers |
|---|---|
| `429Error.kql` | Identifies HTTP 429 (throttling) errors — useful for pinpointing integration or API call volume issues |
| `ConfigPackage.kql` | Tracks configuration package import/export activity |
| `DeadLockBySource.kql` | Breaks down database deadlocks by source process, locking object, and SQL operation type — includes regex-based AL stack trace parsing to identify the extension or codeunit at the top and bottom of the call stack |
| `Email.kql` | Monitors email send activity and failures |
| `Error.kql` | General error query — starting point for unspecified error investigations |
| `FollowSessionID.kql` | Traces a full user session by session ID — useful for reconstructing what happened before an error |
| `JobQueue.KQL` | Comprehensive job queue monitoring covering all 9 lifecycle events (enqueued, started, finished, failed, retried, stopped, cancelled, rescheduled, run-once) — includes a joined query that correlates job queue failures with task scheduler errors to identify root cause |
| `LastLockTimeOUt.kql` | Surfaces the most recent lock timeout events |
| `OData_Finder.kql` | Identifies OData call activity — useful for diagnosing integration performance or unauthorized API access |
| `PageView.kql` | Tracks page load activity by user and page |
| `PermissionErrors.kql` | Surfaces permission-related errors by user and object |
| `Postings.kql` | Monitors posting activity (GL, sales, purchase) — useful for identifying volume patterns or posting failures |
| `RAWErrors.kql` | Raw error output — unfiltered, use for initial triage |
| `StackTraces.kql` | Extracts and parses AL stack traces from error events |
| `TopEvents.kql` | Summary of the highest-volume telemetry events in the environment — good starting point for any investigation |
| `UITimeWaiting.kql` | Measures time users spend waiting on UI operations — useful for identifying slow pages or high-latency scenarios |
| `WhoDidWhatWhen.kql` | Audit-style query tracing user actions by timestamp — useful for change investigations |

### Subfolders

| Folder | Contents |
|---|---|
| `Environment/` | Queries focused on environment-level telemetry — upgrade events, extension lifecycle, environment health signals |
| `Performance/` | Performance-focused queries covering long-running SQL, slow AL execution, and resource saturation patterns |
| `Reports/` | Queries targeting report execution — run times, failures, and usage patterns |
| `Slowness/` | Targeted slowness investigation queries — isolates UI wait time, query duration, and dependency call latency |
| `WebService/` | OData and SOAP web service monitoring — call volume, errors, and slow dependency calls from integrations |

---

## Running a query

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. Navigate to your **Application Insights** resource.
3. Select **Logs** from the left navigation menu.
4. Copy the contents of the relevant `.kql` file.
5. Paste into the Log Analytics query editor.
6. Adjust the time range and any environment-specific filters (tenant ID, company name, environment name).
7. Click **Run**.

Alternatively, open queries directly in **VS Code** using the [KQL extension](https://marketplace.visualstudio.com/items?itemName=josin.kusto-syntax-highlighting) and connect to your Log Analytics workspace.

---

## Customizing queries

Most queries include inline comments flagging the values most likely to need adjustment:

- Time range: `| where timestamp > ago(24h)` — change the duration as needed
- Tenant filter: `| where customDimensions.aadTenantId == '<your-tenant-id>'`
- Environment name: `| where customDimensions.environmentName == '<your-env>'`
- Company name: `| where customDimensions.companyName == '<your-company>'`
- Result limits: `| take 100` or `| limit 50` — reduce when testing, remove for full results

Apply filters as early as possible in the query pipeline to improve performance, especially on high-volume workspaces.

---

## Key telemetry tables

| Table | Description |
|---|---|
| `traces` | The primary table for BC telemetry — most AL and platform events land here via `customDimensions` |
| `requests` | Incoming requests to the BC service |
| `dependencies` | External calls — SQL, APIs, integrations |
| `exceptions` | Unhandled exceptions with stack traces |
| `customEvents` | App-defined events |
| `customMetrics` | App-defined metrics |
| `pageViews` | Browser-side page load data |
| `availabilityResults` | Availability test results |

Most BC-specific telemetry uses the `traces` table with event IDs stored in `customDimensions.eventId`. Filtering on `customDimensions has 'AL0000XXX'` before the `eventId` filter significantly improves query performance on large workspaces.

---

## A note on the job queue / task scheduler join

The `JobQueue.KQL` file includes a query that joins job queue failure events (`AL0000HE7`) with task scheduler error events (`LC0045`) on the scheduled task ID. This pattern is particularly useful when a job queue entry is failing silently — the job queue event shows the failure but not the underlying cause; the task scheduler event often contains the `failureReason` and whether the exception is retriable. Correlating the two gives a complete picture of what went wrong and whether intervention is needed.

---

## Tips

- Start with `TopEvents.kql` on any new environment to understand what telemetry is flowing and at what volume before diving into specific queries.
- Use `FollowSessionID.kql` when you have a user-reported error with a timestamp — pull the session ID from any nearby event and reconstruct the full session context.
- The `DeadLockBySource.kql` stack trace parsing extracts both the top and bottom of the AL call stack, which helps distinguish whether a deadlock is originating in a Microsoft base app object, an ISV extension, or a per-tenant customization.
- Save frequently used queries to your Log Analytics workspace as **Saved Queries** so they're available without returning to this repo each time.

---

## Related resources

- [Microsoft Dynamics 365 BC Telemetry Documentation](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/telemetry-overview)
- [BC Telemetry GitHub (Microsoft)](https://github.com/microsoft/BCTech/tree/master/samples/AppInsights) — the official Microsoft sample library this repo extends and builds on
- [KQL Reference](https://learn.microsoft.com/azure/data-explorer/kusto/query/)
- [Application Insights Overview](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview)

---

## Contributing

Contributions and query suggestions are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Add or update the relevant `.kql` file with inline comments explaining what the query does and when to use it.
4. Submit a pull request with a brief description of the scenario the query addresses.

---

## License

Provided as-is for operational and educational use. Queries may be adapted freely for use in your own BC environments.
