# Sign-in and Sharing

While **Komuta sign-in** is on, anyone who wants to open your service first signs in with a Komuta account; only the people you shared the service with get in. You manage who it is shared with on the **People** tab of the **Access & ports** page.

This page covers both sides: how the service owner manages shares, share links and visitor sessions, and what a visitor sees while signing in.

---

## How a visitor signs in

1. The visitor opens your service's address, for example `https://panel-3a2331e6.edge-1.komuta.app/reports`.
2. Komuta sees that the visitor has no session for this service and sends them to the sign-in page in the Komuta console (`https://console.komuta.io/access/…`).
3. If the visitor isn't signed in to Komuta, they see the **Sign in to continue** page. It says the service's owner limited who can open it with Komuta, shows the service's address under **You're going to**, and offers a **Sign in with Komuta** button. For people without an account it notes: "No account yet? You can create one in seconds with Google or GitHub on the next page. If the page was shared with your e-mail address, use that address."
4. After signing in, the visitor sees **Checking your access**.
5. If their account matches a share, they return to the page they wanted (`/reports` in the example). Otherwise they see the **You don't have access** page (see below).

A visitor who is already signed in to Komuta skips step 3; the redirect completes within a few seconds.

### Session

After a successful sign-in, a session cookie for the service's own address is stored in the visitor's browser:

- The session lasts as long as the service's **Stay signed in for** setting: 15 minutes, 1 hour, 4 hours, **12 hours** (default), 1 day or 7 days (see [Who is signed in](#who-is-signed-in)). It never outlives the visitor's share end date or an "open to everyone" protection end date.
- The session is valid **only for the address signed in to**. If the service has several addresses (for example the `*.komuta.app` address and your custom domain, or a blue-green preview address), each needs its own sign-in.
- Komuta's cookies are removed from the request before it reaches your application; your application never sees or is affected by them.
- Sign-in must be completed within about 10 minutes. If the visitor waits longer, the console says **This sign-in link isn't valid** or the service answers with a short `sign-in link is invalid or expired`; opening the protected page again is enough.

### When sessions end early

Sessions can end before their time, within about 30 seconds. People who still have access sign in again on their next page load: for those already signed in to Komuta this is an automatic redirect, while people who came in through an email share request a new code. Meanwhile non-browser requests (for example the `POST` calls of a single-page app) may get `401` until the page is reloaded.

**Only the people concerned** — once one-by-one sign-out is active on the service (see [When one-by-one sign-out starts](#when-one-by-one-sign-out-starts)):

- You sign a person out on the **People** tab.
- You remove a share, bring its end date forward, or give an end date to a share that had none: the sessions opened through that share end.
- You change the page list of a page-limited share (adding pages included; switching to **Whole site** excepted): the sessions opened through that share end.

Until one-by-one sign-out is active, each of these ends the sessions of **every visitor of this service**.

**Every visitor of the service:**

- You choose **Sign everyone out**.
- Your organization turns off external sharing (on services that have an external share; people who came in through it lose access, everyone else signs in once more).
- Something security-related changes on a visitor's Komuta account: the account is deleted, locked or deactivated, its sign-in credentials or two-factor authentication change, its email address is no longer verified, an account link is removed, or its organization is suspended. Every visitor of the services that person could open signs in again (this can take a little over a minute).

Shortening **Stay signed in for** also ends, at once, the sessions that are older than the new length.

### Who can't sign in

These can't sign in to a protected service: impersonation sessions (acting as another user), requests made with an API key, inactive or locked accounts, and users of inactive or banned organizations. A user can make at most 30 sign-in attempts per minute; beyond that they see "Too many sign-in attempts. Wait a minute and try again."

### Non-browser clients

`GET` and `HEAD` requests without a session to a page that requires sign-in are redirected to the sign-in page (`302`). Other methods (for example a `POST` to an API) are not redirected; they get `401` with the body `sign-in required`. For programs to reach your service, use a [service token](access-protection-machines.md#service-tokens), a [webhook path](access-protection-machines.md#webhook-paths) or the IP allow-list.

---

## Pages visitors see

| Situation | What the visitor sees |
|---|---|
| Signed in and matches a share | The service opens. |
| Not signed in to Komuta | **Sign in to continue** (in the Komuta console). |
| Signed in but matches no share | **You don't have access** (in the Komuta console). |
| Signed in but the page is outside their share | **This page is not shared with you** — "This page is outside the pages shared with you." with a **Pages you can open** list below (at most 20 links). |
| Signed in but the page is protected by an **Only chosen people** rule and they aren't chosen or are outside their time window | **This page is not shared with you** — "This part of the service is open only to the people the owner chose, at the times they set." |
| Came from an address not on the IP allow-list | **Access to this service is restricted** — "This service can only be opened from allowed networks." The page shows the address the service sees under **Your address**; **Copy** it to pass it to the service owner. **Try again** reloads the page. This page offers no sign-in. |
| Opened a path under a **Block completely** rule | `403`, plain text `access denied`. |
| The service no longer asks for sign-in but the visitor came with an old sign-in link | **This service doesn't use Komuta sign-in**. |
| Sign-in can't be checked right now | **Sign-in is unavailable right now** with a **Try again** button. |
| Komuta's access check temporarily can't answer | `503`, plain text `access policy unavailable` or `sign-in unavailable`. With `access policy unavailable` the service opens to nobody; `sign-in unavailable` affects only pages that need sign-in. Try again shortly. |
| Came from a country that isn't on the service's country list | **Access to this service is restricted** (the same page as for the IP list). |
| Sent too many requests | `429`, plain text `too many requests`, with a `Retry-After` header. |
| Used a method that the path doesn't allow | `405`, plain text `method not allowed`, with an `Allow` header. |
| Opened a share link | See [Share links](#share-links). |

The **Access to this service is restricted** and **This page is not shared with you** pages shown on your service appear in Turkish if the visitor's browser language is Turkish, otherwise in English. Requests with methods other than `GET`/`HEAD` (from a browser or a program) get a short plain-text body instead: `access restricted to allowed networks` or `this path is not shared with you`.

### The You don't have access page

This page tells the visitor why they can't get in and what they can do:

- The **Signed in as** box shows which account and which organization (**Organization: …**) they signed in with.
- The **Shared with your email address?** section verifies an email share (see below).
- **Continue with another organization** — if the visitor belongs to several organizations, they can **Switch** to another one here and try again. If you shared the service with "Your organization" or "A linked organization", the visitor may need to continue with that organization.
- **Sign in with another account** — signs in again with a different Komuta account.
- **Go to Komuta** — returns to the Komuta console.

---

## Shares

The **Shared with** list on the **People** tab shows the people and organizations that get in after signing in with Komuta. Add one with **Add share**.

### Share types

| Type | Who gets in |
|---|---|
| **Your organization** | Every active member of this organization. A service can have one. Listed as **Everyone in your organization**. |
| **A member** | One person in this organization. Only active members can be chosen. |
| **A linked organization** | Everyone in another organization you belong to, including people who join later. Only organizations you are also a member of can be chosen. |
| **An email address** | Someone outside your organizations. They sign in with any Komuta account, then confirm the address with a one-time code sent to it. |

**A linked organization** and **An email address** shares are listed with an **External** badge and require your organization to allow external sharing (see below).

A person, organization or address that is already shared can't be added again (the window says "Already shared."); change its end date or page limit with the pencil icon in the list.

### Choices when adding a share

- **What they can open** — **Whole site** (default, "Every page their sign-in opens.") or **Only these pages** ("Their sign-in doesn't open any other page."). For the latter, write one path per line; each path also opens the pages under it (`/reports` opens `/reports/2026` too). Path rules still apply inside these pages. At most 50 pages; `/` can't be used (choose **Whole site** instead). This option can be used only while protection requires Komuta sign-in; otherwise the window says so.
- **Access ends** — optional. Leave it empty to keep access until you remove it. Times are in your account's time zone (**Account → General**); if your device is in a different time zone, the card warns you and shows what the chosen time is on your device. Share end dates have no upper limit; a time in the past can't be chosen.

### How page-limited shares combine

- **The widest share wins.** If someone matches both a page-limited share and a share that opens the whole site, they open the whole site. For example, limiting a member to `/reports` has no effect while **Your organization** has the whole site; the card warns about this.
- Someone who opens a page outside their share sees **This page is not shared with you** and the list of pages they can open.
- Page limits apply only where an identity is needed. A path that is open to everyone, or a place passed without sign-in thanks to the IP list, stays open regardless of page limits.
- If the service has any page-limited share, every session ends when the earliest of the shares the person matched ends.

### Badges and editing in the list

Each row shows the share's name, any page limit ("Only: /a, /b") and its end ("Until …" or **No end date**). Badges:

- **External** — a linked organization or email share.
- **Suspended** — a share suspended because external sharing was turned off. Nobody gets in with it.
- **Expired** — a share whose end time has passed. It stays in the list; give it a new end date with the pencil icon.

The pencil icon edits a share: the type and person can't be changed; the page limit and end date can. Extending or removing the end date, or opening the share to the **Whole site**, doesn't affect open sessions. Adding or bringing forward an end date, or changing the page list, ends the sessions opened through that share; until one-by-one sign-out is active on the service, it ends every open session of this service and everyone signs in once more (see [When sessions end early](#when-sessions-end-early)).

### Removing a share

Remove a share with the trash icon on its row, after the **Remove this share?** confirmation ("… loses access within about 30 seconds, including sessions that are already open. If Komuta cannot yet tell their sessions apart, everyone signed in is asked to sign in again."). Once one-by-one sign-out is active, only the sessions opened through that share end; before that, the other visitors of the service sign in once more too. The share is also removed from any **Only chosen people** rules it was chosen for.

### When shares apply

- Protection must be applied before you can add shares ("Apply protection first, then share the service.").
- Shares apply only while something requires Komuta sign-in: **Require Komuta sign-in** is on, or there is a path rule of type **Komuta sign-in**, **IP list and Komuta sign-in** or **Only chosen people**.
- If sign-in is on but there are no shares, nobody gets past sign-in ("Not shared with anyone yet. Until you add a share, nobody can get past sign-in.").
- **You are added automatically.** When you save a change that makes Komuta sign-in required (including when you turn protection back on), Komuta adds you as **A member** share (no end date, whole site) if you have permission to manage shares and there is no organization share or share in your name yet. This way you don't lock yourself out of your own service. If you shouldn't have access, remove that share.
- Turning protection off doesn't delete shares; when you turn protection back on, the same shares apply again.
- A service can have at most **200** shares.

---

## Share links

A share link lets someone **without a Komuta account** in for a while: a client demo, an outside tester. Whoever holds the link gets in without signing in, until the link ends or you delete it. Links are in the **Share links** section of the **People** tab.

### Creating a link

1. Click **Create link**.
2. In the window:
   - **Name** — name it after who you send it to, for example "Client demo"; the name shows in the access log. 1–64 characters; two links of the same service can't have the same name (case doesn't matter).
   - **Works for** — **1 day**, **7 days** (default), **30 days** or **90 days**. A link always has an end date; through the API it can be at most 365 days away.
   - **What it can open** — **Whole site** (default) or **Only these pages**: one path per line, at most 50; each path also opens the pages under it, and path rules still apply inside. The copied link opens the first of these paths in alphabetical order.
3. Save with **Create link**. The **Your link is ready** screen shows the link; copy it with **Copy**. You can copy it again later from the list.

A link looks like `https://panel.example.com/reports?komuta_link=kl_…`. Send it only to the people it is meant for: anyone holding it gets in.

### What the visitor gets

- **Opening the link** — a browser opens the link (`GET` or `HEAD`). Komuta checks it, opens a session for this address of the service and sends the visitor to the same address without the `komuta_link` part, so the secret leaves the address bar and isn't passed on to other sites. The query string is never recorded.
- **The session** lasts the service's **Stay signed in for** length (12 hours by default) and never past the link's end.
- **Only the link's pages** — other pages answer `403` "Your share link does not open this page." Blocked paths stay blocked, **Only chosen people** paths never open with a link, and the IP allow-list, the country list and the rate limit still apply.
- **An existing session is kept** — a visitor who is already signed in to this service with their Komuta account keeps their own session; they are sent to the same address without the link part.
- **Other methods** (for example `POST`) on a link address get `403` "Open a share link in a browser."
- **A wrong or expired link** gets `403` "This share link is not valid or has expired. Ask the person who sent it for a new one."
- **When you delete the link or it ends**, within about 30 seconds the visitor's next request gets `403` "The share link you opened this site with has ended. Ask the person who sent it for a new one." The visitor is never sent to the Komuta sign-in page.
- These answers are short plain-text responses in English.
- Link visitors aren't listed under **Who is signed in**. Their sessions end when the link is deleted or ends, or when you choose **Sign everyone out**.
- If **Tell my application who signed in** is on, your application receives only `x-komuta-identity`, with `kind: share_link` (see [End Date and Visitor Identity](access-protection-settings.md#headers-your-application-receives)).

> **Link previews count as openings.** Chat apps and mail scanners that fetch a link to show a preview open it just like a visitor. In the access log this appears as "Came in with a share link".

### The list

Each row shows the link's name, the pages it can open (**Whole site** or "Only: …"), its end ("Until …") and when it was last used ("last used …" or "not used yet"). A link that doesn't let anyone in shows a **Not working** badge: it has ended, protection no longer requires Komuta sign-in, or share links are switched off on the platform. Ended links stay in the list until the next link is created.

To delete a link, click the trash icon and confirm **Delete this link?** ("Within about 30 seconds the link stops working and everyone who came in with {name} is shown that it has ended.").

### Rules and limits

- Share links work only while protection requires Komuta sign-in ("Share links work only while Komuta sign-in is required.").
- A service can have at most **50** share links.
- Creating and deleting links needs the **Manage who a protected service is shared with** permission; either access protection permission is enough to see the list.

If the section says "Share links aren't available on this service yet.", share links aren't enabled on your platform yet. If links already exist and the feature is switched off, the section says "Share links are switched off on Komuta for now, so these links don't let anyone in. You can still delete them."

---

## Who is signed in

The **Who is signed in** section of the **People** tab lists the people who signed in to this service with their Komuta account and whose session is still open, one row per person:

- The name (or the email). Someone whose name isn't known is shown as "A visitor from another organization"; people from outside your organization have an **Outside your organization** badge. If the person is signed in on several browsers or devices, "{count} sessions" is shown.
- "Signed in … · last seen … · until …". **Last seen** comes from the access log ("not yet" if they haven't opened a recorded page since signing in).
- Email addresses are shown only to people allowed to view users (for email-share sessions, also to people who can manage shares).
- At most **200** sessions are listed; when there are more, the section says "Showing the most recent sign-ins only."
- Visitors who came in with a share link or a service token aren't listed.

If protection doesn't require Komuta sign-in, the section says "Nobody signs in to this service; turn on Komuta sign-in under Rules to see who is inside."

### Signing people out

Signing out needs the **Manage who a protected service is shared with** permission.

- **Sign out** on a person's row asks **Sign {name} out?**: "Their open sessions end within about 30 seconds; nobody else is affected. To keep them out, also remove the share that lets them in." The person can sign straight back in while a share still lets them in.
- **Sign everyone out** asks **Sign everyone out?**: everyone signed in to this service, including people who came in with a share link, is asked to sign in again within about 30 seconds.

### Stay signed in for

**Stay signed in for** sets how long a sign-in lasts: **15 minutes**, **1 hour**, **4 hours**, **12 hours** (default), **1 day** or **7 days**. It is saved as soon as you choose ("New sign-ins now last {length}. Sessions already open end sooner if they would outlast it.").

- A shorter length also ends older sessions at once; a longer one doesn't extend sessions that are already open.
- It applies to email-share and share-link sessions too.
- Changing it needs the **Manage service access protection** permission.

### When one-by-one sign-out starts

Komuta can end one person's sessions only for sessions it can tell apart. It starts telling them apart when session tracking begins on the service (when Komuta sign-in is turned on, or when this feature reached your platform). Sessions opened before that can stay open for up to 12 hours, so one-by-one sign-out starts **12 hours 10 minutes** after tracking began. Until then:

- signing one person out signs everyone out, and the confirmation turns into **Sign everyone out?**;
- the section says "Some sessions here were opened before Komuta could tell them apart. Until {time}, signing one person out signs everyone out." (or "For now, signing one person out signs everyone out of this service.");
- removing or narrowing a share signs everyone out too.

Komuta also falls back to signing everyone out if a very large number of sessions (more than 500) has been ended one by one within a session's lifetime.

If you don't see this section, managing visitor sessions isn't enabled on your platform yet. Sessions then last 12 hours, and removing or narrowing a share signs everyone out.

---

## Email shares

The person an email address is shared with doesn't need a Komuta account with that address; they sign in with any Komuta account and prove they can read the mailbox.

1. The person opens the service and signs in with any Komuta account (they can create one with Google or GitHub).
2. If their account matches no other share, they see the **You don't have access** page. In its **Shared with your email address?** section, the address field is prefilled with the account's email; they change it if the shared address is different.
3. **Email me a code** sends an **8-digit** code, valid for **10 minutes**, to the address. The email's subject is "Your Komuta access code for {service}", and it shows the Komuta account that asked for the code, the service and the address.
4. The person enters the code and chooses **Verify and continue**; the service opens.

Rules:

- The code works only for the Komuta account that asked for it and only once. A new code is needed for every sign-in; a session opened through an email share lasts the service's **Stay signed in for** length too.
- A code stops working after 5 wrong attempts. After 50 wrong attempts in 24 hours for the same account, service and address, no new codes are sent.
- Code request limits: a user can request at most 20 codes per hour, and at most 5 per hour for the same address. At most 3 codes per hour are sent to an address for the same Komuta account and service; switching browser or device doesn't reset this, and beyond it the screen still says a code was sent but no email goes out. Resending waits 30, 60 and 120 seconds.
- The screen gives the same answer whether or not the address has access ("If {email} has access to this page, we've emailed it a {length}-digit code."), so nobody can guess which addresses a service is shared with.
- If the sign-in link expires before the code would, the screen says "This sign-in link expires before a code would."; open the protected page again and request a code right away. Enter the code without waiting: sign-in must be completed within about 10 minutes of being sent from the protected page.
- The email address must be a plain address: ASCII letters, at most 254 characters; wildcards, spaces, IP addresses and non-English letters aren't accepted.

---

## Allowing external sharing

**A linked organization** and **An email address** shares give access outside your organization. They require your organization to allow external sharing:

- Setting: **Account → Organizations → Allow external sharing**.
- **It is off by default.**
- Changing it needs permission to edit the organization.
- Turning it on applies immediately. Turning it off asks for confirmation (**Turn off external sharing?**).

While it is off, these two types can't be chosen in the **Add share** window and the list says so. If you can change the setting, **Open organization settings** opens it highlighted in a new tab; once you turn it on and come back, the options become available without reloading. Otherwise, ask an organization admin to turn it on.

If you turn it off:

- Linked organization and email shares on all of the organization's protected services become **Suspended**.
- People who came in through these shares lose access within about 30 seconds; the other visitors of these services sign in once more too.
- New external shares can't be added.

If you turn it back on, email shares come back; linked organization shares come back if the link between the two organizations still exists. Komuta checks the link every 5 minutes; if it is broken (no pair of linked, active user accounts remains between the two organizations), the linked organization share is suspended.

---

## Permissions

| Permission | What it allows |
|---|---|
| **Manage who a protected service is shared with** | Adding, editing and removing shares; creating and deleting share links and service tokens; signing visitors out. |
| **Manage service access protection** | Turning protection on and off, changing rules and settings, including **Stay signed in for**. Can see shares but not change them. |

Someone with either permission can see the share list, share links, tokens, **Who is signed in**, the access preview and the access log. Someone with neither, who can still see the service, sees only the number of shares on the **People** tab. In the **Add share → A member** list, members' email addresses are shown only to people allowed to view users.

---

## Related Documents

- [Access Protection](service-access-protection.md) — overview.
- [Rules](access-protection-rules.md) — where Komuta sign-in is required.
- [Access Log](access-protection-activity.md) — who signed in and who used a share link.
- [Reference](access-protection-reference.md) — sign-in error messages.
