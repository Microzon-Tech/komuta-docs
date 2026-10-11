# API Gateway: APIs, Keys and Policies

This page covers connecting APIs to a gateway, defining endpoints, giving callers keys, rate limits and quotas, the other policies the gateway offers, and the responses the clients of your APIs get. Creating a gateway, applying changes, monitoring and gateway-wide settings are on [API Gateway](api-gateway.md).

Every change on this page is saved first. To make it reach traffic, click **Apply changes** at the top of the gateway page.

---

## Connecting an API

An API is a Komuta service connected to the gateway under a path prefix. Today the only source you can choose is an existing Komuta service; an external URL cannot be connected.

1. On the gateway's **APIs** tab, click **Connect API**.
2. In the **Source** section, search for the service and select it. Services in the projects you can access are listed; workloads that take no traffic, such as jobs and cronjobs, and services already connected to this gateway are not.
3. Fill in the **Routing & access** section (see the table below).
4. Choose the **Authentication** mode and, if needed, enter a **Rate limit per key (requests / second)**.
5. Optionally adjust the settings in the **Advanced policy** section.
6. Click **Connect API**. You see "Route saved. Apply your changes to make it live."
7. Click **Apply changes** at the top of the page.

What you may see in the service list:

| State | Meaning |
|---|---|
| **No port** | The service does not expose a port yet and cannot be connected. Add a port on the service's **Ports** tab first ([Access & Ports](services-ports.md)). |
| **Not deployed** | The service can be connected; the route is created now and returns an error until the service's first deployment succeeds. |
| "Komuta will expose … on your private mesh so the gateway can reach it." | The service is opened to the private network. Its own public address is unchanged. |

| Field | Description |
|---|---|
| **API name** | Identifies the API in lists and on traffic charts; it must be unique on the gateway. It is filled with the service's name when you select the service. |
| **Public path** | The prefix callers use, such as `/orders`. It starts with `/` and uses lowercase letters, digits and dashes; at most 64 characters. It is suggested from the service's name. `/__health` is reserved for the gateway's own health check. Two APIs cannot use the same path, but one API's path can sit under another's (`/orders` and `/orders/v2`). |
| **Remove the public path prefix** | When on (the default), the service receives the request without the prefix: a request to `/orders/v1/items` reaches the service as `/v1/items`. When off, it arrives as `/orders/v1/items`. The **Preview the request flow** section under the form shows the result. |
| **Stream responses** | The response travels at the pace your caller reads it instead of being collected by the gateway first. Meant for large downloads and long-lived streams. It cannot be combined with the response cache. |
| **Description** | What the API is for. It appears in the API's reference and OpenAPI document; it does not change how the gateway behaves. |

To change a connected API, select it in the list on the **APIs** tab; its settings open in the panel on the right. Click **Save changes** to save. The target service cannot be changed while editing; to send the path to a different service, remove the API and add it again. If you change the path, the form warns "Callers using {old path} will get a 404 once this is applied."

> **Note:** Requests that come through the gateway may not follow the traffic weights of a canary or blue-green deployment on the target service. For deployment strategies, see [Deployment Strategies](deployment-strategies.md).

> **Note:** Call the API without a trailing `/` after the prefix (`/orders`). A request to `/orders/` gets a redirect (301) to `/orders` instead of the response.

---

## Authentication

Each API uses one of three modes. **API key** is selected on a new API.

| Mode | What the caller sends | Rate limits are counted per |
|---|---|---|
| **API key** | A key issued by the gateway, in the `X-Api-Key` header. The key is accepted only in this header. | Caller (key) |
| **JWT** | A token signed by your identity provider, in the `Authorization: Bearer <token>` header. | Token |
| **Public** | Nothing; there is no authentication. | Client address |

- If you choose **API key** and the gateway has no active key, the form shows "This gateway has no active key yet. Create one on the Keys tab first, or every request to this API is rejected." and the save is refused.
- For **JWT**, the **OIDC discovery URL** must first be set on the **Settings → JWT validation** card; until it is, the form shows "Set the OIDC discovery URL in gateway settings before saving an API with JWT authentication." See [JWT Validation](api-gateway.md#jwt-validation).
- Callers' own limits, API restrictions and usage plans apply only to APIs that require an **API key**. JWT and public APIs do not identify a caller.

**Headers your service receives.** On an API that requires a key, the gateway tells your service who is calling with the `X-Consumer-Username` and `X-Consumer-Custom-ID` (the caller's **Customer ID**) headers. The `X-Api-Key` header does not reach your service unless **Pass the API key on to your services** is on under **Settings**.

**Customer ID header (optional).** Shown only in API key mode. If you want your service to read the customer ID from a header name of your choice (for example `X-Customer-Id`), enter it here. The gateway sets this header on every request and removes any copy the client sent, so your service can trust the value. Headers the gateway uses itself cannot be chosen.

---

## Rate Limit

**Rate limit per key (requests / second)** sets how many requests per second each caller may send to an API. `0` means unlimited; the console accepts up to 10,000.

- The limit is one budget across the API; endpoints without a limit of their own share it.
- Responses served from the cache count toward the limit too.
- A request over the limit gets `429`. See [Responses Your Callers Get](#responses-your-callers-get).
- A caller's own rate limit, or its usage plan's limit, replaces the API's limit for that caller, even when it is higher. See [Caller Limits](#caller-limits).

> **Warning:** Without a counter store, each replica counts the limit on its own. The form says so: "Counted per replica. This gateway may run up to {replicas}, so a limit of {limit}/s admits up to {effective}/s per key across this API…" For an exact limit, see [How Rate Limits Are Counted](#how-rate-limits-are-counted).

---

## API Policies

The **Advanced policy** section of the API form applies to every request to the API. Endpoints inherit these settings, and each one can be changed on an endpoint.

| Setting | Description |
|---|---|
| **Timeout** | How long the gateway waits for your service on this API. Leave empty to use the gateway's timeout. Between 1 ms and 300 seconds, written like `500ms`, `45s` or `2m`. |
| **Retries** | How many more times a request that could not reach your service is tried (0–3). Leave empty for the platform default. |
| **WebSocket** | Lets callers upgrade a connection to a WebSocket. The API's timeout also bounds an idle socket: one that carries nothing for that long is closed. |
| **Response cache** | Keeps responses in a cache for a fixed time. See below. |
| **IP access rules** | Admits or blocks callers by IP address. See below. |
| **Required scopes** | JWT mode only. The token must carry every scope listed here, or the request is refused. At most 16 scopes, each 1–64 characters without spaces. |
| **Deprecation announcement** | Announces with every response that the API is being retired. See below. |
| **Maintenance** | Closes the API with a maintenance message. See below. |
| **Request and response transformations** | Changes headers and the path. See below. |
| **Validation and circuit breaker** | Refuses bad requests and pauses an endpoint that keeps failing. See below. |

Transformations, validation, the circuit breaker and endpoints are not shown on the form for a new API: connect and save the API first, then open it again from the list.

### Response Cache

1. Open the **Response cache** row and switch on **Enable response caching**.
2. Enter the **Lifetime (seconds)** (1–3600; default 60).
3. Optionally enter comma-separated names under **Vary by query parameters** and **Vary by request headers**. With no query parameters listed, the whole query string is used; that is the safe default.

- Only `200` responses to `GET` and `HEAD` requests are cached. Responses with `Cache-Control: private` or `no-store`, or that set a cookie (`Set-Cookie`), are not stored.
- Authentication is part of the cache key: on APIs that require a key, each caller has its own copy; on JWT and public APIs the `Authorization` header is part of the key.
- Responses served from the cache count toward rate limits and quotas too.
- The `X-Komuta-Cache` response header shows the cache result (`HIT`, `MISS`, `EXPIRED`).
- **Clear API cache** marks cached responses unusable in the next configuration; they keep being served until you apply.
- The cache size is set for the whole gateway ([Runtime](api-gateway.md#runtime)). A full cache does not error; it quietly stops storing new responses. You can see how full it is on **Overview → Traffic**.

### IP Access Rules

As the **Access mode**, choose **No restriction**, **Allow only listed addresses** or **Block listed addresses**, and enter the addresses one per line (such as `203.0.113.7`, `10.0.0.0/8` or `2001:db8::/32`). A caller that does not pass the rule gets `403`.

> **Note:** IP rules can be applied only while address filtering is enabled on the platform. If it is not, the save is refused with a message that starts "Address filtering isn't enabled on this platform yet." In that case, contact us through [Support](support-tickets.md).

### Deprecation Announcement

Enter the **Deprecated at** and **Sunset at** dates and, if you like, a **Link** where callers can read about the change. Both dates are optional; the sunset cannot come before the deprecation. The link must be an absolute `http(s)` address of at most 512 characters, without `$` or spaces. Times are in the time zone chosen in your settings.

While the announcement is set, every response from the API carries the `Deprecation`, `Sunset` and `Link` (`rel="deprecation"`) headers. The API shows a **deprecated** badge in the list.

### Maintenance

Switch on **Take this API into maintenance** and write a message of at most 512 characters in **Message to callers**. Once applied, every route of the API answers `503` with this body:

```json
{"error":"maintenance","message":"<your message>"}
```

With an empty message, `"This API is under maintenance."` is sent. On an API that requires a key or a token, only callers that present a valid key or token for this gateway see the message; everyone else gets the usual `401`. On a public API, anyone who requests the path sees it.

### Request and Response Transformations

- Add rules under **Headers sent to the backend** (**Request header rule**) and **Headers returned to the caller** (**Response header rule**). The operations are **Set** (replaces whatever the caller sent), **Add** (leaves theirs in place and sends a second copy), **Remove** and **Rename**. Use **Set** for any header your backend trusts.
- Each header can have one rule per side. Header names use letters, digits and `- _ .` (at most 64 characters); values are one line, at most 512 characters, and cannot contain `$`.
- Values typed into rules are stored in the gateway's configuration and visible to anyone who can read it. With **Read this value from a variable**, you give the name of an environment variable of the gateway's service instead of the value; only the name is stored. If the variable does not exist on the gateway's service, the save is refused.
- Leave **Host header sent to the backend** empty to pass the caller's through.
- **Rewrite the path** takes a pattern and a replacement that starts with `/`. The pattern is applied to the path after the API prefix is removed; requests it does not match pass through unchanged. Use `$1`, `$2` for the pattern's groups.

### Validation and Circuit Breaker

**Validation.** Add required headers under **Headers the request must carry** and JSON body fields under **Fields the request body must satisfy**. For each field you can set a type (`String`, `Integer`, `Number`, `Boolean`, `Array`, `Object`), mark it **required**, and set the smallest/largest value, the shortest/longest length, and for text fields a pattern (such as `^[A-Z]{3}-[0-9]{4}$`) or a list of accepted values.

- A request that does not satisfy the rules is answered `400` by the gateway, naming the field that failed; your service never sees it.
- Validation runs after authentication: a caller without a key or token gets `401` first.
- Body rules can be set only on an endpoint whose methods all send a body (`POST`, `PUT`, `PATCH`); required headers can be set anywhere.
- `X-Request-Id` cannot be made required; the gateway fills it in itself before it checks anything.

**Circuit breaker.** Switch on **Pause an endpoint that keeps failing**:

| Field | Default | Range |
|---|---|---|
| **Failures before pausing** | 3 | 1–100 |
| **Longest pause (seconds)** | 300 | 3–3600 |
| **Failure status codes** | 500, 502, 503, 504 | 5xx codes only |
| **Response code while paused** | 503 | 400–599 |

Failures are counted per path; one endpoint pausing does not affect the rest of your service. The first pause is a couple of seconds and doubles while the failures continue, up to the longest pause; meanwhile callers get the code you chose. Requests refused during a pause still count toward the rate limit.

---

## Endpoints

An endpoint is one path and set of methods under an API. Use one when a single path needs its own cache, limit, timeout or authentication, for example a longer timeout for one slow report on an otherwise fast API. Any request that is not an endpoint keeps reaching your service with the API's own settings.

1. On the **APIs** tab, select the API and click **Add endpoint**.
2. Fill in the **Basics** tab:
   - **Name** — labels the endpoint's traffic on the charts (at most 63 characters).
   - **Summary** and **Description** — shown in the API's reference and OpenAPI document.
   - **Methods** — `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`; at least one.
   - **Path** — relative to the API's prefix. `{name}` matches one segment, and a trailing `/*` covers everything below. Use letters, digits and `- . _ ~`; `/` alone is the API's own prefix.
3. Switch on the policies you want to change. Every policy whose switch is off follows the API.
4. Click **Save** and apply the changes.

| Tab | Policies you can override |
|---|---|
| **Basics** | **Authentication**, **Required scopes**, **Rate limit**, **Deprecation announcement** |
| **Cache** | **Response cache** (lifetime, query parameters, headers; at most 8 names per list) |
| **Transform** | **Change requests and responses** |
| **Protection** | **Timeout**, **Who may call, by address** (at most 60 addresses), **Refuse bad requests**, **Pause when the backend fails** |

- An endpoint's **Rate limit** is counted per caller for this endpoint alone (0–10,000; `0` means no limit on this endpoint). Without a limit of its own, the endpoint shares its API's budget, unless it uses its own authentication, in which case it counts its callers separately.
- An endpoint can require an **API key** only if the gateway has at least one key.
- Two endpoints on the same API cannot match the same method and path. A gateway holds at most 300 endpoints in total.
- To delete an endpoint, click the delete button in the endpoint list on the API's panel and confirm with **Delete** in the "Remove endpoint policy?" window. After you apply, those requests use the API's default policies.

---

## OpenAPI Import and Reference

**Import.** If you have an OpenAPI or Swagger description, you can import endpoints instead of adding them one by one:

1. Select the API and click **Import OpenAPI**.
2. Pick the description with **Choose a file** (JSON or YAML, under 2 MB).
3. To also take the required fields and headers the description states as validation rules, tick **Also bring the request rules the description states**. Only required fields and headers with a plain type are taken; these rules start refusing traffic as soon as you apply.
4. Preview with **See what this adds**, then click **Add {count} endpoints** and apply the changes.

Only endpoints the API does not serve yet are created; existing endpoints keep their settings and nothing is removed. Methods the gateway does not route and paths it cannot express are marked **refused** in the preview. The file must be self-contained: a description that points at another file or a URL is refused, so export it with references resolved.

The imported description is kept on the API ("Description on file: …") and can be downloaded again with **Download {format}**.

**Reference.** **Download reference (OpenAPI)** downloads an OpenAPI document generated from the API's endpoints and policies: the paths under the gateway, how to authenticate, and the responses the gateway itself may give (`400`, `401`, `403`, `429`, `503`). This is the document to hand to your callers. To publish the reference of every API behind a link, use the [Public API reference](api-gateway.md#runtime) setting.

---

## Callers and Keys

A caller is a client that holds one API key: one of your customers, a partner or one of your apps. Callers are listed under **Callers** on the gateway's **Keys** tab, with their name, the start of their key, the APIs they can reach and their status (**active**, **expired**, **revoked**, **no APIs**).

Callers are part of the gateway's configuration: a new, edited or revoked caller reaches traffic on your next apply. The **Keys** tab is visible to members who can edit the gateway and to members granted **Manage API gateway callers and usage plans**.

### Creating a Caller

1. On the **Keys** tab, click **New caller**.
2. Fill in the fields:
   - **Name** — helps you recognise the caller (for example "mobile app").
   - **Contact** — who to tell before you change its limit or revoke it.
   - **Customer ID** — optional. Your services receive it on every request this caller makes, as the `X-Consumer-Custom-ID` header. Up to 128 characters: letters, digits and `. _ : @ + = / ~ -`, starting with a letter or digit.
   - **Expires at** — optional. See [Expiry](#expiry).
   - **Usage plan** — puts the caller on a plan, or choose **No plan — its own limits below**.
   - **Its own rate limit**, **Volume allowance**, **Limit which APIs it may call** — see [Caller Limits](#caller-limits).
3. Click **Save**. The key is shown: "Copy this key now — it is not stored anywhere you can read it again." Copy it and click **Done**.
4. Apply the changes. The key works only after the apply.

Keys start with `kmt_gw_`. Your callers send the key in the `X-Api-Key` header on every request.

> **Warning:** The key is shown only once, when you save, and cannot be read back afterwards. For a lost key, [rotate the key](#rotating-a-key).

### Expiry

If you enter a moment in **Expires at**, the gateway starts refusing the key about a minute after that moment, and keeps refusing it until you extend or clear the date. Unlike revoking, this can be undone. While you have unapplied changes, the key keeps being accepted until your next apply. A moment in the past cannot be entered. Times are in the time zone chosen in your settings.

### Rotating a Key

**Rotate key** on a caller's row issues a new key. The caller keeps everything else: its limits, what it has used of its allowance, its customer ID and its APIs.

Choose the overlap with **Keep the old key working for**: **None — the old key stops at the next apply**, **1 hour**, **1 day**, **7 days** or **30 days**. During the overlap both keys work and count against the same allowance; the list shows the old key as "{prefix}… works until {date}". If the old key leaked, choose **None**. The new key works from your next apply.

### Revoking and Pausing

Click **Revoke** on the caller's row, then **Revoke the key**. Revoking is permanent: the key stops working at your next apply and can never be shown or restored.

- The last active key cannot be revoked while an API or endpoint requires a key: "This is the only active key, and at least one API needs one. Create a replacement key first, then revoke this one."
- To stop a caller temporarily, edit it and clear every API it may reach instead of revoking. The caller is refused everywhere but its key stays valid; the list shows a **no APIs** badge.

### Caller Limits

| Setting | Behaviour |
|---|---|
| **Its own rate limit** | Requests per second for this caller, on every API that requires a key. It replaces that API's own limit for this caller, even when it is higher. When off: "Follows whatever each API sets". |
| **Volume allowance** | How many requests the caller may send **per 24 hours** or **per 30 days**. Counted over a rolling window: the last 24 hours or the last 30 days, not a calendar day or month. A [counter store](#how-rate-limits-are-counted) must be chosen first. |
| **Limit which APIs it may call** | When off, the caller can call every API on the gateway. When on, it can call only the APIs you select and gets `403` on the others. |

> **Warning:** A caller cannot have both its own rate limit and a volume allowance; switching one on switches the other off. A caller with a volume allowance has no per-second limit at all, neither its own nor the one its APIs set. An allowance replaces the rate limit rather than adding to it.

Applying a change to allowances restarts the gateway with a new pod (about 30–60 seconds).

---

## Usage Plans

A usage plan is a named setting that gives many callers the same limit and the same APIs. Change the plan once and every caller on it follows on your next apply.

1. On the **Usage plans** card of the **Keys** tab, click **New plan**.
2. Enter a **Name** (for example "Partner").
3. Choose the **Limit for each caller**: **None — follow whatever each API sets**, **Requests per second** or **Volume allowance** (per 24 hours or per 30 days; counted for each caller separately; needs a counter store).
4. Optionally pick the APIs with **Limit which APIs it may call**.
5. Click **Save**, put callers on the plan and apply the changes.

A caller on a plan has no limits or API list of its own; it uses the plan's. A plan that has callers on it cannot be deleted; move its callers to another plan or to their own limits first.

---

## How Rate Limits Are Counted

Where rate limits and volume allowances are counted is chosen on the **Where counters are kept** card of the **Settings** tab:

| Choice | Result |
|---|---|
| **None — counted per replica, no allowances** (default) | Each replica counts limits on its own; a limit of N allows up to N × (maximum replicas) requests in total. No volume allowances can be given. |
| One of your Valkey instances | Every rate limit is counted in this instance, with one counter across all replicas; scaling does not change what a limit admits. Volume allowances can be used. Each limited request makes one round trip to the instance. The replica ceiling rises to 20. |

- If you have no Valkey instance, create one first ([Valkey](managed-valkey.md)); it is billed separately. An instance that is not running yet cannot be chosen.
- After saving the choice, apply the changes. Until then each replica keeps counting on its own.
- Counters stay in the instance they were written to. Changing or removing the store removes the volume allowances of callers and plans; the console asks you to confirm before it saves.
- If the Valkey instance's password was rotated, click **Refresh credential** and apply the changes.

> **Warning:** While **The counter store cannot be reached** is shown, allowances and rate limits are not enforced: traffic is passed rather than refused, and saves on the gateway are refused. Make sure the Valkey instance is running, then click **Refresh credential**.

---

## Usage

The **Usage** card on the **Keys** tab shows each caller's requests over the last **24 hours** and **7 days**, the **Refused (429)** requests and, if it has a volume allowance, how much it has used and when it resets. Request counts come from the gateway's metrics and are approximate; allowance usage is read from the counter the gateway enforces with and is exact.

Without a counter store, the card says "No Valkey instance is chosen in Settings, so allowances are not being counted."

---

## Responses Your Callers Get

When the gateway refuses a request without passing it to your service, it answers like this:

| Code | When | Body |
|---|---|---|
| `400` | The request breaks a validation rule. | An error message naming the field that failed. |
| `401` | On an API that requires a key, `X-Api-Key` is missing, invalid, expired or revoked; on a JWT API, the token is missing or invalid. | — |
| `403` | The caller may not use this API, the token lacks the required scopes, or an IP rule blocks the request. | — |
| `404` | No API or route on the gateway matches the path. | `{"error_msg":"404 Route Not Found"}` |
| `413` | The request body is over the gateway's **Largest request body** limit. | `{"error":"Gateway error","gateway":"Komuta"}` |
| `429` | The rate limit was exceeded. | `{"error_msg":"Too many requests. X-RateLimit-Reset says when to retry."}` |
| `429` | The caller's volume allowance is used up. | `{"error_msg":"This caller's allowance for the window is used up. X-RateLimit-Reset says when it renews."}` |
| `503` | The API is under maintenance. | `{"error":"maintenance","message":"<your message>"}` |
| The code you chose (default `503`) | The circuit breaker paused the endpoint. | — |
| `502`, `503`, `504` | The gateway got no answer from your service (for example, the timeout passed). | `{"error":"Gateway error","gateway":"Komuta"}` |

- The `404` and `429` bodies are JSON text, but their `Content-Type` header may be `text/plain`; read the body accordingly in your client.
- Error responses your own service returns (for example your own `500` body) reach the caller unchanged.

Response headers the gateway adds:

| Header | When |
|---|---|
| `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` | On routes with a rate limit or a volume allowance. No `Retry-After` is sent; `X-RateLimit-Reset` says when to retry. |
| `X-Komuta-Cache` | On routes with caching on (`HIT`, `MISS`, `EXPIRED`). |
| `X-Request-Id` | On every response when **Request id header** is on. |
| `Deprecation`, `Sunset`, `Link` | On APIs with a deprecation announcement. |

---

## Removing an API

1. On the **APIs** tab, click the trash button next to the API in the list, or **Disconnect API** on the API's panel.
2. In the "Disconnect this API?" window, click **Disconnect API**.
3. Apply the changes.

After you apply, the API's path is no longer routed and requests to it get `404`. The API's endpoints and stored description are removed with it. The target service is not deleted.

---

## Limits

| Item | Limit |
|---|---|
| APIs per gateway | 40 |
| Endpoints per gateway | 300 |
| Public path | At most 64 characters |
| Rate limit (console) | 0–10,000 requests per second |
| Cache lifetime (console) | 1–3600 seconds |
| Retries | 0–3 |
| Required scopes | At most 16, each 1–64 characters |
| Maintenance message | At most 512 characters |
| Announcement link | At most 512 characters |
| Header rule | Name at most 64, value at most 512 characters |
| Customer ID | At most 128 characters |
| Key rotation overlap | At most 30 days |
| Imported OpenAPI file | Under 2 MB |

The gateway's whole configuration also has a size limit. If a save goes over it, it is refused with a message saying by how much; the largest items are validation field rules and header rules. Move rules that every endpoint shares up to the API. Gateway-wide limits are on [API Gateway](api-gateway.md#limits).

---

## Related Documents

- [API Gateway](api-gateway.md) — Creating a gateway, applying changes, monitoring and settings.
- [Valkey](managed-valkey.md) — Valkey instances used as the counter store.
- [Access & Ports](services-ports.md) — Adding a port before connecting a service to the gateway.
- [Deployment Strategies](deployment-strategies.md) — Canary and blue-green deployments.
- [Environment Variables](services-environment-variables.md) — Services' environment variables.
