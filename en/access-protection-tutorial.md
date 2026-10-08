# Setup Guide: From Quick Start to Advanced Scenarios

This guide sets up access protection step by step on one example service. It starts with the simplest setup and adds one capability per level. By the end of the guide, the service does all of these at once:

- Your team gets in with Komuta accounts, and people in the office get in without signing in.
- A customer sees only the pages set aside for them, until a set date.
- The admin panel is open to two people only; internal pages are closed to everyone.
- GitHub webhooks and CI jobs get in without signing in, each in its own way.
- A background service on another cluster reaches the service directly over the private mesh.
- The application recognises who got in through a signed proof of identity.
- Protection ends on a set date, and who got in and who was refused is on record.

Each step has four parts:

- **Do** — what you do on screen. Bold text is the name in the console.
- **Why** — the purpose of the step.
- **Effect** — what changes for visitors after you save.
- **Check** — how to see that it works.

If a section or option is missing, or says it isn't available yet, that feature isn't on for your platform yet.

For every detail of the features, see [Access Protection](service-access-protection.md) and [Reference](access-protection-reference.md); this guide focuses on "how to set it up".

---

## The example scenario

Throughout the guide we use this service:

| | |
|---|---|
| Service | `panel` — an internal management application |
| Address | `https://panel.example.com` (a custom domain added to Komuta) and the service's `*.komuta.app` address |
| Organization | Your team's Komuta organization |
| Customer | Someone outside your organization who uses the address `customer@example.com` |
| Paths | `/` (the application), `/reports` (reports that will also be opened to the customer), `/admin` (admin panel), `/internal` (internal tools), `/webhooks/github` (GitHub notifications), `/api/health` (health check) |

On your own service, follow the same steps with your own paths.

---

## Before you start: planning

The step that saves the most time with access protection is answering these five questions before opening the settings. Each answer maps to a level of this guide.

| Question | Example answer | Level |
|---|---|---|
| Who should be able to open the service? | Our team and a customer | 1, 2 |
| From where should it open? | No sign-in from the office, sign-in from outside | 3 |
| Should some pages be protected more tightly or closed? | `/admin` for two people, `/internal` for nobody | 4 |
| Will non-human clients reach the service too? | A GitHub webhook, a CI health check, a service on another cluster | 5, 6 |
| Should the application recognise who got in? When does protection end? | Yes; at launch | 7, 8 |

**The don't-lock-yourself-out rule:** the Komuta console isn't affected by protection. If a setting goes wrong, you can always fix it from the console or remove protection with **Settings → Open to everyone now**.

---

## Level 0: Preparation

### Step 0.1 — Check that the service is ready

**Do**

1. Open **Service Detail → Configuration → Access & ports**.
2. On the **Network** tab, see that **Public URL** is on and that the **Public addresses** list has at least one address.
3. Make sure the service has been deployed successfully at least once.

**Why** — Access protection checks the traffic that comes to the service's internet address. A service without a public address, or one that has never been deployed, has nothing to protect; the card locks the switch in that case.

**Effect** — Nothing changes yet.

**Check** — The **Public exposure** card on the **Overview** tab should say **Open to everyone**.

### Step 0.2 — Check your permissions

**Do** — Make sure your organization admin has given you these two permissions and edit access to the service:

- **Manage service access protection** — to turn protection on and change rules and settings.
- **Manage who a protected service is shared with** — to share the service with people and create service tokens.

**Why** — The two jobs are separated on purpose: a team lead can manage who gets in without being able to change the rules. Following this guide end to end needs both.

**Effect** — Without them you see the card read-only ("Read-only — you need permission to manage access protection to change these settings.").

### Step 0.3 — Check your custom domain

**Do** — If you use a custom domain (`panel.example.com` in the example), check that its record at your own DNS provider points to Komuta and that it is **not** proxied (orange cloud) in your own Cloudflare account but **DNS only** (grey cloud).

**Why** — Protection covers all addresses of the service together. Komuta can't protect a domain proxied through your own Cloudflare account, and protection doesn't complete, with the warning "A custom domain is proxied through your own Cloudflare account."

**Effect** — Once protection is on, both `panel.example.com` and the `*.komuta.app` address are protected by the same rules.

---

## Level 1: Quick start — only my team

Goal: only members of your organization can open the service, by signing in with their Komuta accounts.

### Step 1.1 — Turn protection on

**Do**

1. Go to the **Rules** tab.
2. Turn on the switch in the header of the **Access protection** card.
3. **Require Komuta sign-in** comes selected; leave it.
4. Press **Apply protection**.

**Why** — The switch only opens the settings and saves nothing; the service runs as before until you press **Apply protection**. This keeps you from leaving protection half done by accident. **Require Komuta sign-in** is the simplest way to know who a visitor is.

**Effect**

- The card shows **Preparing**, then **Applying**, and finally **Protected**. This usually takes a minute or two.
- From **Applying** on, anyone who opens the service is sent to the Komuta sign-in page.
- Komuta also locks the service's pods; requests that skip the gateway and go straight to a pod are refused.
- If you have permission to manage shares, you are added to the share list automatically. This way you don't lock yourself out of your own service.

**Check**

- See that the card says **Protected**.
- Open a private window in your browser and go to `https://panel.example.com`: you should see the **Sign in to continue** page with the service's address under **You're going to**.
- From the command line:

```bash copy
curl -sI https://panel.example.com/ | grep -i -E '^(HTTP|location)'
```

You should see `HTTP/2 302` and `location: https://console.komuta.io/access/…`.

### Step 1.2 — Share the service with your team

**Do**

1. On the **People** tab, press **Add share**.
2. Choose the **Your organization** card.
3. Keep **Whole site** under **What they can open**; leave **Access ends** empty.
4. Save with **Add share**.

**Why** — Sign-in finds out who is coming; a share decides who may get in. Without a share, everyone who signs in stays on the **You don't have access** page.

**Effect** — Every active member of your organization, today's and those who join later, opens the service after signing in. Someone removed from the organization loses access.

**Check** — When a teammate opens the service, they should sign in and return straight to the application. **Signed in** and **Opened the page** rows should appear on the **Activity** tab.

> The quick start is complete at this point. Your service is now open only to your team. The next levels are optional and build on each other in order.

---

## Level 2: Time-limited, narrow access for a customer

Goal: a customer outside the organization sees only the `/reports` pages, and only until a set date.

### Step 2.1 — Allow external sharing

**Do**

1. Open **Account → Organizations**.
2. Turn on **Allow external sharing**. (Changing it needs permission to edit the organization; if you don't have it, ask an organization admin.)

**Why** — Giving access outside the organization is a deliberate decision; that's why this setting is **off by default** and lives at organization level.

**Effect** — The **A linked organization** and **An email address** share types, and **Everyone at a domain** for domains your organization hasn't verified, become available on all of the organization's protected services. If you turn the setting off later, those shares become **Suspended** and the people who came in through them lose access.

### Step 2.2 — Add the customer by email address

**Do**

1. Choose **People → Add share → An email address**.
2. Type `customer@example.com` in **Email address**.
3. Under **What they can open**, choose **Only these pages** and type `/reports` in the field.
4. Enter the project's end date in **Access ends**.
5. Save with **Add share**.

**Why**

- An **email share** doesn't require the customer to join your organization. The customer signs in with any Komuta account and proves they can read the mailbox with a one-time code at every sign-in.
- **Only these pages** is the principle of least privilege: the customer opens `/reports` and the pages under it, not the rest of the application.
- **Access ends** removes the risk of forgetting to take the access away.

**Effect**

- The customer opens the service, signs in to Komuta (creating an account with Google or GitHub in seconds if needed), chooses **Email me a code** in the **Shared with your email address or company domain?** section of the **You don't have access** page, enters the 8-digit code sent to the address and gets in with **Verify and continue**.
- If they open a page outside `/reports`, they see **This page is not shared with you** and the list of pages they can open.
- On the end date, access ends on its own; the share stays in the list with an **Expired** badge.

**Note** — The **Your organization** share opens the whole site, so your team isn't affected. But if you try to limit a teammate with a page-limited member share, it won't work: **the widest share wins**.

**Check** — In the **Access preview** at the bottom of the **Rules** tab, choose the **What a person can open** view and pick the customer's share: you should see **Outside the pages shared with them** for `/` and **Gets in after signing in** for `/reports`.

---

## Level 3: No sign-in from the office, sign-in from outside

Goal: people coming from the office network get in without signing in; people connecting from home or on the road sign in with Komuta.

### Step 3.1 — Add the office addresses

**Do**

1. On the **Rules** tab, type your office's public IP address or range into the **IP allow-list** (one address or CIDR range per line). If you are in the office, you can use the **Add my IP** button.
2. Don't press **Apply protection** yet.

**Why** — An office network usually goes out from a fixed public IP. That address is the reliable source of "this person is connecting from the office": Komuta takes the address only from the value Cloudflare reports and ignores headers the visitor sends themselves.

**Effect** — None until you save. The card marks an invalid line as "Line {n}"; private network ranges (such as `10.x` or `192.168.x`) are refused, because a request from the internet can't come from those addresses.

**Note** — **Add my IP** adds the address the console sees. If your browser reaches the service over a different connection (for example IPv6 to the console, IPv4 to the service), the service sees another address. Adding both your IPv4 and IPv6 addresses is the safe choice.

### Step 3.2 — Choose "Either is enough"

**Do**

1. With **Require Komuta sign-in** on and addresses on the list, the card asks **How should sign-in and the IP allow-list combine?**.
2. Choose **Either is enough**.
3. Press **Apply protection**.

**Why** — The two options give completely different results:

| Option | Employee in the office | Employee at home | Customer |
|---|---|---|---|
| **Require both** | Signs in | Can't get in (403) | Can't get in (403) |
| **Either is enough** | Gets in without signing in | Signs in | Signs in |

**Require both** closes the service completely outside the office; your customer can't get in either. In this scenario we want **Either is enough**.

**Effect**

- People coming from the office address open the application without seeing the sign-in page.
- Everyone else signs in as in Levels 1 and 2; shares apply as before.
- People coming from the office address aren't written to the access log on these pages and aren't identified to the application, even if they signed in earlier: the request passes on its IP address, so their session isn't looked at. This matters in Level 7.

**Check** — In the **Access preview**, choose the **Who can open a page** view and, with **Page** set to `/`, set **Coming from** to **A specific address** and type your office address: for someone who hasn't signed in you should see **Gets in from an allowed network**. With **Outside the allowed networks**, the same row should say **Needs to sign in (whole site)**.

---

## Level 4: Page-by-page protection

Goal: `/admin` is open to two people only, and `/internal` to nobody.

### Step 4.1 — Close the internal tools completely

**Do**

1. In the **Path rules** section of the **Rules** tab, press **Add path rule**.
2. **Path**: `/internal`, **Protection**: **Block completely**.
3. Don't press **Apply protection** yet; we'll apply it together with Step 4.2.

**Why** — Some pages should never open from the internet, even if the application has a bug. A block rule is the strongest rule: sign-in, an allowed IP, a service token or a webhook path can't get past it.

**Effect** — `/internal` and every path below it return `403` to everyone. The rule can't be bypassed with case changes, `%2f`, `..` or similar spelling tricks.

### Step 4.2 — Open the admin panel to two people

**Do**

1. First, on the **People** tab, add the two people who use the panel as **A member**. (The organization share already exists; member shares are there so you can pick people one by one in the rule.)
2. On the **Rules** tab, **Add path rule**: **Path** `/admin`, **Protection** **Only chosen people**.
3. In **Who can open this path**, tick the two people's shares. If you use the panel too, also tick your own share, which was added automatically in Step 1.1.
4. Optionally give someone **From (optional)** and **Until (optional)** times (for example, a consultant for one week only).
5. Press **Apply protection**.

**Why** — The site is open to the whole team, but the admin panel isn't everyone's job. **Only chosen people** narrows this one path without deleting any shares.

**Effect**

- Only the two people you chose get into `/admin`; other team members keep using the rest of the site and see **This page is not shared with you** on `/admin`.
- **Rules only tighten:** even someone in the office must sign in for `/admin`, because this rule asks for the person, not the office address.
- The time window is checked on every request; when it ends, the path closes within a few seconds, including for open sessions.
- Service tokens can never open this path.

**Check** — In **Access preview → Who can open a page**, type `/admin` in **Page**: you should see a green check for the two people, **Not among the people chosen for /admin** for the organization share and the other member shares, **Outside the pages shared with them** for the customer's share, and **Needs to sign in (whole site)** for someone who hasn't signed in. Set **Coming from** to **A specific address** with your office address and the signed-out row turns into **Needs to sign in (/admin)**: someone in the office must sign in on this path too. For `/internal`, everyone should show **Blocked by the /internal rule**.

---

## Level 5: Let machines in

Goal: GitHub webhooks and the CI job's health check get in even though they can't sign in.

### Step 5.1 — An open path for the GitHub webhook

**Do**

1. On the **Machines** tab, press **Webhooks → Open a path**.
2. **Path**: `/webhooks/github`.
3. **Methods**: `POST` only (the default).
4. **Sender addresses (optional)**: type GitHub's webhook address ranges. GitHub publishes this list in the `hooks` field at `https://api.github.com/meta` (for example `140.82.112.0/20`).
5. Save with **Open the path**.

**Why** — GitHub isn't a browser; it can't open the Komuta sign-in page and sign in. An open path opens this one path, only for the method you choose, without asking for sign-in. The sender list is an extra filter; the real guarantee is GitHub's signature.

**Effect**

- `POST /webhooks/github` reaches your application without sign-in. The site's IP list doesn't apply to this path; the only address check is the path's own sender list.
- Other methods, such as `GET /webhooks/github`, are decided as if there were no open path and must pass the site's normal protection.
- In the access log, deliveries appear as **Request on an open path** / **Delivered to an open path**.

**What your application must do** — Without a signature check at the edge (see the recipe below), Komuta doesn't ask who sends requests on this path; your application must verify that a request really comes from GitHub. GitHub adds an `X-Hub-Signature-256` header to every request: the HMAC-SHA256 of the request's raw body with your webhook secret. A Node.js example:

```javascript copy
import { createHmac, timingSafeEqual } from "node:crypto";

export function isFromGitHub(rawBody, signatureHeader, secret) {
  if (!signatureHeader) return false;
  const expected = "sha256=" + createHmac("sha256", secret).update(rawBody).digest("hex");
  const a = Buffer.from(expected);
  const b = Buffer.from(signatureHeader);
  return a.length === b.length && timingSafeEqual(a, b);
}
```

Refuse any request whose signature doesn't match. Also don't accept method-override headers such as `X-HTTP-Method-Override` on this path.

**Check** — **Recent Deliveries** on the webhook page of your GitHub repository settings should show success, and the **Activity** tab should show a **Request on an open path** / **Delivered to an open path** row. If you run `curl -s -o /dev/null -w "%{http_code}\n" -X POST https://panel.example.com/webhooks/github` from your own computer you get `403`: your address isn't on the sender list. (The **Access preview** works out `GET` requests only, so for this `POST`-only path it shows the site's normal rule.)

### Recipe: Receive GitHub webhooks with signature check

Instead of verifying GitHub's signature in your application, you can let Komuta do it at the edge. If the **Signature check at the edge** choice says it isn't available, this isn't enabled on your platform yet; keep the check in your application. The check is chosen when a path is opened, so if you already opened `/webhooks/github` in Step 5.1, close it first.

1. On the **Machines** tab, press **Webhooks → Open a path**: **Path** `/webhooks/github`, **Methods** `POST`, **Signature check at the edge** **GitHub (X-Hub-Signature-256)**. Fill in **Sender addresses** as in Step 5.1 if you like. Press **Open the path**.
2. Under the new path, in **Signing secrets**, press **Add secret**, keep **Generate a strong secret**, press **Add secret** and copy the secret shown. It is shown only once.
3. In your GitHub repository, open **Settings → Webhooks**, set **Payload URL** to `https://panel.example.com/webhooks/github`, **Content type** to `application/json` and paste the secret into **Secret**.
4. Wait about a minute for the secret to reach the gateway.

**Effect** — Only `POST` requests signed with your secret reach your application; anything else on `/webhooks/github` gets `401` (including `GET`), and a body larger than 65,535 bytes gets `413`. A large `push` event can exceed that limit; if your repository sends such events, keep the check in your application instead.

**Check** — GitHub's **Recent Deliveries** should show success (redeliver the `ping` event if needed). `curl -s -o /dev/null -w "%{http_code}\n" -X POST -d '{}' https://panel.example.com/webhooks/github` from a listed address (or with an empty sender list) returns `401`: the request isn't signed. On the **Activity** tab, refusals appear as "Sent a webhook without a valid signature".

**Rotating the secret** — add a second secret, switch GitHub to it, then remove the old one; both work in between. A path holds at most 2 secrets.

### Step 5.2 — A service token for the CI job

**Do**

1. On the **Machines** tab, press **Service tokens → Create token**.
2. **Name**: `github-actions-health`.
3. **Stops working**: for example three months from now.
4. **What it can open**: **Only these pages** and `/api/health`.
5. Press **Create token**. Copy the value on the **Copy your token now** screen and close it with **I saved it**.
6. Store the value in your GitHub repository under **Settings → Secrets and variables → Actions** as `KOMUTA_SERVICE_TOKEN`.
7. Send the token as a header in the CI step:

```yaml copy
- name: Health check
  env:
    KOMUTA_SERVICE_TOKEN: ${{ secrets.KOMUTA_SERVICE_TOKEN }}
  run: |
    status=$(curl -sS -o /dev/null -w "%{http_code}" \
      -H "x-komuta-service-token: $KOMUTA_SERVICE_TOKEN" \
      https://panel.example.com/api/health)
    test "$status" = "200" || { echo "health check returned $status"; exit 1; }
```

Check the status code instead of using `--fail`: if the token isn't read, Komuta answers with a `302` to the sign-in page, and `curl --fail` counts that as success.

**Why**

- A token is a key that stands in for sign-in; it is designed for a program without a browser.
- **Only these pages** and **Stops working** limit the damage a leaked token can do: a leaked token opens only `/api/health`, and only until its end date.
- The value is shown only once; Komuta keeps only a digest of it. That's why it goes into a secret store right away.

**Effect**

- When you create the first token on a service, Komuta refreshes the service's routing; this takes a few minutes, and until then the token seems not to work (`302`).
- The CI request reaches `/api/health` without being sent to sign-in. The token header is removed before it reaches your application.
- **The "Either is enough" choice from Level 3 pays off here:** the token satisfies the sign-in requirement, and since the site rule is "either is enough", the CI machine's IP doesn't have to be on the office list. With **Require both**, the CI machine would also have to come from a listed address; GitHub Actions machines change addresses, so that isn't practical.
- The token can't open `/admin` (people rule) or `/internal` (block, `403 access denied`); on other pages outside its scope it gets `403 this service token cannot open this path`.
- An invalid or deleted token gets `401 invalid service token` (unless it comes from an address on the office list); it isn't sent to sign-in.

**Check**

To run this on your own computer you need the token value; a GitHub secret can't be read back. Define it in your terminal with `export KOMUTA_SERVICE_TOKEN='kst_…'` before you press **I saved it**. If you don't have the value any more, the easiest check is to run the job on GitHub and see the **Health check** step pass. If the variable isn't set, `curl` doesn't send the header at all and you get `302`.

```bash copy
curl -s -o /dev/null -w "%{http_code}\n" -H "x-komuta-service-token: $KOMUTA_SERVICE_TOKEN" https://panel.example.com/api/health
```

You should see `200`. Running the same command for `https://panel.example.com/` gives `403` (outside the token's scope). **Run both from outside the office network** (for example your phone's connection, or CI): a request from the office address passes without sign-in under the "either is enough" rule from Level 3, the token isn't looked at, and both commands return `200`.

---

## Level 6: Let in your service on another cluster

Goal: the `report-worker` service running on another cluster reaches `panel` directly over the private mesh, without Komuta sign-in.

You need this level only if you run your services on different clusters and use the private mesh. If the switch on the **Private mesh** card of the **Network** tab is locked and the card says "Access protection is on for this service. … turn access protection off first", this feature isn't on for your platform yet and a protected service can't join the private mesh; skip this level. If the card says "Works together with Access protection", go on.

### Step 6.1 — Turn on the private mesh and choose the service to allow

**Do**

1. On `panel`, turn on the private mesh on the **Private mesh** card of the **Network** tab.
2. In the **Services that may come in over the private mesh** section of the **Machines** tab, choose `report-worker` in **Choose a service** and press **Allow**.

**Why** — Private mesh traffic doesn't pass Komuta's gateway; it goes straight to the pods, so sign-in or IP rules can't check it. On a protected service, this list decides who may come in over the private mesh.

**Effect**

- Only the listed services reach `panel` directly from your other clusters. If the list is empty, no service can come in over the private mesh.
- Your services on the same cluster aren't affected by this list.
- If a listed service is open to the internet and unprotected, a **Public, unprotected** warning appears: that service can be used as a stand-in to reach this one. Protect it too, or make sure it never forwards what it receives.

**Check** — A request from inside `report-worker` to `panel`'s private mesh address should get an answer; the same request from a service that isn't on the list and runs **on another cluster** should time out (services on the same cluster aren't affected by the list).

---

## Level 7: Your application recognises who got in

Goal: the `panel` application knows who opened a page without a separate sign-in screen; for example, to write it to an audit log.

### Step 7.1 — Turn on visitor identity

**Do**

1. On the **Settings** tab, turn on **Tell my application who signed in** (details on the [End Date and Visitor Identity](access-protection-settings.md#tell-my-application-who-signed-in) page).
2. Wait a few minutes while the section shows the "Getting ready" note.

**Why** — Komuta already knows who the visitor is. This setting passes that to your application as headers on every request; you don't need to write code to verify the same person a second time.

**Effect**

- On every request that needs sign-in, your application receives the `x-komuta-user-email`, `x-komuta-user-id` and signed `x-komuta-identity` headers.
- **The consequence of Level 3:** on requests from the office address the headers are empty, even if the visitor has signed in, because the request passes on its IP address and the session isn't looked at. If your application needs the identity on every page, either choose **Require both** (which also shuts out the customer from Level 2 and the CI job from Level 5), or protect the paths that need an identity (for example `/admin`) with a sign-in rule; on those paths, people in the office sign in too.
- On requests with a service token, only `x-komuta-identity` is filled (`kind: service_token`).

### Step 7.2 — Verify the identity in your application

**Do** — In your application, trust the signed `x-komuta-identity` token rather than the plain headers, and verify it on every request: the signature, `iss` (`komuta-access`), `aud` (your own addresses), `sid` (your own service id), `exp`/`nbf`. Ready-made Node.js and Python examples are on the [End Date and Visitor Identity](access-protection-settings.md#proof-of-identity-jwt) page.

**Why** — Your other services on the same cluster, and the services you allowed over the private mesh, can reach your pods without passing Komuta, so they could send the plain headers themselves. The signed token can't be faked. Checking `aud` and `sid` prevents a valid token issued to another application from being replayed to yours.

**Check** — If you chose yourself on the `/admin` rule, your own email should appear in your application's log when you open `/admin`. (If you didn't, open `/` from outside the office instead.) People who signed in before you turned this on appear without their email until they sign in again (at most the session length, 12 hours by default).

---

## Level 8: End date, monitoring and maintenance

### Step 8.1 — Give protection an end date

**Do**

1. On the **Settings** tab, enter the launch date in **End date and time**.
2. Choose **Then keep it locked** for **When it ends**.
3. Press **Save end date**.

**Why** — **Then keep it locked** is the safe answer to "what if I forget?": when the date comes, protection continues and you get an email; you decide whether to open the service. If you want the service to open on its own at launch, choose **Then open it to everyone**.

**Effect** — 24 hours and 1 hour before the end, and at the end, an email goes to the people who have been given edit access to the service and hold the **Manage service access protection** permission (or the organization admins if there are none). The **Open to everyone now**, **Extend** and **Make permanent** links in the emails open the relevant screen in the console; the action is taken there with your confirmation.

### Step 8.2 — Watch the access log

**Do** — On the **Activity** tab, choose **Show: Refusals**.

**Why** — Refusals show whether the rules work the way you expect: a webhook from an address not on the sender list (**Came from an address that is not allowed**), a path closed to the wrong person (**This page is not shared with them**), a request caught by a block rule (**The page is blocked**) or an expired token shows up here.

**Effect** — The log is kept for 30 days by default (an organization can choose 90 or 365 days), and **Export** downloads it as CSV or JSON. To prevent abuse, refusals of visitors who haven't signed in appear as **Any page** with the first part of the address (`/24` for IPv4, `/48` for IPv6); refusals by a block rule, an open path or a method rule show the rule's path.

### Step 8.3 — Know which changes sign people out

These changes end sessions within about 30 seconds; people who still have access sign in again on their next page (automatic for those signed in to Komuta; people who came in through an email share request a new code):

| Change | Whose sessions end |
|---|---|
| **Sign out** on a person's row under **People → Who is signed in** | That person's |
| Removing a share, adding an end date to it or bringing its end date forward, changing a page-limited share's page list | The sessions opened through that share |
| **Sign everyone out** | Everyone's, share-link visitors included |
| Turning off external sharing | Everyone's on services with an external share |
| Shortening **Stay signed in for** | Sessions older than the new length |

For the first 12 hours 10 minutes after session tracking begins on the service, Komuta can't yet tell sessions apart: signing one person out or changing a share then signs **everyone** out, and the **Who is signed in** section says until when. Adding a new share, extending an end date or adding a rule doesn't affect sessions.

### Step 8.4 — Remove protection when needed

**Do** — **Settings → Remove protection now → Open to everyone now**.

**Effect** — Sign-in, the IP list and path rules stop applying; the service opens to everyone. Shares, share links, service tokens and the private mesh list are kept; the visitor identity setting is kept too and comes back on its own when you turn protection back on with Komuta sign-in. When you turn protection back on, you need to enter Komuta sign-in, the IP list, path rules, webhook paths and the end date again, so keeping a note of your settings makes it easier.

---

## Final state: everything together

At the end of the guide, the `panel` service's settings are:

| Tab | Setting | Value |
|---|---|---|
| Rules | **Require Komuta sign-in** | On |
| Rules | **IP allow-list** | Your office range |
| Rules | **How should sign-in and the IP allow-list combine?** | **Either is enough** |
| Rules | Path rule `/internal` | **Block completely** |
| Rules | Path rule `/admin` | **Only chosen people** (two members) |
| People | **Your organization** | Whole site, no end date |
| People | `customer@example.com` | Only `/reports`, until the project end date |
| People | Two **A member** shares (plus your own share added in Step 1.1) | Whole site |
| Machines | Webhook path `/webhooks/github` | `POST`, GitHub addresses |
| Machines | Service token `github-actions-health` | Only `/api/health`, three months |
| Machines | Services that may come in over the private mesh | `report-worker` |
| Settings | **Tell my application who signed in** | On |
| Settings | Protection end date | Launch date, **Then keep it locked** |

### How a request is decided

To set up the most advanced scenario on your own, it's enough to know the order in which Komuta evaluates each request:

1. **Block rule** — If the path falls under a **Block completely** rule, the request is refused with `403`. Nothing else is looked at.
2. **Method rules** — If a [method rule](access-protection-machines.md#method-rules) covers the path and doesn't allow the method, the answer is `405`. A browser's CORS check is judged by the method it asks for.
3. **Countries** — If the service has a [country list](access-protection-rules.md#countries), a visitor from another country, or whose country is unknown, is refused with `403`. Webhook paths skip this step, unless a share link is being opened on them.
4. **Rate limit** — If the address has used up its [rate limit](access-protection-rules.md#rate-limit), the answer is `429`.
5. **Share link** — If the address carries `?komuta_link=`, the link is checked (the IP lists still apply) and the visitor is sent on to the same address with a session, or refused with `403`.
6. **Webhook (open) path** — If the path is under an open path and the method is chosen, the path's own sender list is checked and the request passes without sign-in. The rules of the site and of other paths don't apply. If the address isn't on the list, the request is refused with `403`; the site's rules aren't tried. If the path has a signature check, the body's signature is checked next (`401` without a valid signature, `413` for a body over 65,535 bytes). A request under a signed path that the path doesn't take gets `401` here too, instead of going on to the site's rules.
7. **The site rule and every matching path rule** — The request must satisfy all of them. The IP address can satisfy a rule's IP condition; on rules with "either is enough", a listed address stands in for sign-in. If a rule can't be met even after signing in (an IP-only rule, or **Require both** from an unlisted address), the request is refused here with `403` and the **Access to this service is restricted** page; no sign-in page is shown. A browser's CORS check passes at this point if **Allow CORS checks without sign-in** is on.
8. **Identity** — If an identity is still needed: if the request carries a service token, only the token is looked at; otherwise the visitor's session (a Komuta sign-in or a share link) and shares are looked at. The share's or link's page limit and the people rule's time window apply here. Without a session, `GET` and `HEAD` requests (from a browser or from `curl` alike) are sent to the sign-in page (`302`); other methods such as `POST` get `401`.

Example requests in this order:

| Request | Result | Why |
|---|---|---|
| From the office, without signing in, `GET /` | Opens | Step 7: site rule is "either is enough", address is listed |
| From home, team member, `GET /` | Opens after sign-in | 7–8: address not listed, organization share exists |
| From the office, `GET /admin`, team member not chosen | **This page is not shared with you** after signing in | 8: `/admin` people rule, person not chosen |
| Anyone, `GET /internal/tools` | `403` | 1: block rule |
| Customer, `GET /reports/2026` | Opens with the email code | 8: email share, page in scope |
| Customer, `GET /` | After sign-in and the email code, **This page is not shared with you** and the pages they can open | 8: outside the page limit |
| GitHub, `POST /webhooks/github` | Opens (the app verifies the signature, or Komuta if you chose a signature check) | 6: open path, sender listed |
| Anyone, `GET /webhooks/github` | Per the site rule | 6 is skipped (method not chosen), 7–8 apply |
| CI with token, `GET /api/health` | Opens | 7–8: the token stands in for sign-in, in scope |
| CI with token, `GET /admin` | `403` | 8: the people rule doesn't accept tokens |

Before you add a new rule, ask yourself: "At which step is this request decided?" If you aren't sure, the **Access preview** shows the same order for pages opened in a browser (`GET`), for people, service tokens and share links alike; it doesn't take the rate limit, other methods such as `POST`, or the private mesh into account.

---

## Common mistakes

| Symptom | Cause | Fix |
|---|---|---|
| Everyone sees **You don't have access** | Sign-in is on but there are no shares | Add a share on the **People** tab. |
| The sign-in page appears even in the office (or **Access to this service is restricted** with **Require both**) | The service sees an address that isn't on the list (for example IPv6 to the console, IPv4 to the service) | Sign in from the office and open a page; the **Address** column of the **Opened the page** row on the **Activity** tab shows the address the service sees (with **Require both**, the **Your address** box on the **Access to this service is restricted** page shows it too). Add that address to the list; add IPv4 and IPv6 together. |
| The customer can't get in at all and gets 403 | **Require both** is selected | Choose **Either is enough**, or add the customer's address to the list. |
| I limited a member to some pages but they open everything | The **Your organization** share opens the whole site | The widest share wins; limit the organization share too, or use a people rule. |
| The CI health check returns `302` | The token never reached the check: the header name is misspelled, the `KOMUTA_SERVICE_TOKEN` secret is empty or saved under another name, or the routing is still being refreshed after the first token | Check that the header is named `x-komuta-service-token` and the secret `KOMUTA_SERVICE_TOKEN`; after the first token, wait a few minutes. |
| The CI token gets `401` | The token expired, was deleted, or its value is incomplete or mistyped | Create a new token and update the secret. |
| The CI token gets `403` | The path is outside the token's scope, or the IP rule is "require both" | Check **What it can open**; set the site rule to **Either is enough**. |
| Webhooks get `403` | The sender list is incomplete or out of date | Add the whole `hooks` list from `https://api.github.com/meta` (IPv6 included); the **Came from an address that is not allowed** row on **Activity** shows the refused network. |
| Webhooks get `302` or `401` | The method isn't chosen, or the path is wrong | Check the open path's methods and path. |
| A signed webhook path answers `401` to every request | No signing secret yet, or the sender signs with another secret | Add the secret under the path and use the same one in the sender; check the **Activity** reason. |
| Some webhooks get `413` | The body is larger than 65,535 bytes, the most a signed path can verify | Keep the signature check in your application for this sender. |
| The application gets empty identity headers on some requests | The visitor came from the office address (even if signed in), or the page doesn't need sign-in | Protect the paths that need an identity with a sign-in rule. |
| The team suddenly had to sign in again | Someone chose **Sign everyone out**, or a share was removed or narrowed while one-by-one sign-out wasn't active yet | This is expected; see Step 8.3. |
| A browser on another site gets a CORS error | The browser's `OPTIONS` check carries no session and gets `401` | Turn on **Machines → Methods and CORS → Allow CORS checks without sign-in**. |
| Requests get `405` | A method rule on the **Machines** tab doesn't allow that method on that path | Add the method to the rule; the `Allow` header lists what the path accepts. |
| Requests get `429` | The address went over the rate limit | Raise the limit, or use a longer time window; remember that sign-in and webhooks count too. |
| A visitor sees **Access to this service is restricted** although no IP list applies to them | Their country isn't on the **Countries** list, or it can't be told | The **Activity** tab shows "Came from a country that is not allowed"; add the country. |
| A share-link visitor sees "The share link you opened this site with has ended." | The link was deleted or ended, or everyone was signed out | Create a new link and send it again. |

---

## Related Documents

- [Access Protection](service-access-protection.md) — overview, status and warnings.
- [Rules](access-protection-rules.md) — details of the IP list, path rules and the access preview.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — share types and what visitors see.
- [Machines and Private Mesh](access-protection-machines.md) — webhook paths, service tokens, private mesh.
- [Access Log](access-protection-activity.md) — reading the log.
- [End Date and Visitor Identity](access-protection-settings.md) — end date and JWT verification examples.
- [Reference](access-protection-reference.md) — limits, responses and error messages.
