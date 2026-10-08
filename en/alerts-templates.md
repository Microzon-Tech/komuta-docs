# Templates, Metrics and Logs

Start with search, category and scope filters in **Alerts → Templates**. **Use** opens a wizard to choose a target and review parameters. Available options depend on accessible resources and workload eligibility.

On your own cluster, metric evaluation requires Prometheus, log evaluation requires Loki, and notification/silence handling requires an appropriately configured Alertmanager. Work with your cluster administrator on missing-component errors. Customers using services on Komuta are not expected to configure connection addresses for these components.

## Which template fits the condition?

| What you want to detect | Starting choice | Reason and limitation |
| --- | --- | --- |
| Sustained resource pressure | Service CPU or memory percentage | Measures proximity to a configured limit. Without a limit, do not rely on this ratio as a starting point. |
| An application restarting repeatedly | Frequent Pod Restarts | Useful for investigating repetition beyond one deployment moment; uses estimated increase over 15 minutes. |
| A service unable to become ready | Pod Not Ready | Tracks readiness rather than resource consumption, excluding completed work from continuous-service assumptions. |
| One known error message | Log text match | Choose your own stable text and source. Appropriate for counting one type of record over five minutes. |
| Many general error logs | High Error Log Rate | Tracks the per-second rate of recognized patterns. Use text match if your custom error code does not match them. |
| Slow responses through a gateway | API Gateway p95 latency | Tracks gateway request latency without assuming high CPU is the only explanation. |
| Increasing server errors at a gateway | API Gateway 5xx error ratio | Measures a percentage of traffic and includes a guard for very low traffic. |
| Node or persistent disk capacity | Node disk / PVC template | Requires your own cluster scope and relevant capacity measurements; does not measure application log size. |

Start with a few conditions for which your team knows what action to take. Identify an owner, destination and first investigation step for each rule. For a connection-error message, that might be service logs and database reachability; raising severity does not repair the connection.

## How should I choose a starting value?

Template defaults are starting points, not universal recommended limits. The numerical workload examples below are hypothetical.

**CPU:** If normal load uses `35–60%` of the limit with brief deployment spikes, the default `80% / 5m` can be evaluated for sustained pressure. If normal usage is already `85%`, investigate capacity and limits before simply raising the threshold. The template’s duration and the metric’s calculation window are distinct.

**Memory:** The default `85% / 10m` tracks sustained usage near the limit. It may not provide enough advance warning for an application that consumes memory rapidly and exits. The OOM termination template tracks a different condition; the two are not interchangeable.

**Restarts:** The service default requires an estimated increase of `3` or more over 15 minutes, with the condition continuing for `2m`. It does not mean three restarts per minute. Compare maintenance/deployment periods with ordinary operation.

**Log text:** For a rare message requiring action, test a `> 0 / 1m` starting point. If occasional harmless errors are expected, `> 10 / 1m` requires at least 11 lines in the last five minutes. A threshold of `10` in a per-second rate template represents a very different volume.

Tune in a cycle: choose a representative normal period → verify data exists → change one threshold or duration → check save/publication → compare actual events during a similar period. Restore the previous setting if the expected benefit does not appear. The [Rules example](alerts-rules.md) shows the screen-level steps.

## Service metric templates

| Template | Condition | Initial duration |
| --- | --- | --- |
| High CPU Usage (Service) | CPU usage exceeds a percentage of the configured positive CPU limit. | 5 minutes |
| High Memory Usage (Service) | Memory usage exceeds a percentage of the configured positive memory limit. | 10 minutes |
| Frequent Pod Restarts (Service) | Estimated restart count over the last 15 minutes is greater than or equal to the threshold. | 2 minutes |
| Pod Not Ready (Service) | A pod expected to be running is not ready; completed and failed pods are excluded from this condition. | 5 minutes |
| Container OOM Killed (Service) | An out-of-memory termination was recorded in the last 10 minutes. | 1 minute |
| Pod Unschedulable (Service) | The workload cannot be placed on a node. | 5 minutes |
| Image Pull BackOff (Service) | The container image cannot be downloaded. | 5 minutes |
| CrashLoopBackOff (Service) | A repeatedly crashing container is waiting to restart. | 3 minutes |
| Container Memory Pressure (Service) | Usage exceeds 80% of a positive memory limit; this template’s threshold is fixed. | 5 minutes |
| Rollout Unavailable Replicas (Service) | An eligible Rollout workload has unavailable replicas. | 10 minutes |

Initial CPU, memory and restart thresholds are `80%`, `85%` and `3`, respectively. The form identifies editable parameters and their bounds. CPU/memory percentages refer to the workload’s limit, not the capacity of the entire physical machine.

**Missing data is not zero.** Without a CPU or memory limit, these percentage templates may produce no result. If the relevant metric is not collected, an absence of events does not prove health. Check available data in the service page.

## Templates for your own cluster

Cluster scope applies to clusters owned by your account and offered in the picker. Cluster versions of high CPU, high memory, frequent restarts and pod-not-ready templates are available, together with:

| Template | Required resource/data |
| --- | --- |
| Deployment Replica Mismatch | Desired and ready Deployment replica counts; initial duration 10 minutes. |
| Node Not Ready | Node readiness; 5 minutes. |
| High Node CPU / Memory Usage | Node resource metrics; 10 minutes. |
| High Disk Usage (Node) | Usage of the node’s root filesystem; 10 minutes. |
| PVC Almost Full | Used and total persistent-volume capacity; 10 minutes. |

Deployments and Rollouts are different workloads. **Rollout Unavailable Replicas (Service)** appears only when Rollout ownership is verified; it is not suitable for every service or scheduled job. Do not treat a Job/CronJob that is expected to complete as a continuously running service.

## API Gateway templates

Selecting a managed **API Gateway** service makes p95 latency, 5xx ratio, 429 response rate and response-cache-full templates available. These are not offered for ordinary application services; the relevant gateway metrics must be collected. The first three start with a five-minute duration, and cache-full with 15 minutes. Follow the form’s units for latency, percentages and per-second rates.

### Read gateway thresholds in the right units

| Condition | Default threshold / duration | Correct interpretation |
| --- | --- | --- |
| p95 latency | `2000` milliseconds / `5m` | The p95 of the latency distribution computed from five-minute rates exceeds 2 seconds. It does not mean every request takes more than 2 seconds. |
| 5xx error ratio | `5%` / `5m` | The ratio must exceed the threshold and total request rate must exceed `0.1 requests/s`. A single error at low traffic may not produce an event. |
| 429 responses | `1 request/s` / `5m` | The per-second rate of 429 responses exceeds the threshold; this is not a percentage. |
| Response cache fullness | `5%` unallocated space / `15m` | Space not yet allocated to the cache falls below this percentage. It may remain low in a warm cache; this does not measure cache-miss or eviction rate. |

Entering `2` for p95 selects **two milliseconds**, not two seconds. Do not infer poor performance from the cache value alone; inspect the workload and other gateway measurements together.

## Log templates

Log rules belong to a service. These templates search application logs for supported text patterns. An application describing the same issue with different wording may not match.

| Template | Measurement | Initial duration |
| --- | --- | --- |
| High Error Log Rate | Per-second rate of error/fail patterns over the last 5 minutes. | 5 minutes |
| Database Connection Errors | Count of connection-error patterns over the last 5 minutes. | 1 minute |
| OOM Killer Detected | Presence of out-of-memory patterns over the last 5 minutes. | 1 minute |
| Slow Database Queries | Count of slow-query patterns over the last 5 minutes. | 1 minute |
| High Authentication Failure Rate | Per-second rate of authentication/unauthorized/forbidden or relevant HTTP status patterns. | 5 minutes |
| Critical Exceptions | Per-second rate of fatal/critical/panic and supported exception patterns. | 2 minutes |
| API Rate Limit Exceeded | Per-second rate of rate-limit and relevant 429 patterns. | 5 minutes |

These templates use default application-log selectors; do not assume they automatically cover telemetry logs in the same way. Use [log text matching](alerts-quick-start.md) to select the telemetry source explicitly. That workflow performs a **count** over a fixed five-minute window, a different measurement from a per-second error-rate template.

A service can have at most **20 enabled log rules**. This limit covers enabled rules from all users for that service. Disabled rules do not count, and enabling a rule checks the limit again.

If an equivalent enabled rule from the same template already exists in the same scope, another enabled rule may be rejected. Changing only its name does not bypass this check; inspect the existing rule. This is not a general deduplication guarantee for arbitrary custom queries.

## Read the comparison and window

| Comparison | Example |
| --- | --- |
| `>` | At threshold 10, a value of 10 is insufficient; the value must exceed it. |
| `>=` | At threshold 3, a value of 3 qualifies. The restart template uses this comparison. |
| `== 0` | A state such as readiness is zero. An absent data series is not automatically zero. |

Templates define their comparison; not every template offers a separate comparison selector. Advanced metric expressions use PromQL, and log expressions use LogQL that returns a numeric result. Loki is the infrastructure evaluating log rules; a query returning only log lines is not a numeric alert condition. You do not need to write a query for your first setup.

Choose threshold, measurement window and duration together. For example, an `80%` threshold with a `5m` duration requires more than one high sample. Successful query validation does not prove that data is available or that the rule has fired.

Next: [Create your first alert](alerts-quick-start.md) · [Investigate missing options](alerts-troubleshooting.md)
