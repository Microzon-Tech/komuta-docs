# Troubleshoot Alerts

First identify the affected stage: **rule → publication → event → notification**. Start by checking the account, service, time range and filters.

## The alert does not fire

1. Check that the rule is enabled and monitors the intended source.
2. Read its [publication badge](alerts-rules.md). **Not confirmed** alone is not failure; observation is limited particularly for log and agent-managed rules.
3. Check that metrics/logs are arriving. CPU/memory percentage templates require positive resource limits.
4. Inspect the comparison, threshold, data window and duration separately. Ten matches do not satisfy `>10`.
5. Check that the exact log text exists in the selected source. Distinguish application output from telemetry, and remove variable timestamps/request details from the match text.
6. Search History for the event. A missing notification does not prove a missing event.

Even when **Test / YAML** passes, no event is expected without data meeting the condition. Do not automatically interpret missing data as zero or healthy.

## The record does not resolve

History shows the last received status. Use current data to check whether the condition has actually cleared. A five- or 15-minute query window may still contain earlier events. If the condition cleared but no resolution arrived, note the times and contact support. Deleting/disabling a rule or sleeping a service does not establish delivery of a resolution.

## A notification does not arrive

- First check that the relevant firing or resolution record exists.
- Inspect the rule’s channel selection, channel enablement and any severity filter. Update the selection if a channel was deleted.
- Check active silences affecting that rule and their scope.
- Review repeat intervals, notification limits and failed/suppressed records in Channels.
- Use **Send test** to try the channel independently. For email, check actual recipients and spam/quarantine; for Slack, the destination channel; for Teams, the webhook, workflow access and run history.
- Verify the actual destination even if the product record is successful. Email provider acceptance is not inbox confirmation.

## Publication is pending or failed

Separate the save result from current publication evidence. **Definition differs** means the older definition may still operate; **Not installed** means it was absent at the last successful check. **Not confirmed** means evidence is insufficient or stale.

For a directly published rule, correct any reported error and use **Retry** when offered. Do not attempt manual publication for automatically delivered rules. Record the last change time and visible error instead of repeatedly recreating the same rule. Do not infer alert installation from service deployment status.

## A template or cluster is missing

| Check | Possible explanation |
| --- | --- |
| You only have services | Your owned-cluster list may be empty; continue with service templates. |
| You want a custom metric query on shared infrastructure | Use a service metric template. Custom metric queries require an eligible owned-cluster scope. |
| You are looking for API Gateway templates | Select a managed API Gateway service. |
| You are looking for the Rollout template | Workload Rollout ownership must be verified. |
| A list cannot load | Inspect the loading/error message and retry; do not interpret failure as no eligible resources. |
| A service or action is missing | Have read, create and edit permissions checked separately. |
| A log rule cannot be created/enabled | A service and source are required; check the limit of 20 enabled log rules per service. |

## A silence cannot be created or is ineffective

Choose at least one valid rule, a single cluster scope and a window ending after it starts. Reselect a deleted rule. Do not assume muting when publication/synchronization failed. Check the local timezone, start, end and selected rules. Read-only access does not allow changes.

## Scope is unclear or notifications are too frequent

An **Account bandwidth** record does not identify one service. When scope cannot be confirmed, do not infer it from a rule name. Do not dismiss every packet drop as normal quota enforcement; [distinguish the drop type](alerts-history.md).

Check duplicate rules for the same service/condition, overlapping channel recipients, short hold durations and notification intervals. Multiple records can also be distinct real events; do not assume they are all one incident. If appropriate, use a short, narrowly scoped [silence](alerts-silences.md) while investigating the cause.

## What can I share with support?

Include these details in a [support ticket](support-tickets.md):

- The page, action and expected outcome.
- Time and timezone, including the latest change.
- Rule type, severity, duration/threshold and visible publication/event/notification status.
- Whether the displayed scope is a service, account or cluster; state when scope could not be confirmed.
- A screenshot with personal and access details removed, plus safe reproduction steps.

Do not share webhook URLs, passwords, tokens, personal information from raw logs, infrastructure access addresses or private resource identifiers. Anonymize service and rule names when needed. If support requires more detail, continue through the secure support channel.

[Back to Overview](alert-guide.md) · [Repeat the first-alert walkthrough](alerts-quick-start.md)
