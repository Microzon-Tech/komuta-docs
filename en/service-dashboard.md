# Service Dashboard

The dashboard is the first screen you see when you open a service. It brings the service's current status, the plan it runs on, the repo and branch it's connected to, and how traffic flows to pods together on a single page. It also works like a control panel: you make strategy, plan, and branch changes quickly from here.

![alt text](img/service-dashboard.png)

---

## Rollout Status

The card shows the service's live status and the name of the active rollout strategy. The status is live — it updates without a page refresh while a deploy is in progress.

You can change the rollout strategy from the settings icon on the card.

| Strategy | Suited for | How it works |
|---|---|---|
| **Blue-Green** | Most services | Two identical environments. Zero downtime, easy rollback; switching to live requires manual approval. |
| **Auto Promote** | Hands-off flows | Same as Blue-Green, but the switch happens automatically once the preview pods are ready. |
| **Canary** | Advanced use | Gradually shifts traffic to the new version. Promoting the new version requires manual management. |

> **Note:** A strategy change takes effect on the next deploy. It doesn't affect a rollout already in progress — that one completes with its current strategy.

![alt text](img/rollout-strategies.png)
---

## Plan / Package

The card shows the resource plan the service runs on, the cluster type, and the CPU and memory defined for the package.

When your service needs more resources, you can change the package from the **Upgrade** button on the card.

> **Tip:** If your service restarts frequently or crashes from out-of-memory errors, this card is the first place to check — the CPU and memory values here are the upper limit the service can use.

![Plan / package card and the package list in the Resource Plan window](img/resource-prices.png)

---

## Git Repository

If you want the next deploy to come from a different branch, you can change the branch from the edit icon on the card. After the change, builds and auto deploys track the new branch. If the **Deploy immediately from the new branch** toggle in the same window is on, the deploy starts right away; if it's off, the change is saved and the first push to the new branch triggers the deployment.

![Git repository card and the change branch window](img/change-branch.png)

---

## MCP ID

The MCP ID is your service's identity value on Komuta. When you give this value to your coding agent (Claude, Codex, etc.), the agent works directly on this service — checking its status, reading its logs, triggering deploys. That means you don't need to go back to the UI for simple operations.

You get the value from the copy icon on the card. If this is the first time you're connecting your agent to Komuta, the MCP ID alone isn't enough — complete the connection setup first: [MCP Setup](mcp-setup.md).

![MCP ID card — identity value and copy icon](img/mcp-id.png)

---
 
## Service Doctor

Service Doctor opens from the button at the top of the page and evaluates the service's operational status in one pass. It examines areas like deployment, runtime, networking, dependencies, reliability, and security, and rolls them up into a single verdict.
 
Alongside the verdict, the panel lists **findings that need attention** (what was detected, why it matters, and what evidence it's based on), **forward-looking risks** (not a problem yet, but a condition that's approaching one), and what needs to change for the verdict to turn healthy. Use **Re-evaluate** to run the analysis again at any time.
 
![Service Doctor panel — verdict header, scope and confidence indicators, findings list](img/service-doctor.png)
 
---

## Deploying a New Version

The **Deploy** button at the top of the page is the fastest way to release a new version of the service.

The window that opens shows the current version tag and asks you to enter a new one. From the same window, you can optionally run this deployment from a different branch instead of the service's default branch — this choice only applies to that deployment and doesn't change the service's permanent branch.

Once the deployment is triggered, you can watch its progress live from the Rollout status card and the topology.

[ADD IMAGE: "Deploy New Version" window — current version, new tag field, and branch selection]

---

## Status Card

Below the info cards, the status card shows service states such as sleep status, security scan, deploy health, and service health all in one place. This is the first place to check whether anything needs your attention — if there's an active alert or an ongoing operation, the card surfaces it.

![Status card — "Everything is fine" state and navigation points](img/smart-status-cards.png)

---

## Topology

The topology on the right side of the page is a live map of the traffic coming into your service. It shows the chain starting from the entry point through to the pods and their attached disks; the map updates as pods are added, removed, or traffic distribution changes.

Its main value is showing at a glance *where* a problem is: if the service looks unhealthy, you can see directly which pod is having trouble or which node isn't receiving traffic.

![Topology map — connections from the HTTPRoute to the pods](img/topology.png)

### Nodes on the Map

| Node | What it represents |
|---|---|
| **HTTPRoute** | The point where traffic enters the service |
| **Service** | The layer that distributes traffic to pods |
| **Preview / Canary** | The copy of the new version not yet receiving all traffic (dashed line) |
| **Pod** | A running copy of your application |
| **Storage** | The persistent disk attached to a pod, shown with its mount path and capacity |

The lines connecting the nodes carry traffic. During a Canary or Blue-Green rollout, you can watch live from these lines how traffic is split between versions.

### Accessing Pod Logs

Clicking a pod on the map opens that pod's logs — no need to go to the logs tab and search for the pod.

![Log window that opens when a pod is clicked in the topology](img/pod-logs.png)

### Restarting the Service

The **Restart** button on the panel recreates all of the service's active pods.

> **Warning:** During a restart, live traffic may see brief errors until the new pods are ready. Avoid using it during peak hours.

---

## Related Documents

- [Deploy Strategies](deployment-strategies.md) — How Blue-Green, Canary, and Auto Promote work.
- [Service Management](service-guide.md) — Service tabs other than the dashboard.
- [MCP Setup](mcp-setup.md) — Connecting your coding agent to Komuta.
- [Monitoring and Logs](monitoring-logs.md) — Log search, metric history, and alerts.
