# Create Your First Alert

By the end of this guide, you will have a rule watching one test log, an explicitly selected notification channel, and a way to follow firing, notification and resolution. You do not need to write a query or own a cluster.

The service `orders-api-demo`, text `DOCS_ALERT_SAMPLE` and channel **Docs test** are illustrative. Use your own test service and a notification destination you can check.

## 1. Complete the preparation

Open the service’s **Logs** page. Can you see a recent application-output log? If not, correct the time range, filters and source first. Creating a rule does not make missing data arrive.

Follow [Channels and Delivery](notification-guide.md) to prepare a channel called **Docs test**. Use **Send test** and confirm the result both in Komuta and at the actual destination. Continue once both checks succeed.

You need create-alert permission, service read access and access to the target service. Channel selection also needs notification read access. If a control is missing, have your role checked against the [permissions table](alert-guide.md).

**Ready to continue:** You can see fresh service logs and find the channel test at the real destination.

## 2. Open log text matching

Go to **Alerts → Rules → New Rule**. Choose **App Level**, select the test service and choose **Log text match**. This opens the form without needing to select an existing log line first.

If a suitable line already exists in the service’s Logs page, use its action menu and **Create alert from this log**. The source and text are then filled from that line; remove variable timestamps and request identifiers from the match text.

## 3. Fill the example rule

| Field | Value for this example | Why? |
| --- | --- | --- |
| Log source | Application output logs | We will look for a line emitted through the application’s standard output/error stream. |
| Name | `Docs sample alert` | Easy to find in Rules and History. |
| Match text | `DOCS_ALERT_SAMPLE` | A stable expression specific to this exercise. |
| Threshold | `0` | At least one match in the last five minutes qualifies; the comparison is `> 0`. |
| Duration | `1m` | Require the condition to hold for one minute. |
| Notification interval | `15m` | Concerns repeat notifications while the condition persists. It does not delay the first firing by 15 minutes. |
| Severity | Warning | Keep this exercise separate from the team’s real emergency workflow. |
| Summary | `DOCS_ALERT_SAMPLE detected in the demo service` | Tell the recipient what to investigate. |
| Notification channels | **Docs test** | Select the destination intentionally; an empty selection does not disable notifications. |

![The actual log-alert form filled with example data: 1 source, 2 match text, 3 threshold and duration, 4 notification destination.](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/alerts/en/log-form.png)

*The image renders the actual Komuta form component locally with example data; no live-account rule was created. Numbers help locate the fields above.*

Read the condition preview. It should show the intended service and source, a `5m` window, threshold `0` and duration `1m`. Create the rule, follow **View alerts** in the confirmation, and confirm that it is enabled on the intended service.

**If the form cannot continue:** Correct an empty name/summary, negative or fractional threshold, or a duration without a unit. This form requires a whole-number threshold of zero or more; enter `60s` or `1m`, not `60`. If the service already has 20 enabled log rules, review those rules first.

## 4. Produce a controlled match

After saving the rule, use your test application’s existing test operation to emit one line containing `DOCS_ALERT_SAMPLE`. These expressions illustrate what could be placed at an appropriate point in your own test application:

```javascript
console.error('DOCS_ALERT_SAMPLE');
```

```python
print('DOCS_ALERT_SAMPLE', flush=True)
```

Running this code in your own computer’s terminal does not add a log to the Komuta service. The line must come from the monitored application and appear in that service’s **Logs** page. Use the application’s normal test path; no production failure is needed.

Select the correct source in Logs and search for `DOCS_ALERT_SAMPLE`. A visible line confirms data input. If it is absent, resolve the [data step](alerts-troubleshooting.md) before investigating History or delivery.

Matching is **case-sensitive literal text containment**. `docs_alert_sample` is not the uppercase expression we selected. This is not a regular-expression field: entering `.*` does not mean “everything.”

## 5. Separate the window, duration and notification interval

The times below are illustrative. Evaluation and transmission schedules can produce different timestamps in real records.

<ol class="docs-alert-timeline" aria-label="Illustrative lifecycle of a single log line" role="list">
<li><strong>12:00 · The line is emitted</strong><p>DOCS_ALERT_SAMPLE reaches the selected service stream. The number of matches in the last five minutes becomes greater than zero.</p></li>
<li><strong>12:00:20 · First positive evaluation</strong><p>In this example, evaluation first observes the condition now. The one-minute hold starts from this evaluation.</p></li>
<li><strong>12:01:20 or later · Firing</strong><p>If the condition held continuously, the rule becomes eligible to fire. The event reaching Komuta and the message reaching its destination can take additional time.</p></li>
<li><strong>Around 12:05 and afterward · Resolution path</strong><p>Without further matches, the first line leaves the window. History updates after a subsequent evaluation and receipt of a resolution event.</p></li>
</ol>

A single line can satisfy the `1m` duration because it remains in the five-minute window. Duration does not require a new line every minute. A threshold of `10` would require at least **11** matches in the same window. Each additional match can help keep the condition true.

`0s` removes only the hold duration. The `15m` notification interval concerns repeat sends. Neither is the log-counting window or a guaranteed delivery time.

## 6. Verify success in four places

| Where? | Expected result | If it differs |
| --- | --- | --- |
| Service → Logs | The test line appears in the intended source. | Check source, time range and whether the application actually emitted the line. |
| Alerts → Rules | `Docs sample alert` is enabled with the intended service and settings. | Correct a save error or wrong scope. **Not confirmed** on a log rule alone is not failure. |
| Alerts → History | A new event belongs to the same rule and service. | Follow the [not-firing path](alerts-troubleshooting.md) using the threshold, duration and available data. |
| Alerts → Channels and the destination | A corresponding sending record and an actual message exist. | If the event exists but the message does not, follow the [notification path](alerts-troubleshooting.md). |

Complete an email sending-success check by inspecting the recipient’s actual inbox. For Slack or Teams, check the intended channel. **Test / YAML** is not a firing button that replaces these checks.

## 7. Complete resolution and cleanup

Stop producing new test lines. Allow previous matches to leave the five-minute window. Find resolution information on the same event in **History**; if a resolution notification was sent, inspect it separately from the firing message in **Channels**.

If the condition no longer holds but no resolution record arrives, follow [resolution troubleshooting](alerts-troubleshooting.md). Once the investigation is complete, disable or delete the test rule. If you created a separate test channel, confirm that no other rule uses it before removing it.

**Completion criteria:** You can locate the rule, show the test line, associate the event with the actual destination message, interpret resolution and clean up the test configuration.

## Next scenario: a CPU template

After the log exercise, open the service’s **Alerts → New Rule → Use a template** and select **High CPU Usage (Service)**. The starting threshold and duration are `80%` and `5m`; choose values after inspecting normal workload and the configured CPU limit. Select channels, review the summary and create.

No event is expected while CPU usage remains normal. Data and a positive CPU limit must be available; you do not need to generate load just to exercise the form. [Template selection and tuning examples](alerts-templates.md) explain which condition fits each situation.

[Manage rules](alerts-rules.md) · [Read event records](alerts-history.md) · [Troubleshoot](alerts-troubleshooting.md)
