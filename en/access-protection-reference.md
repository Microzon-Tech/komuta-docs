# Access Protection: Reference

This page collects the exact values of access protection in one place: limits, states, responses visitors get, headers, error messages, frequently asked questions and a glossary. What each feature does and how to use it is explained in the guide pages; this page only answers "exactly what".

Access protection guides:

- [Access Protection](service-access-protection.md) — overview, turning on and off, status.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — Komuta sign-in, the **People** tab, what visitors see.
- [Rules](access-protection-rules.md) — IP allow-list, path rules, access preview.
- [Machines and Private Mesh](access-protection-machines.md) — webhook paths, service tokens, private mesh.
- [Access Log](access-protection-activity.md) — the **Activity** tab.
- [End Date and Visitor Identity](access-protection-settings.md) — the **Settings** tab.

---

## Limits

| Item | Limit |
|---|---|
| IP allow-list (separately for the site, each path rule, each webhook path) | At most 100 entries, 4600 characters in total; public addresses only; no `/0`; host bits zero |
| Path rules | At most 50 per service (webhook paths included); one rule per path |
| Path length | 2–256 characters; starts with `/`; lowercase letters, digits and `- . _ ~ ! $ & ' ( ) * + , = : @ /` |
| Webhook (open) paths | At most 10; methods `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Shares | At most 200 per service |
| Share page limit | At most 50 pages (including the "only chosen people" paths it is chosen for) |
| Share end date | Must be in the future; no upper limit |
| "Only chosen people" rule | At most 200 people per rule |
| Service tokens | At most 20 per service; name 1–64 characters; end at most 365 days away; at most 50 pages |
| Services that may come in over the private mesh | At most 50; same organization |
| Protection end date | In the future, at most 365 days away |
| Protected addresses (hosts) | At most 50 per service |
| Visitor session | At most 12 hours; never past the share end or an "open to everyone" end |
| Sign-in link | About 10 minutes (sign-in must be completed within this time) |
| Sign-in attempts | 30 per user per minute |
| Email verification code | 8 digits, valid 10 minutes, invalid after 5 wrong attempts |
| Email code requests | 20 per user per hour; 5 per user and address per hour; 3 sends per hour per Komuta account, service and address (30/60/120 s waits) |
| Request path (on a service with path rules or page limits) | At most 1024 bytes; longer gets `400` |
| Access log | Kept 30 days; 15 s batches; 500 rows per service per hour (sign-ins excluded); 50 records per page |
| Access ending after a share is removed | About 30 seconds (all sessions of the service are renewed) |
| Identity JWT | Valid 5 minutes; `nbf` = `iat` − 30 s |

---

## Protection states

| API value | Console (EN) | Console (TR) | Check active |
|---|---|---|---|
| `Disabled` | Off | Kapalı | No |
| `Preparing` | Preparing | Hazırlanıyor | No |
| `Enforcing` | Applying | Uygulanıyor | Yes |
| `Protected` | Protected | Korunuyor | Yes |
| `Disabling` | Turning off | Kapatılıyor | Until the last step |

Steps during `Enforcing` (`RouteFilter` → `PodTokenLock` → `CachePurge`): adding the gateway check to every route, the pod lock, purging the Cloudflare cache. The console doesn't show the step names; it says "{done} of {total} steps completed" (5 steps: preparation, the three enforcing steps, protected). `Disabling` runs in reverse: first the pod lock, then the gateway check is removed.

Expiry action (`ExpiryAction`): `KeepLocked` = **Then keep it locked** (default), `OpenToEveryone` = **Then open it to everyone**.

Share kinds (`ShareKind`): `OwnOrganization` = **Your organization**, `Member` = **A member**, `LinkedOrganization` = **A linked organization**, `Email` = **An email address**.

Combination (`Combine`): `All` = **Require both**, `Any` = **Either is enough**.

The meanings of warning codes (`lastError`) are in [Access Protection → Warnings and what to do](service-access-protection.md#warnings-and-what-to-do). Codes not listed one by one there are platform-side and need nothing from you; for example `capability_*` (the cluster doesn't support this feature yet), `authz_not_confirmed`, `route_filter_not_live`, `pod_token_lock_not_live`, `probe_*` (Komuta's own protection test), `cloudflare_*`, `cluster_*`, `reconcile_*` and other `drift_*` codes.

---

## Responses visitors get

| Situation | HTTP | Response |
|---|---|---|
| Rules met | — | The request is passed to the application. |
| Sign-in required, no session, `GET`/`HEAD` | `302` | Redirect to the Komuta sign-in page (`https://console.komuta.io/access/<service id>?host=…&return=…&state=…`). |
| Sign-in required, no session, other methods | `401` | `sign-in required` |
| Session, but the page is outside the share / not chosen in a people rule, `GET`/`HEAD` | `403` | "This page is not shared with you" HTML page (when out of scope, lists up to 20 pages they can open) |
| The same, other methods | `403` | `this path is not shared with you` |
| IP not on the list (or not verified as coming through Cloudflare), `GET`/`HEAD` | `403` | "Access to this service is restricted" HTML page; the address is shown only for a verified address that isn't on the list |
| The same, other methods | `403` | `access restricted to allowed networks` |
| **Block completely** rule (or while protection is being set up and the address isn't recognised yet) | `403` | `access denied` |
| Sign-in return link invalid, expired or already used | `403` | `sign-in link is invalid or expired` (open the protected page again) |
| Invalid service token | `401` | `invalid service token`, `WWW-Authenticate: KomutaServiceToken realm="komuta"` |
| Valid token, page out of scope | `403` | `this service token cannot open this path` |
| Unreadable path or longer than 1024 bytes (on a service with path rules/page limits) | `400` | `bad request` |
| The same Komuta cookie sent twice | `400` | `duplicate access cookie` |
| Protection data temporarily unavailable | `503` | `access policy unavailable` |
| Sign-in temporarily unavailable | `503` | `sign-in unavailable` |

Every refusal carries `Cache-Control: no-store`. HTML pages appear in Turkish or English depending on the visitor's browser language and load no external resources.

---

## Headers, cookies and reserved paths

| Name | Direction | Description |
|---|---|---|
| `x-komuta-service-token` | Client → Komuta | The service token (`kst_<32 hex>_<43 characters>`). Komuta removes it after checking; it never reaches the application. |
| `x-komuta-user-email` | Komuta → application | The signed-in visitor's email (while visitor identity is on). |
| `x-komuta-user-id` | Komuta → application | The signed-in visitor's Komuta user id (while visitor identity is on). |
| `x-komuta-identity` | Komuta → application | The ES256-signed identity JWT (while visitor identity is on). |
| `x-komuta-access` | Komuta → application | A secret value specific to the service, used for the pod lock. Don't use or log it. |
| `Cache-Control: private, no-store` | Komuta → visitor | Written on every response of a protected service. |
| `__Host-komuta_access` | Cookie | The visitor session; only for that address of the service; at most 12 hours. Not passed to the application. |
| `__Host-komuta_state` | Cookie | A short-lived cookie used during sign-in (10 minutes). Not passed to the application. |
| `/.komuta-access/callback` | Path | Where sign-in returns to. Paths starting with `/.komuta-access` are reserved for Komuta; no rule or webhook path can be defined for them. |

Once visitor identity is active (the "Getting ready" note on Settings is gone), these headers can't be faked by a visitor: they are removed and rewritten on every allowed request that passes Komuta. The signed `x-komuta-identity` can always be verified. For JWT claims and verification rules see [End Date and Visitor Identity](access-protection-settings.md#proof-of-identity-jwt).

**Public keys (JWKS):** `https://api.komuta.io/api/devopszon/access-protection/identity-keys` — anonymous, may be cached for 5 minutes.

---

## Access log reason codes

Technical codes in the access log and their console text:

| Code | Outcome | Console |
|---|---|---|
| `login_complete` | Sign-in | Signed in |
| `session_valid` | Page view | Opened the page |
| `token_valid` | Page view | Opened the page with a service token |
| `open_path` | Delivery | Delivered to an open path |
| `path_not_granted` | Refusal | This page is not shared with them |
| `path_blocked` | Refusal | The page is blocked |
| `ip_not_allowed` | Refusal | Came from an address that is not allowed |
| `untrusted_edge`, `untrusted_source`, `edge_required`, `client_ip_invalid` | Refusal | The request did not come through the Komuta edge |
| `code_invalid` | Refusal | The sign-in link was invalid or expired |
| `code_rejected` | Refusal | Komuta refused the sign-in |
| `token_invalid` | Refusal | Sent an unknown or expired service token |
| `token_not_allowed` | Refusal | The service token cannot open this page |
| `overflow` | Total | More visits this hour, grouped together |

---

## Error messages

Messages shown when an action is refused in the console or the API. Values in curly braces are filled in at the time.

### Protection settings

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:NothingProtected` | Add at least one IP range, turn on Komuta sign-in or add a path rule; otherwise the service would not be protected. |
| `DevOpsZon:AccessProtection:AllowListInvalid` | '{Entry}' is not a valid IP address or CIDR range. Use a form like 8.8.8.8 or 8.8.8.0/24, with the host bits of a range set to zero. |
| `DevOpsZon:AccessProtection:AllowListNotPublic` | '{Entry}' is, or overlaps, a private, reserved or documentation range. Only public internet addresses can be allowed. |
| `DevOpsZon:AccessProtection:AllowListEverything` | '{Entry}' would allow every address. Turn access protection off instead. |
| `DevOpsZon:AccessProtection:AllowListTooLong` | The IP allow list is too long (limit: {Max}). Merge ranges or remove entries. |
| `DevOpsZon:AccessProtection:InvalidCombine` | Choose how the IP allow-list and Komuta sign-in combine: both required, or either one. |
| `DevOpsZon:AccessProtection:PathRuleInvalid` | '{Prefix}' is not a valid path. Start it with /, use letters, digits and - . _ ~ ! $ & ' ( ) * + , = : @ only, and avoid empty, . or .. segments and /.komuta-access. |
| `DevOpsZon:AccessProtection:PathRuleDuplicate` | The path '{Prefix}' has more than one rule. Keep one rule per path. |
| `DevOpsZon:AccessProtection:PathRuleBlockExclusive` | The rule for '{Prefix}' blocks the path, so it cannot also list IP ranges or require sign-in. |
| `DevOpsZon:AccessProtection:PathRulePeopleExclusive` | The rule for '{Prefix}' is for chosen people, so it cannot also list IP ranges. |
| `DevOpsZon:AccessProtection:PathRuleRequiresNothing` | The rule for '{Prefix}' requires nothing. Add IP ranges, turn on Komuta sign-in or block the path. |
| `DevOpsZon:AccessProtection:PathRuleGrantInvalid` | The people chosen for '{Prefix}' are not valid. Choose each person once, keep at most 200, and end each time window after it starts. |
| `DevOpsZon:AccessProtection:PathRuleGrantUnknownShare` | Someone chosen for '{Prefix}' is no longer shared with. Share the service with them first, then choose them for the path. |
| `DevOpsZon:AccessProtection:PathRuleOpenInvalid` | An open path ({Prefix}) lets requests through without sign-in. Choose at least one method, don't add sign-in, blocking or people to it, and don't open the whole site or Komuta's own sign-in path. |
| `DevOpsZon:AccessProtection:PathRuleUnderOpenPath` | The rule for {Prefix} sits under the open path {Open}. Only blocking rules can go under an open path; move the rule or the open path. |
| `DevOpsZon:AccessProtection:PathRulesTooMany` | A service can have at most {Max} path rules. |
| `DevOpsZon:AccessProtection:TooManyOpenPaths` | A service can have at most {Max} open paths. |
| `DevOpsZon:AccessProtection:RulesNotAvailable` | Path rules and the either-one combination are not available yet. |
| `DevOpsZon:AccessProtection:ExpiryInPast` | The expiry must be in the future. |
| `DevOpsZon:AccessProtection:ExpiryTooFar` | The expiry can be at most {MaxDays} days from now. Choose an earlier date or make the protection permanent. |
| `DevOpsZon:AccessProtection:InvalidExpiryAction` | The expiry action is not recognised. Choose to keep the service locked or to open it to everyone. |
| `DevOpsZon:AccessProtection:ProtectionNotEnabled` | Access protection is not turned on for this service. Turn it on before changing its expiry. |
| `DevOpsZon:AccessProtection:StaleRevision` | Access protection was changed in the meantime (expected revision {Expected}, current {Actual}). Reload and try again. |
| `DevOpsZon:AccessProtection:InvalidTransition` | Access protection cannot move from {From} to {To}. |
| `DevOpsZon:AccessProtection:NoPublicUrl` | This service has no public URL, so there is nothing to protect. Turn on the public URL first. |
| `DevOpsZon:AccessProtection:UnsupportedCluster` | Access protection is only available for services running on Komuta's shared hosting clusters. |
| `DevOpsZon:AccessProtection:FeatureDisabled` | Access protection is not enabled on this platform yet. |
| `DevOpsZon:AccessProtection:NotAvailableForOrganization` | Access protection is turned off for this organization. Contact Komuta support to turn it on. |
| `DevOpsZon:AccessProtection:ServiceNotFound` | The service was not found or does not belong to this organization. |
| `DevOpsZon:AccessProtection:IdentityNeedsSignIn` | The visitor's identity can only be passed to the application on a service that asks for Komuta sign-in. Turn sign-in on first. |
| `DevOpsZon:AccessProtection:IdentityNotAvailable` | Passing the visitor's identity is not turned on on this platform yet. |

### Private mesh

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:MeshExposureEnabled` | This service is reachable from your other services over the private network (mesh). That path goes straight to the service and around the access check, so protection cannot be turned on while it is open. Turn off the private network for this service first; if you just turned it off, wait a minute until it has been removed. |
| `DevOpsZon:AccessProtection:MeshBlockedByProtection` | Access protection is on for {ServiceName}. Opening it to the private network (mesh), which connecting to it from another service also does, would let traffic reach it around the access check. Turn access protection off for {ServiceName} first, and wait until it has finished turning off. |
| `DevOpsZon:AccessProtection:MeshPeerInvalid` | Choose other services of this organization; a service cannot be listed for itself. |
| `DevOpsZon:AccessProtection:MeshPeersNotAvailable` | Choosing services that may reach a protected service over the private network is not available yet. |
| `DevOpsZon:AccessProtection:TooManyMeshPeers` | At most {Max} services can reach a protected service directly over the private network. |

`MeshExposureEnabled` and `MeshBlockedByProtection` appear only while using the private mesh together with protection is turned off on the platform.

### Shares and service tokens

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtectionShare:PolicyNotConfigured` | Turn on access protection for this service before sharing it. |
| `DevOpsZon:AccessProtectionShare:MemberNotInOrganization` | The selected person is not an active member of this organization. |
| `DevOpsZon:AccessProtectionShare:OrganizationNotLinked` | You can only share with organizations you are a member of. |
| `DevOpsZon:AccessProtectionShare:LinkedOrganizationIsOwn` | This is your own organization. Share with your organization instead. |
| `DevOpsZon:AccessProtectionShare:OrganizationRequired` | Open an organization to manage access sharing. |
| `DevOpsZon:AccessProtection:ExternalSharingDisabled` | Sharing outside the organization is turned off. |
| `DevOpsZon:AccessProtection:ShareInvalid` | The share details are not valid for the '{Kind}' share type. |
| `DevOpsZon:AccessProtection:ShareNotFound` | The share was not found. |
| `DevOpsZon:AccessProtection:ShareScopeInvalid` | The page '{Prefix}' in this share's scope is not valid, or the scope lists more than 50 pages. |
| `DevOpsZon:AccessProtection:ShareScopeTooWide` | Choosing this person for '{Prefix}' would give their share more than {Max} pages. Remove some pages from their share, or choose them on fewer paths. |
| `DevOpsZon:AccessProtection:TooManyShares` | A service can have at most {Max} shares. Remove a share before adding another. |
| `DevOpsZon:AccessProtection:ServiceTokenNeedsSignIn` | Service tokens work only while protection requires Komuta sign-in. Turn sign-in on first. |
| `DevOpsZon:AccessProtection:ServiceTokenNameInvalid` | A token name must be 1 to {Max} characters. |
| `DevOpsZon:AccessProtection:ServiceTokenNameTaken` | This service already has a token named '{Name}'. |
| `DevOpsZon:AccessProtection:ServiceTokenNotFound` | This service token was not found; it may already be deleted. |
| `DevOpsZon:AccessProtection:ServiceTokenScopeInvalid` | The page '{Prefix}' this token may open is not valid, or more than 50 pages are chosen. Start pages with /; choose none to let the token open the whole site. |
| `DevOpsZon:AccessProtection:TooManyServiceTokens` | A service can have at most {Max} service tokens. Delete one you no longer use. |

### Visitor sign-in

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:AccessDenied` | You do not have access to this page, or the link is no longer valid. |
| `DevOpsZon:AccessProtection:AccessRateLimited` | Too many sign-in attempts. Wait a minute and try again. |
| `DevOpsZon:AccessProtection:AccessRequestInvalid` | The sign-in link is incomplete or malformed. Open the page you wanted again to start over. |
| `DevOpsZon:AccessProtection:SignInNotRequired` | This service does not ask visitors to sign in with Komuta right now. If you cannot open it, it may only be reachable from certain networks; contact the service owner. |
| `DevOpsZon:AccessProtection:EmailChallengeInvalid` | The verification code is wrong or has expired. Request a new code and try again. |
| `DevOpsZon:AccessProtection:EmailChallengeRateLimited` | Too many verification codes were requested. Wait an hour and try again. |
| `DevOpsZon:AccessProtection:CodeInvalid` | The sign-in code is invalid or expired. |
| `DevOpsZon:AccessProtection:CodeAlreadyUsed` | The sign-in code has already been used. |
| `DevOpsZon:AccessProtection:CodeRevoked` | The sign-in code is no longer valid because the service's access settings changed. |
| `DevOpsZon:AccessProtection:CodeTargetNotFound` | The sign-in code does not belong to a protected service. |
| `DevOpsZon:AccessProtection:CodeStoreUnavailable` | Sign-in is temporarily unavailable. Try again shortly. |

---

## Frequently asked questions

**Do visitors need to create a Komuta account?**
If you use Komuta sign-in, yes. Creating an account is free and takes seconds with Google or GitHub. If you don't want visitors to create accounts, use the IP allow-list. Signing in with your own identity provider (SSO) is not supported at the moment.

**Why does my API request get 401 instead of 302?**
`GET` and `HEAD` requests without a session to a path that requires sign-in are redirected to the sign-in page (`302`); other methods get `401`, so that a program doesn't follow the redirect and mistake an HTML page for the answer. For programs, use a [service token](access-protection-machines.md#service-tokens) or the IP list.

**I call my service's API from a browser on another site and get a CORS error.**
The `OPTIONS` preflight request a browser sends carries no cookies, so it gets `401` on a path that requires sign-in. `OPTIONS` can't be chosen on webhook paths either. Keep endpoints that a browser must call from another site outside protection (turn site-wide protection off and protect only the other paths with path rules), or make the request from your server instead.

**I turned protection on, but the site still opens without sign-in.**
Check the status badge: during **Preparing** the check isn't active yet. If it shows **Applying** or **Protected**, your browser may have opened the page from its cache; reload the page. If the problem persists, check the card for a warning.

**I locked myself out.**
The Komuta console isn't affected by protection. In the console, go to the **Rules** tab and add your new address to the IP list (you can see your address on the **Access to this service is restricted** page), or remove protection with **Settings → Open to everyone now**.

**What happens to a sleeping service?**
Protection applies while it sleeps too. Only a visitor who passes the checks can wake the service; someone who hasn't signed in or comes from an address that isn't allowed can't.

**I removed a share; is the person out immediately?**
Yes, within about 30 seconds, including their open sessions. The other visitors of the service sign in once more too.

**Can I sign out one person individually?**
There is no per-person sign-out. Removing a share or bringing its end date forward cuts that person's access, but also ends **every** open session of this service: everyone signs in again on their next page load (automatic for people already signed in to Komuta; people who came in through an email share request a new code). To remove a single person who came in through an organization share, remove them from the organization.

**Can I export the access log?**
Not at the moment. The log is visible in the console for 30 days.

**Can I set rules by country, HTTP method or request rate?**
No. Rules are by address, sign-in and path. Method selection exists only on webhook paths.

**How does my application learn who the visitor is?**
Turn on **Tell my application who signed in** on the **Settings** tab and verify the `x-komuta-identity` JWT. See [End Date and Visitor Identity](access-protection-settings.md#tell-my-application-who-signed-in).

**What if a webhook sender such as GitHub changes its IP ranges?**
The sender address list is optional; the real protection is your application's signature check. If you use the list, keep the ranges the sender publishes up to date, or leave the list empty.

**What happens if I redeploy my service while protection is on?**
Protection isn't affected; the new version goes live with the same protection.

---

## Glossary

| Term | Meaning |
|---|---|
| **Access protection** | The feature that controls, at the Komuta gateway, who can reach a service's public address. |
| **Gateway** | The Komuta layer that internet traffic to your service passes through; the check happens here. |
| **Pod lock** | A protected service's pods accepting only requests from the gateway that passed the check. It prevents bypassing the check. |
| **Komuta sign-in** | A visitor signing in with a Komuta account. |
| **Share** | Permission for a person, organization or email address to sign in to the service. |
| **External sharing** | Sharing outside the organization (a linked organization or an email address). Allowed by an organization setting. |
| **Suspended** | A share that temporarily doesn't work because external sharing was turned off or the organization link was broken. |
| **Page limit (scope)** | A share or token opening only certain paths. |
| **IP allow-list** | The public IP addresses that can reach the service without signing in, or (with "require both") that visitors must come from. |
| **CIDR** | A way of writing an address range, for example `203.0.113.0/24` (256 addresses). |
| **Require both / Either is enough** | How the IP list and sign-in are evaluated together. |
| **Path rule** | Extra protection or a block for a specific path and everything below it. |
| **Only chosen people** | A path rule that opens a path only to chosen shares, optionally within a time window. |
| **Webhook path (open path)** | A path that lets requests with the chosen methods through without sign-in. The application verifies the sender's signature. |
| **Service token** | A secret key that programs send as a header instead of signing in. |
| **Private mesh** | A connection between your clusters that doesn't go out to the public internet; it doesn't pass the gateway. |
| **Access preview** | A tool that shows who gets in where with the saved rules, without changing anything. |
| **Access log** | A 30-day record of sign-ins, page views and refusals. |
| **Protection end date** | When protection ends and what happens then. |
| **Visitor identity** | Passing the signed-in visitor's identity to the application in headers. |
| **JWT / JWKS** | A signed identity token / the published list of public keys used to verify the signature. |
| **Cloudflare** | The network in front of Komuta addresses and custom domains; it reports the visitor's address reliably. |
| **DNS only (grey cloud)** | A Cloudflare record working only as DNS, without proxying. The record of a custom domain added to Komuta must be like this in your own Cloudflare account. |
