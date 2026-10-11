# API Gateway

API Gateway puts one address in front of your Komuta services. It publishes each service under its own path (such as `/orders` or `/payments`), checks requests with an API key or a JWT, applies rate limits and quotas, and can cache responses. Komuta runs the gateway for you: there is no repository, Dockerfile or build.

It is useful when you want to:

- Serve several services under one address or under your own domain.
- Open your API to customers or partners with keys, and give each of them their own limits.
- Refuse bad requests before they reach your service, pause an endpoint that keeps failing, or take an API into maintenance.

You do not need a gateway to publish a single service without authentication; the service's own address is enough.

This page covers setting up and running a gateway. Connecting APIs, endpoints, keys, limits, policies and the error responses your callers get are on [API Gateway: APIs, Keys and Policies](api-gateway-apis.md).

---

## Key Concepts

| Concept | Meaning |
|---|---|
| **Gateway** | A service with its own address that Komuta runs for you. It belongs to a project and a region. |
| **API** | A Komuta service connected to the gateway under a path prefix (such as `/orders`). |
| **Endpoint** | One path and method under an API. It can have its own cache, limit, timeout and validation settings. |
| **Caller** | A client that holds one API key, for example one of your customers or your mobile app. |
| **Usage plan** | A named setting that gives many callers the same limits and the same APIs. |
| **Apply** | Moving your saved changes onto the running gateway. Changes made on a gateway do not reach traffic until you apply them. |

---

## Creating a Gateway

1. Open **Managed Services → API Gateways** ([console.komuta.io/api-gateways](https://console.komuta.io/api-gateways)) and click **New gateway**.
2. Fill in the **New API Gateway** window:
   - **Name** — the gateway's name.
   - **Project** — the project the gateway belongs to.
   - **Region** — where the gateway runs. There is one region today: Germany-East (`de-east`).
   - **Size** — the gateway's CPU and memory package. These are the same packages services use.
3. Once you pick a size, a price note appears below it. Click **Create gateway**.
4. In the **Your gateway's first API key** window, copy the key and store it somewhere safe, then click **I've saved it**. The gateway's page opens.

When the gateway is created you see "Gateway created. It is deploying now." Until the first deployment finishes, the status reads **Provisioning** and the address shows "address pending".

> **Warning:** The first API key is shown only in this window, once, and cannot be viewed again. If you lose it, create a new caller on the **Keys** tab or rotate the key. The first key is listed on the **Keys** tab as the caller named `default`.

The **New gateway** button is visible only to members who are allowed to create managed services.

### Price

| Item | Amount |
|---|---|
| Compute | The size you choose is billed per replica, like any other service. Packages: 0.5 vCPU / 1 GB USD 9, 1 vCPU / 2 GB USD 18, 2 vCPU / 4 GB USD 36, 4 vCPU / 8 GB USD 72 (per month). |
| Gateway fee | USD 9 per month (USD 0.0125 per hour). The create window says "Compute is billed like any other service, plus a 9.00 USD/month gateway fee." |
| Bandwidth | The default Starter plan is free. See [Bandwidth](#bandwidth). |

Prices are in USD, exclude tax and are metered by the hour; a gateway deleted mid-month is charged only for the hours it ran. For example, a gateway running one replica on the 0.5 vCPU / 1 GB package costs 9 + 9 = USD 18 per month. The gateway fee appears on statements under the **API Gateway** category.

When you create a gateway, compute and the gateway fee together are checked against your spending limit; if the monthly estimate would go over the limit, the gateway is not created. See [Billing](billing.md) and the [Pricing](https://www.komuta.io/pricing) page.

---

## The Gateway Page

When you open a gateway, the top of the page shows:

- **Status** — **Ready**, **Provisioning**, **Rolling out**, **Not healthy**, **Failed** or **Stopped**.
- **Upgrade** — changes the gateway's size (package).
- **Address** — the gateway's Komuta address, with a copy button.
- **View service** — opens the page of the service that runs the gateway.
- **Replicas** — sets the replica count and autoscaling.
- **Pending changes bar** — appears when there are unapplied changes. See [Applying Changes](#applying-changes).

| Tab | Contents |
|---|---|
| **Overview** | Traffic and telemetry, an example request (**Try it**), the timeout, when the configuration was last applied, and what the pods are actually running. |
| **APIs** | Connected APIs and their endpoints. |
| **Keys** | Callers, keys, usage plans and usage. Visible only to members who can edit the gateway or manage its callers. |
| **Domains** | Custom domains attached to the gateway. |
| **Bandwidth** | The bandwidth plan and live traffic. |
| **Settings** | Gateway-wide settings, the counter store, the gateway version, configuration copy and deletion. |
| **History** | Configuration changes, and going back to an earlier version. |

The **Try it** card on the **Overview** tab shows a `curl` command for your first API that you can copy. Example: `curl -H 'X-Api-Key: <your key>' https://<gateway-address>/orders`.

The service that runs the gateway appears in the **Apps** list with an **API Gateway** badge. Its environment variables, connected APIs, health checks, replica count and resources are managed by Komuta; change them from the gateway's own page, not from the service page. The service cannot be deleted on its own; delete the gateway instead.

---

## Applying Changes

Changes to APIs, endpoints, settings, callers and usage plans are saved first. When there are saved but unapplied changes, a bar with an **Apply changes** button appears at the top of the page and stays there while you move between tabs. The gateway's card in the list also shows "unapplied changes".

The bar tells you in advance how the apply will happen:

| Text in the bar | What happens |
|---|---|
| "You have unapplied changes. Applying loads them into the running gateway within a few seconds; its pods are not restarted." | The changes are loaded into the running gateway in place. |
| "You have unapplied changes. Applying restarts the gateway with a new pod because … changed: the old one keeps serving until the new one is ready, and the switch takes about 30–60 seconds." | A new pod starts; the old one keeps serving until the new one is ready. The switch takes about 30–60 seconds. |

Changes that need a restart: the engine version, the request body limit, the response cache size, the variables the gateway reads when it starts, the counter store's password, and the callers' allowances.

After you click **Apply changes**, the bar reads "Applying — the gateway is loading the changes." or "Applying — the gateway is restarting." If it does not go well:

| Bar | Meaning and what to do |
|---|---|
| **The changes could not be applied** | The deployment failed and was rolled back. The gateway keeps serving its previous configuration and your changes stay pending. Fix the cause and click **Try again**. |
| **The changes are not confirmed on every pod** | The configuration may already be live on some or all pods; treat it as live. The platform checks again on its own. |
| "Applying is taking longer than expected: ten minutes have passed…" | The gateway did not report the new configuration within ten minutes. The service page shows what the rollout is stuck on. **Try again** becomes available once the deployment behind it gives up. |

The bar is not shown while the first deployment is still running.

**What does not need an apply:** The replica setting and the bandwidth plan take effect as soon as you save them. A key whose expiry has passed starts being refused within about a minute without an apply (if other changes are pending, it keeps being accepted until your next apply).

**Who can apply:** Any member who can edit the gateway. A member who holds only **Manage API gateway callers and usage plans** can apply when every pending change is a caller or usage plan change; otherwise the bar says "Someone who can edit this gateway applies these changes."

### What the Pods Are Serving

On the **Overview** tab, click **Check** on the **What the pods are serving** card. Instead of trusting what was recorded, Komuta asks each pod which configuration it is running and which routes it has loaded. "Every pod is serving the configuration recorded as applied." means everything is in place.

---

## Traffic and Monitoring

The **Traffic & telemetry** section of the **Overview** tab has five tabs:

| Tab | What it shows |
|---|---|
| **Traffic** | **Traffic by API**: requests per second, 4xx/s, 5xx/s and p95 latency for each API. Choose **Last 15m**, **Last 1h**, **Last 6h** or **Last 24h**; the data refreshes every 30 seconds. Next to it, **Traffic by caller** and, when any API caches, how full the **Response cache** is. |
| **Logs** | The gateway service's logs. See [Logs](service-logs.md). |
| **Insights** | The gateway's runtime insights. |
| **Resources** | CPU and memory usage. |
| **Alerts** | The gateway's alert rules. |

Per-API rates are measured over a 5-minute window, so a quiet gateway, or one restarted moments ago, can read as empty in this table. **Traffic by caller** fills only for APIs that require a key; public and JWT APIs do not identify a caller.

**Logs**, **Insights**, **Resources** and **Alerts** are read through the gateway's service. If your account cannot read that service, these tabs stay empty and show a notice.

**Alerts.** Komuta adds two alert rules to every gateway: `<service name> - API 5xx Ratio High` (on a route, more than 5% of responses are 5xx for 5 minutes while the route carries more than 0.1 requests per second) and `<service name> - Callers Throttled` (a route refuses more than 1 request per second with 429 for 5 minutes). For your own rules, use the templates in the **API Gateway** category; see [Templates, Metrics and Logs](alerts-templates.md).

**Notifications.** When an apply fails or the gateway does not serve some routes, Komuta sends an **API gateway configuration problem** notification. Choose where it goes on [Channels and Delivery](notification-guide.md).

Requests per caller and allowance usage are on the **Usage** card of the **Keys** tab; see [Usage](api-gateway-apis.md#usage).

---

## Scale and Size

**Replica count.** Click **Replicas** at the top of the page:

1. For a fixed count, leave **Scale automatically with load** off and enter the **Replica count**.
2. For autoscaling, switch it on and enter **Minimum** and **Maximum**.
3. Click **Save**.

This setting is "Applied straight away — this is not part of the gateway's saved configuration, so it does not wait for an apply." A gateway can run at least 1 and at most 10 replicas. When rate limits are counted in a Valkey counter store and that change has been applied, the ceiling rises to 20 replicas. Each replica is billed as compute.

> **Warning:** Without a counter store, each replica counts rate limits on its own, so a limit of N allows up to N × (maximum replicas) requests per second in total. The dialog says so as well. See [How Rate Limits Are Counted](api-gateway-apis.md#how-rate-limits-are-counted).

**Size.** For more CPU or memory, click **Upgrade** at the top of the page and choose a new package. The response cache is taken from the gateway's memory, so a large cache needs a larger package. For how autoscaling works in general, see [Autoscaling](services-autoscaling.md).

---

## Bandwidth

The **Bandwidth** tab shows the gateway's plan summary, a **Live Traffic** chart and usage gauges. The chart collects samples while you are on the tab, so keep the tab open.

To change the plan, choose a new plan from the list and confirm. The change takes effect at once and does not interrupt traffic.

| Plan | Monthly price | Outbound speed limit |
|---|---|---|
| Starter (default) | Free | 5 Mbit/s |
| Growth | USD 4.99 | 25 Mbit/s |
| Business | USD 14.99 | 100 Mbit/s |
| Enterprise | USD 49.99 | 500 Mbit/s |
| Scale | USD 149.99 | 2 Gbit/s |

See [Bandwidth Plans](bandwidth-plans.md) for details.

---

## Domains

A gateway is reachable at its Komuta address as soon as it is created; a custom domain is optional. To serve all of your APIs under your own domain:

1. On the **Domains** tab, click **Add a domain**; the **Domains** page opens.
2. Add and verify the domain, and attach it to the gateway's service.

Attached domains are listed on the **Domains** tab with their status. See [Custom Domains](custom-domains.md).

---

## Settings

From top to bottom, the **Settings** tab has these cards: **Gateway version**, **Where counters are kept**, **Runtime**, **JWT validation**, **CORS**, **Copy configuration** and **Danger zone**. **Where counters are kept** is described under [How Rate Limits Are Counted](api-gateway-apis.md#how-rate-limits-are-counted).

Change the settings, click **Save**, then apply the changes. If someone saves the same settings while you are editing, a **These settings were changed elsewhere** notice appears; when you save, your changes go on top of theirs.

### Runtime

| Setting | Description |
|---|---|
| **Display name** | The name the console shows for the gateway. |
| **Timeout** | How long the gateway waits for your APIs. Default `10s`. Write it like `500ms`, `45s` or `2m`; at most 300 seconds, and zero is not accepted. It can be changed separately per API and per endpoint. |
| **Request id header** | Adds an `X-Request-Id` to every response, which your callers can quote in a support ticket. Off by default. |
| **Request tracing** and **Trace sampling (%)** | Records a trace for each request; sampling is 1–100. If tracing cannot be switched on for your gateway, saving says so. |
| **Pass the API key on to your services** | When off (the default), the gateway removes the `X-Api-Key` header once it has accepted it, so the key never reaches your code or its logs. Your services learn who called from the `X-Consumer-Username` and `X-Consumer-Custom-ID` headers. Turn it on only for a service that still reads the key itself. |
| **Public API reference** | Publishes a reference of every API on the gateway (paths, how to authenticate, request and response fields) for anyone with the link. The link appears under the switch after you save. It never shows keys or where your services run. Turning it off can take up to a minute to take effect. Off by default. |
| **Response cache size (MiB)** | 8–512 MiB, default 32. Shared by every API that has caching on; held per pod and taken from the package's memory. A value the package cannot hold is refused. Applying a change here restarts the gateway. |
| **Largest cacheable response (KiB)** | 64–32768 KiB, default 512; at most a quarter of the cache size. Larger responses pass through without being cached. |
| **Largest request body (MiB)** | 1–100 MiB, default 10. Requests with a larger body are refused with 413 before they reach your service. Your service's own limit still applies. Applying a change here restarts the gateway. |

### JWT Validation

Fill in this card before you use JWT authentication on an API.

| Setting | Description |
|---|---|
| **OIDC discovery URL** | Your identity provider's discovery address, ending in `/.well-known/openid-configuration` (not the key set's address). It cannot be cleared while any API uses JWT. |
| **JWT audience** | Optional. Must match the `aud` of your tokens. |
| **JWT issuer** | Optional. The `https://` address your provider writes into the `iss` claim. |
| **Signing algorithm** | Default RS256. RS384, RS512, PS256, PS384, PS512, ES256, ES384, ES512 and EdDSA are also available. Shared-secret algorithms such as HS256 are not accepted. |

### CORS

| Setting | Description |
|---|---|
| **Allowed origins** | A comma-separated list. A new gateway starts with `*` (every origin). Leave it empty to allow no cross-origin requests at all. `*` and named origins cannot be in the same list. |
| **Allowed headers** | A new gateway starts with `Origin`, `Authorization`, `Content-Type`, `Accept`, `X-Api-Key` and `X-Requested-With`. |

### Gateway Version

The **Gateway version** card shows the running version. When a new version is available, the card says "Version {version} is available" and you can move to it with **Upgrade to {version}**. Upgrading restarts the gateway one pod at a time; requests in flight finish. Moving back to an older version is not something you can do yourself.

### Copying a Configuration

The **Copy configuration** card moves a gateway's settings, APIs, endpoints and usage plans to another gateway, for example from staging to production.

1. On the source gateway, click **Download**; a `.json` file is downloaded.
2. On the target gateway, click **Import a file** and choose the file (it must be under 2 MB).
3. Optionally tick **Take the gateway settings too** (timeouts, CORS, JWT, tracing, cache, key forwarding) and **Take the usage plans too**. The target gateway keeps its own name and its public reference setting.
4. Choose the target service for new APIs, use **Preview** to see what will be added, updated or left out, and click **Import**.
5. Apply the changes.

APIs with the same prefix are updated in place and keep the service they route to. Nothing the file does not mention is removed. Keys never travel; each gateway issues its own.

---

## Change History

The **History** tab lists every record of the configuration, newest first, with who caused it (the last 50 records). Records are kept for 180 days.

| Record | Meaning |
|---|---|
| **Configuration rendered** | A change was saved. |
| **Apply requested** | Someone clicked **Apply changes**. |
| **Applied** | The gateway is running healthily with the new configuration. |
| **Apply not confirmed** | Not every pod confirmed the change. |
| **Apply failed** | The deployment failed and was rolled back. |
| **Re-rendered by the platform** | Komuta re-rendered the configuration during its own sweep. |
| **Gateway image moved** | The gateway moved to a new version. |

Rows signed "Komuta" are the platform's own work.

**To go back to an earlier version:**

1. Click **Changes** on the row. The **This change** tab shows what changed in that record; the **Going back to it** tab shows what going back would change in the configuration as it is now.
2. Click **Go back to this version**. The change is saved, not applied.
3. Apply the changes.

Going back never touches callers or their keys. APIs added or deleted since, the counter store and the services that APIs route to also stay as they are. Header values typed into transformations are masked as `•1`, `•2` in the history.

---

## Troubleshooting

When the gateway is not working as expected, a notice card appears at the top of the page:

| Card | Meaning and what to do |
|---|---|
| **The gateway is asleep** | The gateway was idle long enough to be hibernated. Traffic or applying changes wakes it; the first request after that waits for the wake-up. |
| **Komuta stopped this gateway after repeated crashes** | The pod restarted repeatedly, so the gateway was scaled to zero and its address does not answer. Fix the cause and apply your changes; the gateway starts with a new pod. |
| **No gateway pod is ready** | The pods are running but none is ready, so requests are not answered right now. The service page has the details. |
| **The last rollout was withdrawn** | The new revision kept crashing, so the platform kept the previous one serving. |
| **Some routes are not being served** | The pod reports healthy, but requests to some routes get 404 because the gateway could not load a policy on them. Check the route's policies and the gateway version, then apply again. |

If the **APIs** tab says "N APIs are configured but not being served", requests to those APIs get 404. The usual causes are a target service that no longer exists, an address the gateway cannot resolve, an IP rule the platform cannot apply, or a caller whose allowance cannot be counted. Check the APIs it names, the **Keys** tab and **Settings**.

For general troubleshooting steps, see [Troubleshooting](troubleshooting.md).

---

## Deleting a Gateway

1. On the **Settings** tab, go to the **Danger zone** card and click **Delete gateway**.
2. Type the gateway's name in the confirmation window and click **Delete**.

Deleting removes the gateway's configuration, its API keys and the service that runs it; attached custom domains are released. The target services connected to the gateway and the Valkey instance you chose as the counter store are not deleted. The gateway fee stops after the hour it is deleted.

> **Warning:** This cannot be undone. Because the keys are deleted too, you will have to give your callers new keys even if you create a new gateway with the same name.

Only members who can edit the gateway can delete it.

---

## Limits

| Item | Limit |
|---|---|
| APIs per gateway | 40 |
| Endpoints per gateway (all APIs together) | 300 |
| Keys that are not revoked, per gateway | 1,000 (expired keys count until they are revoked) |
| Replicas | 1–10; 20 when rate limits are counted in a counter store |
| Timeout | At most 300 seconds (default 10 seconds) |
| Request body | 1–100 MiB (default 10 MiB) |
| Response cache size | 8–512 MiB (default 32 MiB), within the package's memory |
| Largest cacheable response | 64–32768 KiB (default 512 KiB) |
| Configuration file (copy) | Under 2 MB |
| Change history | 180 days; the tab shows the last 50 records |

Limits per API, endpoint, key and policy are on [API Gateway: APIs, Keys and Policies](api-gateway-apis.md#limits). For account-wide limits, see [Limits](limits.md).

---

## Related Documents

- [API Gateway: APIs, Keys and Policies](api-gateway-apis.md) — Connecting APIs, keys, limits, policies and error responses.
- [Valkey](managed-valkey.md) — The store used for rate limit and quota counters.
- [Custom Domains](custom-domains.md) — Serving the gateway under your own domain.
- [Bandwidth Plans](bandwidth-plans.md) — Plans, prices and speed limits.
- [Templates, Metrics and Logs](alerts-templates.md) — API Gateway alert templates.
- [Billing](billing.md) — Spending limit and statements.
