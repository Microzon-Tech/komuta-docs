# Notification Settings

DevOpsZon's notification system offers multi-channel support to keep you informed about alerts, pipeline statuses, and important events. This guide explains how to configure your notification channels.

---

## Notification System Overview

Notifications in DevOpsZon are grouped into three categories:

| Category | Examples |
|----------|----------|
| **Alert Notifications** | Notifications sent when an alert rule is triggered |
| **Pipeline Notifications** | Build success/failure statuses |
| **System Notifications** | Billing, balance, maintenance, and platform announcements |

---

## Notification Channels

Go to the **Notifications** page from the left menu to configure your notification channels.

### Email

| Parameter | Description |
|-----------|----------|
| **Recipient addresses** | The email addresses the notification will be sent to |

Email notifications are sent to your account email address by default. You can add additional recipients.

### Slack

An Incoming Webhook URL is required for Slack integration:

1. Create an Incoming Webhook in your Slack workspace
2. Enter the webhook URL on the **Notifications** page
3. Specify the channel the notification will be sent to
4. Verify by sending a test notification

### Telegram

| Parameter | Description |
|-----------|----------|
| **Bot Token** | The bot token obtained from Telegram BotFather |
| **Chat ID** | The chat or group ID the notification will be sent to |

### Microsoft Teams

| Parameter | Description |
|-----------|----------|
| **Webhook URL** | The Incoming Webhook URL for the Teams channel |

### PagerDuty

| Parameter | Description |
|-----------|----------|
| **Integration Key** | The integration key belonging to the PagerDuty service |

The PagerDuty integration creates an incident based on alert severity and includes it in your incident management process.

### SMS

| Parameter | Description |
|-----------|----------|
| **Phone number** | The phone number the notification will be sent to |

### WhatsApp

| Parameter | Description |
|-----------|----------|
| **Business API** | WhatsApp Business API configuration |

### Webhook (Custom)

Define a custom webhook to send notifications to your own systems:

| Parameter | Description |
|-----------|----------|
| **URL** | The HTTP endpoint the notification will be POSTed to |
| **Headers** | Optional HTTP headers (e.g., Authorization) |

---

## Channel Configuration

### Adding a New Channel

1. Go to the **Notifications** page
2. Click the **Yeni Kanal Ekle** button
3. Select the channel type
4. Enter the required information
5. Send a test notification using the **Test** button
6. Save the channel using the **Kaydet** button

### Linking a Channel to an Alert Rule

Notification channels are associated with alert rules:

1. Go to the **Alert Management** page
2. Create or edit an alert rule
3. In the **Notification Channels** section, select which channels the notification will be sent to
4. You can select multiple channels (e.g., both Slack and email)

---

## In-Panel Notifications

In addition to channel notifications, you receive real-time in-panel notifications in the DevOpsZon Console:

- View new notifications via the notification icon in the top right corner
- Alert, pipeline, and system notifications are listed in chronological order
- You can click a notification to be taken to the relevant page

---

## Notification Flow

The notification process when an alert is triggered:

```
Alert Triggered → Alertmanager → DevOpsZon API → Notification Queue → Delivery to Channel
                                       ↓
                               Real-Time Notification
                               in Panel (SignalR)
```

1. A rule threshold is exceeded in Prometheus or Loki
2. Alertmanager groups the alert and sends it to the DevOpsZon webhook
3. The DevOpsZon API processes the alert and finds the associated notification channels
4. A notification is queued for each channel (RabbitMQ)
5. The queue consumer delivers the notification to the relevant channel
6. At the same time, a real-time update is made on the panel via SignalR

### Retry Mechanism

If a notification delivery fails:
- It is automatically retried
- After multiple failed attempts, it is moved to the Dead Letter Queue (DLQ)
- Messages in the DLQ can be manually reprocessed

---

## Tips

- **Multiple channels:** Define multiple notification channels for critical alerts (e.g., Slack + Email + PagerDuty)
- **Severity-based:** Route Warning alerts to Slack and Critical alerts to PagerDuty
- **Test:** Verify by sending a test notification after adding each new channel
- **Notification fatigue:** Too many notifications cause important ones to be overlooked; set up notifications only for events that require action
