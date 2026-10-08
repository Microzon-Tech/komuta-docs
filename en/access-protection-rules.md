# Rules: Who Can Get In

Rules decide who can get into your service. They are on the **Rules** tab of the **Access & ports** page, in the **Access protection** card. The protection switch and status badge are in this card's header too (see [Access Protection](service-access-protection.md#turning-protection-on)).

Rules have two layers:

1. **For the whole site** — the **Who can get in** section: Komuta sign-in, the IP allow-list and how the two combine.
2. **For specific paths** — the **Path rules** section: opening `/admin` only to signed-in people, closing `/internal` completely, opening `/reports` only to chosen people at chosen times, and so on.

Below them, two more sections apply to every request before anyone signs in: [Countries](#countries) and the [Rate limit](#rate-limit). The HTTP methods each path accepts are set on the **Machines** tab (see [Methods and CORS](access-protection-machines.md#methods-and-cors)).

Changes you make in the card are not saved until you press **Apply protection**. **Countries** and **Rate limit** have their own save buttons. The summary box at the bottom of the card describes in one sentence what is active **Right now** and, if there are unsaved changes, what happens **After you apply**.

---

## Who can get in

This section has two checks that apply to the whole site. One, both or neither can be on (if neither is on, at least one path rule is needed).

### Require Komuta sign-in

When it is on, visitors are sent to sign in with Komuta before they can open your service. Only accounts matching a share on the **People** tab get in. Someone who signs in but matches no share sees the **You don't have access** page.

What visitors see during this, and how shares work, is described in [Sign-in and Sharing](access-protection-sign-in-sharing.md).

### IP allow-list

Only the public internet addresses on the list can reach the service. Write one IPv4 or IPv6 address or CIDR range per line:

```text
8.8.8.8
8.8.8.0/24
2001:4860::/32
```

Rules:

- **Only public internet addresses are accepted.** Private, reserved and documentation ranges, and ranges that overlap them, are refused: `0.0.0.0/8`, `10.0.0.0/8`, `100.64.0.0/10`, `127.0.0.0/8`, `169.254.0.0/16`, `172.16.0.0/12`, `192.0.0.0/24`, `192.0.2.0/24`, `192.168.0.0/16`, `198.18.0.0/15`, `198.51.100.0/24`, `203.0.113.0/24`, `224.0.0.0/4`, `240.0.0.0/4`, `::/128`, `::1/128`, `::ffff:0:0/96`, `64:ff9b:1::/48`, `100::/64`, `2001:db8::/32`, `fc00::/7`, `fe80::/10`, `ff00::/8`.
- **The host bits of a range must be zero.** `8.8.8.0/24` is valid, `8.8.8.5/24` is not.
- **Ranges that cover every address are refused.** `0.0.0.0/0` and `::/0` are rejected; to open to everyone, turn protection off.
- A single address is stored as `/32` (IPv4) or `/128` (IPv6). Numbers with leading zeros (`08.8.8.8`), IPv6 addresses with a `%` zone and the IPv4-mapped IPv6 form (`::ffff:8.8.8.8`) are not accepted.
- Entries can also be separated by commas. An entry written twice is saved once.
- The list takes at most **100** entries and 4600 characters in total.

The card marks invalid lines as you type ("Line {n}"), and **Apply protection** stays disabled until they are fixed.

### Add my IP

The **Add my IP** button adds the public address the Komuta console sees to the list (to the draft only; you still need **Apply protection** to save it):

- On an IPv4 connection only your address is added (`/32`).
- On an IPv6 connection your address can change within its `/64` block, so the whole block is added and the card says so.
- If your address is already listed, the card says "Your address ({entry}) is already in the list."
- If no public address can be determined for your connection, nothing is added.

> **Don't lock yourself out.** The added address is the one the console sees. Your browser may reach the service over a different connection (for example IPv6 to the console, IPv4 to the service), in which case the service sees another address. Check it before applying; if needed, add both your IPv4 and IPv6 addresses. If you do get locked out, the **Access to this service is restricted** page shows the address the service sees; copy it and add it to the list.

### How the address is determined

Komuta takes the visitor's address only from what Cloudflare writes, and separately verifies that the request really came through Cloudflare. Headers the visitor sends themselves, such as `X-Forwarded-For`, `X-Real-IP`, `True-Client-IP` or `Forwarded`, are ignored; nobody can get onto the list by faking an address.

So an IP list (for the site, in a path rule or on a webhook path), the country list and the rate limit require requests to the service to go through Cloudflare. The `*.komuta.app` addresses Komuta gives you and custom domains added under **Domains** work this way. A request that can't be verified as coming through Cloudflare is refused wherever an IP list is required (the access log says "The request did not come through the Komuta edge"); if **Either is enough** is chosen, it is sent to sign-in instead.

### How should sign-in and the IP allow-list combine?

When both checks are on, the card asks this question:

| Option | What happens |
|---|---|
| **Require both** (default) | Visitors must come from a listed address and then sign in with Komuta. Other addresses get 403 and never see the sign-in page. |
| **Either is enough** | Listed addresses get in without signing in; everyone else signs in with Komuta, and only the people and organizations you shared it with get through. Use it for no sign-in from the office, sign-in from outside. |

The question doesn't appear when only one check is on:

- **IP allow-list only** — visitors from the listed addresses get in without signing in; everyone else gets 403. The card points this out with a "Sign-in is off" warning.
- **Komuta sign-in only** — every account that matches a share gets in, whatever its address.
- **Neither, with path rules** — the site itself stays open to everyone; only the paths in path rules are protected.
- **Neither, no path rules** — nothing is left to protect. The button turns into a red **Turn off protection**; pressing it turns protection off after the **Turn off access protection?** confirmation.

---

## Path rules

Path rules protect specific paths without touching the rest of the site. Add them with **Add path rule** in the **Path rules** section; each rule has a **Path** and a **Protection** type. A new rule's default type is **Komuta sign-in**.

| Protection | What it does |
|---|---|
| **Block completely** | Nobody can reach this path or anything below it (403). Signing in, coming from an allowed address or using a service token doesn't help either. |
| **Komuta sign-in** | Visitors to this path must sign in with Komuta; only the people and organizations you shared the service with get through. |
| **IP list** | Only the addresses on the rule's own list can reach this path, on top of the site rule. |
| **IP list and Komuta sign-in** | The rule's own IP list and sign-in are used together; choose **Require both** or **Either is enough** ("How should the IP list and sign-in combine on this path?"). |
| **Only chosen people** | Visitors to this path sign in with Komuta, and only the shares you tick get through, at the times you set. Everyone else you shared with still reaches the rest of the site. |

A rule's IP list follows the same rules as the site list (at most 100 entries, public addresses only), and types with an **IP list** need at least one address. The rule row has its own **Add my IP** button too.

### Rules only tighten

A request must pass the site checks **and** **every** rule that matches its path. A path rule can't remove something the site requires; it only adds conditions. Examples:

- The site requires **Komuta sign-in** and `/ops` has an **IP list** rule → to open `/ops` you must both sign in and come from a listed address.
- If the whole site already requires sign-in, a **Komuta sign-in** rule for `/admin` changes nothing; the card says so under the rule ("The whole site already requires Komuta sign-in, so this rule changes nothing.").
- If `/internal` is **Block completely**, even someone with a session or from an allowed address gets 403.
- With a **Komuta sign-in** rule on `/admin` and an **IP list** rule on `/admin/reports`, both are needed for `/admin/reports`.

If a path must open without sign-in (a webhook, for example), use a [webhook path](access-protection-machines.md#webhook-paths), not a path rule; webhook paths are exempt from the site rules.

### Path matching

- A rule matches its path and everything below it: a `/admin` rule matches `/admin` and `/admin/settings`, but not `/administrator`.
- Case doesn't matter. The path you type is lowercased and a trailing `/` removed: typing `/Admin/` saves `/admin`.
- The query string (`?x=1`) doesn't affect matching.
- **Rules can't be bypassed.** Encoded characters (`/%61dmin`, `%2e%2e`, `%252e`), `//`, `/./`, `/../`, `;parameters`, `\` and case tricks can't be used to skip a rule. Komuta reads the request path every way a server behind it might read it, and applies a rule if any of those readings matches it. This can apply a rule one time too many, but never skips one.
- On a service with path rules or a page-limited share, requests whose path is longer than 1024 bytes, too convoluted to read (layered encoding, too many possible readings) or containing control characters are refused with `400 bad request`. Normal browser requests are not affected.

### Path syntax

- A path starts with `/` and may contain only lowercase letters, digits and `- . _ ~ ! $ & ' ( ) * + , = : @ /`. Non-ASCII characters (for example Turkish letters) and spaces can't be used.
- Empty segments (`//`), `.` and `..` segments, and segments ending in `.` aren't allowed.
- `/` on its own can't be a rule ("“/” covers the whole site. Use the site settings above for that.").
- Paths starting with `/.komuta-access` are reserved for Komuta.
- A path can be at most **256** characters; each path can have only one rule.
- A service can have at most **50** path rules. Webhook paths on the **Machines** tab count towards these 50; the card's counter ("{n} of 50 rules") only counts the rules on this tab.

The card marks an invalid rule as you type; **Apply protection** stays disabled until it is fixed.

### Only chosen people

This type opens a path to only some of the people you shared the service with. For example, with the site open to your whole organization, you can open `/salaries` to two people only, or `/demo` to a customer only during the meeting.

1. Set the rule's type to **Only chosen people**.
2. In **Who can open this path**, tick the existing shares of the service that may open this path.
3. Optionally enter a **From (optional)** and **Until (optional)** time for each person. Times are in your account's time zone; the end must come after the start.
4. Save with **Apply protection**.

Things to know:

- The list consists only of the shares on the **People** tab. Share the service with the person first and save; then choose them here.
- If nobody is chosen, the card warns "Nobody is chosen yet. If you apply now, nobody can open this path."; the rule is valid and nobody can open the path.
- The time window is checked on every request. When it starts, the person can open the path without signing in again; when it ends, the path closes within a few seconds, including for open sessions.
- When you add someone to a rule later and they already signed in before, they may be sent to sign-in once on their first try; this adds the new access to their session.
- When you remove a share chosen for the rule, the person is removed from the rule automatically. If the share is removed elsewhere while you are editing the rule, the card says "# chosen people are no longer shared with."; clean up the draft with **Remove from this rule**.
- This type can't have an IP list, and **service tokens can't open these paths**.
- At most 200 people per rule. If you choose someone whose share is limited to certain pages, the path is added to the pages they can open; a share can't exceed 50 pages in total.

Someone who isn't chosen sees the **This page is not shared with you** page: "This part of the service is open only to the people the owner chose, at the times they set."

---

## Countries

The **Countries** section admits visitors only from the countries you list. Visitors from anywhere else, or whose country can't be told, are refused before sign-in.

1. Type a two-letter ISO country code in the field (for example `TR` or `DE`) and choose **Add**. The list shows each country with its name and code, for example "Germany (DE)".
2. Save with **Save countries** (or undo with **Discard**). The change takes effect within about a minute ("Saved. The gateway applies it within about a minute.").

Rules:

- At most **250** countries; each can be listed once. Lowercase codes are turned into uppercase.
- `XX` (unknown country) and `T1` (Tor) can't be listed. While the list has entries, visitors whose country is unknown or who come over Tor are always refused.
- The country is the one Cloudflare reports, and it is trusted only after Komuta verifies that the request came through Cloudflare, exactly as for the IP list. A request that can't be verified is refused (the access log says "The request did not come through the Komuta edge").
- **It applies to everyone**, before anything else is looked at: visitors who haven't signed in, signed-in visitors, share links, service tokens, browsers' CORS checks and the sign-in step itself.
- **Webhook paths are exempt**; they have their own sender list. A share link opened on a webhook path is still checked.
- A refused visitor sees the **Access to this service is restricted** page, the same page as for the IP list, with the address the service sees. Requests with other methods get `403` with the plain text `access restricted to allowed networks`. The access log says "Came from a country that is not allowed".
- An empty list admits every country ("Visitors from every country are admitted.").
- Changing the list needs the **Manage service access protection** permission.

If you don't see this section, country rules aren't enabled on your platform yet. If a list was saved and the feature is later switched off, the section shows it read-only: "This setting can't be changed on this platform yet; the current list stays in force."

---

## Rate limit

The **Rate limit** section answers `429` to an address that sends more requests than you allow.

1. Enter **Requests** (from 10 to 100,000) and choose a **Time window**: per second, per 10 seconds, per minute, per 10 minutes or per hour.
2. Save with **Save limit**. The section then shows, for example, "Each address may send 120 requests per minute." **Remove limit** turns it off ("No rate limit.").

How it counts:

- **Per address.** Each visitor address has its own budget; IPv6 addresses are counted per `/64` block. The address must be verified as coming through Cloudflare, as for the IP list.
- **Every request counts** once it has passed the block rules, method rules and the country list: pages, files, sign-in steps, share links, service tokens, CORS checks and webhook deliveries.
- **Over the limit**: `429` with the plain text `too many requests` and a `Retry-After` header (seconds to wait). There is no HTML page. The access log says "Sent too many requests".
- **It is approximate.** Each gateway replica counts on its own, in memory, so the real limit can be a few times higher, and a gateway restart starts every count afresh. Use it to slow down floods and scrapers, not as an exact quota.
- Fewer than 10 requests isn't accepted, because signing in alone takes several requests.
- Changing the limit needs the **Manage service access protection** permission.

If you don't see this section, rate limits aren't enabled on your platform yet. If a limit was saved and the feature is later switched off, the section shows it read-only: "This setting can't be changed on this platform yet; the current limit stays in force."

---

## Access preview

**Access preview** shows who gets in where, without changing anything. It appears at the bottom of the **Rules** tab while protection is on, and also while you are editing protection (if you can manage it).

- **With no unsaved changes** it uses the saved rules and shares ("Uses the saved rules and shares as of now.").
- **With unsaved changes in the card** it shows what would happen once you apply them ("See who would get in where once you apply your changes. Nothing is saved."). Komuta checks your draft against the current shares; if applying it would fail, the problems are listed under "Applying these changes would fail:" and the rows appear once you fix them. **Countries** and **Rate limit** aren't part of the draft; the preview uses their saved values.

It has two views:

- **Who can open a page** — type a path in **Page** (for example `/reports`). The list shows a visitor who has not signed in, every share, every service token and every share link that hasn't ended, with whether they get in.
- **What a person can open** — pick a member, share, service token or share link in **Person**. The list shows the site root, every path rule and the pages in their scope.

Two more choices appear when they matter:

- **Coming from** — when the site or a rule has an IP list: **Outside the allowed networks** (default) or **A specific address** (type an IPv4 or IPv6 address, for example your office address).
- **Country** — when the service has a country list: **A listed country** (default) or **A country that is not listed**.

Each row shows a green check (gets in) or a red cross (refused) and the reason:

| Reason | Meaning |
|---|---|
| **Protection is off, so everyone gets in** | Protection isn't on yet. |
| **Open to everyone** | No protection on this path. |
| **Gets in through an open path; the app checks the signature** | A webhook path lets this request through without sign-in. The preview works out `GET` requests only, so this row appears only for open paths that accept `GET`. |
| **Gets in from an allowed network** | Gets in without signing in from an address on the IP list. |
| **Gets in after signing in** | Gets in after signing in with Komuta. |
| **Gets in after signing in from an allowed network** | Needs both an allowed address and sign-in. |
| **Gets in with the service token** / **Gets in with the share link** | The token or link opens this page. |
| **Blocked by the {path} rule** | A **Block completely** rule. |
| **This method is not allowed on {path}** | A method rule on the **Machines** tab doesn't allow `GET` here. |
| "{path} checks webhook signatures and only takes the methods it lists; this request is refused with 401 before it reaches the app." | The page is under a webhook path with a [signature check](access-protection-machines.md#signature-check-at-the-edge); signed paths don't take `GET`. |
| **The visitor's country is not on the list** | Refused by the country list. |
| **Needs an allowed network ({path})** | The address isn't on the list for this path. |
| **Needs to sign in ({path})** | Someone who hasn't signed in. |
| **The service is not shared with them, or the share is paused or ended** | No share, or the share is suspended or expired. |
| **Outside the pages shared with them** | Their share is limited to certain pages. |
| **Outside the paths the service token covers** / **Outside the paths the share link covers** | The page is outside the token's or link's pages. |
| **Not among the people chosen for {path}** | An **Only chosen people** rule. |
| **Only the people chosen for {path} get in; a service token never does** / **…; a share link never does** | Tokens and links never open **Only chosen people** paths. |
| **The service token is unknown or has expired** / **The share link has expired, was withdrawn, or isn't in force yet** | The token or link no longer works. |
| **Their access to {path} starts {date}** / **ended {date}** | Outside the person's time window. |

If someone gets in only through an email share, the row ends with "(by confirming their email)". If saved rules are still rolling out, the preview says so. The preview works out a browser opening a page (`GET`); it doesn't take the rate limit, other methods such as `POST`, or the private mesh into account.

Seeing the preview needs the **Manage service access protection** or **Manage who a protected service is shared with** permission.

---

## Related Documents

- [Access Protection](service-access-protection.md) — turning on and off, status.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — shares and what visitors see.
- [Machines and Private Mesh](access-protection-machines.md) — webhook paths, service tokens, methods and CORS.
- [Reference](access-protection-reference.md) — limits and error messages.
