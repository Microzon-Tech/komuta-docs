# Access Log

The access log shows who signed in to your protected service, which pages signed-in visitors opened, and who was refused and why. It is on the **Activity** tab of the **Access & ports** page. The **Recent activity** box on the **Overview** tab shows the last 5 records; **Show all** takes you to the **Activity** tab.

The log is kept while access protection is on. While protection is off the tab says "The access log is kept while access protection is on." Seeing the log needs either the **Manage service access protection** or the **Manage who a protected service is shared with** permission.

---

## What is recorded

| Event | When | In the list |
|---|---|---|
| **Sign-in** | A visitor signs in with Komuta and returns to the service | **Signed in** |
| **Page view** | A signed-in visitor opens a page | **Opened the page** |
| **Access with a token** | A program opens a page with a service token (`GET`, paths that aren't files) | **Opened the page with a service token** |
| **Webhook delivery** | A request arrives on an open path | **Delivered to an open path** |
| **Refusal** | A visitor is turned away by your rules | The reason for the refusal (table below) |

What is **not** recorded:

- **File requests.** Only `GET` requests whose last path segment has no dot count as page views. Requests such as `/app.js`, `/logo.png` or `/style.css` aren't recorded, so the log shows real page openings.
- **Visits where sign-in isn't needed.** Page views are recorded only on pages that need sign-in, for signed-in visitors and tokens. Nobody is recorded on unprotected paths (paths with no check at all; webhook paths excepted) or on pages passed without sign-in thanks to the IP list, even if the visitor has signed in.
- **Redirects to sign-in.** Sending a visitor who hasn't signed in to the sign-in page, and the `401` returned to programs without a session, are not refusals and don't appear in the log.
- **Platform-side problems.** A response that couldn't be given because of a temporary fault on Komuta's side doesn't count as a visitor refusal.
- **Komuta's signed test sign-in.** To confirm protection works, Komuta sends a test request that acts as signed in; it isn't written to the log. The anonymous checks made while protection is being turned on may, however, show up as a few **Not signed in** refusals on services with an IP list or a **Block completely** rule.

---

## Reading the list

Records are grouped by day; each day starts with the full date. Columns:

| Column | Contents |
|---|---|
| **Time** | When the event was last seen (in the time zone chosen in your account). |
| **Who** | The visitor's name (and email), or one of the labels below. |
| **What happened** | The event or the reason for a refusal. |
| **Page** | The HTTP method and path, for example `GET /reports`. The query string (`?…`) isn't recorded; paths are cut after 256 characters. |
| **Address** | The visitor's IP address (see below). |
| **Times** | How many times the same event happened. |

On narrow screens only **Time** and **Who** are shown; the reason and page appear under the name.

### Labels in the Who column

| Label | Meaning |
|---|---|
| Name and email | A signed-in person from your organization. The email is shown only to people allowed to view users. |
| **Someone from another organization** | A signed-in person from outside your organization (who came in through a linked organization or email share). Their name and email aren't shown. |
| **Service token: {name}** | A program using the service token with this name. |
| **A deleted service token** | A token that was deleted later. |
| **Request on an open path** | A request to a webhook path. |
| **Not signed in** | A visitor who hasn't signed in. |
| **Other visitors** | Other events that didn't fit into the log that hour, gathered in one row (see below). |

### Refusals of visitors who haven't signed in

Refusals of someone who hasn't signed in are recorded with less detail, so the log can't be abused:

- The **Page** column shows **Any page** instead of the real path. This way someone scanning your site can't fill the log with thousands of different paths.
- The **Address** column shows the first part of the network instead of the full address: `/24` for IPv4 (for example `203.0.113.0/24`), `/48` for IPv6.
- Exception: if the refusal comes from a **Block completely** rule or an open path, the **Page** column shows that rule's path (for example `/internal`). You chose those paths yourself, so you can see which rule was hit.

Signed-in visitors' addresses are shown in full.

A visitor's address is shown when the request could be verified as coming through Cloudflare; otherwise the column shows `—`.

### Reasons in the What happened column

| Text | Meaning |
|---|---|
| **Signed in** | Komuta sign-in completed. |
| **Opened the page** | A signed-in visitor opened the page. |
| **Opened the page with a service token** | A program opened the page with a token. |
| **Delivered to an open path** | A request reached a webhook path. |
| **This page is not shared with them** | The visitor signed in, but this page isn't shared with them (their share is limited to certain pages, or the path is open only to chosen people). |
| **The page is blocked** | Hit a **Block completely** rule. |
| **Came from an address that is not allowed** | Came from an address that isn't on the IP allow-list. |
| **The request did not come through the Komuta edge** | The request couldn't be verified as coming through Cloudflare; on a page that requires an IP list, the address wasn't trusted. |
| **The sign-in link was invalid or expired** | The visitor came back with a broken or old sign-in link. |
| **Komuta refused the sign-in** | Komuta refused the sign-in link (for example, it had already been used). |
| **Sent an unknown or expired service token** | Invalid token. |
| **The service token cannot open this page** | The token is valid but this page is outside its scope. |
| **More visits this hour, grouped together** | The total of events that didn't fit into the log. |
| **Refused ({code})** | A reason the console doesn't recognise; the technical code is shown in brackets. |

---

## Grouping and counts

The log doesn't keep every request as a separate row:

- **15-second batches.** If the same event (same person, same outcome, same reason, same method, same path, same address) happens several times within 15 seconds, it is sent as one event with a count.
- **Hourly rows.** The same events are gathered into one row for each hour; the **Times** column grows and **Time** shows when it was last seen.
- **Hourly limit.** At most 500 different rows are kept per service per hour (sign-ins don't count towards this). The rest is gathered in the **Other visitors** / **More visits this hour, grouped together** row.
- **Busy moments.** During a very heavy attack or scan, the number of page views and refusals that can be recorded for one service per 15 seconds is limited, so that other services and sign-ins aren't crowded out. Events above this cap are not recorded at all, not even in the **Other visitors** row. That's why the log is a "who came, who was refused" view, not a security audit log.

Records appear in the list with a delay of a few seconds up to about half a minute.

---

## Filters and pages

- **Time range** — **Last 24 hours**, **Last 7 days** or **Last 30 days** (default).
- **Show** — **Everything**, **Sign-ins**, **Page views**, **Refusals**.
- Each page shows 50 records. Move with **Newer** and **Older**; the bottom says "{from}–{to} of {total}". If the total exceeds 10,000, a `+` appears next to it.

The list doesn't refresh on its own; change a filter or reload the page to see new records.

---

## Retention

Records are kept for **30 days**; a row is deleted 30 days after it was last seen. There is currently no option to export the log.

---

## Related Documents

- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — the pages visitors see.
- [Rules](access-protection-rules.md) — the rules behind refusals.
- [Machines and Private Mesh](access-protection-machines.md) — token and webhook records.
- [Reference](access-protection-reference.md) — technical reason codes.
