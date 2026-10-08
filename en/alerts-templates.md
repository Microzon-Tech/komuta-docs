# Templates, Metrics and Logs

Start with search, category and scope filters in **Alerts → Templates**. **Use** opens a wizard to choose a target and review parameters. Available options depend on accessible resources and workload eligibility.

On your own cluster, metric evaluation requires Prometheus, log evaluation requires Loki, and notification/silence handling requires an appropriately configured Alertmanager. Work with your cluster administrator on missing-component errors. Customers using services on Komuta are not expected to configure connection addresses for these components.

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
