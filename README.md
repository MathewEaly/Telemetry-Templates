# Azure Application Insights KQL Queries

This repository contains Kusto Query Language (KQL) queries for retrieving telemetry data from Azure Application Insights.

These queries can be used to troubleshoot application issues, investigate performance, analyze usage patterns, and explore telemetry collected by Application Insights.

---

## Prerequisites

Before using these queries, ensure you have:

- Access to an Azure subscription
- Reader (or higher) permissions to the target Application Insights resource
- Access to the Azure Portal
- Familiarity with basic Kusto Query Language (KQL) concepts (helpful, but not required)

---

## Running a Query

1. Sign in to the Azure Portal.
2. Navigate to your **Application Insights** resource.
3. Select **Logs** from the left navigation menu.
4. Open the desired `.kql` file from this repository.
5. Copy the contents of the query.
6. Paste the query into the Log Analytics query editor.
7. Modify any parameters or placeholders if necessary (such as timestamps, operation names, request IDs, or environment-specific values).
8. Click **Run** to execute the query.

---

## Customizing Queries

Many KQL queries are designed to be customized before execution.

Common modifications include:

- Changing the time range
- Filtering by cloud role
- Filtering by operation ID
- Searching for a specific user or session
- Filtering by request name or dependency
- Adjusting result limits
- Adding or removing projected columns

Review the query before running it to determine which values should be updated for your environment.

---

## Common Telemetry Tables

Application Insights stores different types of telemetry in separate tables. Some commonly used tables include:

| Table | Description |
|--------|-------------|
| `requests` | Incoming application requests |
| `dependencies` | Calls to external services, databases, or APIs |
| `exceptions` | Application exceptions and stack traces |
| `traces` | Application log messages |
| `customEvents` | User-defined events |
| `customMetrics` | User-defined metrics |
| `pageViews` | Browser page views (web applications) |
| `availabilityResults` | Availability test results |
| `browserTimings` | Client-side performance metrics |

---

## Time Range

Queries can use either:

- The time picker in the Azure Portal
- Explicit KQL filters such as:

```kusto
| where timestamp > ago(24h)
```

If both are used, the query filter further restricts the selected time range.

---

## Tips

- Start with a broader time range, then narrow your search.
- Apply filters early in the query to improve performance.
- Use `take` or `limit` while developing or testing queries.
- Save frequently used queries in your own Azure environment for quick access.
- Review projected columns to reduce unnecessary data.

---

## About Kusto Query Language (KQL)

Kusto Query Language (KQL) is the query language used by Azure Monitor, Log Analytics, and Application Insights to analyze telemetry data.

Microsoft provides comprehensive documentation here:

https://learn.microsoft.com/azure/data-explorer/kusto/query/

---

## Contributing

Contributions are welcome.

If you have additional queries or improvements:

1. Fork the repository.
2. Create a feature branch.
3. Add or update the appropriate `.kql` file.
4. Submit a pull request with a brief description of the changes.

---

## License

This repository is provided as-is for educational and operational use. 
