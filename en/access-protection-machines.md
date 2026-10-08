# Machines and Private Mesh

Komuta sign-in is for people: a sign-in page opens in a browser and the person signs in with their account. Some requests, though, come from a program rather than a person. GitHub tells your application when someone pushes, Stripe sends a notification when a payment happens, your CI pipeline runs a health check after a deploy, a monitoring tool polls your site every minute. These programs can't use a sign-in page.

Access protection offers three ways for such requests, and a section for the HTTP methods and browser CORS checks your service accepts:

| Way | For | Where |
|---|---|---|
| **Webhook path** (open path) | Senders that can't sign in and sign what they send (such as GitHub, Stripe, Slack); Komuta can check the signature | **Machines** tab → **Webhooks** |
| **Service token** | Programs you control (CI jobs, monitoring tools, scripts) | **Machines** tab → **Service tokens** |
| **Services that may come in over the private mesh** | Your Komuta services on other clusters reaching this service directly | **Machines** tab → **Services that may come in over the private mesh** (the private mesh itself is on the **Network** tab) |
| **Methods and CORS** | Allowing only the HTTP methods each path needs, and letting browsers' CORS checks through before sign-in | **Machines** tab → **Methods and CORS** |

The webhook, service token and **Methods and CORS** sections appear only while access protection is on. If protection is off, the tab shows the **Access protection is off** notice and, if you can manage protection, a **Go to Rules** button. The private mesh list is the one exception: if the service's private mesh is on, the list appears and can be filled in as soon as you start turning protection on on the **Rules** tab (before saving).

---

## Webhook paths

A webhook path opens a specific path of the site without Komuta sign-in. For example, while the site is open only to your team, requests GitHub sends to `/webhooks/github` reach your application without signing in.

> **Important:** A webhook path lets anyone send requests without signing in. Something must check that a request really comes from GitHub or Stripe, by verifying the sender's signature (GitHub sends an `X-Hub-Signature-256` header, Stripe a `Stripe-Signature` header). Either choose a [signature check at the edge](#signature-check-at-the-edge), so Komuta refuses unsigned requests before they reach your application, or verify the signature in **your application**. Without either, anyone can send requests to this path.

### Opening a webhook path

1. On the **Machines** tab, in the **Webhooks** section, click **Open a path**.
2. In the **Open a path for webhooks** window:
   - **Path** — the path to open, for example `/webhooks/github`. This path and everything below it is opened.
   - **Methods** — the HTTP methods accepted on this path: `POST`, `PUT`, `PATCH`, `DELETE`, `GET`, `HEAD`. The default is `POST` only; at least one method must be chosen.
   - **Sender addresses (optional)** — one IP address or CIDR range per line. If you fill it in, only requests from these addresses get onto the path. If you leave it empty, requests from any address are accepted; the signature check (at the edge, or in your application) still protects you. You can write ranges the sender publishes here, such as GitHub's webhook addresses (the window shows `140.82.112.0/20` as an example).
   - **Signature check at the edge** — **None, my app checks it** (default), **GitHub (X-Hub-Signature-256)**, **Stripe (Stripe-Signature)** or **Other HMAC-SHA256 header** (see [below](#signature-check-at-the-edge)).
3. Save with **Open the path**. The change takes effect within a few seconds ("Webhook paths are being applied"). If you chose a signature check, add the signing secret under the path next ("After you open the path, add the signing secret under it. Until then every request to it gets 401.").

The list shows each open path with its methods, its sender list ("Only from: …" or "From any address"), its signature check ("GitHub signature checked", for example) and the service's full address. To close a path, click the trash icon on its row; it is closed immediately, without confirmation.

### How webhook paths work

- **No sign-in is asked.** Requests with the chosen methods pass without Komuta sign-in.
- **Only the chosen methods are open.** A request with another method is decided as if there were no open path, so it has to pass the site's normal protection.
- **The site's IP list and parent path rules don't apply.** On an open path, the only address check is the path's own **Sender addresses** list.
- **Block rules still apply.** Only a **Block completely** rule can sit under an open path; for example, with `/webhooks` open, `/webhooks/old` can still be blocked. Sign-in, IP or people rules can't go under an open path. The webhook window refuses a path that overlaps an existing rule with "This path overlaps the rule for /admin. An open path can't share its place with another rule." (`/admin` in the example); if you try to save a rule under an open path on the **Rules** tab, you get a "… sits under the open path …" error.
- **Path matching is case-insensitive.** A request to `/HOOKS/x` is under `/hooks`. A request is opened only when every way its path can be read stays under the open path; `%2f`, `..` and similar tricks can't be used to escape from an open path to another path.
- **Open paths can't be nested.** A second open path can't be opened on, or under, an existing open path.
- **Method-override headers are ignored.** Komuta looks at the request's real method. Your application shouldn't honour headers such as `X-HTTP-Method-Override` or `X-HTTP-Method` on these paths; otherwise requests that act like `DELETE` could be sent to a path you opened only for `POST`.
- **`OPTIONS` can't be opened on a webhook path.** `OPTIONS` isn't one of the methods you can choose, so a browser's CORS check to an open path is decided by the site's normal protection. For endpoints that a browser must call from another site, use [Methods and CORS](#methods-and-cors) instead.
- **Countries don't apply, the rate limit does.** An open path is exempt from the [country list](access-protection-rules.md#countries) (its own sender list is the address check), but its requests count towards the [rate limit](access-protection-rules.md#rate-limit). Block rules and [method rules](#methods-and-cors) are checked before the open path.
- **The access log** records every delivery to an open path as **Request on an open path** / **Delivered to an open path**, under the open path's prefix rather than the full path.

### Signature check at the edge

With a signature check, Komuta verifies the sender's signature on the raw request body before the request reaches your application. A request without a valid signature gets `401` and never reaches your application. Only requests that come through the Komuta gateway are checked: your own services on the same cluster, and services you chose for the private mesh, reach your pods directly without a signature check. If that matters, keep verifying the signature in your application too.

Only the request body is signed (Stripe also signs the time). The sub-path under the signed prefix, the query string and other headers, such as `X-GitHub-Event`, are not covered, so don't trust them on their own.

| Choice | What Komuta checks |
|---|---|
| **None, my app checks it** | Nothing; your application must verify the signature. |
| **GitHub (X-Hub-Signature-256)** | `X-Hub-Signature-256: sha256=<hex HMAC-SHA256 of the body>`. |
| **Stripe (Stripe-Signature)** | `Stripe-Signature: t=<time>,v1=<signature>`: an HMAC-SHA256 of `<time>.<body>`. The time may differ from Komuta's clock by at most **300 seconds**, which stops old requests from being replayed. |
| **Other HMAC-SHA256 header** | An HMAC-SHA256 of the body in the **Header** you choose: `x-hub-signature-256`, `stripe-signature`, `x-signature`, `x-signature-256` or `x-webhook-signature` (only these headers are passed through by the gateway). Optionally a **Value prefix** that comes before the signature (for example `sha256=`; at most 16 characters, no spaces or commas) and the **Encoding** of the signature: hex (default) or base64. |

Rules for signed paths:

- **Only `POST`, `PUT` and `PATCH`.** A signed path takes only requests with a body ("Pick POST, PUT or PATCH; a signed path only takes requests with a body.").
- **Bodies up to 65,535 bytes (about 64 KiB).** The gateway hands at most this much of a body to the check, so a larger body can't be verified and is refused with `413`. Most webhooks are far smaller, but a large event can exceed it; for example a GitHub `push` event with many commits. If your sender sends such events, keep the check in your application instead.
- **Other requests under a signed path get `401`.** A request under a signed path that the path doesn't take (another method such as `GET`, or a path that can be read in more than one way) is refused with `401`; it is not decided by the site's normal rules. This is so a signed path can't be reached around its check.
- **The sender list is checked first.** If the path has **Sender addresses**, a request from another address gets `403` before the signature is looked at.
- **Replays.** GitHub and other HMAC signatures carry no time, so Komuta can't stop someone from re-sending a request they captured, within the time the sender would retry it. Stripe's signature carries a time, so Komuta refuses copies older than 300 seconds. If replays matter, let your application refuse delivery ids it has already processed (GitHub sends `X-GitHub-Delivery`).

What the sender gets:

| Situation | Response |
|---|---|
| Valid signature | The request reaches your application (**Delivered to an open path**). |
| Missing, malformed or wrong signature, or a request under a signed path that the path doesn't take | `401`, plain text `invalid webhook signature` |
| The path has no signing secret yet | `401`, plain text `webhook signature cannot be checked` |
| The body is larger than 65,535 bytes | `413`, plain text `webhook body too large to verify` |

These refusals appear in the access log as "Sent a webhook without a valid signature", "Sent a webhook to a path that has no signing secret yet" and "Sent a webhook body larger than 64 KiB", under the path's prefix.

#### Signing secrets

A signed path checks signatures with the **Signing secrets** listed under it. Until it has one, every request to it gets `401` ("No secret yet. Every request to this path gets 401 until you add the secret the sender signs with.").

1. Under the path, click **Add secret**.
2. In **Where the secret comes from**, choose:
   - **Generate a strong secret** — Komuta creates one. It is shown **only once**: copy it and paste it into the sender's webhook settings (for GitHub, the webhook's **Secret** field).
   - **Use a secret I already have** — paste a secret the sender gave you. For Stripe this is the only choice: paste the endpoint's signing secret from the Stripe dashboard (it starts with `whsec_`).
3. Save with **Add secret**. The secret reaches the gateway within a minute ("The secret is saved and reaches the edge within a minute").

Things to know:

- Komuta keeps the secret encrypted and never shows it again; the list shows only an id and when it was added.
- A secret is 8 to 512 bytes, without spaces or control characters.
- **At most 2 secrets per path**, so you can rotate without downtime: add the new secret, switch the sender to it, then remove the old one. Both work in between.
- Removing a secret asks **Remove secret {key}?**; senders still signing with it are refused within a minute. Removing the last secret of a path means every request to it gets `401` until you add a new one.
- Closing the path, removing its signature check or switching to another provider deletes the path's secrets; a new signed path needs a new secret.
- Seeing and changing signing secrets needs the **Manage service access protection** permission.

If the **Signature check at the edge** choice says "Signature checks at the edge aren't available on this platform yet.", signature checks aren't enabled on your platform yet; verify the signature in your application.

### Limits

- A service can have at most **10** open paths. When the limit is reached, **Open a path** is disabled. Open paths also count towards the total limit of 50 path rules, together with the path rules on the **Rules** tab.
- `/` (the whole site) and Komuta's own sign-in path (anything starting with `/.komuta-access`) can't be opened.
- The path follows the same [syntax as path rules](access-protection-rules.md#path-syntax) (at most 256 characters). The window shows the same message for every invalid path: "Enter a path like /webhooks/github. The whole site can't be opened."
- The sender list takes at most 100 entries and only public internet addresses. If the list has an invalid line, **Open the path** stays disabled.
- A signed path takes only `POST`, `PUT` and `PATCH`, bodies up to 65,535 bytes and at most 2 signing secrets.
- Opening and closing webhook paths, and managing signing secrets, needs the **Manage service access protection** permission; others only see the list.
- Protection made up only of open paths protects nothing; a webhook path makes sense while the site, or at least one path, is under some other protection.

---

## Service tokens

A service token is a secret key that lets a program that doesn't use a browser get through Komuta sign-in. Use it for CI jobs, monitoring tools and your own scripts. The token is added to the request as an HTTP header:

```bash copy
curl -H "x-komuta-service-token: kst_..." https://your-service.example.com/api/health
```

### Creating a token

1. On the **Machines** tab, in the **Service tokens** section, click **Create token**.
2. In the window:
   - **Name** — a name you'll recognise in the access log, for example `github-actions`. 1–64 characters; two tokens of the same service can't have the same name.
   - **Stops working** — optional, but recommended. The token stops working after this time. It can be at most 365 days away. Times are in your account's time zone.
   - **What it can open** — **Whole site** (default) or **Only these pages**. For the latter, write one path per line (at most 50); each path also opens the pages under it (`/reports` opens `/reports/2026` too). Path rules still apply inside these pages.
3. Save with **Create token**.
4. The **Copy your token now** screen opens. The token value is shown **only this once**; Komuta keeps only a digest of it and can never show it again. Take it with **Copy** and keep it in a secret store such as GitHub Actions secrets, never in code. The screen also shows a usage example ("Send it as a header"). Close it with **I saved it**.

If you lose a token, create a new one and delete the old one.

A token looks like `kst_<32 hex characters>_<43 characters>`. Send the value as is, in a single `x-komuta-service-token` header.

### What a token can and can't do

- **It satisfies the sign-in requirement.** A request with a valid token counts as someone who signed in with Komuta; the pages it can open are limited by the token's **What it can open** setting.
- **It can't get past IP rules.** If the site or path requires an IP list, a request with a token must also come from a listed address. Exception: if the rule uses the **Either is enough** combination, the token stands in for sign-in and no address is needed.
- **It can't get past Only chosen people rules.** A path open only to chosen people doesn't open with a token.
- **When the header is present, it decides alone.** If a request has an `x-komuta-service-token` header, the result depends only on the token: an invalid token is refused even if the browser has a valid session.
- **It isn't read where sign-in isn't needed.** On unprotected paths and on places passed without sign-in thanks to the IP list, Komuta doesn't look at the token; the request passes even with an invalid token.
- **It never reaches your application.** Komuta removes the header after checking it; the token value doesn't end up in your application's logs.
- **It needs Komuta sign-in.** Tokens work only while protection requires Komuta sign-in (on the site or in a path rule). If sign-in isn't required, the section says "Service tokens work only while Komuta sign-in is required." and no new token can be created.

### Responses

| Situation | Response |
|---|---|
| Valid token, a page it may open | The request reaches your application. |
| Valid token, a page outside its scope | `403`, body `this service token cannot open this path` |
| Unknown, expired, deleted, malformed or repeated token | `401`, body `invalid service token`, header `WWW-Authenticate: KomutaServiceToken realm="komuta"`. No redirect to sign-in. |

### Tokens in the list and deleting them

The list shows each token's name, the pages it can open ("Only: …" or **Whole site**) and when it stops working ("Until …" or **No end date**). A token that doesn't work shows a **Not working** badge (it may have expired, protection may no longer require sign-in, or the feature may be switched off on the platform). Working tokens have no badge.

To delete a token, click the trash icon and confirm **Delete this token?**. Anything still using the token is refused within about 30 seconds.

### The first token

When the first token of a service is created, Komuta refreshes the service's routing so the header can be checked on the way to the application (nothing is rebuilt). This can take a few minutes, during which the token may look like it doesn't work yet.

### Limits and permissions

- A service can have at most **20** service tokens.
- Creating and deleting tokens needs the **Manage who a protected service is shared with** permission (the same as shares).
- In the access log, pages opened with a token (only `GET` and paths that aren't files) appear as **Service token: {name}**; other requests made with a token, such as `POST`, aren't recorded. Refused token requests are recorded for every method.

---

## Methods and CORS

The **Methods and CORS** section has two settings: a switch that lets browsers' CORS checks through before sign-in, and a list of the HTTP methods each path accepts ("Let browsers send a CORS check before signing in, and allow only the HTTP methods each path needs. A request with any other method is answered 405."). Changes take effect within about a minute ("Saved. The gateway applies it within about a minute.").

### Allow CORS checks without sign-in

When a page on another site calls your service from the browser (for example a front end at `app.example.com` calling an API at `api.example.com`), the browser first sends a CORS check: an `OPTIONS` request without cookies. On a page that needs sign-in, that check would get `401` and the browser would never send the real request. **Allow CORS checks without sign-in** lets these checks through:

- Only a real browser check passes: an `OPTIONS` request with exactly one `Origin` header and exactly one `Access-Control-Request-Method` header naming `GET`, `HEAD`, `POST`, `PUT`, `PATCH` or `DELETE`. Any other `OPTIONS` request still needs sign-in.
- Block rules, the IP allow-list, the country list, the rate limit and the Cloudflare check still apply. (Where sign-in and an IP list are combined with **Either is enough**, the check counts as signed in, so it passes from any address; with **Require both** it must come from a listed address.)
- Method rules are checked against the method the browser asks for, not against `OPTIONS`: with `/api` limited to `GET` and `POST`, a check for `POST` passes and a check for `DELETE` gets `405`.
- Only the check passes. The real request that follows still needs sign-in (cookies, a service token, or an IP list that lets it in), and your application still answers the CORS headers itself.
- It matters only while protection requires Komuta sign-in ("Only matters when Komuta sign-in is on.").

### Method rules

A method rule lists the HTTP methods a path accepts. A request with any other method is answered `405` with the plain text `method not allowed` and an `Allow` header listing the rule's methods (for example `Allow: GET, HEAD`), before sign-in, share links or webhook paths are looked at.

1. Click **Add path**. The first row starts as `/` (the whole site) with `GET` and `HEAD`.
2. Enter the **Path** and tick the **Allowed methods**: `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`.
3. Save with **Save methods** (or undo with **Discard**). **Remove this path** removes a row.

How rules match:

- A rule covers its path and everything below it; `/` covers the whole site.
- The **longest** matching path decides. For example, `/` with `GET`, `HEAD` and `/api` with `GET`, `POST`, `OPTIONS` let `POST /api/orders` through and answer `POST /about` with `405`.
- Paths follow the [path rule syntax](access-protection-rules.md#path-syntax), except that `/` on its own is allowed. Each path can be listed once ("Each path can be listed once."), and each needs at least one method ("Pick at least one method for every path.").
- If a request path can be read in more than one way (encoded characters, `..` and similar), every matching rule must allow the method.
- Komuta's own sign-in path (`/.komuta-access/callback`) is never subject to method rules.
- If there are no method rules, every method is allowed ("Every method is allowed. Add a path to limit it.").
- In the access log, a refusal appears as "Used a method this path does not allow", under the rule's path.

### Limits and permissions

- A service can have at most **50** method rules. They are a separate list and don't count towards the 50 path rules.
- Changing these settings needs the **Manage service access protection** permission.

If you don't see this section, CORS checks and method rules aren't enabled on your platform yet. If settings were saved and the feature is later switched off, the section shows them read-only: "Changing these is not available on this platform yet; the current settings stay in force."

---

## Services that may come in over the private mesh

Komuta's **private mesh** lets your services on different clusters reach each other without going out to the public internet. Private mesh traffic doesn't pass Komuta's access check; it goes straight to the service. That's why on a protected service you choose separately who may come in over the private mesh.

### How it works

- The private mesh itself is turned on and off on the **Private mesh** card of the **Network** tab.
- On a protected service, the services you choose in the **Services that may come in over the private mesh** section of the **Machines** tab reach this service directly **from your other clusters**, without Komuta sign-in or address rules.
- **If the list is empty**, no service can come in directly over the private mesh ("No services chosen. No service can come in directly over the private mesh.").
- **Your services on the same cluster** aren't affected by this list; they keep reaching the service as before.
- If this service's private mesh is off, the list is kept and applies once the private mesh is turned on.
- The list applies only while protection is on. When you turn protection off, the private mesh opens to all your services again as before; the list isn't deleted and applies again when you turn protection back on.

### Adding and removing services

1. Choose another service of your organization in the **Choose a service** list (each appears as "service name · cluster name").
2. Click **Allow**.

To remove one, use the trash icon on its row. Adding and removing are saved immediately, without confirmation.

You may see two warnings in the list:

- **A deleted service** — the service you chose was deleted; you can remove the row.
- **Public, unprotected** — anyone on the internet can open the chosen service, so it can be used as a stand-in to reach this one. Protect it too, or make sure it never forwards what it receives.

### Limits and permissions

- At most **50** services can be chosen; only services of the same organization can be chosen, and a service can't be added to its own list.
- The section is visible only to people with the **Manage service access protection** permission.

> If this feature is not turned on on the platform, the old rule applies: protection can't be turned on for a service that is open to the private mesh, and a protected service can't be opened to the private mesh. The card on the **Network** tab then says "Can't be combined with Access protection". While the feature is on, the card says "Works together with Access protection".

---

## Related Documents

- [Rules](access-protection-rules.md) — protecting the site and paths, countries and the rate limit.
- [Access Log](access-protection-activity.md) — where webhook deliveries and token use appear.
- [End Date and Visitor Identity](access-protection-settings.md) — how a request with a token is introduced to your application.
- [Reference](access-protection-reference.md) — all limits and responses.
