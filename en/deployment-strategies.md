# Deployment Strategies

A deployment strategy decides how a new version of a service is moved onto live traffic: all at once as soon as it is ready, only after you approve it, or step by step. Komuta has three strategies: **Blue-Green**, **Auto Promote** and **Canary**.

You choose the strategy for each service separately. New services you create in the console start with **Auto Promote**; there is no organization-wide setting for a default strategy. Strategies are for long-running services; jobs and cron jobs don't use one.

This page explains how each strategy rolls out, the buttons you can use during a rollout, and what Komuta does when a new version runs into trouble.

---

## Strategies at a Glance

| Strategy | How the new version goes live | What you need to do | Preview address |
|---|---|---|---|
| **Blue-Green** | The new version starts next to the old one; when you approve, all traffic switches to it at once. | Approve with **Promote** | Yes |
| **Auto Promote** | Same as Blue-Green, but the switch happens by itself once all pods of the new version are ready. | Nothing | No |
| **Canary** | Traffic moves to the new version in steps: 20%, 40%, 60%, 100%. | Let it continue at two steps, or promote it in one go | No |

With all three strategies the old version keeps serving traffic while the new one is prepared, so for a while both versions run side by side. A pod counts as "ready" when it passes the service's **Readiness** probe. You set this probe up on the [Health Probes](services-health-probes.md) page.

> **Note:** Komuta doesn't offer a classic "rolling update" strategy that replaces pods one by one. These three are the only options.

The **Rollout status** card on the **Overview** page and the **Rollout Console** write Blue-Green as **Blue/Green**.

---

## Blue-Green

1. The new version starts in a separate preview environment next to the live version. Live traffic stays on the old version.
2. When the new version's pods are ready, the rollout waits for your approval. The **Rollout Console** on the **Overview** page shows this stage as **Ready** or **Paused**, along with "Preview is ready. Promote to make it active." and a **Promote** button.
3. Try the new version on the preview address.
4. Press **Promote**. All traffic switches to the new version at once.
5. The old version's pods keep running for about a minute so in-flight requests can finish, then they are removed.

The new version doesn't go live until you approve it; the rollout waits at this point. A rollout that is waiting for your approval is not counted as stuck by the [Rollout Watchdog](#rollout-watchdog).

> **Note:** The approval step isn't only for code changes. When you apply environment variables with **Apply Changes**, a new version is created too, and it waits for your approval before going live.

**Preview address:** If the service has its public address turned on, Komuta creates a separate preview address for Blue-Green services. It is the service address with `-preview` added to its first part:

| | Address |
|---|---|
| Service address | `https://panel-3a2331e6.edge-1.komuta.app` |
| Preview address | `https://panel-3a2331e6-preview.edge-1.komuta.app` |

The console doesn't show the preview address separately; you work it out from the service address as above. A preview address is only created for the Komuta address; your custom domains have no preview counterpart. If you have turned on [access protection](service-access-protection.md) for the service, the preview address is protected by the same rules.

If the new version's pods are unhealthy, Komuta refuses to promote it, so the faulty version never receives traffic. The console then tells you the preview has unhealthy pods and suggests cleaning it up with **Abort**.

---

## Auto Promote

Auto Promote is Blue-Green without the approval step:

1. The new version starts next to the live version.
2. Once all pods of the new version pass the **Readiness** probe and stay ready for at least 10 seconds, traffic switches to the new version by itself.
3. The old version's pods are removed about a minute later.

Pods being ready is the only condition for the switch. No threshold such as error rate or response time is defined or checked. Your **Readiness** probe should therefore show that the application can really serve requests; a probe that only shows the process is up can let a version that isn't ready go live.

If the new version's pods never become ready, the switch doesn't happen and the old version keeps serving traffic. See [If a New Version Fails](#if-a-new-version-fails) for what happens next.

No preview address is created for Auto Promote.

---

## Canary

Canary moves traffic to the new version in fixed steps. The steps are the same for every Canary service and can't be changed:

| Step | New version's share of traffic | What happens next |
|---|---|---|
| 1 | 20% | Waits until you continue. |
| 2 | 40% | Waits 5 minutes, then continues by itself. |
| 3 | 60% | Waits until you continue. |
| 4 | 100% | The new version takes all traffic and the rollout completes. |

For a step to advance, the new version's pods must be ready and stay ready for at least 10 seconds.

- **Resume** moves the rollout to the next step. The button shows the next traffic share, for example `40%`.
- **Promote** skips all remaining steps and gives the new version all traffic.
- **Abort** sends all traffic back to the old version.

While the rollout runs, the **Rollout Console** shows which step you are on and the new version's share of traffic. The **Deployment → History** page also shows **Canary progression** and the share of each version (**Stable** and **Canary**).

The steps apply to new code versions. When you apply environment variables with **Apply Changes**, Komuta puts the change live directly without going through the steps.

Canary has no separate preview address; the new version receives part of the traffic that comes to the service's normal address. If the new version is in an error state, Komuta refuses to promote it in one go; you first need to go back with **Abort**.

> **Warning:** Canary weights and the Blue-Green switch apply to traffic that arrives at the service's public addresses (the Komuta address and the domains you have added). Requests that come through Komuta API Gateway may not follow these weights and may also reach pods of a new version that hasn't been promoted yet. The same may apply to requests that arrive over the private network through **Connections**. If you publish your service behind [API Gateway](api-gateway.md), don't rely on gateway traffic moving to the new version gradually.

---

## Choosing and Changing the Strategy

1. On the service's **Overview** page, click the **Change strategy** icon on the **Rollout status** card. You can also open the same window from **Change strategy** in the **Current version** section of the **Deployment → History** page.
2. In the **Change rollout strategy** window, choose **Blue-Green**, **Auto Promote** or **Canary**. The service's current strategy is marked **Current**.
3. Click **Apply strategy**.

You see "Rollout strategy updated. Will apply on the next deploy." The new strategy is used for the service's next deployment. A rollout that is already in progress is not affected; it finishes with its current strategy.

You need edit permission on the service to change its strategy. Without it, the icon isn't shown.

If you create the service with a stack, choose the strategy in the manifest with the service's `rollout.strategy` field: `auto-promote`, `blue-green` or `canary`. If the field is left out, `auto-promote` is used. See the [Stack Manifest Reference](stack-manifest-reference.md) for details.

---

## Rollout Console and Actions

While a rollout is in progress, the **Rollout Console** appears in the status card on the **Overview** page. It shows the strategy and how far the rollout has got:

| Strategy | Stages shown in the console |
|---|---|
| **Blue-Green** | **Creating** → **Ready** (or **Paused**) → **Promoting** → **Active** |
| **Auto Promote** | **Creating** → **Ready** → **Active** |
| **Canary** | A traffic bar that moves from 0% to 100% in steps (20%, 40%, 60%, 100%) |

The same actions are available in two places: the **Rollout Console** on the **Overview** page and the action bar on the **Deployment → History** page:

| Rollout Console | History page | What it does | When to use it |
|---|---|---|---|
| **Promote** | **Promote** | Gives the new version all traffic. For Blue-Green it promotes the preview; for Canary it skips the remaining steps. | While a Blue-Green or Canary rollout is in progress. Not shown in the console for Auto Promote, where the switch happens by itself. |
| **Resume** | **Resume** | Continues a paused rollout from where it stopped. For Canary it moves to the next step. | While the rollout is paused. In the console it only appears for Canary. |
| — | **Pause** | Holds the rollout where it is. | While a rollout is in progress. Only on the History page. |
| **Abort** | **Abort** | Cancels the rollout and removes the new version's pods. Traffic stays on, or goes back to, the old version. | Before the rollout completes. |
| — | **Retry** | Starts an aborted rollout again with the same version. | Only when there is an aborted rollout. |
| **Restart** | **Restart** | Recreates the pods of the running version. The version doesn't change. | In the console while a rollout is in progress; on the History page at any time. |

- The **Rollout Console** buttons are only shown to users with edit permission on the service. **Abort** and **Restart** ask for confirmation first: "Abort rollout?" and "Restart app pods?".
- **Retry** doesn't change the version. If the rollout was aborted because of a bug in the new version, you get the same result; fix the bug and deploy a new version first.
- During a restart, live traffic may see brief errors until the new pods are ready.

> **Warning:** The action bar on the **Deployment → History** page doesn't ask for confirmation; **Abort** and **Restart** run as soon as you click them.

To go back to an earlier version, use the **Rollback** action on the deployment records of the **Deployment → History** page. A rollback deploys the version you choose again and puts it live without waiting for approval; a rollout in progress is aborted. See [Deployment History](service-deployment-history.md) for details.

You can also do all of these actions (pause, resume, promote, abort, retry, rollback, restart) from your coding agent through Komuta MCP. See [Komuta MCP](mcp-tools.md) and [Installing the Komuta MCP Server](mcp-setup.md).

---

## If a New Version Fails

Komuta never rolls a version that has gone live back to an earlier version on its own. If the new version runs into trouble before it goes live, the old version keeps serving traffic.

| Situation | What happens | What you should do |
|---|---|---|
| With Blue-Green or Auto Promote, the new version's pods don't become ready | Traffic doesn't switch to the new version; the old version keeps running. Komuta refuses to promote an unhealthy preview. | Fix the bug and deploy a new version, or clean up the preview with **Abort**. |
| With Canary, the new version is in an error state | Komuta refuses to promote the new version in one go. | Send traffic back to the old version with **Abort**. |
| The new version's pods keep crashing while the old version is healthy | [Crash protection](#crash-protection) aborts the new version; the old version stays live. | Fix the bug and deploy again. |
| The rollout doesn't finish within the timeout | The [Rollout Watchdog](#rollout-watchdog) sends a notification and, depending on your setting, may abort the rollout. | Open the service in the notification and look at the rollout. |
| A version that has gone live keeps crashing and none of the service's pods is ready | [Crash protection](#crash-protection) scales the service to zero replicas. | Fix the bug and deploy a new version, or go back to an earlier version with **Rollback**. |

---

## Rollout Watchdog

The Rollout Watchdog is an organization setting that decides what Komuta does with deployments that get stuck without finishing. It applies to every service in the organization.

You reach it from **Account → Organization → Rollout Watchdog**: `https://console.komuta.io/account/rollout-watchdog`

| Field | What it sets |
|---|---|
| **Custom policy** | While off, the platform default applies: "Off = platform default (notify only after 10 minutes)." Turn it on to set the two fields below yourself. |
| **Stuck timeout (seconds)** | A value between 60 and 86400 seconds (24 hours). A rollout that hasn't finished within this time is considered stuck. |
| **Remediation action** | **Notify only**: the deployment is marked as timed out and a notification is sent; the service isn't touched. **Abort rollout**: the stuck rollout is aborted automatically and the old version keeps serving traffic; for Canary, all traffic goes back to the old version. |

Save with **Save Changes**.

The watchdog follows these rules:

- A stuck rollout is one that has neither finished nor is waiting for your approval. Waiting for Blue-Green approval, or a Canary waiting at a step, doesn't count as stuck.
- One notification is sent for each stuck deployment. In the event list on the **Notifications & Alerts** page this event is called **Deployment stuck**; see [Channels and Delivery](notification-guide.md).
- An automatic abort only happens while an old version is still running. On a service's first deployment there is no version to go back to, so Komuta doesn't abort; it only notifies.
- The same deployment is aborted automatically at most once.

If the new version can't become ready for a long time, Komuta treats the deployment as failed and applies the same setting. There is one difference in this case: if this setting has never been saved in your organization and an old version is still running, Komuta aborts the new version automatically. If **Custom policy** was saved as off, only a notification is sent.

> **Note:** You need permission to view the Rollout Watchdog to see the page. Without permission to change it, the page opens read-only and says "Read-only — you need the Manage Rollout Remediation permission to change this policy."

---

## Crash Protection

When a service's application container keeps crashing and restarting, Komuta steps in after a certain number of restarts in a row. Only crashes of your application's own container count; failures in preparation steps that Komuta adds to the pod don't trigger this protection.

Komuta tells two situations apart:

| Situation | What Komuta does | What the status card shows |
|---|---|---|
| The new version's pods keep crashing while the old version is healthy | Aborts only the new version. The old version keeps serving traffic. | **New deployment failed; stable version kept online** |
| None of the service's pods is ready and the pods keep crashing | Scales the service to zero replicas. The service doesn't serve requests until a fixed deployment is made. | **Service automatically stopped after repeated restarts** |

This card appears in the status card on the **Overview** page and contains:

- the name of the crashing pod, the error reason and the number of restarts,
- **Detected cause** and **Last crash log**,
- an **Open logs before the stop** link that opens the logs from before the stop,
- what you need to do and when the protection kicked in.

A notification is also sent when the protection kicks in. In the event list on the **Notifications & Alerts** page it is called **Service stopped after repeated crashes**. The event is also recorded in the **Restart history** in the status card.

**To bring the service back:**

1. Find the cause in the **Last crash log**, or in the logs opened with **Open logs before the stop**.
2. Fix the bug and commit the change.
3. Deploy a new version of the service.

When the new version is deployed, the protection is lifted by itself and "Crash protection lifted — a new version was deployed" appears in the **Restart history**. If the new version keeps crashing too, the protection kicks in again.

> **Warning:** Retrying the same version is not a reliable way to lift the protection. If the bug is in the version itself, the same version crashes again and the protection kicks in again; redeploying an unchanged version can also leave the service stopped. To bring the service back, fix the bug and deploy a new version.

---

## Related Documents

- [Deployment History](service-deployment-history.md) — Released versions, rollout actions and rollback.
- [Service Dashboard](service-dashboard.md) — The rollout status card, the status card and the topology.
- [Health Probes](services-health-probes.md) — The probes that decide when a new version counts as "ready".
- [Pipelines](service-pipeline-guide.md) — The build and release process that produces a new version.
- [API Gateway](api-gateway.md) — Services behind a gateway.
- [Channels and Delivery](notification-guide.md) — Where stuck-deployment and crash-protection notifications are sent.
- [Komuta MCP](mcp-tools.md) — Running rollout actions from your coding agent.
