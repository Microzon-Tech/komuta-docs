# Troubleshooting Alerts

Start at the first missing stage: **data → enabled rule → publication/evaluation → event → notification**. Use the correct account, service and time range throughout. Record each result before moving on so you do not repeatedly change settings without isolating the cause.

## Which path should I follow?

| Symptom | Start here |
| --- | --- |
| Expected logs or metrics are missing | **1. Is source data available?** below |
| Data exists but History has no event | **2. Is the rule condition actually met?** |
| History has an event but no message arrived | **An event exists, but no notification arrived** |
| Even test messages do not arrive | **Follow the channel test result** |
| An event is awaiting resolution | **A record is not resolving** |
| Saving or publication fails | **Publication is pending or failed** |
| A form or option is missing | **A template, resource or action is missing** |

## No event: three checks in order

### 1. Is source data available?

**Where:** The relevant service’s **Logs** or metrics screen. Set the time range to cover the expected event.

**Expected for logs:** A new line containing the match text in the source selected by the rule. The [first example](alerts-quick-start.md) uses `DOCS_ALERT_SAMPLE`. Clear search filters and compare application output versus telemetry selection. A line visible only in another source means the rule is watching a different stream.

**Expected for metrics:** A current value and the information required by the template. CPU/memory percentage needs a positive resource limit. A missing graph or limit does not establish “zero usage and a working rule.”

**If data is missing:** Verify that the application emits the record/measurement, the source is correct and you have access. On your own cluster, resolve collection/evaluation component errors with its administrator. Do not adjust thresholds before this stage works.

**If data exists:** Note the service and source, then continue to the second check.

### 2. Is the rule condition actually met?

**Where:** **Alerts → Rules**, search by name → details or **Edit**.

**Expected:** The correct scope, an enabled rule, the intended comparison and duration. Text matching requires the stable fragment with the same letter case; expressions such as `.*` are not special patterns. Remove changing timestamps and request identifiers from the match text.

**If the condition is not met:** Correct the specific cause. `>10` needs 11 lines in the same five-minute window; exactly 10 is insufficient. A `1m` duration requires the condition to continue for a minute. Do not compare count and per-second rate templates using the same numerical threshold. For gateway 5xx alerts, review the low-traffic guard in the [template table](alerts-templates.md).

**If the condition is met:** Record the latest change time and continue to the third check. You do not need to recreate the rule.

### 3. What do publication and History show?

**Where:** The same rule’s publication badge and check time, then **Alerts → History**.

**Expected:** A current publication match for supported metric rules, followed by the relevant event if the condition and duration are satisfied. Follow the publication path below for **Definition differs**, **Not installed** or operation errors. For log and agent-managed rules, **Not confirmed** alone does not establish a publication failure.

Expand the History period to include your test and clear the status filter. If the row appears, move from data/rule investigation to **notification** investigation. If data, condition and supported publication checks are correct but no event is received, include the times and observed states in a support ticket. Do not repeatedly create rules based on an assumed delivery deadline.

A successful **Test / YAML** validation does not complete these three checks or generate real data/events.

## An event exists, but no notification arrived

1. Open the **History** row and identify the rule, verified scope, event type and time. Search for firing and resolution messages separately.
2. Inspect channel selection under **Rules → Edit**. Update the selection if the destination was deleted. Empty selection uses eligible active channels; choose explicitly if you want one destination.
3. In **Channels**, confirm the destination is active and any severity filter accepts the rule. Correct a target/filter mismatch, then follow the next controlled event.
4. In **Silences**, check active windows covering this rule and their synchronization state. If planned maintenance explains suppression, leave the rule intact. Inspect all overlapping windows.
5. Search delivery records in **Channels** using the period, channel and status filters for the event. Also review repeat interval and notification limits.

| Result | Next step |
| --- | --- |
| Failed delivery | Read the displayed error and run the channel check below. Correct invalid destinations or access problems in channel settings. |
| Suppressed delivery | Inspect the displayed restriction and notification limits. Repeated test sends do not resolve a limit. |
| Pending/processing delivery | Do not count it as complete; check its later state. Record times for persistent failures or stalled processing. |
| Successful delivery, no message at the destination | Inspect the actual recipient/channel, mail filters or Teams workflow execution. For email, success records provider acceptance. |
| No record after clearing filters | Check rule channel selection, enablement, severity, silence and repeat interval together. Include the event information when contacting support. |

The platform notification subscription matrix is separate from rule channel selection. Receiving another type of notification does not validate this rule’s routing.

## Follow the channel test result

**Where:** **Channels → Manage in notification settings**, relevant channel → **Send test**. This requires permission to send tests. Evaluate one test before starting another.

| Result / destination | Check and expected outcome |
| --- | --- |
| Test action missing or unavailable | Have notification-edit permission and permission-loading state checked. Read access alone is insufficient. |
| Test was not sent | Inspect channel enablement, required fields and the displayed restriction. This is not a successful attempt at the destination. |
| Email | Confirm addresses were saved as recipient chips. Check the actual inbox, spam/quarantine and company filters; verify every required recipient on a multi-recipient channel. |
| Slack | Confirm the stored address is an Incoming Webhook linked to the intended destination and app access still works. Verify success with a message in that channel. |
| Teams | Use a Workflows webhook rather than a Teams channel link. Check workflow enablement, ownership, authentication option and run history; a successful execution should produce a message in the intended channel. |

**If the test arrives but the real event does not:** Return to rule routing above. **If the test also fails:** Repair [channel setup](notification-guide.md), then run one new test to verify the outcome. A channel test does not fire the rule; do not expect it to create the same kind of row in event History.

## A record is not resolving

**Where:** The open event’s details and the service’s current logs/metrics.

1. Does new data still meet the condition? If so, investigate the underlying issue rather than expecting resolution.
2. If new matches have stopped, compare the last match time with the query window. The text example retains lines for five minutes; restart templates look back 15 minutes. New matches can extend the period in which the condition holds.
3. When the data no longer meets the condition, follow subsequent evaluation and receipt of resolution information. **History** shows the last received state, not a live query result.
4. If the state does not change, record the last match, firing, investigation and latest rule-change times, then contact support.

Disabling/deleting a rule or putting the service to sleep does not confirm resolution of an existing record. If resolution is recorded but its message is missing, follow the notification path for the resolution type.

## Publication is pending or failed

**Where:** **Rules**, the relevant row’s publication state and any operation details.

- **Definition differs:** Reopen the rule and check the saved settings. Compare the last change with the publication check time. The older definition may still evaluate.
- **Not installed:** Should the rule be enabled? Absence may be consistent with a disabled rule. If enabled, inspect the direct/automatic publication result.
- **Not confirmed:** Read the rule type first. For log or agent-managed rules this badge may reflect observation limits. For supported metric rules, inspect loading errors or an old check time and check again for a current result.
- **Operation failed:** Address the displayed error. Use **Retry** when offered for direct publication; do not look for a manual action on automatically published rules.

If the result persists, include setting, operation and check times in a support ticket. A healthy service deployment is not substitute evidence for rule publication. [Publication badge reference](alerts-rules.md).

## A template, resource or action is missing

| Where / symptom | Check | How to continue |
| --- | --- | --- |
| Own-cluster list is empty | Does the account own a cluster, and did the list load without error? | If you only have services, continue with **App Level** templates. Retry a failed load. |
| Custom metric expression is unavailable | Is the service on shared infrastructure? | Use a service template; advanced metric expressions require eligible own-cluster scope. |
| API Gateway / Rollout template is missing | Is the service type and verified workload eligible? | Gateway templates need a managed API Gateway; Rollout needs verified Rollout ownership. |
| Create/edit action is missing | [Permissions](alert-guide.md) and managed-rule state | Read, create and edit are separate permissions; automatically managed rules cannot be changed. |
| Log rule cannot be created/enabled | Source/scope, form errors and active log rule count | Check the limit of 20 enabled log rules per service, across all users. |
| The same template cannot be created again | Is an equivalent enabled rule already present? | Inspect the existing rule; do not retry with only a different name. |

## Silence creation fails or messages continue

**Creation error:** Select at least one valid rule, one cluster scope and an end after the start. Remove deleted rules from the selection. If synchronization fails, do not assume a successful silence record was created.

**Created but messages continue:** Compare the message’s rule with the selected rules and its event/send time with the window and local timezone. The window may not have started or may have ended. Check synchronization; another rule’s message is outside this window’s scope. The [maintenance example](alerts-silences.md) provides before/during/after checks.

## Unclear scope or too many messages

**Account bandwidth** does not identify one service. For **Scope could not be verified**, do not guess from the rule name. Distinguish [packet-drop types](alerts-history.md); they do not all mean ordinary quota limiting.

For message volume, first use **History** to distinguish different events from repeats of the same event. Then inspect duplicate rules watching the same condition, shared recipients across channels, duration and repeat interval. Collapsing records in the display does not combine future sends. For temporary maintenance noise, use a narrow silence with a defined end; investigate persistent underlying problems.

## Fill your support request with observed results

Use your own observations to complete this [support ticket](support-tickets.md) example. Values are illustrative; include the screen you checked when reporting “present” or “missing.”

```text
Action: Service log text-match rule
Expected: An event after one test line, then a message in the selected channel
Time / timezone: [date, time, timezone]
Scope: [verified service / cluster / account / not verified]
Rule: Enabled; threshold 0; duration 1m; repeat interval 15m
Data check: [source selected in Logs and last match time]
Publication: [displayed badge, check time and any error]
History: [event present/missing, firing/resolution time]
Channel: [type, test outcome, delivery record state]
Silence: [window covering this rule present/absent]
Latest change and steps tried: [...]
```

You may add screenshots with personal and access information removed. Exclude webhook URLs, passwords, tokens, personal information in raw logs, infrastructure access addresses and private resource identifiers. Anonymize service/rule names when needed, and share further details requested by support through the secure support channel.

[Back to Overview](alert-guide.md) · [Run the controlled first example](alerts-quick-start.md)
