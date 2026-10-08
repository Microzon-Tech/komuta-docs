# Channels and Delivery

Komuta Alerts uses **Email, Slack and Microsoft Teams** channels. **Alerts → Channels** shows configured channels and delivery records. To add or change a channel, follow **Manage in notification settings** to **Notifications & Alerts**.

## Prepare a destination

| Channel | Required configuration |
| --- | --- |
| Email | A channel name and at least one valid recipient address. Add each address to the list before saving. |
| Slack | An Incoming Webhook URL created for the target conversation. A regular Slack channel link is insufficient. |
| Microsoft Teams | An HTTPS webhook URL created through Teams Workflows. A Teams channel’s browser link cannot receive alerts. |

Creating a channel requires notification-create permission. Editing, enabling/disabling, deleting and **Send test** require notification-edit permission. Confirm that the channel is active. Treat webhook URLs as credentials; do not include them in public documentation, screenshots or support messages.

## Email: add recipients and check the actual inbox

Select **Email**, add the intended recipients and save. Customers do not enter a Resend key or SMTP server in this form; sending uses Komuta’s email infrastructure through Resend.

After **Send test**, check the actual inbox, spam/quarantine folders and your organization’s mail filtering. Success for a channel with multiple recipients may not mean success for every recipient; verify important recipients individually.

Komuta’s email delivery record represents **provider acceptance**. Resend’s `email.sent` event concerns an accepted send request; `email.delivered` concerns delivery to the recipient’s mail server. Neither establishes that the message was seen in the inbox or read. Do not assume the Komuta page exposes every provider event. [Resend event definitions](https://resend.com/docs/webhooks/event-types).

### Complete example: the Docs test email channel

1. Open the channel creation form through **Alerts → Channels → Manage in notification settings**.
2. Set the name to **Docs test** and type to **Email**. The name must contain at least two characters.
3. Enter a test address you can actually check, then press **Add** or Enter. The address must appear as a separate recipient chip; do not leave it only in the input box.
4. Optionally enter “Test destination for the documentation example” as the description. Save and confirm that the channel appears as active.
5. Use the channel’s **Send test** action. Check both the result in Komuta and the actual mailbox. Verify every required destination when there are multiple recipients.
6. Return to the [first alert form](alerts-quick-start.md) and select **Docs test**. A successful channel test does not automatically connect the rule to that channel.

![Sample email channel form: 1 channel name, 2 email type, 3 recipient added to the list.](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/alerts/en/channel-form.png)

*The real form component is rendered locally with sample data. `alerts@example.com` is illustrative; replace it with an address you can check. No channel was created and no message was sent to prepare this image.*

**If a recipient will not add:** Check that the address is valid and not already listed. After pasting a list separated by commas, semicolons or lines, count the resulting chips rather than assuming every address was accepted. Remove an incorrect chip and enter the corrected address.

## Slack: use an Incoming Webhook

Enable Incoming Webhooks for your Slack app, create a webhook for an authorized destination channel and save the generated URL in Komuta. You also need the appropriate Slack access for a private channel. [Slack setup guide](https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks/).

Verify the test in the target Slack channel. Removing the webhook, archiving the channel or changing app access can cause delivery failures; test again with a valid destination.

## Microsoft Teams: a channel link is not a webhook

1. Open **Workflows** for the target Teams channel.
2. Create a suitable webhook workflow, such as **Send webhook alerts to a channel**, and choose the team and channel.
3. Save and copy the generated webhook URL into a **Microsoft Teams** channel in Komuta.
4. Use **Send test**, then verify the Teams message and, if necessary, the workflow run history.

Komuta sends a webhook request; the form does not establish a Microsoft user session or OAuth connection. The workflow’s authentication setting must accept this call. If organizational policy prevents that, work with your Teams administrator on a suitable connection. [Microsoft webhook setup](https://support.microsoft.com/en-us/workflows/send-messages-in-teams-using-incoming-webhooks).

Keep the workflow enabled with a valid owner. Add a co-owner when appropriate: an owner leaving can affect continuity. [Microsoft’s ownership and Workflows guidance](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook).

## Route a rule to channels

Choose notification channels explicitly when creating or editing an alert. If the selection is empty, matching active channels are used; it does not mean “send to nobody.” Channel enablement and any configured severity filter also affect routing.

The event-channel matrix in **Notifications & Alerts** controls subscriptions to platform events. It is separate from an alert rule’s own channel selection. Receiving a deployment-completed message does not prove that a particular metric/log rule routes to the intended channel.

### Channel ready and rule connected: the final check

Suppose you connected the **Warning** rule `Docs sample alert` to **Docs test**. Reopen the rule: is the correct channel selected? Is the destination active in the channel list? Does any severity filter accept Warning? Then follow the controlled example’s **History** event and its delivery.

If tests arrive but actual event messages do not, inspect rule channel selection, active silences, repeat interval and failed/suppressed records before rebuilding the connection. If the test fails too, repair channel configuration first. The [decision steps](alerts-troubleshooting.md) identify the next screen for each result.

## Separate tests from delivery records

**Send test** checks a channel connection. It does not test the rule’s metrics/logs, publication, firing or resolution. If the test reports that nothing was sent, check channel enablement, settings and notification limits. Tests can affect a channel’s attempt statistics; do not expect the same History row as a normal alert event.

In **Channels**, use period, channel, status and search filters. Distinguish firing from resolved notification types. Expand collapsed identical records when needed; collapsing is a display convenience.

| Status | What it establishes |
| --- | --- |
| Pending/processing record | A record exists in the sending flow; delivery is not complete. |
| Successful/delivered record | The product recorded a successful sending attempt. For email this is provider acceptance; verify actual recipients separately. |
| Failed record | A sending attempt failed; inspect the error and destination settings. |
| Suppressed record | A limit or restriction prevented sending; it is not a delivery. |
| No record | No record matches the selected period/filters. This alone proves neither a lack of firing nor a healthy channel. |

Account for scope when comparing period-based delivery statistics with all-time channel attempts. With no attempts there is no success-rate evidence. Notification limits, repeat intervals, silences and provider errors can affect the result; do not assume immediate, guaranteed or exactly-once delivery.

Next: [Missing notifications](alerts-troubleshooting.md) · [Create your first alert](alerts-quick-start.md)
