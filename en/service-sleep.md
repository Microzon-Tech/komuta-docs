# Service Sleep and Always On

A web service on the free package is put to sleep when it gets no traffic for a while: its replica count goes to zero and it uses no resources. The first request to its public URL wakes it up on its own. The free services in your organization share a monthly pool of awake hours; when the pool runs out, free services sleep until the 1st of the month.

This page covers which services sleep and when, what visitors see while a service wakes up, waking and sleeping a service by hand, the free hours pool, the Git account that free services need, and the **Always on** add-on that keeps a service running.

---

## Which Services Sleep?

Automatic sleep only applies to the free package. Services on a paid package are never put to sleep for being idle.

| Service | Sleeps when idle? |
|---|---|
| Web service on the free package with its **Public URL** on | Yes. It is put to sleep after 30 minutes without traffic. |
| Service on the free package with its **Public URL** off, or exposed to your other services over the [Private Network (Mesh)](services-ports.md) | No. Its callers could not wake it, so it is not put to sleep automatically; but every hour it stays awake comes out of the pool. |
| Free service with the **Always on** add-on | No. It does not use the pool either. |
| Service on a paid package | No. |
| Jobs and CronJobs | No. Sleep only applies to long-running services. |

**What counts as traffic?** Only HTTP requests that reach the service's public URL from outside. Calls from your other services over the private network, outbound connections your app makes, and Komuta's health checks do not keep the service awake.

**When is a service not put to sleep?**

- A newly created or newly deployed service is left awake for a while, so its first traffic has time to arrive.
- A service is not put to sleep while a build or deployment is running.
- A service that has just woken up is not put back to sleep right away.
- If the traffic data cannot be read, the service is not put to sleep in that round. Because the check runs at intervals, a service may go to sleep a little after its 30 quiet minutes.

> **Note:** The free package runs a single replica without autoscaling. The other free package rules and prices are on the [Pricing](https://www.komuta.io/pricing) page.

---

## What Visitors See While a Service Wakes Up

The first request to a sleeping service's public URL wakes it. Waking usually takes about a minute; the exact time also depends on how fast your app starts.

**Visitors using a browser** see the **This service is Waking up** page if the service cannot answer right away: "It will be ready in a few seconds. This page refreshes on its own once it is ready." The page checks the service every few seconds and loads the requested page once the service is ready. The visitor does not need to do anything. The page is shown in Turkish when the browser language is Turkish, otherwise in English.

**API clients and other non-browser calls** are held until the service is ready, then passed on to it. If the service is not ready in time, the response is `503` with a `Retry-After` header. Clients with a short timeout (for example services that send webhooks) may get an error on the first request during a wake-up; add retries to those clients, or keep the service [Always on](#always-on-add-on).

**For services with access protection**, a sleeping service is woken only after the visitor passes the protection. Visitors who are not signed in or not on the allow-list cannot wake it. Details: [Service Access Protection](service-access-protection.md).

> **Warning:** Calls from your other services over the private network do not wake a sleeping service. A sleeping service cannot answer those calls; wake it from the console.

### When a request cannot wake the service

In some cases a request cannot wake the service. The visitor then sees an information page that does not refresh on its own. The page is shown in Turkish or English, following the visitor's language preference.

| Title the visitor sees | Reason |
|---|---|
| **This service is paused for this month** | The organization's free hours for this month are used up. |
| **This service is on hold** | The free plan is not linked to a Git account. |
| **This service is stopped** | The service was put to sleep from the console without **Allow incoming requests to wake the service**, or the platform team stopped it. |
| **This service is unavailable** | The account is suspended. The visitor is not told why. |

---

## Watching the Sleep State

**Apps list.** On the [Apps](https://console.komuta.io/services) list, sleeping services are labeled **Sleeping** and services that are waking up are labeled **Waking up**. The **Sleeping** status filter lists both. If waking fails, the service shows as **Failed**.

**Service Dashboard.** While a service sleeps, the **Sleep state** section comes forward in the status card on the [Dashboard](service-dashboard.md). Its title tells you the state:

| Title | Meaning |
|---|---|
| **App is asleep** | The service is asleep. The line below says why (see [Sleep Reasons](#sleep-reasons)). |
| **App is going to sleep** | The service is being put to sleep; it will be asleep in a few seconds. |
| **App is waking up** | The service is starting. The section updates on its own when it is ready. |
| **Sleep transition failed** | The last sleep or wake attempt did not finish. The error text is shown on the **Error:** line. |

The **Put to sleep at** line shows when the service went to sleep. If it was put to sleep for being idle, the section also shows how many requests were counted in the 30 minutes before, for example **Probe observed 0 requests in the idle window**. If you think "but it had traffic", look at this line first.

**Notifications.** To be notified when a service goes to sleep and wakes up, connect the **App put to sleep** and **App woke up** events in the **App Lifecycle** group to a channel in the event-channel matrix on **Account → Notifications & Alerts**. Setting up channels: [Channels and Delivery](notification-guide.md).

---

## Waking a Service by Hand

1. Open the service's **Dashboard**.
2. In the **Sleep state** section of the status card, select **Wake up**.
3. The button changes to **Waking up…** and an "App is waking up..." message appears. The section goes away when the service is ready.

The **Wake up** button is shown while the service sleeps and after a **Sleep transition failed**; use it to retry a failed transition too. If waking is blocked, the button shows an error message (see [Troubleshooting](#troubleshooting)).

There are other ways to wake a service:

- **Deploying a new version.** A deployment started with **Deploy** or by a push wakes a sleeping service first. Applying environment variable changes also wakes it.
- **Moving to a paid package.** After the package change, a sleeping service is woken automatically (see [Turning Sleep Off for Good](#turning-sleep-off-for-good)).

---

## Putting a Service to Sleep by Hand

You can put a service on the free package to sleep without waiting, for example to stop it using pool hours while you are not using it.

1. At the top of the service page, select **Hibernate**.
2. In the **Put {service} to sleep?** window, choose how it wakes up:
   - Default: "It stays asleep until you wake it from the console; incoming requests do not wake it." Visitors see the **This service is stopped** page.
   - With **Allow incoming requests to wake the service** checked, the service sleeps now and wakes on the first request, like a service that went to sleep for being idle.
3. Confirm with **Hibernate**. The message "Sleep request processed. Check the service status to confirm the result." appears.

A service put to sleep so that requests do not wake it is woken with **Wake up**, with a new deployment, or by moving it to a paid package.

> **Note:** The **Hibernate** button is only shown for services on the free package that are awake and do not have the **Always on** add-on. It is not shown for services in maintenance mode either.

> **Note:** If the service woke up a short while ago, or a build or deployment is running, the sleep request is not applied. That is why you should check the service status after confirming; if the service is still awake, try again a little later.

---

## Sleep Reasons

The line under the **App is asleep** title says why the service is asleep. How it wakes up depends on the reason.

| Text on the card | Reason | How does it wake up? |
|---|---|---|
| "This app was put to sleep to save energy; its replica count was scaled down to zero when there was no traffic." | No traffic for 30 minutes. | On the first request, with **Wake up**, or with a deployment. |
| "This app was put to sleep manually; its replicas stay at zero until it is woken up." | Someone in your organization used **Hibernate**. | With **Wake up** or a deployment. Also on the first request if **Allow incoming requests to wake the service** was chosen. |
| "This month's free hours are used up, so this free-plan app sleeps until the pool resets." | The organization's monthly free hours pool is used up. | When the pool resets on the 1st. To start it now, use **Always on** or a paid package. |
| "Free-plan apps need a Git account linked to the organization. This app sleeps until one is linked under Account → External Logins." | The free plan is not linked to a Git account. | When a Git account is linked, or the service moves to a paid package. |
| "This app is asleep because the account is suspended for billing. It wakes up once the account is reactivated." | The account is suspended for unpaid statements. Paid services can sleep for this reason too. | When the balance is paid and the account is active again. |
| "The platform team put this app to sleep; its replicas were scaled down to zero." | The Komuta platform team stopped the service. | When the platform team releases the hold. Open a [support ticket](support-tickets.md) if needed. |
| **In maintenance** title | The service was put into maintenance mode. | With **End maintenance**. |

### Sleep history

The console shows the current sleep reason and, for an idle sleep, the request count from the last decision. The full list of a service's sleep and wake events is not shown on a separate console page; ask your code agent connected over MCP for it (see [Managing with MCP](#managing-with-mcp)).

---

## Monthly Free Hours Pool

The free services in your organization share a total of **450 awake hours** each month. The pool belongs to the organization and follows the calendar month:

- Every minute a service on the free package stays awake comes out of the pool. A sleeping service uses no hours.
- Services with the **Always on** add-on, services on a paid package, and Jobs/CronJobs do not use the pool.
- The pool resets on the 1st of every month at 00:00 UTC. Unused hours do not carry over to the next month.
- Hours used this month by a service you delete still count against the pool.

> **Note:** The pool resets with the calendar month (the 1st, UTC). The billing cycle runs from the 6th to the 6th; these are different dates.

### Where to see the usage

| Place | What it shows |
|---|---|
| **Free hours this month** bar above the **Apps** list | Used and total hours, and hours left. Select it to open the per-service breakdown, **Other services** (including deleted services), the reset time, and when the pool runs out if usage keeps its pace (**Runs out … at this rate**). |
| Label next to each free service on the list | **… this month** (hours the service used this month), **Always on**, or **Hours used up** once the pool is used up. |
| **Free hours** section of the status card on the service Dashboard | Under the **Free hours pool** title: the organization's usage plus **This service**, **Remaining** and **Resets**. |
| **Free hours usage** section of the status card | The **This service uses a lot of free hours** notice, when the service was awake for more than three quarters of the last 24 hours, or when the pool will run out before the month ends at the current pace. |

Dates and times on these screens are shown in UTC. An **Estimated** label next to a value means part of that month's usage is based on an estimate.

> **Tip:** If an uptime monitor or a browser tab left open sends regular requests to the service, it never sleeps and quickly uses up the pool. If the service has to stay on all the time, use the [Always on](#always-on-add-on) add-on.

### Warnings

| Usage | In the console | Email |
|---|---|---|
| 80% | A "You have used 80% of this month's free hours" notice at the top of the page. It reminds you that free services sleep until the reset once the pool runs out. | "You have used 80% of this month's free hours" |
| 100% | A "Your free hours for this month are used up" notice. | "This month's free hours are used up" |

The **View usage** button on the notice opens the Apps list; the close button hides the notice for that browser session. The emails are sent once per threshold each month, to the organization admins and to the person who linked the free plan's Git account.

### When the pool runs out

When the pool runs out, the free services in the organization (except those with **Always on**) are put to sleep and stay asleep until the 1st of the month (UTC):

- Requests do not wake them; visitors see the **This service is paused for this month** page.
- **Wake up** cannot wake them.
- Deployments and service changes (such as environment variables, ports and replica count) are refused. Git pushes do not start a build.

There are two ways to start a service before the month ends:

- Turn on the [Always on](#always-on-add-on) add-on for that service. The service wakes up once the add-on is active.
- [Move the service to a paid package](#turning-sleep-off-for-good). Moving to a paid package works even when the pool is used up.

When the pool resets on the 1st, the block is lifted. A service with a public URL wakes on its first request; Komuta wakes the other free services itself. You can also start a service right away with **Wake up**.

---

## Git Account for Free Services

The free plan works with a personal Git account linked to the organization (GitHub, GitLab or Azure DevOps). In most cases Komuta links the Git account that owns the free service's repository by itself, and you do not need to do anything.

If that link cannot be made (the owner cannot be read from the repository address, or that account already uses the free plan in another organization), the console shows the **Your free services could not be tied to a Git account** notice at the top. The free services in the organization are then put to sleep, cannot be woken by requests or with **Wake up**, and cannot be deployed. Visitors see the **This service is on hold** page. Services on a paid package are not affected.

To link an account:

1. Select **Link Git account** on the notice, or open **Account → External Logins**.
2. On the **Free plan Git account** card, select **Bond with this account** next to your personal account in the **Your Git accounts** list.
3. If no suitable account is listed, confirm your GitHub account with **Link GitHub**, or use **Open integrations** to connect your GitLab or Azure DevOps account and then come back to this page. Connecting a Git account: [Integrations](integrations.md).

Once an account is linked, the block is lifted; the services are woken by requests or by Komuta.

- Only personal accounts can be used. Organization accounts and self-hosted Git servers cannot hold the free plan.
- A Git account can use the free plan in one organization only.
- Only the person who linked the account can release the link. A released account cannot be used for another organization's free plan for 30 days, and not before the end of the month. Releasing the link also ends the **Always on** add-ons in the organization.

---

## Always On Add-on

**Always on** is a paid add-on that keeps a single service on the free package awake all the time. The service does not sleep even without traffic, and it does not use the free hours pool. If you turn it on while the pool is used up, the service wakes up and keeps running.

### Price and billing

- **USD 3 per billing cycle + tax.** The price is shown on the add-on card and in the confirmation window. If the price is going to change on a future date, the card shows the new price and its start date in advance.
- Billing starts once the service is awake and running. Each billing cycle you pay only for the part of the cycle the add-on is on.
- Coupon and campaign discounts do not apply.
- Billing stops as soon as you turn the add-on off.
- The add-on counts toward your monthly spending limit. How the limit works: [Billing](billing.md).

### Requirements

The add-on is only available for web services on the free package; the card is not shown for services on a paid package, or for Jobs and CronJobs. If a requirement is missing, the card shows what to do under **To turn it on:**

| Shown on the card | What to do |
|---|---|
| **Use a single replica, no autoscaling** | The service must run a single replica with autoscaling off. See [Autoscaling](services-autoscaling.md). |
| **Add a verified default card** | Set a verified card as the default on **Account → Payment Methods**. |
| **Link GitHub to the free plan** | The organization's free plan must be linked to a Git account. See [Git Account for Free Services](#git-account-for-free-services). |
| **Pay the balance to lift the suspension** | The account is suspended. Pay the unpaid statements on **Account → Statements**. |
| **Pay the overdue invoice** | A payment is overdue. Pay it on **Account → Statements**. |
| **Raise your spending limit** | The add-on would take this month's estimated spend over your limit. Your limit is on **Account → Wallet**. |
| **Wait for the platform hold to be released** | The platform team is holding the service asleep. |

Organization admins and members with this permission can turn the add-on on and off. Without the permission, the card says "Ask an administrator in your organization to change Always on."

### Turning it on

1. On the service's **Dashboard**, switch to the **Always on** section of the status card. If the service is asleep, the **Keep my app always awake** link in the **Sleep state** section also brings you here.
2. Turn on the switch on the card.
3. In the **Turn on Always on for {service}?** window, read the price and terms, then confirm with **Turn on**.
4. The card shows the **Activating…** label: "Waking the service up. Billing starts once it is running."
5. Once the service is running, the label changes to **On** and the card shows since when the add-on has been on. On the Apps list, the service's label becomes **Always on**.

If it takes long, the card says "This is taking longer than usual. It keeps trying in the background and nothing is charged until the service is running."; select **Check again** to refresh the state. If activation does not finish, nothing is charged and the card says "The last attempt to turn it on did not finish, so nothing was charged. You can try again."

### Turning it off

1. Turn off the switch in the **Always on** section.
2. In the **Turn off Always on for {service}?** window, select **Turn off**.

Billing stops right away. The service goes back to the normal sleep cycle: it sleeps when idle and uses the pool again. If this month's free hours are used up, it goes to sleep right away.

### When does the add-on end on its own?

- When the service moves to a paid package (a service on a paid package does not sleep anyway).
- When the service is deleted.
- When the free plan's Git account link is released.

> **Warning:** **Always on** does not keep a service awake if the account is suspended or the platform team stops it. While the add-on is on, the service cannot be put to sleep by hand or put into maintenance mode; turn the add-on off first.

---

## Turning Sleep Off for Good

If a service should never sleep, move it to a paid package. A service on a paid package is not put to sleep for being idle and does not use the free hours pool.

1. On the service's **Dashboard**, select **Upgrade** on the **Plan / tier** card.
2. In the **Resource Plan** window, choose a paid package.
3. Confirm with **Apply Plan**.

If the service is asleep, it is woken automatically after the package change. If the service has the **Always on** add-on, the add-on ends. Package prices depend on CPU and memory; for example the 0.5 vCPU / 1 GB package is USD 9 per month. All prices are on the [Pricing](https://www.komuta.io/pricing) page.

> **Note:** If the monthly estimate would go over the organization's spending limit, the upgrade is refused. See [Billing](billing.md).

**Always on or a paid package?** If the free package's resources are enough and you only need the service not to sleep, **Always on** costs less. If you need more CPU or memory, several replicas, or autoscaling, move to a paid package.

---

## Managing with MCP

Your code agent connected over MCP can read a service's sleep state and its sleep and wake history, wake a service or put it to sleep, see the organization's free hours usage, and turn the **Always on** add-on on and off. Changes such as putting a service to sleep or turning on the add-on are made with your confirmation. Setup and tools: [MCP Setup](mcp-setup.md), [MCP Tools](mcp-tools.md).

---

## Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| Visitors see **This service is paused for this month** | The free hours pool is used up. | Wait for the 1st, or add **Always on** to the service, or move it to a paid package. |
| Visitors see **This service is on hold** | The free plan is not linked to a Git account. | Link a Git account on **Account → External Logins**. |
| Visitors see **This service is stopped** | The service was put to sleep by hand so that requests do not wake it. | Select **Wake up** in the console. |
| The **This service is Waking up** page does not change for a long time | The app does not start or does not pass its health check. | On the Dashboard, check **Sleep transition failed** and the **Error:** line, then the [logs](service-logs.md). |
| **Sleep transition failed** | The service was not ready in time, or the transition did not finish. | Retry with **Wake up**. If the problem is in the app, deploy a fixed version; the deployment goes ahead even if waking fails. |
| Calls from other services do not reach the sleeping service | Private network calls do not wake a sleeping service. | Start it with **Wake up**. If it has to stay on, use **Always on** or a paid package. |
| The service never sleeps and the pool runs out fast | Regular requests (an uptime monitor, an open tab), or the service's **Public URL** is off or it is exposed on the private network. | Check the **Free hours usage** notice on the status card and the service's label on the list. Stop the tool that sends the requests, or use **Always on**. |
| The service did not sleep after **Hibernate** | The service had just woken up, or a build or deployment was running. | Wait a little and try again. |
| A webhook or API client times out on the first request | The service was asleep; waking takes about a minute. | Add retries to the client, or use **Always on**. |

**Error messages when you select Wake up** (in these messages, "keep awake" is the **Always on** add-on):

- "This month's free plan awake hours are used up, so this service stays asleep until the 1st of next month (UTC). Turn on keep awake for this service or move it to a paid plan to wake it now."
- If the Git account is missing, the message says that free plan services need a linked Git account and ends with: "Link your personal Git account or move this service to a paid plan to wake it."
- "This service cannot be woken while your account is suspended for unpaid statements. Pay the outstanding balance and it wakes again."
- "A platform administrator is holding this service asleep. It cannot be woken until the hold is released."

**When you deploy or change settings while the pool is used up:** "This month's free plan awake hours are used up, so changes to this service are blocked until the 1st of next month. Turn on keep awake for this service or move it to a paid plan to make changes now."

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — The status card and the **Plan / tier** card.
- [Access & Ports](services-ports.md) — Public URL and private network settings.
- [Service Access Protection](service-access-protection.md) — How protected services are woken.
- [Autoscaling](services-autoscaling.md) — Replica count and autoscaling.
- [Billing](billing.md) — Billing cycle, statements and spending limit.
- [Integrations](integrations.md) — Connecting a Git account.
- [Channels and Delivery](notification-guide.md) — Setting up channels for sleep and wake notifications.
- [MCP Tools](mcp-tools.md) — Managing services with a code agent.
