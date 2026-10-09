# Sign-in and Sharing

While **Komuta sign-in** is on, anyone who wants to open your service first signs in with a Komuta account; if the service is shared with their email address or domain, with a one-time code sent to that mailbox; or, if it is shared with your organization's identity provider, with their account there. Only the people you shared the service with get in. You manage who it is shared with on the **People** tab of the **Access & ports** page.

This page covers both sides: how the service owner manages shares, share links and visitor sessions, and what a visitor sees while signing in.

---

## How a visitor signs in

1. The visitor opens your service's address, for example `https://panel-3a2331e6.edge-1.komuta.app/reports`.
2. Komuta sees that the visitor has no session for this service and sends them to the sign-in page in the Komuta console (`https://console.komuta.io/access/…`).
3. If the visitor isn't signed in to Komuta, they see the **Sign in to continue** page. It says the service's owner limited who can open it with Komuta, shows the service's address under **You're going to**, and offers a **Sign in with Komuta** button. For people without an account it notes: "No account yet? You can create one in seconds with Google or GitHub on the next page. If the page was shared with your e-mail address, use that address."
4. After signing in, the visitor sees **Checking your access**.
5. If their account matches a share, they return to the page they wanted (`/reports` in the example). Otherwise they see the **You don't have access** page (see below).

A visitor who is already signed in to Komuta skips step 3; the redirect completes within a few seconds.

If the service has an **An email address** or **Everyone at a domain** share, the **Sign in to continue** page also shows, under **Sign in with Komuta**, an **or** divider and the **Shared with your email address or company domain?** section: the visitor can get in with a code sent to their mailbox, without a Komuta account (see [Without a Komuta account](#without-a-komuta-account)).

If the service is shared with one of your organization's identity providers, the page also shows a **Continue with {name}** button under **Sign in with Komuta** for each of them: the visitor signs in at your Microsoft Entra ID, Google Workspace, Okta or other provider instead (see [Sign in with your organization's identity provider](#sign-in-with-your-organizations-identity-provider)).

### Session

After a successful sign-in, a session cookie for the service's own address is stored in the visitor's browser:

- The session lasts as long as the service's **Stay signed in for** setting: 15 minutes, 1 hour, 4 hours, **12 hours** (default), 1 day or 7 days (see [Who is signed in](#who-is-signed-in)). It never outlives the visitor's share end date or an "open to everyone" protection end date.
- Visitors without a Komuta account (who signed in with an email code or with your identity provider) stay signed in for **at most 12 hours**, even when **Stay signed in for** is 1 day or 7 days.
- The session is valid **only for the address signed in to**. If the service has several addresses (for example the `*.komuta.app` address and your custom domain, or a blue-green preview address), each needs its own sign-in.
- Komuta's cookies are removed from the request before it reaches your application; your application never sees or is affected by them.
- Sign-in must be completed within about 9 minutes. If the visitor waits longer, the console says **This sign-in link isn't valid** or the service answers with a short `sign-in link is invalid or expired`; opening the protected page again is enough.

### When sessions end early

Sessions can end before their time, within about 30 seconds. People who still have access sign in again on their next page load: for those already signed in to Komuta this is an automatic redirect, people who came in through an email share request a new code, and people who came in through your identity provider choose **Continue with {name}** again. Meanwhile non-browser requests (for example the `POST` calls of a single-page app) may get `401` until the page is reloaded.

**Only the people concerned** — once one-by-one sign-out is active on the service (see [When one-by-one sign-out starts](#when-one-by-one-sign-out-starts)):

- You sign a person out on the **People** tab.
- You remove a share, bring its end date forward, or give an end date to a share that had none: the sessions opened through that share end.
- You change the page list of a page-limited share (adding pages included; switching to **Whole site** excepted): the sessions opened through that share end.

Until one-by-one sign-out is active, each of these ends the sessions of **every visitor of this service**.

**Every visitor of the service:**

- You choose **Sign everyone out**.
- Your organization turns off external sharing (on services that have an external share; people who came in through it lose access, everyone else signs in once more).
- You limit an existing identity provider share to groups, or remove or replace a group in its list (adding groups or clearing the list doesn't sign anyone out).
- The groups claim of an identity provider changes (on services that have a group-limited share of that provider).
- Something security-related changes on a visitor's Komuta account: the account is deleted, locked or deactivated, its sign-in credentials or two-factor authentication change, its email address is no longer verified, an account link is removed, or its organization is suspended. Every visitor of the services that person could open signs in again (this can take a little over a minute).

Shortening **Stay signed in for** also ends, at once, the sessions that are older than the new length.

If sign-in with an email code without a Komuta account, or with identity providers, is switched off on your platform (or the service stops requiring Komuta sign-in), new sign-ins of that kind stop at once. Sessions that are already open end within 12 hours at the latest; choose **Sign everyone out** to end them now.

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
- The **Shared with your email address or company domain?** section verifies an email or domain share (see below).
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
| **An email address** | Someone outside your organizations. They confirm the address with a one-time code sent to it, either after signing in with any Komuta account or without an account. |
| **Everyone at a domain** | Anyone with an email address at a domain such as `@example.com`, optionally its subdomains too. They confirm their address at the domain with a one-time code, with or without a Komuta account. See [Domain shares](#domain-shares). |
| **Your identity provider** | Everyone who signs in with one of your organization's identity providers (Microsoft Entra ID, Google Workspace, Okta or another OIDC provider), optionally only members of chosen groups; no Komuta account is needed. Listed as **Everyone who signs in with {provider}**. See [Sign in with your organization's identity provider](#sign-in-with-your-organizations-identity-provider). |

**A linked organization**, **An email address** and **Your identity provider** shares are listed with an **External** badge and require your organization to allow external sharing (see below). An **Everyone at a domain** share counts as external too, unless the domain is one your organization has verified (see [Verified domains](#verified-domains)); then it has a **Verified domain** badge instead.

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

- **External** — a linked organization, email or identity provider share, or a domain share for a domain your organization hasn't verified.
- **Verified domain** — a domain share under a domain your organization has verified; it counts as your organization's own.
- **Suspended** — a share suspended because external sharing was turned off, or because the domain of a domain share is no longer verified. Nobody gets in with it.
- **Expired** — a share whose end time has passed. It stays in the list; give it a new end date with the pencil icon.

The pencil icon edits a share: the type and person can't be changed; the page limit and end date can. Extending or removing the end date, or opening the share to the **Whole site**, doesn't affect open sessions. Adding or bringing forward an end date, or changing the page list, ends the sessions opened through that share; until one-by-one sign-out is active on the service, it ends every open session of this service and everyone signs in once more (see [When sessions end early](#when-sessions-end-early)). For an **Everyone at a domain** share the domain can't be changed. **Also addresses at its subdomains** can be turned off (this ends the sessions opened through the share), but turned on only for a verified domain. A share that covers subdomains of a domain that is no longer verified stays **Suspended**; the window saves it only once the domain is verified again or the subdomains option is turned off. For a **Your identity provider** share the provider can't be changed, but its groups (**Only these groups (optional)**) can (see [Sharing a service with a provider](#sharing-a-service-with-a-provider)).

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

The **Who is signed in** section of the **People** tab lists the people who signed in to this service, with their Komuta account, with an email code or with your identity provider, and whose session is still open, one row per person:

- The name (or the email). People from outside your organization have an **Outside your organization** badge; someone from another organization whose name isn't known is shown as "A visitor from another organization" (visitors without a Komuta account have their own labels, below). If the person is signed in on several browsers or devices, "{count} sessions" is shown.
- "Signed in … · last seen … · until …". **Last seen** comes from the access log ("not yet" if they haven't opened a recorded page since signing in).
- Email addresses are shown only to people allowed to view users (for sessions opened through an email, domain or identity provider share, also to people who can manage shares).
- Someone who signed in with an email code without a Komuta account is shown by their email address, with the **Outside your organization** badge ("Signed in with an e-mail code" to people who can't see the address). They can be signed out like anyone else.
- Someone who signed in with your identity provider is shown by the name and email the provider sent (the email only if Komuta accepted it, see [What your application receives](#what-your-application-receives)), with the **Outside your organization** badge. The name and email are shown only to people allowed to view users or manage shares; everyone else, and everyone when the provider sent neither, sees "Signed in with {provider}" ("Signed in with an identity provider" if the provider has since been removed). They can be signed out like anyone else.
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
- It applies to email-share and share-link sessions too. Visitors without a Komuta account (email code or identity provider) stay signed in for at most 12 hours, even with a longer setting.
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

The person an email address is shared with doesn't need a Komuta account with that address; they prove they can read the mailbox, either after signing in with any Komuta account (below) or without an account (see [Without a Komuta account](#without-a-komuta-account)).

1. The person opens the service and signs in with any Komuta account (they can create one with Google or GitHub).
2. If their account matches no other share, they see the **You don't have access** page. In its **Shared with your email address or company domain?** section, the address field is prefilled with the account's email; they change it if the shared address is different.
3. **Email me a code** sends an **8-digit** code, valid for **10 minutes**, to the address. The email's subject is "Your Komuta access code for {service}", and it shows the Komuta account that asked for the code, the service and the address.
4. The person enters the code and chooses **Verify and continue**; the service opens.

Rules:

- The code works only once, and only for the Komuta account that asked for it (without an account: only in the browser that asked for it). A new code is needed for every sign-in; a session opened through an email share lasts the service's **Stay signed in for** length too.
- A code stops working after 5 wrong attempts. After 50 wrong attempts in 24 hours for the same account, service and address, no new codes are sent. After **200 wrong codes in 24 hours for one address on a service** from people without a Komuta account, nobody without an account gets new codes for that address on that service until the period ends. People signing in with a Komuta account aren't blocked this way: each of them still stops at 50, and 500 wrong codes for one address on a service only raise an alarm at Komuta. Wrong codes without an account never stop codes for Komuta accounts, and the other way round.
- Code request limits: a user can request at most 20 codes per hour, and at most 5 per hour for the same address. At most 3 codes per hour are sent to an address for the same Komuta account and service; switching browser or device doesn't reset this, and beyond it the screen still says a code was sent but no email goes out. Resending waits 30, 60 and 120 seconds.
- The screen gives the same answer whether or not the address has access ("If {email} has access to this page, we've emailed it a {length}-digit code."), so nobody can guess which addresses a service is shared with.
- Sign-in must be completed within about 9 minutes of being sent from the protected page, so enter the code without waiting; the code screen says how many minutes are left. With less than 2 minutes left no new code is sent and the screen says "This sign-in link is about to expire."; open the protected page again and request a code.
- The email address must be a plain address: ASCII letters, at most 254 characters; wildcards, spaces and IP addresses aren't accepted. An address at an internationalised domain is typed in its `xn--` form (`ali@xn--irket-idb.com.tr` for `şirket.com.tr`).


### Without a Komuta account

Someone an email address or domain is shared with can also get in without a Komuta account:

1. They open the service. On the **Sign in to continue** page, under **Sign in with Komuta** and the **or** divider, is the **Shared with your email address or company domain?** section.
2. They type their address and choose **Email me a code**. The email says the code was asked for by "Someone without a Komuta account, on the service's sign-in page" and that the code only works in the browser where it was requested.
3. They enter the code on **Enter the code** and choose **Verify and continue**; the service opens.

Things to know:

- The section appears only while the service has an active **An email address** or **Everyone at a domain** share (not suspended, not expired). If your platform hasn't turned on sign-in without an account yet, it doesn't appear and visitors use their Komuta account as before.
- Someone who has a Komuta account can use either way; through the code they are an email visitor, not their account.
- The visitor isn't a Komuta user. Your organization sees them by their email address: in the access log and in **Who is signed in** by the address (people who can't see emails see **Someone who signed in with an e-mail code** in the log). The same address is the same visitor on all your organization's services, and a different one in other organizations.
- If visitor identity is on, your application receives the visitor's email and an id of the form `eml:<32 hex>` instead of a Komuta user id (see [Settings](access-protection-settings.md#headers-your-application-receives)).
- Removing the share, or the share ending, closes their session like everyone else's.

Limits without an account, counted per network (an IPv4 address, or an IPv6 `/64`) instead of per account:

- At most 60 code requests per hour per network and service, and at most 5 per hour for the same address. At most 3 codes per hour are sent to an address from one network.
- A wider limit counts the whole IPv6 `/48` (or the IPv4 address): at most 240 code requests per hour per service, so a visitor can't get around the limits by changing addresses inside their range.
- An address receives at most 10 codes per hour from visitors without an account on one service, and 30 across your organization; above that the screen still says a code was sent but no email goes out. These limits don't affect people signing in with Komuta.
- A network that sends too many requests in a short time is refused for a moment (a code request then shows "Too many codes were requested. Wait an hour and try again.", even though a minute is usually enough).
- After 50 wrong codes in 24 hours for the same network, service and address, no new codes are sent.

---

## Domain shares

An **Everyone at a domain** share lets in anyone who can read a mailbox at that domain, for example the whole team at `@example.com`, without adding people one by one.

- Write the domain only, such as `example.com` (`@example.com` also works). Internationalised domains are accepted; they are stored and listed in their `xn--` form (for example `şirket.com.tr` becomes `xn--irket-idb.com.tr`), and visitors must type their address in that form on the access page.
- **Also addresses at its subdomains** lets in `@team.example.com` and similar too. It can be chosen only for a domain your organization has verified, because anyone who controls a subdomain could otherwise get in.
- Public mail services and shared domains where anyone can get an address (`gmail.com`, `outlook.com`, `yahoo.co.uk`, `co.uk`, `onmicrosoft.com` and similar) can't be shared ("Anyone can get an e-mail address at '{Domain}', so a share for it would let anyone in."). Share with single email addresses instead.
- Visitors sign in exactly like email shares: they confirm their address at the domain with an **8-digit** code in the **Shared with your email address or company domain?** section, after signing in with any Komuta account or without an account. A Komuta account's own email address never opens a domain share by itself.
- Someone who leaves the company can't receive new codes once their mailbox is closed; a session that is already open lasts until it ends (**Stay signed in for**).

Extra limits protect the domain's mailboxes:

- One Komuta account can request codes for at most **3 different addresses** of the same domain share in 24 hours.
- After **200 wrong codes in 24 hours** on a domain share, no new codes are sent for it until the period ends.
- These two limits don't affect someone who also has their own **An email address** share; they keep getting codes through that share (within its own limits).
- Without a Komuta account the requester is the visitor's network, so an office behind one address isn't held to 3: up to **20 different addresses** of a domain share get codes from one network in 24 hours. Wrong codes entered without an account are counted apart: 200 of them pause codes only for visitors without an account, never for people signing in with Komuta.
- Visitors without a Komuta account get at most **100 codes per hour** for one domain share, across all their networks and addresses.
- Like email shares, the screen gives the same answer whether or not an address has access.

## Verified domains

Verify a domain your organization owns so that shares with everyone at that domain count as your own:

- Setting: **Account → Organizations → Verified domains**. Changing it needs permission to edit the organization.
- Choose **Add domain**, then add the TXT record shown at your DNS provider: name `_komuta-verify.<domain>`, value `komuta-verify=<code>`. Choose **Check now** (at most once every 10 seconds); Komuta also checks every 6 hours.
- A verified domain also covers its subdomains: verifying `example.com` makes shares for `team.example.com` your own too.
- Keep the record in place. If the record is missing on 3 checks in a row and was last found at least 2 days ago, or DNS for the domain gives no answer and the record was last found more than a week ago, the domain stops counting as verified (**Record no longer found**).
- At most **20** domains per organization. Each organization verifies its own domains; verification isn't shared with linked organizations.

What changes when a domain is verified, lapses or is removed:

- A share under a verified domain keeps working while external sharing is off.
- When a domain stops counting as verified (or is removed), its shares count as external again: while external sharing is off they are **Suspended**, and a share that also covers subdomains is suspended in every case. People who came in through them lose access within about 30 seconds; the other visitors of these services sign in once more too. They come back once the domain is verified again.

---

## Sign in with your organization's identity provider

Visitors without a Komuta account can sign in with your organization's own Microsoft Entra ID, Google Workspace, Okta or any other OpenID Connect (OIDC) provider. You add the provider once for the organization, then share services with **Everyone who signs in with {provider}**, optionally only with certain groups.

### Adding a provider

- Setting: **Account → Organizations → Visitor sign-in with your identity provider**. Changing it needs permission to edit the organization. If you don't see it, it isn't enabled on your platform yet.
- First create an application (OIDC client) at your identity provider, then choose **Add identity provider** and enter its details:

| Field | Contents |
|---|---|
| **Provider** | **Microsoft Entra ID**, **Google Workspace**, **Okta** or **Other OIDC provider**. |
| **Name** | Visitors see it on the sign-in page as "Continue with {name}". At most 64 characters. |
| **Directory (tenant) ID** (Microsoft Entra ID) | The Directory (tenant) ID from your app registration's overview, or its v2.0 issuer URL. Only accounts of this directory can sign in. |
| **Workspace domain** (Google Workspace) | Your Google Workspace domain, such as `example.com`. Only accounts of this Workspace can sign in. |
| **Issuer URL** (Okta, other providers) | The `https` issuer URL; for Okta, the issuer of your authorization server, for example `https://your-org.okta.com/oauth2/default`. When you add the provider, Komuta reads its configuration from `/.well-known/openid-configuration`. |
| **Client ID**, **Client secret** | From the application you created. The secret is stored encrypted and is never shown again; when it expires, edit the provider and enter a new one (leave the field empty to keep the current one). |
| **Groups claim** | Optional; not for Google Workspace. The ID token claim that lists the visitor's groups; `groups` if left empty. |

- After you add the provider, Komuta shows its **Redirect URI**: `https://console.komuta.io/access/sso/callback/{providerId}`. Each provider has its own. Register it at your identity provider as a web redirect URI, exactly as shown and without wildcards; otherwise visitors can't finish signing in. You can copy it again from the list.
- An organization can add at most **5** providers.
- Later you can change the name, the client ID, the client secret and the groups claim. Changing the groups claim signs out everyone signed in to the services that have a share limited to groups on this provider, members of your organization included; they sign in once more. The provider type, directory, Workspace domain and issuer can't be changed; add the provider again instead.
- A provider can't be removed while shares use it ("Remove the shares that use this identity provider first.").

Komuta refuses providers that would let anyone in:

- For **Okta** and **Other OIDC provider**, issuers anyone can sign in at: Microsoft's shared `common`, `organizations` and `consumers` endpoints and personal Microsoft accounts, `accounts.google.com` (use the Google Workspace choice instead), and public sign-in services such as GitHub, GitLab, Apple, Facebook, LinkedIn, Slack or Discord ("Anyone can sign in at '{Issuer}', so it would let anyone in. Use your organization's own tenant or domain.").
- For **Microsoft Entra ID**, a **Directory (tenant) ID** that isn't your own directory's ID: `common`, `organizations`, `consumers` and the personal Microsoft account tenant are refused as not valid ("The identity provider setting 'Issuer' is not valid.").
- An issuer that isn't a plain `https` address with a domain name (no IP address, port, query or fragment).
- For Google Workspace, a public mail domain such as `gmail.com`. Every Google sign-in is bound to the Workspace domain: an account from another Workspace, or a personal Google account, is refused.
- For Microsoft Entra ID, an account from another directory is refused.

### Sharing a service with a provider

On the **People** tab choose **Add share → Your identity provider** and pick the provider under **Identity provider**. If your organization hasn't added one yet, the window links to **Account → Organizations**. A service can have one share per provider.

- **Only these groups (optional)** — one group per line, at most 20, each at most 256 characters. Leave it empty to let everyone who signs in with this provider in; with groups, only visitors whose ID token lists at least one of them in the groups claim get in.
  - Configure your provider to include the groups claim in the ID token.
  - Microsoft Entra ID sends group object IDs (GUIDs), not group names.
  - Groups match exactly, including upper and lower case: `Admins` doesn't match `admins`.
  - Google Workspace doesn't send groups, so its shares can't be limited to groups; neither can a provider without a groups claim.
  - If a user is in too many groups, Microsoft Entra ID leaves the groups out of the token (group overage). Komuta then can't tell their groups, so group-limited shares refuse them; shares without groups still let them in.
- **What they can open** and **Access ends** work as for other shares.
- It counts as external sharing: it needs **Allow external sharing**, carries the **External** badge, and is **Suspended** while external sharing is off.
- Limiting an existing share to groups, or removing or replacing one of its groups, signs everyone signed in to the service out, including members of your organization; they sign in once more. Adding groups or clearing the list doesn't. If a provider's groups claim changes, every service with a group-limited share of that provider signs everyone out the same way.

### What the visitor sees

1. On the **Sign in to continue** page the visitor chooses **Continue with {name}** and is sent to your identity provider (Microsoft and Google ask which account to use).
2. After signing in there, they see **Finishing your sign-in** in the Komuta console and then return to the page they wanted.

If it doesn't work, the page shows one of these, with a **Back to the protected page** button (or **Go to Komuta**):

| Page | When |
|---|---|
| **This sign-in has expired** — "This sign-in has expired or was opened in another browser. Open the protected page again." | Sign-in took too long, or was finished in another browser. |
| **Sign-in didn't complete** — "The identity provider did not sign you in." | The provider answered with an error, for example the visitor cancelled or isn't allowed to use the application. |
| **You don't have access** — "You signed in, but your account isn't in a group this page was shared with, or its share has ended. Ask the page owner to add you." | No share lets them in: they aren't in the share's groups, or the share has ended. |
| **You don't have access** — "This page is only shared with certain groups, but your identity provider didn't send your groups, so we couldn't check them. Ask your administrator or the page owner." | The share is limited to groups and the provider didn't send the visitor's groups (for example Microsoft Entra ID group overage). |
| **You don't have access** — "The organization this page belongs to turned off sharing outside the organization, so signing in with this provider is paused." | External sharing is off. |
| **You don't have access** — "Your account doesn't have access to this page." | Any other refusal. If the provider's groups claim was changed while the visitor was signing in, they open the protected page and sign in again. |
| **This sign-in isn't available anymore** — "Signing in with this provider isn't available for this page anymore." | The provider was removed, or signing in with an identity provider was switched off on Komuta. |
| **Too many sign-in attempts** — "Too many sign-in attempts. Wait a few minutes and try again." | Over the limits below. |
| **We couldn't sign you in** — "Something went wrong while finishing the sign-in. Open the protected page again to retry." | Anything else, for example the provider's answer couldn't be verified. |

Starting the sign-in can also fail on the sign-in page itself: "Signing in with this provider isn't available for this page anymore." (the share is gone or suspended), "Too many sign-in attempts. Wait a few minutes and try again." (too many sign-in starts from one network, see below) or "We couldn't start the sign-in. Try again in a moment."

Things to know:

- Sign-in, including the step at your provider, must be completed within about 9 minutes, in the same browser.
- The visitor isn't a Komuta user. The same account at the same provider is the same visitor on all your organization's services, and a different one in other organizations.
- Their sessions last the service's **Stay signed in for**, but at most 12 hours.
- At most 30 sign-in starts and 30 sign-in finishes per hour per network (an IPv4 address, or an IPv6 `/64`) and service.
- In the access log, people allowed to view users or manage shares see their accepted email, or the name the provider sent; everyone else sees **Someone who signed in with {provider}**. In **Who is signed in** they appear as described in [Who is signed in](#who-is-signed-in).
- Removing the share, the share ending or external sharing being turned off ends their sessions within about 30 seconds, like other shares. Disabling someone at your identity provider stops their new sign-ins; a session that is already open lasts until it ends (at most 12 hours) unless you sign them out on the **People** tab.

### What your application receives

If **Tell my application who signed in** is on (see [End Date and Visitor Identity](access-protection-settings.md#headers-your-application-receives)):

- `x-komuta-identity` carries `kind: sso`, and `x-komuta-user-id` and the JWT's `sub` are `sso:` followed by 32 lowercase hex characters.
- `x-komuta-user-email` is filled only when the provider marks the address as verified **and** the address is at one of your organization's [verified domains](#verified-domains) (or their subdomains), or, for Google Workspace, at the Workspace domain. Otherwise it is empty and the JWT has no `email`. This way a provider can't name an address your organization doesn't own.
- Komuta takes an email from the provider only when the ID token says it is verified: `email_verified`, or for Microsoft Entra ID `xms_edov` (the address's domain is verified) for a member of your directory (`acct` is `0`). Microsoft Entra ID sends none of these by default: to pass members' emails on, add the optional claims `email`, `xms_edov` and `acct` to the ID token in your app registration (**Token configuration** → **Add optional claim** → **ID**). Guests of your directory and visitors whose token lacks these claims reach your application without an email; tell them apart by the `sso:` id. Identify people in your application by the `sso:` id, not by the email: an administrator of your directory can change a user's email address. The name the provider sent is shown only to people allowed to view users or manage shares, in **Who is signed in** and the access log.

---

## Allowing external sharing

**A linked organization**, **An email address** and **Your identity provider** shares give access outside your organization. They require your organization to allow external sharing:

- Setting: **Account → Organizations → Allow external sharing**.
- **It is off by default.**
- Changing it needs permission to edit the organization.
- Turning it on applies immediately. Turning it off asks for confirmation (**Turn off external sharing?**).

While it is off, these types can't be chosen in the **Add share** window and the list says so. If you can change the setting, **Open organization settings** opens it highlighted in a new tab; once you turn it on and come back, the options become available without reloading. Otherwise, ask an organization admin to turn it on. **Everyone at a domain** can still be chosen while it is off, but only for a domain your organization has verified; such shares are your organization's own and don't need this setting (see [Verified domains](#verified-domains)).

If you turn it off:

- Linked organization, email and identity provider shares on all of the organization's protected services become **Suspended**, and so do domain shares whose domain your organization hasn't verified. Domain shares under your own verified domains keep working.
- People who came in through these shares lose access within about 30 seconds; the other visitors of these services sign in once more too.
- The **Continue with {name}** buttons disappear from the sign-in page.
- New external shares can't be added.

If you turn it back on, email, domain and identity provider shares come back (except a domain share that also covers subdomains of a domain that isn't verified; it stays suspended until the domain is verified again); linked organization shares come back if the link between the two organizations still exists. Komuta checks the link every 5 minutes; if it is broken (no pair of linked, active user accounts remains between the two organizations), the linked organization share is suspended.

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
- [End Date and Visitor Identity](access-protection-settings.md) — the headers your application receives.
- [Reference](access-protection-reference.md) — sign-in error messages.
