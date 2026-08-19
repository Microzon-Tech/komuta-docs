# Monitoring and Log Management

Komuta lets you monitor the Kubernetes resource status and centralized logs of your services. For request performance, traces, endpoints, network dependencies, and data coverage, the [Observability Guide](observability-guide.md) should be used instead.

---

## Service Dashboard

Each service's **Dashboard** tab is the central monitoring point that shows that service's live status.

### Live Traffic Visualization

The main component of the Dashboard is an interactive map that visually shows the flow of traffic coming into your service:

- **Ingress point:** Traffic entry point and hostname
- **Pods:** Each pod's status is shown with a color code
  - **Green:** Healthy and receiving traffic
  - **Yellow:** Starting up or not ready
  - **Red:** Unhealthy or failed
- **Traffic arrows:** Traffic flow between pods
- **Rollout status:** The status of each group in Canary/Blue-Green strategies

> This visualization updates in real time — no page refresh is needed.

### Status Cards

Summary cards on the Dashboard:

| Card | Display |
|------|----------|
| **Pod Status** | Running / Pending / Failed pod counts |
| **Replica Count** | Current / Desired / Ready replicas |
| **HPA** | Current / Min / Max replicas (if HPA is active) |
| **Resource Consumption** | CPU and memory usage percentages |
| **Last Pipeline** | Status and duration of the latest build |
| **Active Alerts** | Number of triggered alerts |

---

## Resource Monitoring

Service and cluster screens show the resource status provided by Kubernetes. This information is different from the trace-derived RED data within Observability.

### Core Metrics

| Metric | Description | Unit |
|--------|----------|-------|
| **CPU Usage** | CPU resource consumed by the service | mCPU / core |
| **Memory Usage** | Amount of RAM consumed by the service | MiB / GiB |
| **Replica Status** | Desired, current, and ready pod counts | count |
| **HPA Status** | Minimum, maximum, and current replica target | count |

For HTTP request, error, and latency measurements, use the **Observability → Trace-Derived RED** screen. Since network flows do not carry byte or packet counts, Observability flow counts should not be interpreted as bandwidth.

### Resource Monitoring (Resources Tab)

In **Service Management** → **Resources** tab:

- CPU request and limit values along with current consumption
- Memory request and limit values along with current consumption
- Resource usage trend

---

## Log Management

DevOpsZon centrally collects all service logs through the **Komuta Logs** infrastructure.

### Viewing Logs

Access your service's logs from **Service Management** → **Logs** or the **Observability → Log** screen:

1. **Date range:** Select the time period you want to view
2. **Search:** Search for keywords within the logs
3. **Pod filter:** View the logs of a specific pod
4. **Real-time:** Follow new logs live

### Log Search

You can use the following methods in the log search field:

| Method | Example | Description |
|--------|-------|----------|
| **Plain text** | `error` | All logs containing "error" |
| **Case sensitivity** | `ERROR` | Search with exact match |
| **Multiple terms** | `database connection failed` | Logs containing all terms |

### Pipeline Logs

Access the logs of pipeline runs from the **Pipeline Summary** tab:

1. Click on the relevant pipeline run
2. In the task list, click the step whose logs you want to see
3. Detailed build and deploy logs are displayed

---

## Real-Time Updates

DevOpsZon offers instant notifications through its real-time communication infrastructure:

| Event | Notification |
|------|--------------|
| **Pipeline progress** | Status is updated as each task completes |
| **Pod status change** | Pod starting, stopping, error states |
| **Rollout progress** | Canary/Blue-Green steps |
| **Addon provisioning** | Provisioning steps and completion |
| **Alert triggered** | New alerts are shown instantly |

All of these updates are automatically reflected on the panel without requiring a page refresh.

---

## Cluster Events

Cluster events related to your service are shown on the Dashboard:

| Event Type | Examples |
|-----------|----------|
| **Normal** | Pod created successfully, health check passed |
| **Warning** | Image pull error, insufficient resources, probe failed |

These events are the first resource to check during troubleshooting.

---

## Troubleshooting Guide

### Pod Won't Start

| Possible Cause | Where to Check | Solution |
|-------------|-------------|-------|
| Image not found | Dashboard → Events | Check the registry connection and image name |
| Insufficient resources | Resources tab | Increase request/limit values or add a node |
| Crash loop | Logs tab | Find the cause of the error from the application logs |

### Service Not Responding

| Possible Cause | Where to Check | Solution |
|-------------|-------------|-------|
| Readiness probe failed | Health Probes tab | Check the probe endpoint and its parameters |
| Port mismatch | Ports tab | The port the application listens on must match the configured port |
| Out of memory (OOM) | Dashboard → Events | Increase the memory limit |

### Pipeline Failed

| Possible Cause | Where to Check | Solution |
|-------------|-------------|-------|
| Build error | Pipeline Logs | Check the Dockerfile |
| Git access error | Pipeline Logs → fetch-credentials | Check the Git connection |
| Registry push error | Pipeline Logs → build-and-push | Check the registry connection |

---

## Tips

- **Keep the Dashboard open:** Thanks to real-time updates, don't miss instant changes
- **Use it together with alerts:** Do proactive monitoring by defining alert rules for metric thresholds
- **Log level:** Reduce unnecessary log volume by using a configurable log level (DEBUG, INFO, WARN, ERROR) in your application
- **Time range:** Improve query performance by selecting a narrow time range in log search
- **Check evidence status:** Before interpreting empty results, verify on the Observability → Data Health screen that the source and coverage are ready

---

## Related Document

- [Observability Guide](observability-guide.md) — traces, RED, logs, network dependencies, SLOs, anomalies, and data health
