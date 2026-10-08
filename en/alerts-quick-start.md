# Create Your First Alert

For your first rule, choose one service, a clear condition and a tested notification channel. “Example application” and the log text below are fictional examples.

## Before you begin

- Check that you are in the correct account and can access the target service.
- You need create-alert permission. Selecting a service requires service read access; selecting channels requires notification read access.
- Confirm that the service produces the relevant metrics or logs. Choosing a template does not create missing data.
- Prepare a channel using the [Channels guide](notification-guide.md). Leaving channel selection empty does not disable notifications: matching active channels may be used.

## Path 1: Start from a service template

1. Open your service’s **Alerts** page and select **New Rule**.
2. Choose an eligible template. For example, **High CPU Usage (Service)** suits a service with a CPU limit and available CPU metrics.
3. Review the editable template values. A threshold of `80%` compares CPU usage against the configured CPU limit. Service scope is filled automatically.
4. Choose channels and a notification interval. Review the name, condition and target, then create the rule.
5. Return to the list and check enablement and [publication](alerts-rules.md). With normal CPU usage, no event is expected; you do not need to place load on a real service.

The template’s duration and severity are starting values. After creation, use **Edit** to change permitted fields if needed. The metric expression remains protected on shared infrastructure.

## Path 2: Start from a log line

1. In the service’s **Logs** page, open the line’s action menu and select **Create alert from this log**.
2. Review **Match text**. Use a stable, non-sensitive part of the message, such as `example-operation-failed`, instead of a timestamp, request identifier or personal data.
3. A **Threshold** of `0` and **Duration** of `1m` requires the number of matches in the last five minutes to remain greater than zero for one minute. A single matching line can satisfy that condition.
4. Review severity, name, summary and channels. Create the rule and use **View alerts** in the confirmation to return to alerts.

> **The counting window and duration are different.** This form counts over five minutes. A threshold of `10` means more than 10 matches: at least 11. `0s` removes the hold duration; it does not remove collection, evaluation or notification latency.

When opened from a log line, the source comes from that line. Application output logs and telemetry logs are separate streams. Seeing a record in a merged view does not mean the rule counts both streams. If the source cannot be established, select a service or line with a verifiable source.

## Path 3: Start from global Alerts

1. Open **Alerts → Overview**, or **Rules → New Rule**.
2. Select **App Level** and choose your service. Cluster scope is also offered when you own a cluster.
3. Use **Use a template** for the same template workflow. After selecting a service, **Log text match** also opens the text-matching form without first choosing a log line.
4. Select the log source for text matching, or fill the template parameters. Review channels and the final summary, then create.

**Templates → Use** opens the same wizard with the template selected. Its scope stays fixed; choose a compatible service or cluster. **Manual (Advanced)** opens the detailed query form and is not required for a first setup.

## Verify it safely

| Check | What to look for |
| --- | --- |
| Saved rule | The rule appears on the intended service with the expected settings. |
| Publication | Read the current observation for a metric rule. Log rules may not have this confirmation; **Not confirmed** alone does not mean failure. |
| Condition | Verify that the monitored data really meets the condition and hold duration. |
| Event | Find the corresponding new record in **History**. |
| Notification | Inspect **Channels**, then verify the message in the actual inbox or destination channel. |
| Resolution | After the condition clears and the relevant data window passes, check for a resolution record. |

In a suitable test environment, have your application emit a harmless test log. Do not break a production service or introduce a real failure; use non-sensitive text. Disable or delete a temporary test rule when finished. **Test / YAML** validates a rule; **Send test** checks a channel. Neither alone proves the complete workflow.

Next: [Manage rules](alerts-rules.md) · [Troubleshoot](alerts-troubleshooting.md)
