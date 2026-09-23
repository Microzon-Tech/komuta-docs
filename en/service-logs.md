# Logs

The Logs tab is where a service's container output is searched, filtered, and watched live. Tracing an error, finding the step where a request failed, or creating an alert from a log line is all done from here.

![The full Logs tab — filter bar, volume chart, and log lines together](https://cdn.komuta.io/docs/tr/images/logs/logs-overview.png)

---

## Filters

Values in the filter bar are a draft — they don't change the table until the **Apply** button is pressed. The one exception is field filters added from a line, which rerun the query the moment they're added.

### Time Range

A single query scans at most a **12-hour** range; the default is **Last 1 hour**. A **Custom range** lets the start and end time be entered manually. Going further back is done with **Load older logs**.

### Log Level

The level filter narrows results to one of `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`, and `FATAL`.

For lines without a level tag, Komuta tries to infer the level from the message itself — common patterns like `[10:51:10 INF]`, `level=error`, `WARN -` are recognized. If none match, the line is treated as levelless and drops out of the list once a level filter is applied.

### Search Box

The search box works in two modes. In plain text mode, the entered phrase is searched inside the log message, case-insensitively. In regex mode, the phrase is interpreted as an RE2 pattern. The mode is switched with the icon next to the box.

In plain text mode the matched phrase is highlighted in the results; in regex mode it is not.

### Field Filters

When the search box is written as `field:value`, the query runs as an indexed field search instead of a text scan through the message — noticeably faster than plain-text search. Supported fields and their aliases:

| Field | What it carries | Accepted spellings |
|---|---|---|
| `trace_id` | The W3C trace id identifying a request's end-to-end trace | `trace_id`, `traceId`, `traceparent` |
| `correlation_id` | The application's own correlation id | `correlation_id`, `correlationId`, `cid` |
| `request_id` | The id of a single request | `request_id`, `requestId`, `rid`, `reqid` |
| `span_id` | The id of a single step within a trace | `span_id`, `spanId` |
| `error_code` | The error code produced by the application | `error_code`, `errorCode`, `errcode`, `err` |

Example — fetches every line for a given request.

```text
request_id:abc-123
```

A field tag on a log line, or the **Filter by this** button in the detail panel, turns that same field into a pinned filter.

A field filter only returns results if the application actually produces that field in its logs; if it doesn't, the filter matches no lines.

### Trace Context

The **Only lines with a trace id** toggle narrows results to lines carrying a valid trace id. This doesn't mean the line's trace detail is also stored — it only says the log can be linked to a trace.

Live tail can't be used while this filter is on, since live tail follows a stream rather than querying a fixed time range.

![The filter bar — time range, level, search box, and pinned field filters](https://cdn.komuta.io/docs/tr/images/logs/log-filters_v2.png)

---

## Timeline and Older Logs

### Volume Chart

The volume chart helps find *when* an error started: clicking a bar narrows the table to just that time slice.

The chart only works within a single service's scope and while live tail is off.

### Loading Older Logs

**Load older logs** moves further into the past without hitting the 12-hour query limit: each press shifts the search further back and appends the found lines to the end of the list.

---

## Live Tail

While the **Live tail** toggle is on, new log lines are added to the top of the list as they arrive and the list follows the stream. In this mode the volume chart and pagination are turned off — the screen shows the stream, not a query result.

Live tail only works within a single service's scope and can't be used while the trace context filter is on.

---

## Log Line Detail

### Detail Panel

Clicking a line opens the full record: the unclipped message, any JSON content, the structured fields the line carries, and its tags. The message and field values can each be copied individually, and the **Filter by this** button next to a field filters the logs down to that value.

![Log detail panel — message, JSON content, and structured fields sections](https://cdn.komuta.io/docs/tr/images/logs/log-detail.png)

### Jumping to a Trace

If a line carries a valid trace id, **Search trace** jumps to that trace's detail; the link is anchored to the log's timestamp. On lines without a trace id, this link is replaced by an explanation of why: Komuta doesn't match a log to a trace based on its text or timestamp alone.

### Creating an Alert from a Log

The bell icon above a line creates a pre-filled log alert from it. The match text is derived from the line's first line; the timestamp and level prefix are stripped out.

| Field | What it does |
|---|---|
| **Match text** | Log lines containing this text are counted. It should be trimmed down to the unchanging part of the message. |
| **Threshold** | The alert fires once the match count *exceeds* this value. `0` means a single match is enough; `10` means more than 10 are needed. |
| **Duration** | How long the condition must hold continuously. `0s` fires immediately; `2m` filters out momentary spikes. |
| **Severity** | The alert's severity level. |

The counting window is fixed and can't be changed: **5 minutes**. The duration field always requires a unit — `30s`, `2m`, `1h`; a bare number is invalid.

For a rare but critical line, threshold `0` and duration `0s` are used: if `OutOfMemoryException` occurs even once, the alert fires instantly.

For an error that's already occasionally seen, the threshold is raised: match text `payment gateway timeout`, threshold `10`, duration `2m`. Occasional errors don't trigger an alert; it fires only if more than 10 matches occur within five minutes and persist for two minutes.

The created rule is added to the service's alert rules and edited from there.

![Create Log Alert window — match text, threshold, and duration fields](https://cdn.komuta.io/docs/tr/images/logs/log-alert-create.png)

---

## Cross-Application Search

When a single request touches more than one service, keeping the search limited to one service breaks the trail. The **Search across all my apps instead** option in the scope bar runs the same query across multiple services and merges the results in chronological order; each line carries the service it came from.

### Correlation Filter Requirement

This mode won't run without at least one `trace_id`, `correlation_id`, or `request_id` filter. `error_code` alone isn't enough.

### Narrowing the Searched Services

By default, every service the user can access is searched. The service picker lets specific services be checked to narrow the scope. No more than **25 services** can be searched at once — past that limit the query is rejected and narrowing the scope is requested.

### Partial and Truncated Results

Two states in cross-application search signal an incomplete result, and each means something different:

- **Partial result** — some services couldn't be queried. Lines may be missing; zero results doesn't mean those services have no logs.
- **Truncated result** — the number of matched lines exceeded the merge limit (500 lines), and only the most recent matches are shown. Narrowing the time range reveals the rest.

This mode has no pagination, live tail, or volume chart.

![Cross-application search — scope bar, service picker, and merged lines carrying the service name](https://cdn.komuta.io/docs/tr/images/logs/cross-service-search.png)

---

## Sharing and Exporting

### Link to a Filtered View

**Copy link** produces a link carrying the entire current view: time range, level, search phrase and mode, trace context toggle, pinned field filters, and cross-application scope. Whoever opens the link — provided they have access — sees the same filtered table, with no need to rebuild the filters.

> **Note:** The link doesn't carry access rights. If the other person doesn't have permission to view the service, the link won't work.

### Export

**Export** downloads the lines currently loaded on screen as a plain text file. Each line carries its timestamp, level, and message; in cross-application search, the originating service is also written. Export is limited to the loaded lines — not the full query, only what's been loaded.

---

## Komuta Platform Logs

Not every line in the list is application output; Komuta also writes its own lines — for example, platform steps that run before the application starts. These lines are marked separately so they aren't confused with the application's errors.

---

## Troubleshooting

**Query timed out.** The scanned window or number of filters exceeded the log backend's response time. Rerun with a narrower time range or fewer filters.

**Time range too wide.** A single query scans at most 12 hours. Narrow the range; go further back with **Load older logs**. This warning usually comes from an old link being opened.

**Filter pattern too long / too many filters.** Shorten the search pattern or remove some of the pinned field filters.

**Too many applications to search at once.** In cross-application search, narrow the scope below 25 services with the service picker.

**Empty result.** Widen the time range or remove filters. If searching by a field filter, make sure that field is actually produced in the application's logs — if it isn't, the filter matches no lines.

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — the service's live status; clicking a pod in the topology opens this same log panel.
- [Observability Guide](observability-guide.md) — traces, RED metrics, and data health.
- [Alert Guide](alert-guide.md) — management of log alerts and other rules.
