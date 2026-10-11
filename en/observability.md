# Observability

The Observability screens show how a service behaves in production: how many requests it receives, which endpoints slow down or return errors, where a single request spends its time across services, and how your success targets are holding up. Start here to find when and on which endpoint a problem began, to inspect a slow request step by step, or to see the effect of a deploy.

Searching logs is a separate screen: [Logs](service-logs.md). The Observability screens link to logs but do not replace them.

---

## Reaching the Screens

There are two entry points:

- **From the service page.** Open the service and click the **Observability** tab in the left menu. The service's **Overview** screen opens and the left menu switches to the Observability menu.
- **From the main menu.** On the [**Observability**](https://console.komuta.io/observability) page under the **Monitoring** heading, open the service catalogue with **Browse services**, then use **Open observability** on the service's row.

The service's Observability menu:

| Menu | What it shows | Requires Observability to be enabled? |
|---|---|---|
| **Overview** | Request volume, error rate, response time, changes and the endpoints that take the most time | Yes |
| **Trace** | Traces of individual requests and their detail | Yes |
| **Metrics** | Measured CPU and memory usage | No |
| **Dependencies and L7** | Network flows into and out of the service | Yes |
| **Endpoints** | Requests, errors and latency per endpoint | Yes |
| **Reliability → SLO and error budget** | Success targets and error budget | Yes |
| **Reliability → Anomalies and alerts** | Statistical deviations from the baseline | Yes |
| **Reliability → Alerts** | The service's alert rules and silences | No |
| **Reliability → Release impact** | Before-and-after comparison of deploys | Yes |
| **Optimize → Cost** | Cost attribution and telemetry efficiency | Yes |
| **Optimize → Demand forecast** | Demand forecast from past requests | Yes |
| **Optimize → Telemetry storage** | Stored telemetry, quota and cost | Yes |
| **Utilities → Dashboards** | Service dashboards you build yourself | Yes |
| **Utilities → Natural-language query** | Ask questions about the telemetry in plain language | Yes |

The top of the page shows the service and environment, an **Alerts** shortcut and, while the subscription is active, a **Cancel subscription** button.

---

## Enabling Observability

Observability is enabled per service. On a service where it is not enabled, the paid screens show an **Advanced observability — paid plan** preview instead; **Metrics** and **Alerts** are always available.

1. Open one of the service's paid Observability screens and click **Upgrade to unlock**.
2. In the **Enable Observability** dialog, choose how much of the service's telemetry to keep (see **Storage and Cost** below). The dialog calculates the estimated monthly cost for your choice.
3. Confirm with **Enable Observability**.

When you confirm, the service is redeployed once and tracing is added to your application. If the service has never been deployed, tracing is added on its first deploy. Data appears once the redeployed version starts receiving requests.

**Who can use it?**

- Only organization admins can enable Observability. Other members see "Ask an administrator in your organization to upgrade." instead of the button.
- Viewing the paid screens needs its own permission. Admins have it; if a member sees **Service Insights permission is required**, an admin selects the member on the **Access Control** page and grants **View observability** in the **Advanced observability** row of the **Module capabilities** card. The same row also holds **Edit SLOs**, **View anomalies**, **Natural-language query** and **Edit custom dashboards**. Details: [Organizations and Team Members](organizations-team.md).

**Turning it off.** While the subscription is active, use the **Cancel subscription** button at the top of the page. When you confirm, collection of traces, endpoint analytics and SLOs stops, the screens return to the paid-plan preview and the service is redeployed to remove tracing. Data already collected stays until it expires, and its storage is no longer billed. Your SLOs and dashboards are kept and come back when you enable it again.

---

## How Data Gets In

**Automatic tracing.** On a service with Observability enabled, Komuta adds tracing to your application itself; no code change or environment variable is needed. The language is detected from the runtime image in your Dockerfile:

| Language | Automatic tracing |
|---|---|
| .NET (including ASP.NET) | Yes. The runtime image must be based on an official Microsoft .NET image, Debian, Ubuntu, distroless or Alpine; on a base image Komuta does not recognize, tracing is not added. |
| Java (including Spring Boot, Quarkus, Micronaut) | Yes |
| Node.js (including Next.js, NestJS, Express, Fastify, Koa) | Yes |
| Python (including FastAPI, Django, Flask) | Yes |
| Go, Ruby, PHP, Rust and others | No; use the manual setup below |

Tracing is added to long-running services; no traces are collected for Jobs and CronJobs.

**Manual setup (unsupported languages).** For an unsupported language the **Trace** screen shows a message such as "Automatic tracing isn’t available for Go" and a **Set up OpenTelemetry** button. The **Connect this service with OpenTelemetry** wizard it opens has three steps:

1. **Overview** — Why setup is needed in the code.
2. **Instrument code** — Install the OpenTelemetry API, SDK, OTLP exporter and the instrumentation package for your web framework. Initialize the SDK as early as possible at application startup; enable the server, outgoing HTTP, database and messaging instrumentation your service uses; keep W3C trace context on incoming and outgoing requests.
3. **Deploy** — Tick "I added the OpenTelemetry SDK and the required instrumentation libraries to the application code." and click **Integrate and deploy**.

You do not enter the telemetry address, protocol, service identity or sampling settings; Komuta adds them and redeploys the service. This setup sends traces only, not metrics or logs. Applying the last step requires service edit permission.

**Matching traces to logs.** To see related logs inside a trace, the application must write its logs to stdout in a way that carries the trace id. Automatic matching covers .NET, Java, Python and Node.js:

| Language | What to do |
|---|---|
| .NET | Write logs through Microsoft.Extensions.Logging (ILogger). If Serilog is your only logging pipeline, forward events through ILogger as well, or emit `trace_id` and `span_id` fields. |
| Java | Write logs through SLF4J with Logback or Log4j2. |
| Node.js | Write structured logs with Pino, Winston or Bunyan. |
| Python | Use the standard `logging` module. If you use Loguru or structlog, bridge them to standard logging or emit `trace_id` and `span_id` fields. |
| Other languages | Propagate W3C trace context and write structured `trace_id` and `span_id` fields in your logs. |

Redeploy the service after changing the logger.

**Turning automatic tracing off.** If automatic tracing is incompatible with an application, it can be turned off for that service: set the service's `security.autoInstrumentation.disabled` field to `true` in the Stack file. Details: [komuta.yaml Manifest Reference](stack-manifest-reference.md).

---

## Overview

**Overview** answers "is the service working, compared with when, and where is the time going?" on one screen.

**Time range.** Choose **1 hour**, **6 hours**, **24 hours** or **7 days**; the default is 1 hour. The 1-hour and 6-hour ranges refresh every minute; 24 hours and 7 days show the state at the time they were read. The **Compare with yesterday** switch compares each value with the same range one day earlier; with 7 days selected the switch becomes **Compare with last week**. The range and comparison are kept in the address bar, so anyone you share the link with opens the same view.

**Status line.** The screen opens with a one-line status:

| Status | Meaning |
|---|---|
| **Healthy** | Server errors are below 1% and no alert is open |
| **Needs attention** | Server errors (5xx) are at 1% or more, or an alert is open |
| **Something is wrong** | Server errors are at 5% or more, or a critical alert is open |
| **No requests in this range** | Nothing called the service, or tracing is not switched on for it |
| **Status unclear: part of the data is delayed** | Part of the data for the range has not arrived; the numbers may still change |
| **The data could not be read** | The telemetry could not be read; this says nothing about the service's health |
| **Status could not be determined** | The evidence is not enough to say whether the service is working |

Client errors (4xx) never change the status. A **Partial data** badge next to the status means part of the range's data is missing; the **Data source** section at the bottom of the page says what is missing.

**Indicators.** **Requests / min**, **Server errors (5xx)**, **Client errors (4xx)** and **Response time (p95)**. With comparison on, each shows its change against yesterday or last week. The response time tile also gives p50 and p99.

**Charts.** **Requests by status** splits requests into 2xx, 3xx, 4xx and 5xx; **Response time** shows latency over time. Changes such as deploys and restarts are marked on the charts.

**Changes and alerts.** Deploys, failed deploys, restarts (with their cause, such as out of memory, health check failed or failed to start), crash loops, and alerts opened or resolved in the range.

**Where the time goes.** The endpoints that take the largest share of total time. **All endpoints** takes you to the **Endpoints** screen.

**Resources now.** The service's current CPU and memory usage and their share of the limit; details are on the **Metrics** screen.

The links at the bottom open **Open traces**, **Open logs**, **All endpoints** and **Dependencies and L7** with the same range. Because the Logs page reads at most 12 hours, the link becomes **Open logs (last 12 hours)** for 24 hours and 7 days.

---

## Endpoints

**Endpoints** shows requests, errors and latency for each route of the service on its own row. It reads the same data as Overview, so the two screens never disagree.

The counting window can be **1 hour**, **6 hours** or **24 hours**. If you came from Overview it opens with the range you had there; 7 days becomes 24 hours.

| Column | What it shows |
|---|---|
| **Endpoint** | Method and route template (e.g. `GET /api/orders/{id}`) |
| **Trend** | Request trend across the window |
| **Requests** | Total number of requests |
| **Faults** | Server error (5xx) rate; the count in brackets |
| **Refused** | Client error (4xx) rate; the count in brackets |
| **Worst bucket p95** | The p95 latency of the worst 5-minute bucket in the window |

> **Note:** The latency column is not the p95 of the whole window; it is the p95 of the worst 5-minute bucket. Rows without a latency digest show **no sketch** instead of a value; when you filter by a duration threshold those rows are left out rather than counted as fast.

**Filtering and sorting.** Search by route or method. The health filter takes **All**, **Failing** (any 5xx), **No errors** (no 4xx or 5xx at all) and **Unrated** (no requests to rate). Under **Thresholds** you enter **At least this many requests** and **Worst bucket p95 at least (ms)**. Sort by **Most traffic**, **Most errors**, **Slowest bucket** or **Route**.

**Special rows and badges.**

| Row / badge | Meaning |
|---|---|
| **Requests without a route** | Requests no route answered; most are 404s. They are not named by their path, so every address a scanner tries does not become an endpoint. Search their traces by time or status. |
| **Other requests (cardinality limit)** | Once the service passes its limit of distinct endpoints, the remaining requests are grouped in this row. Their traces keep their real routes. |
| **Health check** | Part of the row's requests are the platform's health checks; the tooltip gives the share. These requests are counted in the totals. |
| **Framework route** | The route is the template your application's framework reported. |
| **Inferred route** | The framework reported no route, so the template was inferred from the observed request path. |

Komuta replaces the variable parts of request paths (numbers, UUIDs, long hexadecimal values, tokens, email addresses, IPs) with placeholders, so `/orders/123` and `/orders/456` land on the same row.

**Endpoint detail.** Clicking a row opens four tabs:

- **Traffic over time** — Requests, errors and p95 in **5 minutes** (last 6 hours), **1 hour** (last 7 days) and **1 day** (last 30 days) buckets. The axis stops where stored buckets run out; that does not mean traffic stopped.
- **Status codes** — Count, share and p95 for each HTTP status code. Clicking a code opens the traces that returned it.
- **Traces** — The endpoint's traces.
- **Logs** — The endpoint's newest traces, matched to the service's log lines by trace id. **Download these lines** saves them as `.ndjson` or `.log`.

---

## Traces

A trace is the end-to-end record of one request or background job. Each step is a *span*: the incoming request, a database query, a call to another service. The **Trace** screen lists traces; selecting one opens its detail beside the list.

### Searching

1. Choose the time range: **Last 15 minutes**, **Last 1 hour**, **Last 6 hours** or **Last 24 hours**. The default is 1 hour.
2. Narrow it if needed: **Errors only**, a minimum and maximum **Duration (ms)**, **Method**, **Route** (chosen from the route catalogue), **Job name** for background jobs, **Queue or topic** for queue consumers.
3. Choose the sort order: **Newest**, **Longest operation** or **Errors first**. A page shows 5 or 20 traces.

If you are looking for a specific trace (for example the `trace_id` in a log line), type it in the **Open by Trace ID (32 hexadecimal characters)** box and click **Open trace**. The direct lookup searches only inside the selected time range; widen the range if the trace is older.

> **Note:** If the selected range has no traces when the screen opens, it widens the range step by step (1 hour, 6 hours, 24 hours) and says so: "No matching traces were returned for Last 1 hour, so the window was widened to Last 6 hours." This stops once you choose a range yourself.

**Live updates.** While the list is open the window advances every minute: **Following newest** appears with the **Newest** sort, **Live window** with the other sorts. Moving to the next page or opening a trace by ID holds the window (**Paused**); **Follow newest again** resumes it.

**Accounted for switch.** By default the list shows only traces that can be accounted for: an incoming request or a named background job. One-span outbound calls with neither behind them are hidden. Turn the switch off to see them all (**Including unexplained**); those rows carry the **Unexplained** badge.

**Reading the rows.** Each row shows the root operation (method and route, or job name), the status (**OK**, **Error**, **Not reported**), the duration and the span count. Duration is wall-clock elapsed time; long-lived connections (**Long-running connection**) and scheduled jobs (**Scheduled job**) are labelled separately. The **Longest operation** sort ranks by the root operation's duration, not by CPU time or the sum of span durations.

---

## Trace Detail

When you select a trace, its detail opens with two views:

- **Span waterfall** — Spans on a time axis, in their call hierarchy. Each row shows the span's duration and the share spent in its own code (**self**). The header shows the critical path: the chain of spans that decides when the operation finished, and how much of it was spent in outgoing calls versus the service's own code. If there are errors, the **first failure** link and the **Previous failure** / **Next failure** buttons take you to the failing spans. Consecutive repeated rows are folded.
- **Service map** — The services that took part in the trace and the calls between them; each node shows its call count, error count and self time.

Clicking a span opens the **Span evidence** panel: service, span kind, start, duration, HTTP method and route, status message, **Governed attributes**, **Resource attributes** and **Span events**.

**Correlated logs.** Log lines carrying the trace's id are listed below the trace. A badge says how they were matched: **Matched by trace id** is an exact match; **Matched by time window** is approximate, and some lines may belong to other requests. To see a line in the logs page with its neighbours, use **Open this line in the logs page**.

**Download evidence report.** Downloads the trace's id, evidence state and spans as a Markdown file; handy for attaching to a support ticket or an incident record.

You may see one of these warnings in the detail:

| Warning | Meaning |
|---|---|
| **Partial span detail: … of … spans shown** | The trace has more spans than the safety limit (at most 2,000 spans are shown). The waterfall is not the full execution path. |
| **… spans have missing parents** | Some spans' parent spans were not received; these rows are shown outside the hierarchy. |
| **Clock disagreement** | Some spans' clocks disagree with the root's; their positions are approximate, their durations are correct. |
| **No retained trace spans found** | No spans were retained for this trace in the selected range. Normal and fast traces may not be retained by sampling. |

### Comparing Traces

- **Compare with similar traces** — Lists the 10 slowest traces of the same operation (same method and route) in the same time window and states where this trace ranks within that set. The rank applies only to that set; it is not a percentile over all traffic.
- **Open the expanded presentation** — Opens the trace in a wide view. **Compare with another trace** there opens a second trace (from the slowest of the same operation, or from the traces loaded in the list) beside it with its own waterfall. This is how you put a fast and a slow request side by side.

For traces whose root span has no route or method, no comparable set can be built; the screen says so and asks you to pick the second trace from the loaded list.

---

## Sampling and Evidence Labels

The numbers on these screens come from two different sources; knowing the difference is what lets you read them correctly.

**Counts are not sampled.** Request, error and latency values on Overview and Endpoints come from counters that count every request. The **Record** row in the **Data source** section at the bottom of Overview states this: **Span counters taken before sampling — every request counted**.

**Traces are sampled.** Not every request's trace is kept. Error and slow traces have separate keep rules and are kept at a higher rate; only part of the normal, fast traces are kept. That rate appears at the top of the Trace screen as **Normal trace retention**. So the trace list is not an exact count of requests, and not finding a request's trace does not mean the request did not happen.

Labels you will often see:

| Label | Meaning |
|---|---|
| **Exact evidence** | The query read every source completely; the result is exact. |
| **Qualified evidence** | The result is real data but may be incomplete (sampling, or a delayed or unreadable source). |
| **Evidence coverage** | How much of the requested data could be read. Shows **Not measured** when unknown. |
| **Source watermark** | The point up to which data has fully arrived. Newer records may not have arrived yet. |
| **Partial data** | On Overview, the status rests on part of the range rather than all of it. |
| **Latest refresh was refused** | The latest read disagreed with itself and was not accepted; the screen keeps the last valid reading, which is not advancing. |

The overall state of the result also appears above the trace list:

| State | Meaning |
|---|---|
| **Real records · partial traffic** | The traces shown are real, but because only part of normal traffic is kept, the list is not an exact total of all requests. This is the usual state for sampled traces. |
| **Verified empty** | The query completed fully and there really are no traces in the range. |
| **Delayed** | The source data is behind; recent traces may be missing. |
| **Unavailable** / **Incompatible** | The query did not complete; no exact or "healthy" result is shown. |

> **Note:** The Observability screens never show unreadable data as "healthy" or "zero". A card titled "… could not be read" is about the read, not about the service. Try again in a moment, or choose a narrower range.

---

## SLO and Error Budget

An SLO is the success rate an endpoint, or the whole service, must meet over a period. The error budget is the number of failed requests allowed without breaking that target.

1. On the **Reliability → SLO and error budget** screen, click **New SLO**.
2. Enter a **Display name**.
3. Fill in **Method** and **Path** (e.g. `GET` and `/api/checkout`). Leave both empty for an SLO that covers the whole service (**Service-wide SLO**).
4. Choose a **Target (%)**: 99%, 99.5%, 99.9%, 99.95% or 99.99%. The default is 99.9%.
5. Choose a **Window**: **7 days** or **30 days**. The default is 30 days.
6. Save with **Create SLO**.

If you are unsure which target to pick, the **Recommend** button uses AI to suggest a target and window from the endpoint's last 30 days of traffic; values you accept are filled into the form.

Each SLO card shows the target, the current success rate, the error budget used and the status:

| Status | Meaning |
|---|---|
| **In budget** | Less than 25% of the error budget is used |
| **Watch** | 25–75% of the budget is used |
| **Burning fast** | 75–100% of the budget is used |
| **Exhausted** | The budget is used up; the target cannot be met in this window |
| **Window warming** | There is no data for the whole window yet; values cover only the days with data |
| **Unknown** | The status could not be calculated |

**SLO notifications.** Komuta evaluates SLOs every hour and notifies your organization's notification channels when the status gets worse: **Exhausted** is critical, **Burning fast** is a warning and **Watch** is informational. Reminders follow while the budget keeps burning, and one notification is sent when the SLO recovers. Notifications are paused while the window is warming. To set up channels: [Channels and Delivery](notification-guide.md).

---

## Other Screens

| Screen | What it is for |
|---|---|
| **Metrics** | The service's measured CPU and memory usage. A missing series is shown as unknown, never as zero. Available without enabling Observability. |
| **Dependencies and L7** | Network flows into (**Inbound**) and out of (**Outbound**) the service; HTTP error rate and p95 where HTTP was seen. Flows are counted, not requests: one long-lived connection is one flow. |
| **Anomalies and alerts** | Compares each endpoint's latency, error rate and request volume with its baseline over the last 7 days and shows only statistically supported deviations. A new endpoint is not scored until its baseline has built up. |
| **Alerts** | The service's alert rules, firing history and silences. Details: [Alerts: Overview](alert-guide.md). |
| **Release impact** | Compares the hour after each deploy with an earlier hour; when evidence is incomplete it does not claim cause and effect. |
| **Cost** | The service's cost attribution, telemetry volume and AI-assisted savings suggestions. |
| **Demand forecast** | A forecast of the coming days' request demand from past volume. Resource headroom is not inferred from it. |
| **Dashboards** | Build and save your own service dashboards from a catalogue of widgets. |
| **Natural-language query** | Ask questions about the service's telemetry in Turkish or English; you see the returned table and the read-only query that ran. The generated narrative is guidance, not evidence. Each organization can run 30 queries per hour. |

---

## Storage and Cost

For a service with Observability enabled, traces, network flows and runtime security events are stored and billed by the amount stored. Logs are not part of this. Size is measured as the telemetry arrives (uncompressed), and the bill uses the same figure.

**Price.** USD 0.000342 per GB-hour (about USD 0.25 per GB-month), excluding tax. Usage is metered hourly and appears in the **Observability** category on your statement. When Observability is enabled on a service, the terms at that moment (the included space, any free period, and the price) are locked to the service; changing the storage choice later keeps them. The current terms are shown in the enable dialog and on the **Account → Observability Storage** page.

**Storage choice.** When enabling, and later in the **How much to keep** section of **Optimize → Telemetry storage**, you pick one of two options:

| Option | What happens |
|---|---|
| **Stay within …** | Once data passes the included space, the oldest data is deleted to make room; the service stays at the included size. You never pay for more than the included space. This is the default. |
| **Grow past …** | Older data is kept longer; you pay for every GB the service stores. Set an upper limit with **Stop growing at**; once it is reached, the oldest data is deleted. Without a limit the service grows up to the platform's per-service ceiling. |

If there is a free period, the included space is free during it; with the grow option, anything above the included space is charged from the start. Once the free period ends, every GB the service stores is charged (at most the included space with the stay-within option).

The **Cost calculator** uses the service's measured daily average (the last 7 full days), or a value you enter, to show the size the service will level off at, when it will fill up and the monthly cost. Shrinking the choice deletes part of what the service stores right away; the calculator says how much.

> **Warning:** Deleted telemetry cannot be restored. Read the calculator's warning before changing the choice.

**The Telemetry storage screen** shows the service's status, the stored size (split into **Traces**, **Network flows** and **Runtime security events**), the **Oldest data kept** and the **Last measured** time. You change the storage choice with **Save choice**; only organization admins can do this.

| Status | Meaning |
|---|---|
| **Not measured yet** | The service is measured a few minutes after it sends telemetry. |
| **Within quota** | Stored data is below the quota. |
| **…% full** | 80% or more of the quota is used. |
| **Over quota** | The quota is exceeded; the oldest data is being deleted to make room. |
| **Refusing new data** | Stored data exceeds the quota plus its tolerance; new traces, network flows and security events are not kept. |
| **Not kept** | Observability is not enabled on the service. |

When a service refuses new telemetry, a "… is refusing new telemetry" notice appears at the top of the console. You have three options: switch to the grow option, raise the growth limit, or change nothing. If you change nothing, the oldest data is deleted and new data is accepted again once the service is back under the limit; data that arrives in between is lost. Notices also go to the organization's billing email addresses when the quota is nearly full, when it is exceeded and when data starts being refused.

**Account → Observability Storage.** Shows every service's usage, status and monthly estimate in one table, along with the **Estimated monthly total**, the **Price per GB / month** and any free period. On the **Cycle budget** card you can enter a USD amount per billing cycle. The budget is a warning threshold, not a spending cap: the card tells you when the cycle-end forecast or the collected charges reach it, but collection does not stop.

**How long is data kept?** Unless the quota deletes it sooner, traces are kept for at most 14 days, network flows for 7 days and runtime security events for 30 days. The counts behind Overview and Endpoints are kept separately; the endpoint traffic chart reaches back up to 30 days.

---

## Privacy

**What is collected?** For each trace: the request path and route template, HTTP method and status code, durations, calls between services and the attributes your application's libraries add to spans (for example database query text). For network flows: the source and destination of the connection, the HTTP path and DNS queries. Logs are collected separately and live on the [Logs](service-logs.md) screen.

**Masking.** Komuta strips sensitive information from trace data when storing and showing it:

- Attributes such as `authorization`, `cookie`, `set-cookie` and connection strings, and headers whose name contains token, secret, password, apikey or credential, are removed entirely.
- Bearer and Basic credentials, assignments such as `password=`, `token=` and `api_key=`, JWTs, private keys, and user names and passwords inside URLs are replaced with `REDACTED`.
- Sensitive query parameters in URLs (key, secret, password, signature, code, token and similar) are masked.
- A value is shown up to 4,096 characters; at most 48 attributes and 32 events per span are visible.

Removed values are noted in the span panel as "… value(s) omitted by policy or source limits", "Sensitive content was removed by the telemetry redaction policy." or "Sensitive SQL fragments were removed by the database-statement redaction policy."

> **Warning:** Database query text is kept with only secret-looking fragments removed. Personal data such as email addresses, IP addresses or names is not masked in traces. Avoid putting personal data in URL paths, query parameters and SQL text.

**Access.** Data is shown only inside your organization, to users with the relevant permission; every query is scoped to a single service and its environment.

**AI features.** When you use **Natural-language query**, your question and up to 200 rows returned by the query are sent to an AI model to produce the answer. The SLO **Recommend** button and the suggestions on the **Cost** screen also use aggregated endpoint statistics for the same purpose.

---

## Using It with MCP

If you have connected your coding agent to Komuta, the agent can search the same retained traces you see in the console, open a trace's detail and read the service's logs; for example, you can ask it to "find the 5 slowest requests in the last hour and summarize the waterfall of the first one". Setup: [Installing the Komuta MCP Server](mcp-setup.md). Tool list: [Komuta MCP](mcp-tools.md).

---

## When Data Is Missing

| Symptom | Likely cause | What to do |
|---|---|---|
| The screen shows the **Advanced observability — paid plan** preview | Observability is not enabled on the service | An organization admin enables it with **Upgrade to unlock**. |
| **Service Insights permission is required** | The user lacks the view permission | An admin grants **View observability** under **Module capabilities** → **Advanced observability** on the **Access Control** page. |
| **No requests in this range** right after enabling | The service has not been redeployed yet or has received no requests | Check that the deploy finished on the [Deployment History](service-deployment-history.md) page and send the service a request. |
| "Automatic tracing isn’t available for …" on the **Trace** screen | The application's language is not covered by automatic tracing | Follow the **Set up OpenTelemetry** wizard. |
| No traces at all for a .NET service | The runtime image's base was not recognized | Use a runtime image based on an official .NET image, Debian, Ubuntu, distroless or Alpine in your Dockerfile and redeploy. |
| No traces for a Job or CronJob | Tracing is not added to these workload types | Use the logs. |
| A known request's trace cannot be found | The trace was sampled out, is outside the selected range or is past its retention | Widen the range; error and slow requests are kept at a higher rate. When you arrive from a log line, **No retained trace spans found** describes this case. |
| **Correlated logs** is empty in a trace | The logger is not supported or logs do not go to stdout | Set up the logger according to the language table in **How Data Gets In** and redeploy. |
| New traces stop arriving and the status is **Refusing new data** | The service exceeded its storage limit | On the **Telemetry storage** screen, switch to the grow option or raise the limit. |
| A card says "… could not be read" | The read did not complete | Try again in a few minutes, or choose a narrower range. |

---

## Related Documents

- [Logs](service-logs.md) — Searching service logs; jumping from a log line to its trace.
- [Service Dashboard](service-dashboard.md) — The service's current state and Service Doctor.
- [Alerts: Overview](alert-guide.md) — Alert rules, history and silences.
- [Channels and Delivery](notification-guide.md) — Channels that receive SLO and alert notifications.
- [Organizations and Team Members](organizations-team.md) — Granting observability permissions to members.
- [komuta.yaml Manifest Reference](stack-manifest-reference.md) — Turning automatic tracing off from the Stack file.
- [Billing and Wallet](billing.md) — Statements, wallet and spending limit.
- [Komuta MCP](mcp-tools.md) — Querying traces and logs with your coding agent.
- [Pricing](https://www.komuta.io/pricing) — Current prices.
