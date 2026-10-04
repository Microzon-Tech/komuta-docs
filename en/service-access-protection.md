# Service Access Protection

Access protection decides who can open your service's public URL. Use it to open an application that is still in development only to your team, a customer, or your office network — without writing a sign-in screen into the application itself.

Access protection is included in every plan. It is managed in two places:

- The **Access protection** card on the **Service Detail → Configuration → Access & ports** page — every setting is here.
- The **Service protection** wizard, opened from the protection card on the service overview — to set up or edit protection step by step (see [Service protection wizard](#service-protection-wizard)).

While protection is off, the card only shows its on/off switch. While the service is open to everyone, the **Set up access protection** button in the **Public exposure** summary at the top of the page turns the card on with **Require Komuta sign-in** already selected. Sign-in, the IP allow-list, path rules and share settings appear when protection is turned on and are hidden again when it is turned off. You need permission to edit the service to change the card; without it, the card is read-only.

---

## Protection Types

The **Who can get in** section of the card has two controls that apply to the whole site. You can turn on either one or both. On top of these, you can add [path rules](#path-rules) for specific paths.

| Control | What it does |
|---|---|
| **Require Komuta sign-in** | Visitors are sent to Komuta to sign in first. Only accounts matching a share in the **Shared with** list get in. |
| **IP allow-list** | The service can only be reached from the public addresses in the list. Visitors from other addresses see the **Access restricted** page (HTTP 403), unless **Either is enough** is chosen. |

How the controls combine:

- **Both on** — the card asks how they combine (**How should sign-in and the IP allow-list combine?**).
  - **Require both** (default) — the visitor must come from a listed address **and** sign in with an account that matches a share. Other addresses get 403.
  - **Either is enough** — visitors from listed addresses get in without signing in; visitors from other addresses don't get 403, they are sent to Komuta sign-in and get in if they match a share. Use it for sign-in-free access from the office network and sign-in from anywhere else.
- **IP allow-list only** — visitors from listed addresses get in without signing in; everyone else gets 403.
- **Komuta sign-in only** — every account that matches a share gets in, from any address.
- **Both off, with path rules** — the site itself stays open to everyone; only the paths in your path rules are protected (the **Public exposure** summary at the top of the page shows "Specific paths are protected").
- **Both off, no path rules** — there is nothing to protect; after a **Turn off access protection?** confirmation, applying turns protection off and the service opens to everyone.

Settings are saved with the **Apply protection** button.

---

## IP Allow-List

Enter one IPv4 or IPv6 address or CIDR range per line (for example `8.8.8.8` or `8.8.8.0/24`). Rules:

- Only public internet addresses are accepted. Private and reserved ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `100.64.0.0/10`, loopback, link-local, documentation ranges, `fc00::/7` and so on) and ranges that overlap them are refused.
- The host bits of a range must be zero: `8.8.8.0/24` is valid, `8.8.8.5/24` is not.
- `0.0.0.0/0` and `::/0`, which cover every address, are refused; if you want the service open to everyone, turn protection off.
- The list holds at most 100 entries. An entry written twice is saved once.

### Add my IP

The **Add my IP** button adds the public address the Komuta console sees for you:

- On an IPv4 connection, only your address is added.
- On an IPv6 connection, your address can change within its `/64` block, so the whole `/64` block is added.

The added address is the one the console sees. Your browser may reach the service over a different connection (for example IPv6 to the console, IPv4 to the service); in that case the service sees another address, and the card warns you about it. Check the address before applying so you don't lock yourself out; if needed, add both your IPv4 and IPv6 addresses.

If you are locked out, the **Access restricted** page shows the address the service saw. You can copy it and add it to the list from the console.

### Cloudflare requirement

The IP allow-list (for the site or in a path rule) requires requests to the service to go through Cloudflare (proxied). Komuta-generated `*.komuta.app` hosts and custom domains added under **Domains** work this way. While the IP allow-list is on, requests that don't come through Cloudflare are refused (with **Either is enough** they are sent to sign-in instead).

---

## Path Rules

Path rules protect specific paths without touching the rest of the site: for example, opening `/admin` only to your signed-in team while the site is public, closing `/internal` completely, or limiting `/ops` to the office network. Add them with **Add path rule** in the **Path rules** section of the card and save with **Apply protection**.

Each rule has a **Path** and a **Protection** type:

| Protection | What it does |
|---|---|
| **Block completely** | Nobody can reach this path or the paths under it (HTTP 403). Signing in or coming from an allowed address doesn't grant access either. |
| **Komuta sign-in** | Visitors to this path must sign in with Komuta; only the people and organizations you shared the service with get through. |
| **IP list** | Only the addresses in the rule's own list can reach this path. |
| **IP list and Komuta sign-in** | The rule's own IP list and sign-in are used together; choose **Require both** or **Either is enough** (same meaning as for the site). |

### Rules only tighten

A request must pass the site's controls **and** every rule that matches its path. A path rule can't remove something the site requires; it only adds conditions. Examples:

- The site requires **Komuta sign-in** and `/ops` has an **IP list** rule → to reach `/ops` a visitor must sign in and come from a listed address.
- If the whole site already requires sign-in, a **Komuta sign-in** rule for `/admin` changes nothing; the card says so under the rule.
- If `/internal` is **Block completely**, someone with a session or from an allowed address also gets 403.

### Path matching

- A rule matches the path itself and the paths under it: a `/admin` rule matches `/admin` and `/admin/settings`, not `/administrator`.
- Matching is case-insensitive; if you type `/Admin/` it is saved as `/admin`.
- The query string (`?x=1`) doesn't affect matching.
- Encoded characters (`%61dmin`), `//`, `/./`, `/../`, `;parameters` and `\` can't be used to get around a rule; Komuta checks the request in every form your server could read it. On a service with path rules, requests whose path is longer than 1024 characters, too convoluted to read, or containing control characters are refused with HTTP 400.

### Path format and limits

- A path must start with `/` and may only contain lowercase letters, digits and `- . _ ~ ! $ & ' ( ) * + , = : @ /`. Non-ASCII characters and spaces are not allowed.
- Empty segments (`//`), `.` and `..` segments and segments ending in `.` are not allowed.
- `/` on its own can't be a rule; use the **Who can get in** settings above for the whole site.
- Paths starting with `/.komuta-access` are reserved for Komuta.
- A path can be at most 256 characters, and each path can have only one rule.
- A service can have at most 50 path rules.

The card shows a problem with a rule as you type, and **Apply protection** stays disabled until it is fixed.

---

## Komuta Sign-In and Shares

With **Require Komuta sign-in** on (or when a visitor reaches a path that requires sign-in), a visitor who opens your service is sent to the Komuta sign-in page. After signing in they see a "Checking your access" screen, and if their account matches a share they are taken back to the page they wanted to open.

Who can get in is set with **Add share** in the **Shared with** list at the bottom of the card:

| Share type | Who gets in |
|---|---|
| **Your organization** | Every active member of this organization. |
| **A member** | One person in this organization. |
| **A linked organization** | Everyone in another organization you belong to, including people who join later. |
| **An email address** | Someone outside your organizations. They sign in to Komuta with any account, then confirm the address with a one-time code sent to it. |

Things to keep in mind:

- You need to turn on access protection before you can add shares.
- Shares only apply while something requires sign-in: **Require Komuta sign-in**, or a path rule that includes **Komuta sign-in**.
- If sign-in is on but there are no shares, nobody can get past sign-in.
- Each share can have an optional **Access ends** date. Leave it empty to keep access until you remove it. You can change the date later with the pencil icon in the list; an expired share is shown with the **Expired** label.
- A service can have at most 200 shares.

### Email shares

A person the service was shared with by email signs in to Komuta with any account when they open the service. In the "Shared with your email address?" section of the **You don't have access** screen, they enter their address and choose **Email me a code**. An 8-digit code, valid for 10 minutes, is sent to the address; after entering it and choosing **Verify and continue**, the service opens.

Only someone who can read that mailbox gets in, and they need a new code each time they sign in. Code requests are limited per hour.

### External sharing permission

**A linked organization** and **An email address** shares require your organization to allow sharing outside it. An organization admin turns this on with the **Account → Organizations → Allow external sharing** setting.

While it is off, the **Add share** dialog, the wizard's **Who can sign in?** step and the share list say so. If you can change the setting, the **Open organization settings** button opens it in a new tab with the setting highlighted; when you turn it on and come back, the options become available without reloading the page. If you can't, ask an organization admin to turn it on.

If it is turned off, these shares can't be added; existing ones are suspended and shown with the **Suspended** label, and people who signed in through them lose access. When it is turned back on, suspended shares apply again; for linked organization shares to come back, the link between the organizations must still exist.

### Removing access

When you remove a share, the people it covered lose access within about 30 seconds, including sessions that are already open. Turning protection off does the opposite: the service becomes reachable by everyone again.

Turning protection off does not delete your shares; if you turn protection back on, the same shares apply again. Sign-in, IP allow-list and path rule settings are not kept; choose them again when you turn protection back on.

---

## What Visitors See

| Situation | What the visitor sees |
|---|---|
| Signed in and matches a share | The service opens. |
| Signed in but matches no share | The **You don't have access** page. |
| Not coming from an address in the IP allow-list (**Require both**, or IP allow-list only) | The **Access restricted** page (HTTP 403). |
| Not coming from an address in the IP allow-list (**Either is enough**) | The Komuta sign-in page; the service opens if they sign in and match a share. |
| Opening a path with a **Block completely** rule | HTTP 403, "access denied". |
| Opening a path that requires an IP list from an address not in it (unless the rule uses **Either is enough**) | The **Access restricted** page (HTTP 403). |

The **You don't have access** page shows the signed-in account and organization. From there the visitor can verify with an email code, choose **Continue with another organization** to switch to another organization they belong to, or choose **Sign in with another account**.

The **Access restricted** page says the service can only be opened from allowed networks and shows the address the service saw, with a copy button. No sign-in option is offered on this page; if the address is not in the list, signing in does not grant access either. With **Either is enough**, the sign-in page appears instead of this page.

For non-browser clients: on a service (or a path) that requires sign-in, requests without a session that are not `GET`/`HEAD` (for example a `POST` to an API) are not redirected to sign-in; they receive HTTP 401. For services that need machine-to-machine access, the IP allow-list is a better fit.

---

## Protection End Date

While protection is on, the card shows a **Protection ends** section. Here you choose when protection ends and what happens then:

- **Then keep it locked** — when the end date arrives, protection stays as it is; the service is not opened until you decide.
- **Then open it to everyone** — when the end date arrives, protection is removed and the service opens to everyone.

The end date is optional and can be at most 365 days away. Times are in the time zone chosen in your settings. Protection without an end date stays on until you turn it off.

For protection with an end date, the people who can edit the service (or the organization admins, if there is nobody else) are emailed one day and one hour before the end, and again when it ends. The links in the email go straight to **Open to everyone now**, **Extend** or **Make permanent**.

Other buttons in this section:

- **Make permanent** — removes the end date; the service stays protected until you turn protection off.
- **Open to everyone now** — removes protection right away: sign-in, the IP allow-list and path rules stop applying, and anyone with the URL can open the service. Your shares are kept.

---

## Service protection wizard

On the service overview, click the **Service protection** card (**Set up protection**, or **Manage protection** when protection is on). The window shows the current protection; **Set up protection** (**Edit protection** when protection is on) starts the wizard. It asks for the same settings as the card, step by step; nothing changes until you review and apply.

1. **Protection method** — **Komuta sign-in**, **Specific IP addresses**, **Sign-in and specific IPs** (choose **Require both** or **Either is enough** here), or **Path rules only** (offered when path rules are available).
2. **Allowed addresses** — for methods that use IPs.
3. **Path rules** — optional; at least one rule is required with **Path rules only**.
4. **Who can sign in?** — when the site or a path rule requires sign-in; shares are added here.
5. **Duration** — the protection end date and what happens when it is reached (**Keep the protection rules** or **Open to everyone**).
6. **Review changes** — saved with **Apply protection**.

---

## Status and Progress

The badge in the top-right corner of the card shows the protection status:

| Status | Meaning |
|---|---|
| **Off** | No protection; the service is open to everyone. |
| **Preparing** | Protection is being prepared. |
| **Applying** | Protection is being applied to the service. |
| **Protected** | Protection is in effect. |
| **Turning off** | Protection is being removed. |

While protection is being turned on, the card shows a progress bar, the current step and the time of the last status change. If a change you made to a protected service is still rolling out, the card shows "Your latest change is being rolled out." Wait for the card to show **Protected** to be sure the setting is fully in effect.

If an attempt to apply protection fails, the card shows the error; Komuta retries automatically.

If the service's protection was changed elsewhere (another tab or a teammate), the card loads the latest settings and asks you to make your change again.

---

## How It Works With Other Features

- **Sleep mode** — a sleeping service is woken only after the visitor passes the checks. Visitors who haven't signed in, or who are not on the allow-list, cannot wake a protected service.
- **Caching** — responses of protected services are never cached by Cloudflare. The **Cache purge** step shown on the card while protection is applied clears content that was cached earlier.
- **Domains** — protection applies to both the service's `*.komuta.app` host and the custom domains added under **Domains**.

---

## Requirements

- The service must have a public URL; a service without one has nothing to protect.
- The service must run on Komuta's shared hosting clusters.
- The IP allow-list requires requests to the service to go through Cloudflare (see [Cloudflare requirement](#cloudflare-requirement)).

---

## Related Documents

- [Access & Ports](services-ports.md) — The page that holds the access protection card.
- [Ingress and Domains](ingress-domains.md) — `*.komuta.app` hosts and custom domains.
- [Service Sleep Mode](service-sleep.md) — Putting idle services to sleep and waking them up.
