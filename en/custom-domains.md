# Custom Domains and Service Addresses

Every service that serves HTTP gets an HTTPS address from Komuta when it is deployed. You don't have to set anything up for it. If you want visitors to reach the service at your own domain, such as `api.example.com`, you connect that domain on the **Domains** page and add a few DNS records at your DNS provider. This page explains both: the address Komuta gives you, and connecting, checking and removing your own domain.

---

## Your Service's Komuta Address

Komuta creates the address during the service's first successful deploy. Until then the services list shows **Creating URL** or **URL pending** instead of an address ("The public URL is created during deploy. It will appear after the next successful deploy.").

The address looks like this:

```
order-api-3a2331e6.edge-1.komuta.app
```

- The first part is the service's name followed by the first 8 characters of the service's ID. The name is lowercased, Turkish and accented letters are converted to plain Latin letters, spaces, `_` and `.` become `-`, other characters are dropped, and at most 40 characters of the name are used.
- The rest of the address belongs to the region the service runs in and always ends in `.komuta.app`.
- The address works over HTTPS straight away; its certificate is managed by Komuta. Requests over plain HTTP are redirected to HTTPS with a `301`.
- Requests go to the service's primary port. See [Access & Ports](services-ports.md).

The address keeps working after you connect your own domain; both lead to the same service.

### Where you see the addresses

Open **Service Detail → Configuration → Access & ports** and the **Network** tab. The **Public addresses** card lists every address of the service:

| Badge | Meaning |
|---|---|
| **Komuta domain** | The address Komuta created. |
| **Custom domain** | A domain you connected. Custom domains are listed before the Komuta address. |
| **Pending** | The custom domain isn't active yet. |
| **Redirects to {host}** | The custom domain sends its visitors to another domain of the service (see [Redirecting one domain to another](#redirecting-one-domain-to-another)). |

The copy button next to each address copies its full URL, and **Manage domains** opens the **Domains** page.

### Public URL

The **Public URL** switch on the same card decides whether the service can be reached from the internet at all. When it is off, the card says "Not published — the service isn't reachable from the internet." Turning it off asks **Turn off the public URL?** ("The service's routes are removed and its domains stop answering."): the Komuta address and every custom domain stop answering.

Jobs and cron jobs don't serve HTTP, so they get no address and can't have a custom domain.

### Blue-Green preview address

A service deployed with the Blue-Green strategy also answers at a preview address, where you can try the new version before promoting it. It is the Komuta address with `-preview` added to the first part, for example `order-api-3a2331e6-preview.edge-1.komuta.app`. Custom domains have no preview address. See [Deployment Strategies](deployment-strategies.md).

### Clearing the edge cache

**Clear edge cache** on the **Public addresses** card clears the cached copy of the service's files on all of its addresses. Komuta also does this after each deploy, so you need it only when you want a change to show up without a new deploy. It can be used once a minute.

---

## Before You Connect a Domain

- **A service with a public URL.** The domain is connected to a service that serves HTTP and whose **Public URL** is on. Only such services appear in the **Target App** list.
- **Access to the domain's DNS.** You will create three records at the company where the domain's DNS is managed.
- **Start with a subdomain.** `www.example.com` or `api.example.com` is the easiest case. A root domain (`example.com`) needs a DNS provider that supports CNAME flattening; see [Root (apex) domains](#root-apex-domains).
- **Each address is a separate domain.** `example.com` and `www.example.com` are added separately. A service can have several custom domains.
- **No wildcards.** `*.example.com` isn't accepted; add each subdomain on its own.
- **Internationalised domains** are entered in their `xn--` form (for example `xn--irket-idb.com` instead of the form with Turkish letters).
- **One service per domain.** A domain can be connected to only one service in Komuta at a time, across all organizations.
- **Permission.** Adding a domain needs permission to add domains and edit access to the target service. Checking and removing a domain needs permission to edit domains.

---

## Connecting a Custom Domain

### 1. Add the domain

1. Open [**Domains**](https://console.komuta.io/domains) under **Configuration** in the left menu.
2. Click the **+** button (**Add domain**). If you have no domains yet, click **Add Your First Domain**.
3. In the **Add Custom Domain** window, enter the domain in **Hostname**, for example `api.example.com`.
4. Choose the service in **Target App**.
5. Click **Add Domain**.

You see "Domain added. Add the DNS records shown to your DNS provider, then use Refresh Validation to check the status." and the new domain opens. Komuta updates the service's routes for the domain in the background; you don't need to deploy.

### 2. Create the DNS records

The **DNS Records to Add** section lists the records Komuta needs. Create each one at your DNS provider with exactly the type, **Name** and **Expected** value shown. Use the copy buttons next to the values to avoid typos.

For `api.example.com` the records look like this:

| Purpose | Type | Name | Value |
|---|---|---|---|
| **Traffic Routing** | CNAME | `api.example.com` | `origin.komuta.app` |
| **Ownership Verification** | TXT | `_cf-custom-hostname.api.example.com` | The code shown in the console |
| **Certificate Validation** | CNAME | `_acme-challenge.api.example.com` | `api.example.com.<code>.dcv.cloudflare.com`, as shown in the console |

- The ownership code and the certificate validation value are unique to your domain. Always take them from the console.
- Set the TTL to **Auto** or the lowest value your provider allows.
- Many DNS panels add your domain to the name automatically. If yours does, enter only the part in front of it (for example `api` or `_acme-challenge.api`).
- You add the certificate validation record once. Certificates are then issued and renewed automatically.
- Leave all records in place after the domain is active. The traffic record carries the visitors, and certificate renewals depend on the certificate validation record.
- Domains connected earlier may point to an address like `origin.edge-1.komuta.app`. They keep working, and you don't need to change them.

The **?** button on the section opens **How to add DNS records** with the same steps.

> **Note:** If the console lists `_acme-challenge` **TXT** records instead of the certificate validation CNAME, their values change with each certificate and several can share one name: add all of them.

### 3. Wait for activation

Above the records you see how far you are, for example "2 of 3 records configured". Each record shows its own state and when it was last checked. While a domain that isn't active yet is open, the page refreshes itself every 30 seconds.

Once Komuta has verified ownership and the certificate is issued, the domain's status becomes **Active** and the service answers at `https://api.example.com`.

- DNS changes usually take a few minutes to appear, and can take up to 24 hours.
- To check right away, click **Refresh Validation**. You see "Validation refreshed. If you just changed DNS, refresh again in a few minutes." The button is shown while the domain isn't active or needs attention. Refreshing also re-applies the domain to the service's routes.
- Komuta also checks the domain's status in the background, so the status changes even if you close the page.

> **Warning:** A domain that hasn't become active within 14 days of being added is removed automatically. You can add it again at any time.

---

## Statuses

### Domain status

The badge next to each domain in the list, and the counters at the top of the page, use these statuses:

| Status | Meaning |
|---|---|
| **Active** | Ownership is verified and the certificate is in use. Visitors reach the service. |
| **Provisioning** | Komuta is waiting for the DNS records or for the certificate. Nothing is wrong yet. |
| **Updating** | The certificate of an already working domain is being rolled out again. No action needed. |
| **Action Required** | The domain needs you, for example because an error was reported, DNS no longer points to Komuta, the certificate expired or the check timed out. The **Issues Detected** card and the **Sync Status** card say what to fix. |

### Sync Status

The **Sync Status** card shows two separate statuses with a one-line explanation under each, plus **Last Synced** and, if there was one, **Last Error**.

**Provider Status** is about the domain itself:

| Status | What the console says |
|---|---|
| **Active** | Traffic for this domain reaches Komuta. |
| **Pending** | Waiting for the DNS records below to be found. |
| **Updating**, **Deploying**, **Migrating** | No action needed. |
| **Under review** | The provider is reviewing this domain. Contact support if this does not clear. |
| **DNS moved** | DNS no longer points to Komuta. Check the traffic record. |
| **Blocked** | The provider blocked this domain. Contact support. |
| **Deleted** | The provider no longer has this domain. Remove it here and add it again. |
| **Not reported yet** | The provider has not reported this status yet. |

**SSL Status** is about the certificate:

| Status | What the console says |
|---|---|
| **Active** | The certificate is issued and in use. |
| **Starting**, **Issuing**, **Deploying** | The certificate is on its way. No action needed. |
| **Awaiting validation** | Waiting for the certificate validation record to be found. |
| **Expiring soon** | The certificate could not be renewed yet. Check the certificate validation record. |
| **Expired** | The certificate expired. Check the certificate validation record, then refresh. |
| **Timed out** | The provider stopped waiting. Check the DNS records, then refresh. |

### DNS record states

| State | Meaning |
|---|---|
| **Configured** | The record was found with the expected value. |
| **Pending** | The record is waiting to be verified. |
| **Different value** | DNS returns another value; **Observed** shows what it returns. Correct the record. |
| **Not detected** | "No answer from public DNS: the record is missing, or the check could not complete." |
| **Not detected yet** | The record hasn't been checked yet. |

Records that still need work are grouped under **Action required ({count})**, the finished ones under **Configured ({count})**. Once the domain is active, records that don't match are moved to **For reference ({count})** and need no action.

---

## Root (Apex) Domains

A root domain such as `example.com` is marked **Apex** on its detail panel; other domains are marked **Subdomain**. Domains like `example.com.tr` or `example.co.uk` count as root domains too.

DNS doesn't allow a normal CNAME record at the root of a domain. If the console asks for a CNAME there, your DNS provider must support CNAME flattening (also called ALIAS or ANAME). The console notes that a flattened record answers with IP addresses, so the **Traffic Routing** row may stay undetected even after the domain becomes active. That is expected.

To serve both `example.com` and `www.example.com`, add both domains to the same service. You can then redirect one to the other.

---

## TLS Certificates

You don't upload or renew certificates yourself.

- **Komuta address:** HTTPS works from the first deploy.
- **Custom domain:** the certificate is issued after the certificate validation record is found and renewed automatically as long as that record stays in place. Progress is shown under **SSL Status**. If it shows **Expiring soon** or **Expired**, check the certificate validation record and click **Refresh Validation**.

---

## Redirecting One Domain to Another

The **Redirect** card on a domain's detail panel sends that domain's visitors to another domain of the same service with a permanent (`301`) redirect. The path and query string are kept, and the redirect keeps answering while the service sleeps.

1. In **Redirect**, choose the target domain. **Don't redirect — serve the service** turns the redirect off.
2. Confirm the window that asks **Redirect {source} to {target}?**.

"Redirect saved. It goes live with the route update, usually within a minute or two." The domain then shows **Redirects to {host}**.

- Only other **Active** custom domains of the same service can be targets. If there is none, the card says "To redirect this domain, add another domain to the same service and verify it."
- Redirects don't chain. A domain that other domains already redirect to can't redirect itself.
- Browsers remember permanent redirects, so visitors who already saw one may keep following it for a while after you turn it off.

---

## Removing a Domain

1. Select the domain on the **Domains** page and click **Remove**.
2. The window explains that the domain will be removed permanently. Click **Remove Domain**.

You see "Domain deleted." Komuta removes the domain from the service's routes in the background, and the domain stops leading to your service. Domains that redirected to it serve the service again.

- Deleting a service removes its custom domains as well.
- After removing a domain, delete its DNS records at your DNS provider if you no longer use them.
- You can add the same domain again later, to the same or another service.

---

## Access Protection and Custom Domains

If the service uses [access protection](service-access-protection.md), it covers the custom domain too once the domain is active.

- Visitors sign in separately for each address of the service. See [Sign-in and sharing](access-protection-sign-in-sharing.md).
- If the domain's DNS is in your own Cloudflare account and the record is proxied, protection can't cover it and the console says "A custom domain is proxied through your own Cloudflare account. Set its DNS record to 'DNS only' (grey cloud) so Komuta can protect it."
- A protected service can have at most 50 addresses, including its Komuta address. Above that the console says "This service has more domains than one protection policy can cover. Remove custom domains you no longer need."

---

## Limits

| Limit | Value |
|---|---|
| Domain length | Up to 253 characters, each part up to 63 |
| Wildcard domains | Not supported |
| Services per domain | 1, across all organizations |
| Domains per service | No fixed limit (50 addresses with access protection) |
| Adding domains | A daily limit per organization (UTC day) and an hourly limit per network. Removing a domain doesn't give back the count. |
| Time to activate | 14 days, then the domain is removed automatically |

See also [Limits](limits.md).

---

## Common Problems

**"Enter a valid hostname (e.g. www.example.com)"**
The form accepts letters, digits, `-` and `.` only, and the name must end in a domain extension. Remove `*`, `http://`, a trailing `.` or a path, and enter internationalised domains in their `xn--` form.

**The service isn't in the Target App list, or you see "No apps available to attach."**
Only services with an HTTP endpoint and the **Public URL** turned on are listed. Turn on **Public URL** under **Access & ports → Network**. Jobs and cron jobs can never be selected. If your organization has no such service, switch to the organization that owns it or create one first.

**"'{ServiceName}' does not serve public HTTP traffic, so a custom domain cannot be attached to it. Turn on its public URL first; jobs and cron jobs cannot have custom domains."**
Same cause as above, for example when the public URL was turned off in the meantime.

**Adding the domain fails with an error**
- The domain is already connected to a Komuta service, in your organization or another one. Remove it there first.
- Your organization has reached the daily limit for adding domains. Try again the next day (UTC).

**A record stays at Not detected or Different value**
Compare **Observed** with **Expected**. The most common causes are a missing record, a typo, the zone name added twice (`api.example.com.example.com`) or a record still waiting for DNS changes to spread. Fix the record, wait a few minutes and click **Refresh Validation**.

**"No DNS records were returned for this domain yet. Use Refresh Validation to check again; contact support if this persists."**
Click **Refresh Validation**. If the list stays empty, open a [support ticket](support-tickets.md).

**The Traffic Routing row of a root domain stays undetected, but the domain is Active**
This is expected with CNAME flattening. No action needed.

**An active domain switched to Action Required with DNS moved**
The traffic record no longer points to Komuta. Restore the CNAME record shown in the console.

**Provider status Deleted**
Remove the domain on the **Domains** page and add it again, then check the DNS records.

**Provider status Under review or Blocked**
Open a [support ticket](support-tickets.md).

**The change is refused because of the account or the free plan**
Domain changes follow the same rules as other changes to a service. You may see "Your account is suspended for unpaid statements, so changes to this service are blocked. Pay the outstanding balance to make changes again." or, for a free service, "This month's free plan awake hours are used up, so changes to this service are blocked until the 1st of next month. Turn on keep awake for this service or move it to a paid plan to make changes now." See [Service Sleep](service-sleep.md).

---

## Related Documents

- [Access & Ports](services-ports.md) — Ports, the public URL and the service's addresses.
- [Service Access Protection](service-access-protection.md) — Protecting the service's addresses with sign-in and IP rules.
- [Deployment Strategies](deployment-strategies.md) — Blue-Green and the preview address.
- [API Gateway](api-gateway.md) — A gateway gets its custom domain the same way: add it on the **Domains** page and attach it to the gateway's service.
- [Stack Manifest Reference](stack-manifest-reference.md) — Declaring domains in a Stack.
- [Support Tickets](support-tickets.md) — Getting help when a domain doesn't become active.
