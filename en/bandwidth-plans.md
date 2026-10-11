# Bandwidth Plans

Every service has a bandwidth plan. The plan sets how fast each of the service's pods can send data out. Inbound traffic is unlimited and not billed.

New services start on the free **Starter** plan. If your service serves large files, answers many users, or downloads feel slow, upgrade the plan from the **Bandwidth** tab on the service page.

---

## What a Plan Limits

| Traffic | Speed limit |
|---|---|
| **Outbound (egress)** — responses you return to users, downloads and other data your service sends out | The plan's egress speed, applied to each pod separately |
| **Inbound** — requests and uploads coming into your service | None; inbound traffic is not billed |

The speed limit is per pod. If your service runs 3 pods, each pod can send up to the plan's speed, so total capacity grows with the pod count.

The plan does not limit the number of requests your service receives. Only the speed of outgoing data is limited, regardless of how many requests arrive per second.

---

## Plans

| Plan | Monthly fee | Egress speed (per pod) |
|---|---|---|
| **Starter** (default) | Free | 5 Mbps |
| **Growth** | USD 4.99 | 25 Mbps |
| **Business** | USD 14.99 | 100 Mbps |
| **Enterprise** | USD 49.99 | 500 Mbps |
| **Scale** | USD 149.99 | 2 Gbps |

Prices are in USD and exclude tax. You can see the current values on the plan cards in the **Bandwidth** tab and on the [Pricing](https://www.komuta.io/pricing) page.

Komuta assigns the **Starter** plan to every new service automatically. For services you create with a Stack, you can choose the starting plan with the `bandwidth.plan` field in the manifest (for example `Growth`); see the [Stack Manifest Reference](stack-manifest-reference.md).

---

## The Bandwidth Tab

Open the service from the **Apps** page and select **Bandwidth** in the left menu (`https://console.komuta.io/services/<service-id>/bandwidth`). The tab has four parts.

### Plan summary

The strip at the top shows the service's current state:

- **Current plan** — the plan name, its monthly fee and an **Active** badge.
- **Egress (per pod)** — the speed limit the plan applies to each pod.

### Bandwidth history

The chart plots the service's **Inbound** and **Outbound** traffic over time.

- Choose a time range of **5m**, **15m** or **1h**. While the tab is open, the chart adds a new sample every few seconds.
- **Pause** freezes the chart and **Resume** continues the live stream. The button next to them refreshes the data immediately.
- The tiles at the top show the latest sample. The **Outbound** tile relates the rate to the plan limit; the **Inbound** tile reads **Not limited or billed**.
- The summaries below are **Avg inbound**, **Avg outbound**, **Peak inbound**, **Peak outbound**, **30-day inbound** and **30-day outbound**.

### Live utilization

The gauge in the side panel shows outbound traffic as a percentage of the plan limit, with **of …bps limit** below it. The value is not capped and can show more than 100%. Values are one-minute averages; queueing can make them differ from the speed your client measures.

When sending is being slowed to the plan speed, a **Pacing … · … ms average wait** line appears under the gauge. Inbound traffic is shown on a separate row with an **Unlimited** badge.

### Available plans

All plans are listed as cards. Each card shows the monthly fee and the **Egress (per pod)** speed. The default plan carries a **Default** badge; the button on the plan your service uses reads **Current plan**.

### States in the chart and gauges

| What you see | What it means |
|---|---|
| **Live** | Data is current and samples are arriving. |
| **Paused** | You stopped the chart. Continue with **Resume**. |
| **Collecting samples…** | There are no samples in the selected range yet. The chart fills as traffic arrives. |
| **No traffic recorded yet** | There is no live sample for the service. |
| **Unavailable** | Traffic monitoring is not reporting a fresh sample. The screen reads **Traffic monitoring is not reporting a fresh sample. Usage is unknown, not zero.** This does not affect the limit or billing. |

---

## Changing the Plan

1. In the **Bandwidth** tab, scroll down to **Available plans**.
2. Click the button on the card of the plan you want. It reads **Upgrade** for faster plans and **Downgrade** for slower ones.
3. Review the **Confirm plan change** dialog:
   - the **From** and **To** plans with their monthly fees,
   - the new plan's **Egress (per pod)** speed,
   - the fee difference (**Your monthly bill increases by $…** or **decreases by**),
   - on a downgrade, the warning **Lower limits may cause traffic throttling if your app is already near capacity.**
4. Confirm with **Confirm upgrade**, **Confirm downgrade** or **Confirm change**. Click **Cancel** to back out.

After you confirm, the notification **Plan "Growth" saved. Cluster synchronization is in progress.** appears and the plan summary shows the new plan.

You need permission to edit the service to change its plan.

### When the change takes effect

The plan is saved immediately. The service is then redeployed with the same version: no new build runs, and the pods are recreated with the new speed limit. The new limit applies once the new pods take live traffic.

| Deployment strategy | New limit |
|---|---|
| **Auto Promote**, **Canary** | The new pods go live automatically once they are ready. |
| **Blue-Green** | The new pods wait in preview. The live pods keep the old limit until you approve the switch with **Promote** on the **History** page. |

> **Warning:** Billing switches to the new plan from the hour the plan is saved. On Blue-Green services, remember to promote; otherwise you pay for the new plan while running at the old speed.

See [Deployment History](service-deployment-history.md) for strategies and the **Promote** action.

### When the plan cannot be changed

| Situation | Message you see |
|---|---|
| The service has never been deployed | **This service has no manifest Git repository URL; cluster sync cannot run.** Try again after the first deployment completes. |
| The upgrade would exceed your monthly spending limit | **This operation would bring your estimated monthly spend to … USD, above your current spending cap of … USD. Your cap needs to be at least … USD higher to continue. …** |
| An invoice could not be collected | **Your account is in a grace period because an invoice could not be collected. Operations that increase your monthly spend are paused until the outstanding balance is settled. …** |
| The account is suspended | **Your account is suspended for outstanding invoices. Settle the balance from Account → Billing to resume operations.** |

The spending limit is checked only for changes that raise the monthly fee. Downgrading or returning to **Starter** is never blocked by it.

---

## When a Service Hits Its Speed Limit

When your service tries to send faster than the plan allows, Komuta does not reject requests, return errors or drop connections. The data a pod sends is slowed down to the plan speed. Your users notice this as longer downloads and large responses arriving more slowly.

Komuta automatically creates an **Egress Bandwidth Quota Exceeded** alert rule for every service. It fires when the busiest pod's outbound traffic stays above 90% of the plan speed for 5 minutes. The rule watches the plan's speed limit. See [Rules and Publication](alerts-rules.md) for rules Komuta creates automatically.

If you hit the limit often, you have two options:

- **Upgrade the plan** — each pod sends faster.
- **Run more pods** — the limit is per pod, so total capacity grows. [Autoscaling](services-autoscaling.md) adds pods when load rises.

---

## Billing

**Plan fee.** The monthly fee is charged by the hour: the hourly rate is 1/720 of the monthly fee. The fee accrues for as long as the plan is assigned to the service, even with no traffic; if you change plans mid-month, each plan is charged for the hours it was in place. **Starter** is free.

**Inbound traffic.** Inbound traffic is never billed on any plan.

**Billing period.** A period runs from the 6th of one month to the 6th of the next (UTC); statements are issued on the 6th. The plan fee appears on the statement as a line for the service.

**Where you see it.**

- The service's **Invoice** tab — the service's items for the current period.
- **Account → Current Spend** — all of the organization's spending in the current period (visible to organization admins).

**Spending limit.** Plan fees count toward the organization's monthly estimate. If an upgrade would push the estimate above your spending limit, the plan is not changed. The limit is checked only at the moment of the change; running services are not stopped.

---

## API Gateway Bandwidth

API Gateways have a bandwidth plan too. It is chosen from the same plans as services and works the same way: it limits the outbound traffic of each gateway pod, and inbound traffic is unlimited. Responses the gateway returns to clients are outbound traffic, so the gateway's plan sets how fast your APIs can respond. The services behind it keep their own plans as well.

A new gateway starts on the **Starter** plan. To change it:

1. Open the gateway from the **API Gateways** page.
2. Go to the **Bandwidth** tab. The plan summary, the **Live utilization** panel and the plan cards are the same as on services. The **Live Traffic** chart only plots samples collected while the tab is open; it does not load past data.
3. Choose a plan and confirm.

The note at the bottom of the tab reads **Changing the plan re-syncs the gateway's manifests. Traffic is not interrupted.** The gateway's bandwidth plan is billed separately from the API Gateway's own monthly fee. For setting up and configuring the gateway, see [API Gateway](api-gateway.md).

---

## Related Documents

- [Autoscaling](services-autoscaling.md) — Growing total egress capacity by adding pods under load.
- [Deployment History](service-deployment-history.md) — Deployment strategies and the **Promote** action.
- [Billing](billing.md) — Billing period, statements and the spending limit.
- [API Gateway](api-gateway.md) — Setting up and configuring the managed API Gateway.
- [Rules and Publication](alerts-rules.md) — Rules Komuta creates automatically.
- [Stack Manifest Reference](stack-manifest-reference.md) — The `bandwidth.plan` field.
- [Pricing](https://www.komuta.io/pricing) — Current plan prices.
