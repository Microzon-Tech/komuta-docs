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
