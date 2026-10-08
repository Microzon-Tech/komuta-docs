# Alerts: Overview

Use Komuta Alerts to monitor service metrics and logs, and configure rules that notify your team by email, Slack or Microsoft Teams. Start with a template, turn a log line into an alert, and manage the same rules from a service or the global **Alerts** menu.

[Create your first alert](alerts-quick-start.md) · [Choose a template](alerts-templates.md) · [Connect a channel](notification-guide.md)

## Where should I start?

| Your goal | Reading path | What you will be able to do |
| --- | --- | --- |
| Set up your first alert | [Completed log example](alerts-quick-start.md) → channel test → event and resolution | Follow one log line through to a message at the real destination. |
| Choose a useful service condition | [Template decision table](alerts-templates.md) → normal workload → parameters | Explain the measurement unit and why you chose the threshold. |
| Change an existing rule | [Field reference and change example](alerts-rules.md) → publication check | Distinguish saved settings from observed publication. |
| Investigate a received message | [Sample event record](alerts-history.md) → verified scope → affected service | Identify the event and resource to investigate. |
| Find a missing message | [Troubleshooting decisions](alerts-troubleshooting.md) | Narrow the issue to data, rule, publication or delivery. |
| Silence maintenance notifications | [Thirty-minute maintenance example](alerts-silences.md) | Create a window for selected rules with a clear start and end. |

## Essential terms

**Scope** identifies the service, your own cluster or account-level record involved. A **metric** is a numerical measurement such as CPU percentage; a **log** is a text record produced by your application. A log alert turns matching records into a numerical condition too.

A **threshold** is the boundary used in the comparison. The **data window** defines how far back each evaluation looks; **duration** defines how long the condition must continue before firing. **Notification interval** concerns repeats of an ongoing condition. “More than 0 matches in the last 5 minutes, continuing for 1 minute, with a 15-minute repeat interval” describes three distinct choices.

**Severity** communicates priority. A **channel** is a message destination. An **event** is a firing/resolution record received by Komuta. A **silence** suppresses notifications for selected rules during a defined period. See these concepts together in the [first example’s timeline](alerts-quick-start.md).

## Four steps in an alert

<ol class="docs-alert-flow" aria-label="Alert flow">
<li><span>01</span><strong>Define the rule</strong><p>Choose a service, condition, duration and notification channels.</p></li>
<li><span>02</span><strong>Check publication</strong><p>Read the saved definition’s publication state and check time.</p></li>
<li><span>03</span><strong>Follow the event</strong><p>Find the last received firing or resolution record in History.</p></li>
<li><span>04</span><strong>Verify the message</strong><p>Check the delivery record and receipt at the actual destination separately.</p></li>
</ol>

**Saving, publishing, firing and receiving a notification are separate stages.** Success at one stage does not confirm completion of the next.

## What is in the Alerts menu?

| Page | Use it to… |
| --- | --- |
| **Overview** | Review rule and publication summaries, recent events, frequently firing rules, starter templates and notification health when permitted. |
| [**Rules**](alerts-rules.md) | Find, filter, create and perform permitted operations on rules. |
| [**History**](alerts-history.md) | Inspect the last received status, scope, firing/resolution times and available event details. |
| [**Silences**](alerts-silences.md) | Mute notifications for selected rules during a time window. |
| [**Templates**](alerts-templates.md) | Start from a condition suitable for a service or a cluster you own. |
| [**Channels**](notification-guide.md) | Inspect configured channels and delivery records; open notification settings to manage channels. |

Overview counts relate to the loaded scope and stated period. If data could not be loaded, do not interpret that as zero events or a healthy system. The number of records **awaiting resolution** is not a live count of alerts currently firing.

## Which scope should I choose?

| Your situation | Where to start |
| --- | --- |
| You have services on Komuta, without your own cluster | Choose **App Level**. Service metric templates, log templates and text matching are available. You do not need to enter infrastructure addresses. |
| You also own a cluster | In addition to service rules, use **Cluster Level** metric templates for your own clusters offered in the picker. Advanced metric queries are available in an eligible scope. |
| You are investigating account bandwidth alerts | Read the [scope explanation](alerts-history.md). An account aggregate cannot identify a single responsible service. |

On shared infrastructure, use service templates for metrics instead of custom metric queries. Custom log queries remain scoped to a service; cluster-wide log rules are not supported. An empty owned-cluster list does not prevent you from using service alerts.

## Read and change permissions

Operations depend on your role’s permissions and access to the particular resource. A role name alone does not establish what you can do.

| Operation | Permission needed |
| --- | --- |
| Overview, Rules, History, Silences, Templates and rule validation | Read alerts |
| Create, create from a template and duplicate | Create alerts |
| Edit, enable/disable, delete, publish and silence | Edit alerts, plus change access to the relevant rule |
| Select services and verify service eligibility | Read services and access the relevant service |
| Alerts → Channels and channel selection | Read notifications; Alerts pages also require read access to alerts |
| Add a channel | Create notifications |
| Edit, enable/disable, delete or test channels; change notification routing | Edit notifications |

Read access does not allow sending a test message or changing a rule. Action controls may remain unavailable while permissions load. Notification read access can provide access to the notification page without general settings-view permission; change permissions are checked separately.

## Help from the mascot

Use **Guide** in the alert wizard to follow the scope, template, parameter and channel steps. Guided tours also cover service alerts, creating an alert from a log, and following alerts after creation.

Ask “What does this publication state confirm?” or “Does this record refer to a service or the whole account?” Help uses the page’s verified scope, loading/error state, visible record summary and form step. This context does not represent the entire history, raw logs or the recipient’s inbox. The mascot cannot bypass access permissions; do not treat unknown scope or delivery as verified. Tours explain fields and navigation; review and complete creation or other changes in the relevant form.

## A useful order to follow

1. Add a [notification channel](notification-guide.md) and test it at the actual destination.
2. Create your [first alert](alerts-quick-start.md) for one service and one condition.
3. Check [publication](alerts-rules.md), then [event history](alerts-history.md).
4. Reduce unnecessary repetition with thresholds, duration and [silence windows](alerts-silences.md).

Use [troubleshooting](alerts-troubleshooting.md) when a stage does not behave as expected.
