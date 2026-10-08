# History, Scope and Resolution

**Alerts → History** shows alert events received by Komuta. Use the period, search and status filters. Expand a row to inspect available times, scope, trigger value and notification-record count.

> **History shows the last received status.** An open record alone does not confirm that an alert is still firing. A resolution update may not have arrived yet.

## From firing to resolution

1. The condition occurs in the rule’s data source and continues for its configured duration.
2. When a firing event reaches Komuta, the historical record is created or updated.
3. Notification routing selects matching channels; silences and notification limits can affect the outcome.
4. A received resolution event updates the record to resolved.

**Awaiting resolution** means a resolution has not yet been recorded. **Resolved** means resolution information was saved. Do not confuse **Pending** in event history with the publication queue. History is neither the rule’s live evaluation screen nor the recipient’s delivery acknowledgment.

Disabling/deleting a rule or putting a service to sleep does not prove an old event resolved. A five-minute log window may still contain earlier matches. Follow the [resolution checks](alerts-troubleshooting.md).

## Sample record: follow one test log

The following records are **illustrative, not live events or delivery evidence**. They use `Docs sample alert` and `orders-api-demo` from the [first alert example](alerts-quick-start.md). Times are on the same day and in the same timezone; they do not promise a delivery delay.

| When you check | What you read in History | Supported conclusion |
| --- | --- | --- |
| 12:02 | Rule: Docs sample alert; verified service: orders-api-demo; firing time: 12:01:20; no resolution information | A firing record for this service was received. Investigate the current condition in its logs. |
| 12:03 | The same event is still awaiting resolution | No new resolution information is visible. Do not interpret this as a new firing or a live status update. |
| 12:06 | The same event has resolution time 12:05:40 and status resolved | Resolution information was received. Review the selected period and other rows before concluding that no new events occurred. |

**Match it to a message:** Compare the rule name and verified scope first, then the event time. In **Channels**, look for the firing delivery in the same period and destination. For example, a successful send to **Docs test** at 12:02 means that attempt was recorded as successful. Find the email in the recipient’s mailbox separately. If a resolution send appears at 12:06, inspect it as a separate delivery.

**Expected outcome:** Distinguish one event’s open and resolved states and find its associated delivery. Do not combine different services or events at different times just because their rule names match.

## An investigation sequence

1. Set the **History** period to include the event time. Search for the rule name; clear the status filter if the expected row is missing.
2. Open the row. Note its displayed scope and firing/resolution times. If a value is available, read its unit alongside the template; a number is not always a percentage.
3. For a verified service, inspect its metrics or logs over the same time range. For account scope, investigate account traffic without assigning it to one service.
4. If a message is missing, continue to **Channels**. If the event is missing too, start with the [not-firing checks](alerts-troubleshooting.md).
5. If scope cannot be verified, include the visible status and times in a [support ticket](support-tickets.md). Do not infer a new scope from raw labels.

## Which service is affected?

| Displayed scope | What it establishes |
| --- | --- |
| Verified service name | The record is associated with that service’s rule. Inspect the service. |
| **Account bandwidth** | The alert concerns an account aggregate; it does not identify a single service. |
| Verified name of your own cluster | The rule is cluster-scoped. Do not infer a single service. |
| **Alert scope could not be confirmed** | Reliable scope information is unavailable. Do not infer a service from the rule name or historical text. |

For example, **Example application — High memory usage** can be associated with a service; **Account Bandwidth Packet Drops** is a different scope. Multiple services may contribute to account traffic. A record count is not a count of dropped packets.

## Packet-drop alerts have different meanings

| Alert type | What it describes | First check |
| --- | --- | --- |
| Account bandwidth packet drops | An account aggregate of datapath drops, separated from drops associated with a full shaping queue. | Investigate possible policy, traffic-identity or resource issues with support; do not dismiss it as ordinary quota enforcement. |
| Account bandwidth queue pressure | Where the required measurements exist, drops associated with a full shaping queue and bandwidth cap. | Review traffic demand and bandwidth capacity. |
| Account bandwidth scheduler packet drops | Packet drops from a finite network scheduling queue. | Request investigation of queue memory, traffic behavior and infrastructure saturation. |
| Account bandwidth emergency cap | On infrastructure that provides the required metrics, a protective bandwidth cap is active after policy expiry. | Ask support to inspect policy renewal; do not treat this as ordinary quota usage. |
| Service-associated high/critical packet drop rate | Drop rate from the network-observability source, different from the account bandwidth counter. | Inspect service network conditions and available drop reasons. This record alone does not identify the root cause. |
| Ingress/egress bandwidth quota alert | A bandwidth-usage condition in the relevant rule. | Check usage, direction and threshold; do not automatically treat it as evidence of packet loss. |

A listed template or managed rule does not prove that all required measurements are produced on that infrastructure. If scope or source cannot be confirmed, do not attribute the event to a particular service, quota exceedance or failure as a definite cause.

## What does the notification count mean?

The count in History reflects the product’s recorded notification information. It does not confirm that every recipient received, opened or noticed a message. Inspect [Channels](notification-guide.md) for more detail, and distinguish firing notifications from resolution notifications.

Next: [Check delivery](notification-guide.md) · [Prepare safe support information](alerts-troubleshooting.md)
