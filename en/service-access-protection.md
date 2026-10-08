# Access Protection

Access protection decides who can open your service's address on the internet. Use it to open an application that is still in development only to your team, a customer or your office network; to close an admin panel; or to open a page to chosen people at chosen times. You don't need to build a sign-in screen into your application or change its code.

Access protection is included in every plan.

---

## At a glance

With access protection you can:

- **Require Komuta sign-in.** Visitors sign in with a Komuta account; only the people, organizations and email addresses you share the service with get in.
- **Limit by IP address.** The service opens only from networks you choose (for example your office). It can be combined with sign-in as "require both" or "either is enough".
- **Protect path by path.** Keep the site open while `/admin` is open only to signed-in people, close `/internal` completely, or open `/reports` to chosen people at chosen times.
- **Let machines in.** Open paths that don't ask for sign-in for webhook senders such as GitHub and Stripe; give CI and monitoring tools a service token.
- **Set an end date.** End protection on a date; when it ends, keep the service locked or open it to everyone.
- **See who got in.** The access log shows sign-ins, the pages people opened and the visitors who were refused.
- **Tell your application who it is.** The signed-in visitor's email and a signed proof of identity can be passed to your application as headers.

---

## How it works

Every request to a protected service passes a check at the Komuta gateway before it reaches your application:

1. A visitor opens your service's address.
2. Komuta checks the request against your rules: is the IP address on the list, does the visitor have a valid session for this service, is there a special rule for this path?
3. If the rules are met, the request is passed to your application. If sign-in is needed, the visitor is sent to the Komuta sign-in page. If access is not allowed, the visitor sees an explanatory page and the request never reaches your application.

The check can't be bypassed. When protection is turned on, Komuta also locks your service's pods: they accept only requests that come from the gateway, having passed the check. A request that tries to skip the gateway and reach a pod directly is refused. (Your own services on the same cluster, and the services you chose for the private mesh on the **Machines** tab, keep reaching your pods directly.)

Protection applies to all public addresses of the service together: the `*.komuta.app` address Komuta gives you, custom domains you added under **Domains**, and the preview address of a blue-green deployment.

---

## Quick start

These recipes cover the most common setups. All settings are on **Service Detail → Configuration → Access & ports**. For a step-by-step walkthrough that explains why each step is done and what it changes, up to the most advanced scenario, see the [Setup Guide](access-protection-tutorial.md).

### Only my team can open it

1. On the **Rules** tab, turn on the **Access protection** switch. **Require Komuta sign-in** comes selected.
2. Press **Apply protection**. If you have permission to manage shares, you are added to the share list automatically.
3. On the **People** tab, choose **Add share → Your organization**.

Every member of your organization can sign in with their Komuta account and open the service; everyone else stays at the sign-in page.

### Show it to a customer

1. Make sure your organization allows external sharing: **Account → Organizations → Allow external sharing** (off by default).
2. Turn protection on as above.
3. On the **People** tab, choose **Add share → An email address** and type the customer's address. Setting an **Access ends** date is a good idea.

The customer signs in with any Komuta account (created in seconds with Google or GitHub), then confirms the address with the 8-digit code sent to it.

### Only from the office network

1. On the **Rules** tab, turn protection on and turn **Require Komuta sign-in** off.
2. Type your office's public IP address or range into the **IP allow-list** (use **Add my IP** if you are connecting from there).
3. Press **Apply protection**.

Visitors from the listed addresses get in without signing in; everyone else sees the **Access to this service is restricted** page.

### No sign-in from the office, sign-in from outside

Turn on **Require Komuta sign-in**, enter your office addresses in the **IP allow-list**, and choose **Either is enough** for "How should sign-in and the IP allow-list combine?".

### Keep the site open, protect only the admin panel

1. Turn protection on and turn **Require Komuta sign-in** off.
2. In the **Path rules** section, **Add path rule**: path `/admin`, protection **Komuta sign-in**.
3. Press **Apply protection**, then add the people who use the panel on the **People** tab.

### Let a GitHub webhook in too

On a protected service, on the **Machines** tab, use **Webhooks → Open a path** to open `/webhooks/github` for `POST`, and verify GitHub's signature (`X-Hub-Signature-256`) in your application.

---

## The screen: Access & ports

Access protection is managed in tabs on **Service Detail → Configuration → Access & ports**:

| Tab | Contents | Guide |
|---|---|---|
| **Overview** | A summary of the service's current exposure, how traffic reaches the service, and recent access events. | This page |
| **Rules** | The protection switch, status, **Who can get in** (Komuta sign-in, IP allow-list), **Path rules**, **Access preview**. | [Rules](access-protection-rules.md) |
| **People** | Who the service is shared with. | [Sign-in and Sharing](access-protection-sign-in-sharing.md) |
| **Machines** | Service tokens, webhook paths and services that may come in over the private mesh. | [Machines and Private Mesh](access-protection-machines.md) |
| **Activity** | The access log. | [Access Log](access-protection-activity.md) |
| **Network** | Public addresses (public URL), ports and the private mesh. | [Machines and Private Mesh](access-protection-machines.md#services-that-may-come-in-over-the-private-mesh) |
| **Settings** | Protection end date, telling your application who signed in, and **Open to everyone now**. | [End Date and Visitor Identity](access-protection-settings.md) |

The tab is kept in the URL as `?tab=` (`rules`, `people`, `machines`, `activity`, `network`, `settings`), so you can share a link to a tab.

The **Public exposure** card on the **Overview** tab states the situation in one line: **Open to everyone**, **Protection is being applied**, **Protected**, **Specific paths are protected**, **Protection is being removed**, **Protection can't finish** (the private mesh is on and skips the access check), **Not on the internet**, **Private mesh only** (no public address; the service is reached only over the private mesh) or **Access status unavailable**. Below it, the active settings are listed as short chips (for example "Komuta sign-in", "3 allowed addresses", "2 path rules", "5 shares", "Until …"). While the service is open to everyone, **Set up access protection** takes you to the **Rules** tab and starts turning protection on. If there is any warning (including ones you need to fix), a **Needs attention** badge appears next to the headline.

If you can't view the service's protection, only the **Overview** and **Network** tabs are shown. If access protection is turned off for your organization, services that aren't protected also show only these two tabs; services that are already protected stay protected and can still be managed.

---

## Turning protection on

1. On the **Rules** tab, turn on the switch in the header of the **Access protection** card. This doesn't save anything yet; the settings open with **Require Komuta sign-in** selected.
2. Choose the checks you want: Komuta sign-in, the IP allow-list, path rules or a combination (see [Rules](access-protection-rules.md)). At least one is needed.
3. Press **Apply protection** ("Access protection is being applied").
4. If you chose Komuta sign-in, share the service on the **People** tab. When a save makes sign-in required, you are added automatically if you have permission to manage shares.

To back out, press **Cancel** or turn the switch off; the unsaved draft is discarded.

You can also set protection up step by step from the **Service protection** card on the service overview (see [Service protection wizard](#service-protection-wizard)).

---

## How long it takes

- **Turning it on the first time** usually takes a minute or two. Komuta refreshes the service's routing and applies the pod lock; this may show up as a deployment, but nothing is rebuilt.
- The check takes effect shortly after protection reaches **Applying**, once the service's routing has been refreshed. During **Preparing** the service still runs as before (open to everyone).
- **On a protected service**, changes to rules, the IP list and shares usually take effect within a few seconds to half a minute. Meanwhile the card says "Your latest change is being rolled out."
- **Turning it off** also finishes within a few seconds to a couple of minutes; the check is removed in the last step of **Turning off**.

---

## Status

The header of the card on the **Rules** tab shows a status badge (no badge is shown while protection is off; the switch is just off):

| Badge | Meaning | Check active? |
|---|---|---|
| **Off** | No protection; the service is open to everyone. | No |
| **Preparing** | Your settings are being written to the gateway. | No |
| **Applying** | The check is active; the pod lock and cache purge are being completed. | Yes (a short delay is possible in the first seconds while routes refresh) |
| **Protected** | Protection is fully in place. | Yes |
| **Turning off** | Protection is being removed; once done, anyone with the URL can open the service. | Yes, until the last step |

While protection is being turned on, the card shows a 5-step progress bar ("{done} of {total} steps completed"). The steps are, in order: preparation, the gateway check, the pod lock, the cache purge and protected; the bar disappears once protection is complete. Along the way Komuta tests protection itself: it confirms that a request without sign-in is really refused, that a signed-in request gets through and that protected responses are not cached.

If someone changed the same protection before you (in another tab, or a teammate), the card loads the latest settings and says "Access protection was changed elsewhere. The latest settings are loaded — make your change again." This way nobody overwrites someone else's change without knowing.

---

## Warnings and what to do

If protection can't be completed or something goes wrong, a warning box appears on the **Rules** tab (and at the top of the other tabs), with the technical code below it ("Code: …"). Komuta doesn't count temporary waiting conditions as errors for 15 minutes, and retries automatically in every case.

### Things you need to fix

Box title: "Protection can't be completed until this is fixed."

| Code | Message (first sentence) | What to do |
|---|---|---|
| `service_not_deployed` | This service has not been deployed yet. | Deploy the service. |
| `no_hosts` | This service has no public address that can be protected. | Turn on **Public URL** on the **Network** tab. |
| `too_many_hosts` | This service has more domains than one protection policy can cover. | Remove custom domains you no longer need (limit: 50 addresses). |
| `host_invalid` | One of this service's route hostnames is not a valid domain name. | Fix or remove the domain. |
| `host_not_owned` | This service routes a domain that isn't one of its verified domains, so protection cannot include it. | Verify the domain for this service, or remove it. |
| `host_claimed_elsewhere` | One of this service's domains is already protected by another service. | Remove the domain from one of the two services. |
| `host_not_public` | One of this service's domains points to a private or reserved address. | Point the DNS record to Komuta, or remove it. |
| `host_dns_unresolved` | One of this service's domains does not resolve in DNS yet. | Check the DNS record; protection continues on its own once it resolves. |
| `custom_domain_o2o` | A custom domain is proxied through your own Cloudflare account. | In your own Cloudflare account, set that record to **DNS only** (grey cloud). |

### Things Komuta takes care of

- **"Protection is not fully in place."** (`exposed`, `drift_route_filter`, `drift_pod_token_lock`) — The service may be reachable without the access check right now. Komuta has been alerted and is restoring protection. If you want to close a sensitive service completely meanwhile, you can turn the public URL off temporarily on the **Network** tab.
- **`bypass_policy_present`** — Another network rule lets traffic reach the service around the access check. If you just turned off the private mesh, this clears on its own once the change rolls out.
- **`drift_mesh_peers`** — The private mesh still lets in more services than you chose; this clears on its own within a few minutes.
- **Other codes** — "A platform-side problem stopped protection from being applied. Nothing is needed from you."

In these cases, as in the cases you need to fix above, the card on the **Overview** tab shows a **Needs attention** badge.

### When it can't be turned on

| Message on screen | Why | What to do |
|---|---|---|
| "Access protection works through the public URL. Turn on the public URL first." | The service's public URL is off. | Turn on **Public URL** on the **Network** tab. |
| "This service has no public address yet. You can turn on access protection after its first deployment." | The service has never been deployed. | Deploy the service. |
| "Locked while Private mesh is on" | The private mesh is on, and using the private mesh together with protection is turned off on the platform. | Turn off the private mesh, or see [private mesh](access-protection-machines.md#services-that-may-come-in-over-the-private-mesh). |
| "Access protection is only available for services running on Komuta's shared hosting clusters." | The service runs on your own cluster or an unsupported cluster. | — |

Job and cron job services have no public URL, so they have no access protection.

---

## Turning protection off

There are two ways, with the same result:

- **Open to everyone now** — in the **Remove protection now** section at the bottom of the **Settings** tab. Turning off the switch on the **Rules** tab opens the same confirmation. Confirmation: **Open this service to everyone?** ("The service is being opened to everyone").
- **Removing every rule** — if you remove sign-in, the IP list and all path rules on the **Rules** tab, the button turns into a red **Turn off protection** and asks **Turn off access protection?**.

The check is removed in the last step of **Turning off**; after that, anyone with the URL can open the service.

**What is kept, what is not:**

| Kept (applies again when you turn protection back on) | You need to set again |
|---|---|
| Shares (including suspended ones) | The Komuta sign-in choice |
| Service tokens | The IP allow-list |
| The list of services that may come in over the private mesh | Path rules and webhook paths |
| Telling your application who signed in (comes back on by itself when you turn protection on again with Komuta sign-in; switched off if the new protection doesn't ask for sign-in) | The protection end date |

The access log is not kept, and can't be viewed, while protection is off; earlier records are kept for 30 days and show up on the **Activity** tab again if you turn protection back on.

---

## Service protection wizard

On the service overview (dashboard), the **Service protection** card shows the protection status and a short summary (for example "Komuta sign-in · No IP restriction"). Clicking the card opens the **Current protection** window, which also contains the share list, the protection end date and **Open to everyone now**. **Set up protection** (or **Edit protection** if protection is on) starts the wizard:

1. **Protection method** — **Komuta sign-in**, **Specific IP addresses**, **Sign-in and specific IPs** (where you choose **Require both** or **Either is enough**) or **Path rules only**.
2. **Allowed addresses** — for methods that use IPs.
3. **Path rules** — optional; at least one rule is needed if you chose **Path rules only**.
4. **Who can sign in?** — when sign-in is needed and you can manage shares; shares are added here. You are added to the list automatically ("(you)"); remove yourself if you shouldn't have access.
5. **Duration** — the protection end date and what happens when it is reached (**Keep the protection rules** or **Open to everyone**).
6. **Review changes** — current and new settings side by side; save with **Apply protection**.

Nothing changes until you review and apply. If a step fails while saving, the successful steps are kept and the service is never opened to everyone on its own. Webhook paths, service tokens, the access log and visitor identity are only on the **Access & ports** page (the window links to it as **Advanced settings in Access & ports**).

The wizard doesn't let you set up protection while the service's private mesh is on; use the **Access & ports** page in that case.

---

## How it works with other features

- **Sleep mode** — A sleeping protected service is woken up only after a visitor passes the checks. Visitors who haven't signed in, or who aren't on the allow-list, can't wake a protected service. Protection applies while the service sleeps too.
- **Caching** — Every response of a protected service gets the header `Cache-Control: private, no-store` (replacing the value your application sends), and Cloudflare doesn't cache the service's responses. Previously cached content is purged when protection is turned on. So a protected page can never be shown from a cache to someone who hasn't signed in.
- **Domains** — Protection applies to the service's `*.komuta.app` address and every active custom domain added under **Domains**. When you add a new domain, protection covers it too. Your custom domain's DNS record must not be proxied (orange cloud) in your own Cloudflare account; it must be **DNS only**.
- **Blue-green deployment** — The preview address is protected too; visitors sign in to the preview address separately.
- **Private mesh** — Private mesh traffic doesn't pass the gateway. On a protected service you choose on the **Machines** tab which services may come in directly from your other clusters (see [Machines and Private Mesh](access-protection-machines.md#services-that-may-come-in-over-the-private-mesh)).
- **Your application** — Komuta's session cookies and the service token header are removed from the request; your application doesn't see them. Every allowed request carries the `x-komuta-access` header that Komuta adds for the pod lock. It is a secret value specific to your service: you don't need to use it, so don't log it or forward it anywhere.
- **Not supported** — Browser CORS preflight requests (`OPTIONS`) from another site carry no session, so they can't pass paths that require sign-in; `OPTIONS` can't be chosen on webhook paths either. Visitors signing in with your own identity provider (SSO) is not supported at the moment; visitors need a Komuta account.

---

## Requirements and permissions

- The service needs a public URL and at least one public address.
- The service must run on Komuta's shared hosting clusters.
- Wherever an IP list is used, requests must come through Cloudflare; Komuta addresses and custom domains added to Komuta work this way.
- Access protection is on by default for organizations. If it has been turned off for your organization, no new protection or share can be added ("Access protection is turned off for this organization. Contact Komuta support to turn it on."); existing protections keep working.

| Permission | What it allows |
|---|---|
| **Manage service access protection** | Turning protection on and off; Komuta sign-in, the IP allow-list, path rules, webhook paths, the private mesh list, the end date, visitor identity; **Open to everyone now**. |
| **Manage who a protected service is shared with** | Managing shares and service tokens. |

Both also need edit access to the service. People without the permission see the settings read-only ("Read-only — you need permission to manage access protection to change these settings.").

---

## Access protection guides

- [Setup Guide](access-protection-tutorial.md) — step-by-step setup from quick start to the most advanced scenario.
- [Rules](access-protection-rules.md) — Komuta sign-in, the IP allow-list, path rules, the access preview.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — shares, email shares, external sharing, what visitors see.
- [Machines and Private Mesh](access-protection-machines.md) — webhook paths, service tokens, the private mesh.
- [Access Log](access-protection-activity.md) — the **Activity** tab.
- [End Date and Visitor Identity](access-protection-settings.md) — protection end date, reminders, passing identity to your application, JWT verification.
- [Reference](access-protection-reference.md) — limits, responses, headers, error messages, FAQ, glossary.

Related: [Service Ports](services-ports.md) — the ports part of the **Access & ports** page.
