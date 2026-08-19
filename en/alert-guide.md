# Alert Management

DevOpsZon's alert system allows you to proactively monitor the health of your services and infrastructure. By defining metric- and log-based rules, detect issues before your users do.

---

## How Does the Alert System Work?

The DevOpsZon alert pipeline consists of the following stages:

```
Metric/Log Collection → Rule Evaluation → Alert Triggering → Notification Sending
 (Komuta Metrics    →    (Alert rule)  → (Komuta Alerts) → (Email/Slack/...)
  / Komuta Logs)
```

1. **Komuta Metrics** collects metrics from the cluster
2. **Komuta Logs** collects application logs
3. Defined rules are evaluated periodically
4. When the condition is met, an alert is triggered via **Komuta Alerts**
5. An instant notification is sent to your notification channels
6. The dashboard is updated in **real time**

---

## Alert Scopes

In DevOpsZon, alerts are managed in two scopes:

### Global Alerts

Manage all application-wide alert rules from the **Alerts** page in the left menu. On this page:
- Alerts for all clusters and services are listed
- You can filter by cluster or service
- You can create new rules or edit existing ones

### Service-Level Alerts

Manage alerts belonging only to the selected service from the **Service Management** → **Alert Management** tab. This lets you focus on the health of a single service.

---

## Creating an Alert Rule

### Step 1: Choose the Rule Type

| Type | Description | Query Language |
|-----|----------|-----------|
| **Metric-based** | Based on metrics such as CPU, memory, request count | Metric query language |
| **Log-based** | Based on patterns in application logs | Log query language |

### Step 2: Define the Query

**Metric-based examples:**

| Scenario | Metric Query |
|---------|----------------|
| CPU above 80% | `rate(container_cpu_usage_seconds_total[5m]) > 0.8` |
| Memory above 90% | `container_memory_working_set_bytes / container_spec_memory_limit_bytes > 0.9` |
| High 5xx error rate | `rate(http_requests_total{status=~"5.."}[5m]) > 0.05` |
| Pod restart count increased | `increase(pod_container_status_restarts_total[1h]) > 3` |

**Log-based examples:**

| Scenario | Log Query |
|---------|----------------|
| Error log detected | `{app="my-service"} |= "ERROR"` |
| High exception count | `count_over_time({app="my-service"} |= "Exception" [5m]) > 10` |

### Step 3: Set the Severity

| Level | Use Case |
|--------|----------------|
| **Info** | For informational purposes; does not require urgent action |
| **Warning** | A situation requiring attention; may become a problem soon |
| **Critical** | Urgent action required; the service may be affected |
| **Emergency** | The system is completely affected; immediate action is essential |

### Step 4: Set the Duration

Specify how many minutes the condition must be continuously met for the alert to be triggered. This prevents temporary fluctuations from creating false alarms.

| Duration | Use |
|------|----------|
| **1 minute** | Fast detection for instant issues |
| **5 minutes** | General purpose; suitable for most scenarios |
| **15 minutes** | Trend-based issues; filters out short-lived fluctuations |

### Step 5: Choose the Notification Channel

Select which channels will receive a notification when the alert is triggered. You can select multiple channels.

### Step 6: Test It

Before saving your rule, verify it with the **Test** button. The test runs your query against the cluster and shows whether the alert would be triggered under the current state.

### Step 7: Deploy to Kubernetes

Your saved rule is automatically deployed to the Kubernetes cluster as a PrometheusRule CRD, and monitoring begins.

---

## Alert Templates

DevOpsZon offers ready-made alert templates for commonly used scenarios:

| Template | Description |
|--------|----------|
| **High CPU Usage** | CPU limit is about to be exceeded |
| **High Memory Usage** | Memory limit is about to be exceeded |
| **Pod Restart Loop** | Pod is continuously restarting |
| **5xx Error Rate** | HTTP 5xx errors are increasing |
| **Disk Capacity** | Disk capacity is running low |
| **Database Connection Pool** | Connection pool is about to fill up |

> You can use the templates directly or customize them to create new rules.

---

## Notification Channels

You can receive alert notifications through the following channels:

| Channel | Configuration |
|-------|-------------|
| **Email** | SMTP settings and recipient addresses |
| **Slack** | Channel integration via webhook URL |
| **Telegram** | Bot token and chat ID |
| **Microsoft Teams** | Incoming webhook URL |
| **PagerDuty** | Incident management via integration key |
| **SMS** | Phone number and SMS provider settings |
| **WhatsApp** | Business API integration |
| **Webhook** | POST request to a custom HTTP endpoint |

You can configure notification channels from the **Notifications** page.

---

## Alert Silencing

You can temporarily silence certain alerts during planned maintenance or known issues:

1. In the alert list, click the **Sustur** (Silence) button next to the rule you want to silence
2. Set a duration (e.g., 2 hours, 1 day)
3. Optionally add a description
4. At the end of the specified duration, the alert automatically becomes active again

---

## Addon Alerts

When managed services (PostgreSQL, RabbitMQ, Valkey) are created, basic alert rules are automatically defined:

| Service | Automatic Alerts |
|--------|-------------------|
| **PostgreSQL** | High CPU, memory, disk, connection pool, replication lag |
| **RabbitMQ** | Queue capacity, memory, connection count |
| **Valkey** | Memory usage, connection count |

These alerts are active by default and can be customized according to your needs.

---

## Tips

- **Tiered alerting:** Define Warning and Critical alerts with different thresholds for the same metric
- **Duration setting:** Too short durations produce false alarms, too long durations cause late detection
- **Test:** Always test each rule before saving it
- **Notification fatigue:** Too many alerts cause important alerts to be overlooked; only define alerts that require action
