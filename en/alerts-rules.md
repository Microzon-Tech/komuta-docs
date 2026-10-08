# Rules and Publication

**Alerts → Rules** brings together accessible rules in your account. **Alerts** within a service provides the corresponding operations in that service’s context.

## Find and inspect a rule

Use search and the available scope, severity, enablement and publication filters. A rule missing from a filtered view has not necessarily been deleted. Read its name, service/scope, duration, enablement switch and publication badge together.

Severity levels are **Info, Warning, Critical and Emergency**. Severity communicates priority and interacts with channel severity filters; it is not a promise of automatic remediation or escalation.

## How should I fill in the fields?

| Field | A useful choice | Form limit or behavior |
| --- | --- | --- |
| Name | `Orders API — connection errors` identifies the service and condition | Required, up to 128 characters in the text-match and edit forms. |
| Summary | `Database connection errors appeared in the last 5 minutes; inspect service logs.` | Required, up to 256 characters in those forms. Omit secrets and customer data. |
| Match text | A stable fragment such as `connection refused` | Up to 512 characters in the log text form; case-sensitive literal containment. |
| Threshold | `0` to detect at least one matching log line | A non-negative integer in the text form. Template units and limits vary. |
| Duration | `30s`, `5m`, `1h` | A number and unit; use `60s` or `90s`, not `60` or `1m30s`. `0s` removes the hold duration. |
| Notification interval | For example, `15m` for an ongoing condition | At least 5 minutes when supplied. Leaving it blank does not disable notifications. |
| Severity | The priority agreed by your team | Check that the channel’s severity filter accepts it. |
| Channels | An active channel for the team responsible | Empty selection routes to eligible active channels. Select a target explicitly if you want one. |
| Advanced query | A numerical condition in an eligible scope when a template is insufficient | Up to 2,000 characters in the edit form. Follow query locks. |

Template creation parameters and existing-rule edit fields are not always interchangeable. For example, you cannot change a shared-infrastructure metric expression in the edit dialog. If you need another threshold, prepare the new definition through an appropriate template and deliberately manage any overlap between enabled old and new rules.

## Example: reduce notifications from brief CPU spikes

After observing the normal workload, you decide that CPU increases lasting over five but less than ten minutes do not require action. This is an illustrative tuning decision; do not hide a real capacity problem by extending the duration.

1. Find the correct service’s CPU rule in **Rules**. Record its current `80%` threshold, `5m` duration, severity and channels.
2. Select **Edit** and change only duration to `10m`. The same condition must now continue longer; CPU limits and the data window remain separate settings.
3. Save and reopen the form to confirm that `10m` persisted. Then inspect the publication check time and any operation result.
4. Follow automatic publication for automatically delivered rules. Use the direct publication flow if it is offered. If **Definition differs** persists, do not assume the new duration is installed.
5. Compare service measurements and **History** during the next comparable workload. The expected effect is fewer events from brief spikes, not immediate resolution of an existing open record.
6. If the result is unsuitable, restore `5m` and repeat the save/publication checks.

If you use **Duplicate** to prepare another configuration, the copy starts disabled. Enabling both rules does not create a special comparison mode; both can generate notifications.

## Available operations

| Operation | Effect and considerations |
| --- | --- |
| **New Rule** | Create from a template, service log text match or an advanced query in an eligible scope. Creating an enabled rule also starts its publication flow. |
| **Edit** | Change permitted name, query, duration, severity, summary and notification fields. Metric expressions on shared clusters are locked. |
| **Enable / Disable** | Change enablement and the associated published definition. This does not turn an open historical record into proof of resolution. |
| **Duplicate** | Copy settings to a new rule. The copy starts disabled; review its name and channels before enabling it. |
| **Test / YAML** | Validate the rule and see a configuration preview when available. This does not cause a real firing or send a notification. |
| **Deploy / Undeploy** | Available for rules that support direct publication. Rules delivered through the automatic publication flow do not offer these operations. |
| **Delete** | After confirmation, start removal of the rule and its published definition. This is not a history-clearing operation. |

Select multiple user-created rules to enable, disable, deploy/undeploy eligible rules, change severity or delete. Review both successful and failed counts in bulk results: one successful item does not mean the entire selection succeeded.

## Managed rules

Rules automatically provisioned by Komuta differ from rules you create. In **Rules**, these managed rows do not offer editing, deletion, enable/disable, duplication or bulk changes. You can inspect details and history. **Test / YAML** may report that testing is unsupported for bandwidth-agent-managed rules.

Automatic publication does not make every rule read-only. A service metric rule you created may be published automatically while still allowing you to edit supported fields. Follow the available controls. To reduce notifications temporarily, use [Silences](alerts-silences.md) for eligible rules.

## Publication request versus observation

A save or deploy response describes the operation’s result. For supported rules, the publication badge compares the installed definition with the saved definition at the last check.

| Badge | Meaning | Next step |
| --- | --- | --- |
| **Published** | The installed definition matched the saved definition at the check time. | Verify evaluation, events and delivery separately. |
| **Definition differs** | The installed and saved definitions differ. The installed version may still evaluate. | Review the latest change and publication operation. |
| **Not installed** | The last successful check did not find the rule in its authorized source. | Check enablement and the publication path. |
| **Not confirmed** | Sufficient current publication evidence is unavailable. | Do not equate missing evidence with failed publication; investigate any reported error. |

Read the badge’s check time. A stale or unavailable observation does not count as current confirmation. The current publication observation does not confirm installation of log rules or bandwidth-agent-managed rules; these can show **Not confirmed**.

A publication request can be pending, processing or failed. If **Retry** is offered after failed direct publication, inspect the error first. Do not expect a manual deploy button for automatically delivered rules. Recreating a rule just because it is pending can create duplicates.

A healthy/synchronized service deployment, sleep or wake-up does not by itself prove that an alert rule is installed, paused or evaluating. Do not assume a fixed publication or delivery time.

## Duration and notification interval

- **Duration** is how long the condition must continue before firing, for example `2m` or `30s`.
- **Notification interval** concerns repeated notifications. A supplied value must be at least five minutes; it is not a guaranteed delivery schedule.
- The query’s **data window** is separate. A condition looking back five minutes can remain true after new errors stop.

For brief spikes, review duration. For persistent unnecessary repetition, review the threshold and notification interval. [Templates](alerts-templates.md) explains comparisons and missing data.

Next: [Read History](alerts-history.md) · [Investigate publication](alerts-troubleshooting.md)
