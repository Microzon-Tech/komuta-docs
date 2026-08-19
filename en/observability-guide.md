# Observability Guide

Komuta Observability shows, in a single workspace, **what happened** in a service, **where it slowed down**, and **what evidence a decision is based on**. The screens present only the data the platform has actually collected; if data hasn't arrived, instead of drawing an empty chart they clearly state why the source is unavailable.

This guide helps non-technical users interpret the screens correctly. For Kubernetes resource usage such as CPU and memory, use the [Monitoring and Log Management](https://www.komuta.io/docs/guides/monitoring-logs) guide; for runtime threats, use the [Runtime Security](https://www.komuta.io/docs/security/runtime-security-guide) guide.

---

## How Do You Access Observability?

1. Open a service in Komuta Console.
2. Go to the **Observability** section from the left menu.
3. Check that the correct **service** and **environment** are selected at the top.

All queries are scoped server-side to this service and environment. A user can only see data for the organizations and services they are authorized for.

---

## What Do the Screens Show?

### Overview

Overview summarizes the service's state over the recent window with three key signals:

- **Rate:** Number of observed requests
- **Errors:** Number and rate of failed requests
- **Duration:** p50, p95, and p99 latency values

This trio is briefly called **RED**. The values are produced from the trace spans that were collected. If the trace collection policy applies sampling, the request count is not the sum of all real traffic but **the count of the sampled traffic that was collected**. This is why the "Sampled" or "Qualified" label on the screen matters.

Overview also shows service dependencies. If evidence cannot yet be produced for "highlighted findings" or a "change timeline," the section clearly states this as "unavailable." This does not mean the service is healthy or unhealthy.

### Investigate

The **Investigate** section lists the investigation records Komuta generates based on evidence. An investigation can bring together:

- What happened?
- What was the impact on the user or service?
- What changes coincided with it in time?
- What is the strongest evidence and the supporting evidence?
- What evidence is missing?
- What is the recommended next step and verification window?

A candidate cause is not presented as a definitive root cause. If evidence is insufficient, the confidence level and the missing pieces remain visible.

### Trace

The Trace screen shows a request's journey through the service as spans. In the search section, you can narrow down by route, HTTP method, error status, and duration range.

When a trace is selected, the following are shown:

- The span waterfall and start order
- Each span's duration and its own execution time
- Error status and status message
- Safely exported attributes and events
- Quality warnings such as clock mismatch, missing spans, or truncated data
- Logs that can be correlated with the trace ID

Sampling can be applied to traces. Important signals like errors and slowness are kept at a higher priority, while normal traffic may be sampled per policy. For this reason, do not interpret trace count directly as "total user requests."

### Log

The Log screen searches service records in the central log store. The following filters can be used together:

- Time range
- Level: fatal, error, warning, info, debug, trace, or unknown
- Namespace, pod, and container
- Contains text / matches regex
- Excludes text or regex match
- Supported correlation fields: trace, request, or correlation ID

The histogram and the log table use the same filters. This way, the density shown in the chart and the records below it answer the same question. The "Load older records" action uses cursor-based pagination; exports include only the results the user can access.

> Log messages contain the content your application writes. Do not log passwords, access keys, personal data, or session tokens.

### Trace-Derived RED

This screen shows the RED data from Overview in more detail, broken down by endpoint:

- Traffic and errors in five-minute buckets
- Request and error rate per endpoint
- p50, p95, and p99 values for the worst bucket
- A "no sketch" warning for buckets where a latency summary cannot be produced

This screen **does not show CPU, memory, disk, node capacity, or Prometheus metrics**. These resources are monitored in your service's **Resources** and related cluster screens.

### Dependencies and L7

Komuta Flow Agent collects observed network flows belonging to the service. The screen can present the following information:

- Source and destination service/pod/namespace
- Protocol and destination port
- Allowed or dropped flow
- HTTP method, URL/route, status code, and observed latency
- DNS query and response code
- Traffic direction and node

Flow records do not carry byte or packet counts. For this reason, network bandwidth, packet rate, or transfer size cannot be calculated from the flow count on this screen. The "HTTP flow" count is also not the same measurement as the trace-based request count.

### Endpoints

The Endpoints screen ranks endpoints by request count, error rate, and worst latency bucket. You can jump from an endpoint to the Trace screen to examine the relevant samples.

If there is no data in a time bucket, Komuta does not automatically treat it as zero. Especially when a new rollup schema is deployed, the start of coverage may appear as a gap in the chart.

### Reliability

#### SLO and Error Budget

An SLO is a service's success commitment for a given period. For example, a target such as "99.9% of this endpoint's requests must succeed" can be defined.

Komuta:

- Creates SLOs at the service level or per endpoint
- Shows the measured success rate
- Calculates the remaining and consumed error budget
- Distinguishes cases where the budget is being consumed quickly or has been fully exhausted
- Shows the status as **Unknown** rather than assuming "healthy" when telemetry is missing

Creating or editing an SLO may require separate permissions.

#### Anomalies

The Anomalies screen compares current endpoint signals against a moving historical baseline. If the baseline hasn't filled sufficiently during the initial period, a "warming up" warning is shown; early results are evaluated with lower confidence.

An anomaly, on its own, is not evidence of a failure or an attack. The magnitude of the deviation, the time range, and the related trace/log evidence should be examined together.

#### Release Impact

Release Impact compares the measurement windows before and after a deployment. Error rate, latency, and endpoint changes can be shown on the same timeline.

- Overlapping deployments remain visible.
- No score is given if telemetry is missing.
- If correlation data is absent, it is not claimed that the deployment caused the change.
- "High" confidence is not assumed if the confidence value is unknown.

### Optimize

#### Cost and Telemetry Efficiency

Service cost, in the organization context, is distributed from the measured counters for the current billing period. The "finalized time" information shows up to which point billing data is complete.

Telemetry efficiency analysis can use metrics such as sampled day count, requests, unique paths, and storage rows. AI-generated titles and recommendations are **suggestions**; the measurement table and applied rules are shown separately.

#### Demand Forecast

Demand Forecast produces short- and long-term demand projections from historical request volume. If the confidence interval and historical coverage are insufficient, this is indicated on the screen.

This surface only forecasts **demand**. It does not produce outcomes such as CPU/memory limits, HPA ceiling, node headroom, or "capacity will run out in how many days."

### Dashboards

Dashboards let you build persistent service dashboards from the widget catalog Komuta allows. Depending on your permissions, you can create, edit, or view dashboards read-only.

If a saved dashboard's query fails, the screen does not show this as "no dashboard"; it presents an error and a retry option. Unknown or no-longer-supported widget types are not run.

### Natural Language Telemetry Query

On this surface, you can ask questions about your service in Turkish or English. Komuta converts the question into a **read-only** query scoped to the tenant and service, against the allowed datasets.

As a result, the following can be seen:

- The actual data table
- The read-only SQL that was executed
- Query duration and tool information
- AI summary or error explanation

The AI narration is not evidence; base your decisions on the result table and the query.

### Data Health

The Data Health screen answers not the question "is the service healthy?" but "can I trust the data on this screen?"

The main information shown:

- Gateway, schema, and policy version
- Whether data ingestion and the query layer are ready
- Status of the Flow, trace, and runtime-security sources
- Observed, accepted, rejected, buffered, and lost record counts
- Source watermark value and last acceptance time
- Coverage ratio and reason codes explaining any gaps

If there are rejected or lost records, the results on the screen are not considered "complete." An empty result can only be interpreted as "not observed" if the source is healthy and coverage is sufficient.

---

## Reading Evidence Labels

Komuta Observability carries an evidence status on every managed response:

| Label | Meaning |
|--------|--------|
| **Complete** | The data sources required by the query were available for the window |
| **Partial** | A result exists, but at least one required source or time range is missing |
| **Unavailable** | Not enough source data could be read to produce a reliable result |
| **Exact** | The result was calculated from all records within the scope |
| **Sampled** | The result was calculated from accepted sampled telemetry |
| **Approximate** | The result uses an approximate calculation or an upper/lower bound |

**Coverage percentage** shows how much of the required sources responded. **Watermark** indicates the last time the source was reliably complete. A query ID allows the support team to track the same query.

---

## How Does Data Reach Komuta?

```text
Application and cluster sensors
        │
        ├─ Trace: OpenTelemetry / Alloy
        ├─ Network flows: Hubble / Komuta Flow Agent
        ├─ Runtime events: Tetragon and KubeArmor / Security Agent
        └─ Logs: Loki ingestion pipeline
        │
        ▼
Komuta Telemetry Gateway
        │  validation, tenant/service ownership, policy, and record accounting
        ▼
ClickHouse (telemetry) + Loki (log) + PostgreSQL (investigation records)
        │
        ▼
Komuta API
        │  authorization, scope, query limits, and evidence envelope
        ▼
Komuta UI
```

The interface does not connect directly to ClickHouse or Loki. Every query is authorized through the Komuta API and Gateway, scoped to the service, and subject to duration/row limits.

---

## Retention Periods

The default telemetry retention policy varies by data type:

| Data type | Default raw data retention |
|-----------|-----------------------------|
| Network flows | 30 days |
| Runtime-security events | 30 days |
| Trace spans | 14 days |

Five-minute or hourly summary tables may be retained longer for some product screens. Log retention period depends on the organization plan and the log store policy. The date range selectable in the interface may be further restricted by the relevant API's safe query limit.

---

## Privacy and Secure Use

- Sensitive keys are scrubbed before runtime-security evidence is stored, and large content may be truncated. The screen separately flags truncated or corrupted evidence.
- Flow records may carry potentially sensitive fields such as pod/IP, DNS query, and HTTP URL. Grant Observability permissions only to users who need them.
- Application logs carry the message the developer wrote; do not log secrets or personal data.
- Trace attributes and events are limited by size, count, and security policies.
- Keep exported reports at the same privacy level as production data.

---

## Quick Troubleshooting

### Chart is empty but the service is receiving traffic

1. Open the **Data Health** screen.
2. Confirm that the relevant source is ready and accepting data.
3. Check whether the watermark has reached the selected time range.
4. If coverage is partial or unavailable, share the reason code with the support team.

### Trace count is lower than the actual request count

The trace collection policy may apply sampling. For total workload, evaluate the appropriate RED or traffic signal together with the sampling label on the screen, rather than the trace count alone.

### Log histogram doesn't match the table

Make sure the same time range and filters are in effect. Refresh the query after changing a filter. If the problem persists, share the query ID with the support team.

### "Unknown" appears instead of "Healthy"

Komuta does not assume missing telemetry is healthy. Check the source coverage in Data Health; the status is recalculated once the data is complete.

### Demand forecast won't load

There may not be enough history for the forecast, or the relevant query route may be unavailable. This does not mean capacity has run out.

---

## Related Documents

- [Monitoring and Log Management](https://www.komuta.io/docs/guides/monitoring-logs)
- [Alert Management](https://www.komuta.io/docs/guides/alert-guide)
- [Runtime Security](https://www.komuta.io/docs/security/runtime-security-guide)
- [Security Center](https://www.komuta.io/docs/security/security-center-guide)
- [Access Control](https://www.komuta.io/docs/security/access-control-guide)
