# Rules and Publication

**Alerts → Rules** brings together accessible rules in your account. **Alerts** within a service provides the corresponding operations in that service’s context.

## Find and inspect a rule

Use search and the available scope, severity, enablement and publication filters. A rule missing from a filtered view has not necessarily been deleted. Read its name, service/scope, duration, enablement switch and publication badge together.

Severity levels are **Info, Warning, Critical and Emergency**. Severity communicates priority and interacts with channel severity filters; it is not a promise of automatic remediation or escalation.

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
