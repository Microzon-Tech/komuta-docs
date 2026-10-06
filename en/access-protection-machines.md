# Machines and Private Mesh

Komuta sign-in is for people: a sign-in page opens in a browser and the person signs in with their account. Some requests, though, come from a program rather than a person. GitHub tells your application when someone pushes, Stripe sends a notification when a payment happens, your CI pipeline runs a health check after a deploy, a monitoring tool polls your site every minute. These programs can't use a sign-in page.

Access protection offers three ways for such requests:

| Way | For | Where |
|---|---|---|
| **Webhook path** (open path) | Senders that can't sign in and send their own signature (such as GitHub, Stripe, Slack) | **Machines** tab → **Webhooks** |
| **Service token** | Programs you control (CI jobs, monitoring tools, scripts) | **Machines** tab → **Service tokens** |
| **Services that may come in over the private mesh** | Your Komuta services on other clusters reaching this service directly | **Machines** tab → **Services that may come in over the private mesh** (the private mesh itself is on the **Network** tab) |

The webhook and service token sections appear only while access protection is on. If protection is off, the tab shows the **Access protection is off** notice and a **Go to Rules** button. The private mesh list is the one exception: if the service's private mesh is on, the list appears and can be filled in as soon as you start turning protection on on the **Rules** tab (before saving).

---

## Webhook paths

A webhook path opens a specific path of the site without Komuta sign-in. For example, while the site is open only to your team, requests GitHub sends to `/webhooks/github` reach your application without signing in.

> **Important:** Komuta doesn't check who sends requests on a webhook path; it only opens the door. **Your application** must check that a request really comes from GitHub or Stripe by verifying the sender's signature. For example, GitHub sends an `X-Hub-Signature-256` header and Stripe a `Stripe-Signature` header. If your application doesn't verify the signature, anyone can send requests to this path.

### Opening a webhook path

1. On the **Machines** tab, in the **Webhooks** section, click **Open a path**.
2. In the **Open a path for webhooks** window:
   - **Path** — the path to open, for example `/webhooks/github`. This path and everything below it is opened.
   - **Methods** — the HTTP methods accepted on this path: `POST`, `PUT`, `PATCH`, `DELETE`, `GET`, `HEAD`. The default is `POST` only; at least one method must be chosen.
   - **Sender addresses (optional)** — one IP address or CIDR range per line. If you fill it in, only requests from these addresses get onto the path. If you leave it empty, requests from any address are accepted; the signature check still protects you. You can write ranges the sender publishes here, such as GitHub's webhook addresses (the window shows `140.82.112.0/20` as an example).
3. Save with **Open the path**. The change takes effect within a few seconds ("Webhook paths are being applied").

The list shows each open path with its methods, its sender list ("Only from: …" or "From any address") and the service's full address. To close a path, click the trash icon on its row; it is closed immediately, without confirmation.

### How webhook paths work

- **No sign-in is asked.** Requests with the chosen methods pass without Komuta sign-in.
- **Only the chosen methods are open.** A request with another method is decided as if there were no open path, so it has to pass the site's normal protection.
- **The site's IP list and parent path rules don't apply.** On an open path, the only address check is the path's own **Sender addresses** list.
- **Block rules still apply.** Only a **Block completely** rule can sit under an open path; for example, with `/webhooks` open, `/webhooks/old` can still be blocked. Sign-in, IP or people rules can't go under an open path. The webhook window refuses a path that overlaps an existing rule with "This path overlaps the rule for /admin. An open path can't share its place with another rule." (`/admin` in the example); if you try to save a rule under an open path on the **Rules** tab, you get a "… sits under the open path …" error.
- **Path matching is case-insensitive.** A request to `/HOOKS/x` is under `/hooks`. A request is opened only when every way its path can be read stays under the open path; `%2f`, `..` and similar tricks can't be used to escape from an open path to another path.
- **Open paths can't be nested.** A second open path can't be opened on, or under, an existing open path.
- **Method-override headers are ignored.** Komuta looks at the request's real method. Your application shouldn't honour headers such as `X-HTTP-Method-Override` or `X-HTTP-Method` on these paths; otherwise requests that act like `DELETE` could be sent to a path you opened only for `POST`.
- **CORS preflight requests (`OPTIONS`) don't pass an open path.** `OPTIONS` isn't one of the methods you can choose. A webhook path isn't suitable for endpoints that a browser must call from another site.
- **The access log** records every delivery to an open path as **Request on an open path** / **Delivered to an open path**, under the open path's prefix rather than the full path.

### Limits

- A service can have at most **10** open paths. When the limit is reached, **Open a path** is disabled. Open paths also count towards the total limit of 50 path rules, together with the path rules on the **Rules** tab.
- `/` (the whole site) and Komuta's own sign-in path (anything starting with `/.komuta-access`) can't be opened.
- The path follows the same [syntax as path rules](access-protection-rules.md#path-syntax) (at most 256 characters). The window shows the same message for every invalid path: "Enter a path like /webhooks/github. The whole site can't be opened."
- The sender list takes at most 100 entries and only public internet addresses. If the list has an invalid line, **Open the path** stays disabled.
- Opening and closing webhook paths needs the **Manage service access protection** permission; others only see the list.
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

- [Rules](access-protection-rules.md) — protecting the site and paths.
- [Access Log](access-protection-activity.md) — where webhook deliveries and token use appear.
- [End Date and Visitor Identity](access-protection-settings.md) — how a request with a token is introduced to your application.
- [Reference](access-protection-reference.md) — all limits and responses.
