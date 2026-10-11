# Troubleshooting and FAQ

This page collects the problems people run into most often on Komuta. Each entry starts with what you see (with the console message where there is one), then the likely cause, then what to do. Where more detail helps, the entry links to the page that covers it.

If the steps here do not solve the problem, go to **Getting Help**.

| What you see | Section |
|---|---|
| A pipeline **Failed**, or a build does not start | **Build Fails** |
| The deploy finished but the address does not open, returns an error code or responds slowly | **Deployed, but the Service Does Not Open** |
| The service is asleep, slow to open or will not wake | **Service Is Asleep or Will Not Wake** |
| You get an error when creating something | **I Can't Create Something** |
| A repo is missing or the Git connection fails | **Git Connection** |
| A domain stays in **Provisioning** or **Action Required** | **Custom Domain** |
| The service cannot connect to its database | **Database Connection** |
| The logs, metrics or traces screen is empty | **Logs, Metrics and Traces Are Empty** |

---

## Build Fails

### The pipeline shows "Failed"

**What you see:** On the service's **Pipelines** tab the run is **Failed**.

**What to do:**

1. Open the run. The **Why did it fail?** section shows the error lines of the failing task; if no line was marked as an error, it lists the last lines.
2. If the automatic repair record says "The application itself failed to compile — a Dockerfile change cannot fix that.", the error is in your code; fix it and push.
3. To have the error explained, use the **Resolve with AI** menu, or **Fix with MCP** if Komuta MCP is set up.
4. **Retry** builds the latest commit of the branch. If the error is still in that commit, push the fix first.

Details: [Pipelines](service-pipeline-guide.md).

### The Dockerfile pull request is not merged

**What you see:** On the service dashboard, **Action required: merge the Dockerfile pull request**. In the deploy dialog: "An AI Dockerfile pull request is open and not yet merged. Merge it into your branch first, or this deploy may fail."

**Cause:** The repo had no Dockerfile and AI generated one, or AI repaired a broken Dockerfile. The change is opened as a pull request on a separate branch. Until it is merged into the service's branch, later builds run on the old state and fail.

**What to do:** Use **Open PR** on the notice, review the pull request, merge it into the service's branch, then deploy again. Pressing **Deploy** a second time in the deploy dialog skips the warning, but the build will most likely fail again.

### The build is waiting in the queue, or cannot be queued

**What you see:** A **Queued #N** chip in the service list; or, when you start a build: "Your plan's build queue is full. Wait for queued builds to start or cancel some of them, or upgrade your plan for a larger queue."

**Cause:** The same service's previous build is still running, your account tier's concurrent build limit is full, or build capacity is busy at the moment. Every queued build shows why it is waiting.

**What to do:** Wait, or cancel builds you do not need. Concurrent build and queue limits per tier are on the [Build Queue](build-queue.md) page; your account tier is shown under **Account → Wallet**.

> **Note:** Builds do not stop when your build minutes run out. Every billing cycle includes 2,000 minutes; each minute beyond that is billed at USD 0.008 on that cycle's statement. Time spent in the queue is not counted. You can see your usage on the **Build minutes** card under **Account → Current Spend**.

---

## Deployed, but the Service Does Not Open

Start with the status card on the service dashboard; its title tells you what kind of problem it is.

| Card title | What it means | What to do |
|---|---|---|
| **Deployment in progress** | The new version is still starting. | Wait for it to finish; the card updates on its own. |
| **Published, but the URL is not responding** | The pods are ready, but the public address returns a 5xx error or does not answer at all. | See **Wrong port** and **Health probes fail**. |
| **Container is not starting** | The app crashes while starting. | See **The app crashes on start**. |
| **New deployment failed; stable version kept online** or **Service automatically stopped after repeated restarts** | Restart protection stepped in. | See **Restart protection**. |
| **Deployment sync failed: changes did not reach the cluster** | The latest change could not be applied. | Try **Re-sync**; if it persists, fix the cause based on the message on the card and deploy again. |
| **App is asleep** | The service is asleep. | See **Service Is Asleep or Will Not Wake**. |
| **In maintenance** | Maintenance mode is on; visitors see a maintenance page (HTTP 503). | Press **End maintenance** when you are ready. |

### Wrong port

**What you see:** **Published, but the URL is not responding**. If Komuta can tell from the app's logs that it listens on another port, the card shows a line like `expected port 8080, app is listening on 3000 — …`.

**Cause:** Traffic is sent to the **Target** value of the service's primary port. If the app listens on a different port, the pods look ready but requests never reach the app.

**What to do:**

- Make the app read the `PORT` environment variable. Komuta sets it to the **Target** value of the primary port; if you defined `PORT` yourself, your value is left alone.
- Or, on the service's **Access & ports** page, set the primary port's **Target** to the port the app actually listens on, and deploy again.
- An app that tries to listen on a port below 1024 (such as 80) cannot start; the card shows the hint "Container is binding to a privileged port (<1024)". Move the app to port 1024 or above (such as 8080), or grant the permission with **Open capability settings** on the hint.
- If the service does not need a public address (a background worker, for example), you can turn the address off with **Manage public URL** on the card.

Komuta's address check counts only 5xx responses and no response at all as failures; an app that returns `401`, `403` or `404` is considered up.

Details: [Access & Ports](services-ports.md), [Environment Variables](services-environment-variables.md).

### The app crashes on start

**What you see:** **Container is not starting**. Below the card are the last log lines of the previous container and, for known cases, a hint.

The cause of every restart is kept in the **Restart and crash history** window. Open it from the "N container restarts in the last 7 days" card on the service dashboard or from the **Logs** page; **Open the logs around this moment** takes you to the logs of that moment.

| Sign in the console | Cause | What to do |
|---|---|---|
| "Container was killed for exceeding its memory limit" | The app used more memory than its package allows. | Reduce memory use, or move to a larger package with **Change resource plan**. |
| "The application exited on its own (exit code 0); a service process must keep running." | The main process finished and exited. | Keep the main process in the foreground; do not start it in the background and exit. |
| "The application exited with an error (exit code N)." | The app failed on start. | Fix the error in the last log lines. A missing environment variable or secret is a common cause. |
| "Container is binding to a privileged port (<1024)" | Privileged port. | Follow the steps under **Wrong port**. |

> **Note:** After you add or change an environment variable, the value does not reach the running service until you press **Apply Changes**.

### Restart protection

**What you see:** When a pod restarts again and again, Komuta stops the loop and shows one of two cards:

- **New deployment failed; stable version kept online** — Only the new version is pulled back; the previous healthy version keeps serving traffic.
- **Service automatically stopped after repeated restarts** — The service is scaled to zero replicas and does not answer requests.

The card shows the **Detected cause**, the **Last crash log** and an **Open logs before the stop** link.

**What to do:** Fix the detected error, commit the change and deploy the service again. A new deployment that starts healthy lifts the protection automatically.

> **Warning:** Retrying the same version does not bring the service back. The protection lifts only when a new deployment with a real change (code or configuration) starts healthy.

### Health probes fail

**What you see:** A pod that fails its readiness probe gets no traffic. A container that fails its liveness probe is restarted, and the crash history says "The container stopped answering its health check, so Kubernetes restarted it." Repeated restarts can trigger restart protection.

**Common causes and fixes:**

- **The probe path or port is wrong.** A newly enabled probe checks the `/health` path on port 80 by default. Change the **Path** and **Port** fields to match your app.
- **The app starts slowly.** Add a **Startup** probe, or increase **Initial delay (s)**.
- **The app becomes too slow to answer.** As the console hint puts it, a blocked thread or a slow dependency can cause this. Review the timeout and the failure threshold.

Details: [Health Probes](services-health-probes.md).

### The address returns 502, 503 or 504

| Code | Common cause | What to do |
|---|---|---|
| `503` | No ready pod to take the traffic: the new version did not start, restart protection stopped the service, or the readiness probe is failing. Maintenance mode and the pages of a service that cannot be woken also return `503`. | Find the cause on the status card of the service dashboard. |
| `502` | The app closed the connection or could not give a valid response. | Check the service logs for the same time. |
| `504` | The app did not answer in time. | Look at slow requests and dependencies (database, external APIs). |

> **Note:** For `502` and `504` responses, the body your app returned may not reach the caller; the caller sees a short plain-text body such as `error code: 502` or `error code: 504` instead. Look for the error details in the service logs, not in the response body.

### Responses or downloads are slow

**Cause:** The service's bandwidth plan puts a speed limit on each pod's outbound (egress) traffic. When a pod reaches the limit, responses slow down; large files and heavy traffic are where this shows first. Inbound traffic is not limited.

**What to do:** On the service's **Bandwidth** page, check whether outbound traffic is close to the **Egress (per pod)** limit. If it is at the limit, choose a higher plan. Details: [Bandwidth Plans](bandwidth-plans.md).

### Visitors are sent to a sign-in page or blocked

**What you see:** Visitors are redirected to **Sign in to continue**, or see **You don't have access** after signing in. Webhook and CI calls get `401` or `403`.

**Cause:** Access protection is on for the service and the request does not meet its rules: the visitor is not signed in, the service is not shared with their account, or their IP address or country is not on the allowed list.

**What to do:** Add the person to the shares or review the rules; use a service token for calls from machines. Every response a visitor can get is listed on [Access Protection: Reference](access-protection-reference.md). See also [Access Protection](service-access-protection.md) and [Machines and Private Mesh](access-protection-machines.md).

---

## Service Is Asleep or Will Not Wake

A service on the free package goes to sleep after 30 minutes without traffic and wakes on the first request; waking can take about a minute. Meanwhile a visitor in a browser sees the **This service is waking up** page, which refreshes on its own once the service is ready. Services on paid packages do not go to sleep on their own.

If the service cannot be woken, the visitor sees an information page (HTTP 503) and you see the **App is asleep** card on the service dashboard:

| What the visitor sees | Description in the console | What to do |
|---|---|---|
| **This service is paused for this month** | "This month's free hours are used up, so this free-plan app sleeps until the pool resets." | Free services share 450 awake hours a month. Once the pool is used up, they cannot be woken or deployed until the 1st of the month (UTC). If you do not want to wait, add the **Always on add-on** to the service or move it to a paid package. |
| **This service is on hold** | "Free-plan apps need a Git account linked to the organization. This app sleeps until one is linked under Account → External Logins." | Link your GitHub account under **Account → External Logins**, or connect your own GitHub, GitLab or Azure DevOps account on the **Integrations** page. Paid packages do not need this. |
| **This service is unavailable** | "This app is asleep because the account is suspended for billing. It wakes up once the account is reactivated." | Pay the outstanding statement under **Account → Statements**. |
| **This service is stopped** | "This app was put to sleep manually; its replicas stay at zero until it is woken up." | The service was put to sleep from the console, and incoming requests do not wake it. Start it with **Wake up** on the dashboard. If the description says "The platform team put this app to sleep", open a support ticket. |

**If a wake attempt failed**, the card shows **Sleep transition failed**; try again with **Wake up**.

**If free hours run out quickly**, something that sends regular requests (an uptime monitor or a browser tab left open, for example) may be keeping the service awake. Every awake hour counts against the pool.

Details: [Service Sleep](service-sleep.md).

---

## I Can't Create Something

| Message you see | Cause | What to do |
|---|---|---|
| "This operation would bring your estimated monthly spend to … USD, above your current spending cap of … USD. …" | The new resource or plan upgrade would take the monthly estimate over the spending limit. | Add funds to the wallet or verify your card; both raise the limit. You can also ask for more with **Request a limit increase** under **Account → Wallet**. |
| "Your account is in a grace period because an invoice could not be collected. …" or "Your account is suspended for outstanding invoices. …" | There is an unpaid statement. | Pay it under **Account → Statements**. |
| "You can publish at most 1 service on the free plan. Choose a paid plan to deploy more services." | The organization's free service allowance is used. | Pick a paid package for the new service, or move the existing free service to a paid package. |
| **Link a personal Git account first** or **This Git account is used by another organization** | The free plan is tied to a personal Git account, and a Git account can use the free plan in one organization only. | Link your personal account with **Link Git account**, use a different Git account, or pick a paid package. |
| **You don't have access to this view** — "Your role is missing the … permission. Ask an administrator to grant it on your role." | Your role lacks the permission. | Ask an organization admin to add the permission to your role. |
| "This area is managed by your organization administrators. Ask one of them if you need access." | Billing pages are open to organization admins only. | Ask an admin for help. |
| "Verify your email address before creating an organization. Open the confirmation link we sent to your inbox, then try again." | Your email address is not confirmed. | Open the link in the confirmation email; it is valid for 24 hours. If the email did not arrive, request a new one with **Didn't receive it? Resend** on the confirmation screen. |
| "You have reached the maximum of 5 organizations per account." | An account can be in at most 5 organizations. | The limit is fixed; use one of your existing organizations. |

> **Note:** The spending limit is checked only when you create something or move it to a bigger plan; resources that are already running are not stopped because of it.

Details: [Billing](billing.md), [Organizations and Team](organizations-team.md).

---

## Git Connection

| What you see | Cause | What to do |
|---|---|---|
| "No repositories returned for this account." in the wizard, or **No repositories found** on the **Integrations** page | The connection has no access to any repository. If the GitHub App was installed with **Selected repositories**, only those repositories appear. | Add the repository to the installation on GitHub with the settings button in the **GitHub App Installations** list. Check that a GitLab token has the `api` scope, and that an Azure DevOps PAT has Code (Read) and Service Connections (Read, query, manage). |
| **Couldn't load repositories** | The connection may have been removed or suspended, or its credentials may no longer be valid. | Reconnect it. |
| **Provider not supported yet** | The connection belongs to a provider that is not supported. | Use a GitHub, GitLab or Azure DevOps connection. |
| "This installation was removed on GitHub. Reconnect to restore auto-deploy." | The Komuta App was removed on the GitHub side. | Press **Reconnect**. |
| "This installation is suspended on GitHub. Re-enable it there to restore access." | The installation was suspended on GitHub. | Re-enable the installation on GitHub. |
| **Token expired** | The Azure DevOps PAT has expired; auto-deploy builds fail at clone time. | Enter a new PAT with **Update token**. GitLab connections have the same **Update token** action. |

> **Warning:** Do not remove a Git connection that is in use. Once it is removed, services built from its repositories keep running but can no longer pull source, and the connection's auto-deploy subscriptions are deleted.

Details: [Integrations](integrations.md).

---

## Custom Domain

**What you see:** On the **Domains** page the domain stays in **Provisioning** or **Action Required**.

**What to do:**

1. Open the domain and look at the **DNS Records to Add** list. Add or correct, at your DNS provider, every row that says **Not detected yet**, **Different value** or **Not seen in DNS**.
2. Wait for the DNS change to spread; this usually takes a few minutes and at most 24 hours.
3. Press **Refresh Validation**.

**If the certificate is not issued**, look at the **SSL Status** row:

| SSL Status | Meaning and what to do |
|---|---|
| **Awaiting validation** | The certificate validation record has not been found yet. Add or correct it. |
| **Timed out** | The provider stopped waiting. Check the DNS records, then refresh validation. |
| **Expired** | The certificate could not be renewed. Check the certificate validation record, then refresh validation. |

Other cases:

- If the **Provider Status** shows **DNS moved**, DNS no longer points to Komuta; check the traffic record.
- If you see **Under review** or **Blocked**, open a support ticket.
- On an apex domain (such as `example.com`), your DNS provider must support CNAME flattening (ALIAS or ANAME). The traffic routing row may then look undetected even though the domain is active.

Details: [Custom Domains](custom-domains.md).

---

## Database Connection

If your service cannot connect to a managed PostgreSQL, Valkey or RabbitMQ instance, check these in order:

1. **Is the instance ready?** No connection info exists while the instance is **Provisioning**; wait until it is **Active**.
2. **Are the details right?** Copy the host, port, user and password from **Show connection info** on the instance. On PostgreSQL, if **Rotate database password** was used, the old password no longer works and existing connections are dropped.
3. **Did the value reach the service?** After adding the connection info as environment variables, press **Apply Changes**.
4. **Are you using the right address?** If the instance has a **Private address** (also listed among the connection strings as **Connection String (private network)**), use it from your services on Komuta; it works in every access mode. When **Network access** is **Private only**, there is no public address and nothing outside Komuta (your computer, a CI job) can connect. In **Public, IP allow-list** mode, addresses not on the list are refused.
5. **Did you hit the connection limit?** On PostgreSQL the number of concurrent connections depends on the plan; keep your app's connection pool below that limit.

For connections between your own apps, use **Connect an app** on the service's **Connections** page; **Add as env variable** writes the generated address to the service.

Details: [PostgreSQL](managed-postgresql.md), [Connections](services-connections.md).

---

## Logs, Metrics and Traces Are Empty

**If the logs are empty:**

- Logs come from the container's output. Make sure the app writes to standard output (stdout) or standard error (stderr).
- If you see **No logs found**, widen the time range and clear the filters. The default range is the last hour.
- A service that is asleep, in maintenance or stopped by restart protection writes no new logs.

**If the metric charts are empty** ("No data for the selected window."): no pod may have been running in that range. If the service is asleep or was just deployed, change the range.

**If the traces are empty:**

- Automatic instrumentation supports .NET, Java, Node.js and Python. If your service uses another language, add the OpenTelemetry SDK to the app with **Set up OpenTelemetry** on the traces screen.
- **No matching traces returned** does not prove that no requests came in; because of sampling, some requests are not recorded. Widen the time range.
- A "… is refusing new telemetry" notice means the service is over its storage limit; traces sent in the meantime are not kept. Review the limit under **Account → Observability Storage**.

Details: [Logs](service-logs.md), [Observability](observability.md).

---

## Getting Help

**Support ticket.** Click the **Support** icon (life buoy) in the top bar, or open [console.komuta.io/support](https://console.komuta.io/support), and use **Create Ticket**. Include:

- The name of the service or resource.
- When the problem started (UTC).
- What you did and the exact text of the message you saw.
- For server errors, the **Support reference** shown in the notification (use **Copy** next to it).

Details: [Support](support-tickets.md).

**Platform status.** You can follow whether there is a Komuta-wide outage at [status.komuta.io](https://status.komuta.io).

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — The service's live state and deploy actions.
- [Pipelines](service-pipeline-guide.md) — Build stages, logs and automatic Dockerfile repair.
- [Build Queue](build-queue.md) — Why builds wait.
- [Service Sleep](service-sleep.md) — How free services sleep and wake.
- [Billing](billing.md) — Spending limit, wallet and statements.
- [Bandwidth Plans](bandwidth-plans.md) — The per-pod outbound speed limit.
- [Limits and Quotas](limits.md) — Platform-wide limits.
- [Support](support-tickets.md) — Opening and following support tickets.
