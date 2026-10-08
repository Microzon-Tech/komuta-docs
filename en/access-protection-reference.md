# Access Protection: Reference

This page collects the exact values of access protection in one place: limits, states, responses visitors get, headers, error messages, frequently asked questions and a glossary. What each feature does and how to use it is explained in the guide pages; this page only answers "exactly what".

Access protection guides:

- [Access Protection](service-access-protection.md) — overview, turning on and off, status.
- [Setup Guide](access-protection-tutorial.md) — step-by-step setup from quick start to advanced scenarios.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — Komuta sign-in, the **People** tab, what visitors see.
- [Rules](access-protection-rules.md) — IP allow-list, path rules, access preview.
- [Machines and Private Mesh](access-protection-machines.md) — webhook paths, service tokens, private mesh.
- [Access Log](access-protection-activity.md) — the **Activity** tab.
- [End Date and Visitor Identity](access-protection-settings.md) — the **Settings** tab.
- [Access Protection in a Stack Manifest](stack-manifest-access.md) — the `access` block of a Stack service.

---

## Limits

| Item | Limit |
|---|---|
| IP allow-list (separately for the site, each path rule, each webhook path) | At most 100 entries, 4600 characters in total; public addresses only; no `/0`; host bits zero |
| Path rules | At most 50 per service (webhook paths included); one rule per path |
| Path length | 2–256 characters; starts with `/`; lowercase letters, digits and `- . _ ~ ! $ & ' ( ) * + , = : @ /` |
| Webhook (open) paths | At most 10; methods `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Signed webhook paths | Methods `POST`, `PUT`, `PATCH` only; body at most 65,535 bytes (about 64 KiB); at most 2 signing secrets per path; secret 8–512 bytes, no whitespace or control characters; Stripe timestamps within 300 seconds; HMAC value prefix at most 16 characters |
| Method rules | At most 50 per service (separate from path rules); methods `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`; `/` allowed; one rule per path |
| Countries | At most 250 two-letter ISO codes; `XX` (unknown) and `T1` (Tor) can't be listed |
| Rate limit | 10–100,000 requests per 1–3600 seconds (console: per 1, 10, 60, 600 or 3600 seconds); per address, IPv6 per `/64`; counted per gateway replica, so approximate |
| Shares | At most 200 per service |
| Share page limit | At most 50 pages (including the "only chosen people" paths it is chosen for) |
| Share end date | Must be in the future; no upper limit |
| Domain shares | Domain only (no name before `@`); public mail and shared domains refused; subdomains only for a verified domain; codes for at most 3 different addresses per Komuta account and share in 24 hours; no new codes after 200 wrong codes in 24 hours |
| Verified domains | At most 20 per organization; TXT `_komuta-verify.<domain>`; checked every 6 hours and on **Check now** (once per 10 s); lapses once the record has been missing for 2 days (at least 3 checks in a row), or after a week without a DNS answer |
| "Only chosen people" rule | At most 200 people per rule |
| Service tokens | At most 20 per service; name 1–64 characters; end at most 365 days away; at most 50 pages |
| Share links | At most 50 per service; name 1–64 characters; end date required (console: 1, 7, 30 or 90 days; API: at most 365 days away); at most 50 pages |
| Services that may come in over the private mesh | At most 50; same organization |
| Protection end date | In the future, at most 365 days away |
| Protected addresses (hosts) | At most 50 per service |
| Visitor session | 15 minutes, 1 hour, 4 hours, 12 hours (default), 1 day or 7 days (**Stay signed in for**); never past the share end, the link end or an "open to everyone" end; 12 hours while visitor sessions aren't managed on your platform |
| **Who is signed in** list | At most 200 sessions |
| One-by-one sign-out | Starts 12 hours 10 minutes after session tracking began on the service; falls back to signing everyone out if more than 500 sessions are ended one by one within a session's lifetime |
| Sign-in link | About 9 minutes (sign-in must be completed within this time); no new email code with less than 2 minutes left |
| Sign-in attempts | 30 per user per minute |
| Email verification code | 8 digits, valid 10 minutes, invalid after 5 wrong attempts |
| Email code requests | 20 per user per hour; 5 per user and address per hour; 3 sends per hour per Komuta account, service and address (30/60/120 s waits) |
| Email codes without a Komuta account | Counted per network (IPv4 address or IPv6 `/64`) and service: 60 requests per hour, 5 per address per hour, 3 sends per address per hour, 50 wrong codes per address in 24 hours; an address receives at most 10 such codes per hour per service and 30 across the organization; a domain share sends codes to at most 20 different addresses per network in 24 hours; their wrong codes count toward a separate 200-in-24-hours limit that pauses only codes without an account |
| Request path (on a service with path rules or page limits) | At most 1024 bytes; longer gets `400` |
| Access log | Kept 30 (default), 90 or 365 days, per organization; 15 s batches; 500 rows per service per hour (sign-ins excluded); 50 records per page |
| Access log export | CSV or JSON; inside the retention period; at most 50,000 rows; one export per organization at a time |
| Access ending after a share is removed, a person is signed out or a link is deleted | About 30 seconds (before one-by-one sign-out is active, removing a share ends every session of the service) |
| Identity JWT | Valid 5 minutes; `nbf` = `iat` − 30 s |

---

## Protection states

| API value | Console (EN) | Console (TR) | Check active |
|---|---|---|---|
| `Disabled` | Off | Kapalı | No |
| `Preparing` | Preparing | Hazırlanıyor | No |
| `Enforcing` | Applying | Uygulanıyor | Yes (a short delay is possible in the first seconds) |
| `Protected` | Protected | Korunuyor | Yes |
| `Disabling` | Turning off | Kapatılıyor | Until the last step |

Steps during `Enforcing` (`RouteFilter` → `PodTokenLock` → `CachePurge`): adding the gateway check to every route, the pod lock, purging the Cloudflare cache. The console doesn't show the step names; it says "{done} of {total} steps completed" (5 steps: preparation, the three enforcing steps, protected). `Disabling` runs in reverse: first the pod lock, then the gateway check is removed.

Expiry action (`ExpiryAction`): `KeepLocked` = **Then keep it locked** (default), `OpenToEveryone` = **Then open it to everyone**.

Share kinds (`ShareKind`): `OwnOrganization` = **Your organization**, `Member` = **A member**, `LinkedOrganization` = **A linked organization**, `Email` = **An email address**, `EmailDomain` = **Everyone at a domain**.

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
| Method not allowed by a method rule | `405` | `method not allowed`, `Allow: <the rule's methods>` |
| Country not on the list (or unknown), `GET`/`HEAD` | `403` | "Access to this service is restricted" HTML page, with the address |
| The same, other methods | `403` | `access restricted to allowed networks` |
| Over the rate limit | `429` | `too many requests`, `Retry-After: <seconds>` |
| Valid share link opened | `302` | Redirect to the same address without `komuta_link` (a session is opened; an existing Komuta session is kept) |
| Wrong or expired share link | `403` | `This share link is not valid or has expired. Ask the person who sent it for a new one.` |
| Share link opened with a method other than `GET`/`HEAD` | `403` | `Open a share link in a browser.` |
| Share-link visitor, page outside the link | `403` | `Your share link does not open this page.` |
| Share-link visitor after the link was deleted or ended | `403` | `The share link you opened this site with has ended. Ask the person who sent it for a new one.` |
| Browser CORS check while **Allow CORS checks without sign-in** is on | — | Passed to the application (the real request still needs sign-in) |
| Signed webhook path: missing or wrong signature, or a request the path doesn't take | `401` | `invalid webhook signature` |
| Signed webhook path without a signing secret | `401` | `webhook signature cannot be checked` |
| Signed webhook path, body larger than 65,535 bytes | `413` | `webhook body too large to verify` |
| Protection data temporarily unavailable | `503` | `access policy unavailable` |
| Sign-in temporarily unavailable | `503` | `sign-in unavailable` |

Every refusal carries `Cache-Control: no-store`. HTML pages appear in Turkish or English depending on the visitor's browser language and load no external resources.

---

## Headers, cookies and reserved paths

| Name | Direction | Description |
|---|---|---|
| `x-komuta-service-token` | Client → Komuta | The service token (`kst_<32 hex>_<43 characters>`). Komuta removes it after checking; it never reaches the application. |
| `x-komuta-user-email` | Komuta → application | The signed-in visitor's email (while visitor identity is on). |
| `x-komuta-user-id` | Komuta → application | The signed-in visitor's Komuta user id, or `eml:<32 hex>` for someone who signed in with an email code without an account (while visitor identity is on). |
| `x-komuta-identity` | Komuta → application | The ES256-signed identity JWT (while visitor identity is on). |
| `x-komuta-access` | Komuta → application | A secret value specific to the service, used for the pod lock. Don't use or log it. |
| `Cache-Control: private, no-store` | Komuta → visitor | Written on every response of a protected service. |
| `__Host-komuta_access` | Cookie | The visitor session (also for share-link visitors); only for that address of the service; lasts the service's session length. Not passed to the application. |
| `__Host-komuta_state` | Cookie | A short-lived cookie used during sign-in (10 minutes). Not passed to the application. |
| `komuta_link` | Query parameter | Carries a share link (`kl_…`). Komuta removes it with a redirect; it never reaches the application or the access log. |
| `Retry-After` | Komuta → visitor | Seconds to wait, on a `429` from the rate limit. |
| `Allow` | Komuta → visitor | The methods a path accepts, on a `405` from a method rule. |
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
| `link_opened` | Sign-in | Came in with a share link |
| `link_valid` | Page view | Opened the page with a share link |
| `link_invalid` | Refusal | Opened an unknown or expired share link |
| `link_not_allowed` | Refusal | The share link does not open this page |
| `link_ended` | Refusal | Came back after the share link ended |
| `method_not_allowed` | Refusal | Used a method this path does not allow |
| `country_not_allowed` | Refusal | Came from a country that is not allowed |
| `rate_limited` | Refusal | Sent too many requests |
| `signature_invalid` | Refusal | Sent a webhook without a valid signature |
| `signature_key_missing` | Refusal | Sent a webhook to a path that has no signing secret yet |
| `webhook_body_too_large` | Refusal | Sent a webhook body larger than 64 KiB |
| `overflow` | Total | More visits this hour, grouped together |

`preflight` (a browser CORS check let through by **Allow CORS checks without sign-in**) is a reason the gateway uses, but it isn't recorded in the access log.

---

## Error messages

Messages shown when an action is refused in the console or the API. Values in curly braces are filled in at the time.

### Protection settings

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:NothingProtected` | Add at least one IP range, turn on Komuta sign-in or add a path rule; otherwise the service would not be protected. |
| `DevOpsZon:AccessProtection:TurnOffWithOpen` | To turn access protection off, use the turn-off action; a change that removes every IP range, sign-in and path rule is not saved. |
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

### Webhook signatures

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:PathRuleSignatureInvalid` | The signature check on {Prefix} is not valid. A signed path is open, accepts only POST, PUT and PATCH, and uses GitHub, Stripe or an HMAC header the gateway forwards. |
| `DevOpsZon:AccessProtection:WebhookSignaturesNotAvailable` | Webhook signature checks are not available on this platform yet. |
| `DevOpsZon:AccessProtection:WebhookKeyPathNotSigned` | {Prefix} is not an open path with a signature check, so it cannot hold a webhook secret. |
| `DevOpsZon:AccessProtection:TooManyWebhookKeys` | A signed path can hold at most {Max} webhook secrets; remove the old one after the sender uses the new one. |
| `DevOpsZon:AccessProtection:WebhookKeyNotFound` | The webhook secret was not found. |
| `DevOpsZon:AccessProtection:WebhookSecretInvalid` | A webhook secret is {Min} to {Max} bytes without spaces or control characters. |

### Countries, rate limit and methods

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:CountryInvalid` | {Country} is not a two-letter ISO country code (unknown and Tor codes cannot be listed). |
| `DevOpsZon:AccessProtection:CountriesTooMany` | A service can list at most {Max} countries. |
| `DevOpsZon:AccessProtection:RateLimitInvalid` | A rate limit allows {Min} to {Max} requests per 1 to {MaxSeconds} seconds; 0 requests turns it off. |
| `DevOpsZon:AccessProtection:MethodRuleInvalid` | The method rule for {Prefix} is not valid. Use a path prefix and one or more of GET, HEAD, POST, PUT, PATCH, DELETE and OPTIONS, each path at most once. |
| `DevOpsZon:AccessProtection:MethodRulesTooMany` | A service can have at most {Max} method rules. |
| `DevOpsZon:AccessProtection:MethodRulesNotAvailable` | CORS preflight and method rules are not available on this platform yet. |

`MethodRulesNotAvailable` is also the answer when adding countries or a rate limit on a platform where these aren't enabled.

### Access preview

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:ExplainPathInvalid` | The path {Path} cannot be explained. Give a plain path that starts with /, without encoded characters, backslashes, ';', empty or '.' segments, outside /.komuta-access. Such paths are read more strictly and never open more than the plain path does. |
| `DevOpsZon:AccessProtection:ExplainMethodInvalid` | The HTTP method is not valid. Use a method name such as GET or POST. |
| `DevOpsZon:AccessProtection:ExplainAddressInvalid` | {Address} is not a valid IPv4 or IPv6 address. |
| `DevOpsZon:AccessProtection:ExplainVisitorInvalid` | The {Kind} visitor is incomplete or mixed: a member needs a user, a share needs a share id, an e-mail visitor needs a valid address, a token needs a token id, a share link needs a link id, and an anonymous visitor carries none of them. |

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
| `DevOpsZon:AccessProtection:DomainInvalid` | '{Domain}' is not a domain name such as example.com. |
| `DevOpsZon:AccessProtection:DomainOpenToAnyone` | Anyone can get an e-mail address at '{Domain}', so a share for it would let anyone in. Share with single e-mail addresses instead. |
| `DevOpsZon:AccessProtection:DomainSubdomainsNeedVerification` | Subdomains of '{Domain}' can only be included once the organization has verified the domain. |
| `DevOpsZon:AccessProtection:DomainSharesNotAvailable` | Sharing with everyone at a domain is not available yet. |
| `DevOpsZon:AccessProtection:VerifiedDomainExists` | '{Domain}' is already on the organization's list. |
| `DevOpsZon:AccessProtection:VerifiedDomainNotFound` | This domain is not on the organization's list. |
| `DevOpsZon:AccessProtection:TooManyVerifiedDomains` | An organization can verify at most {Max} domains. |
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

### Share links and sessions

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:ShareLinksNotAvailable` | Share links are not turned on on this platform yet. |
| `DevOpsZon:AccessProtection:ShareLinkNeedsSignIn` | Turn on Komuta sign-in first; a share link lets someone in without signing in. |
| `DevOpsZon:AccessProtection:ShareLinkNameInvalid` | Give the link a name of at most {Max} characters. |
| `DevOpsZon:AccessProtection:ShareLinkNameTaken` | A link named {Name} already exists. |
| `DevOpsZon:AccessProtection:ShareLinkEndRequired` | A share link needs an end date. |
| `DevOpsZon:AccessProtection:ShareLinkScopeInvalid` | The link's path {Prefix} is not a valid path. |
| `DevOpsZon:AccessProtection:TooManyShareLinks` | A service can have at most {Max} share links. |
| `DevOpsZon:AccessProtection:ShareLinkNotFound` | That share link no longer exists. |
| `DevOpsZon:AccessProtection:SessionLifetimeInvalid` | Choose a session length of 15 minutes, 1 hour, 4 hours, 12 hours, 24 hours or 7 days. |
| `DevOpsZon:AccessProtection:SessionsNotAvailable` | Managing visitor sessions is not turned on on this platform yet. |
| `DevOpsZon:AccessProtection:SessionNotFound` | That session has already ended. |

### Access log

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:AccessLogRetentionInvalid` | Keep the access log for 30, 90 or 365 days. |
| `DevOpsZon:AccessProtection:AccessLogExportRangeInvalid` | Choose a range that starts before it ends and lies within the days the access log is kept. |
| `DevOpsZon:AccessProtection:AccessLogExportTooLarge` | This range holds more than {Max} entries. Choose a shorter range or one kind of entry. |
| `DevOpsZon:AccessProtection:AccessLogExportBusy` | Another export of your organization's access log is still running. Try again in a moment. |
| `DevOpsZon:AccessProtection:AccessLogExportUnaudited` | The export could not be recorded in the audit log, so it was not made. Try again in a moment. |

### Stacks

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:DeclaredProtectionRefused` | The declared access protection could not be set up for {Service}; the service was not provisioned. |
| `DevOpsZon:AccessProtection:DeclaredProtectionInvalid` | The declared access protection for {Service} needs sign-in or an allow-list, and sharing with the organization needs sign-in; the service was not provisioned. |

The validation codes of the manifest's `access` block are listed in [Access Protection in a Stack Manifest](stack-manifest-access.md#validation).

### Visitor sign-in

| Code | Message |
|---|---|
| `DevOpsZon:AccessProtection:AccessDenied` | You do not have access to this page, or the link is no longer valid. |
| `DevOpsZon:AccessProtection:AccessRateLimited` | Too many sign-in attempts. Wait a minute and try again. |
| `DevOpsZon:AccessProtection:AccessRequestInvalid` | The sign-in link is incomplete or malformed. Open the page you wanted again to start over. |
| `DevOpsZon:AccessProtection:SignInNotRequired` | This service does not ask visitors to sign in with Komuta right now. If you cannot open it, it may only be reachable from certain networks; contact the service owner. |
| `DevOpsZon:AccessProtection:EmailChallengeInvalid` | The verification code is wrong or has expired. Request a new code and try again. |
| `DevOpsZon:AccessProtection:EmailChallengeRateLimited` | Too many verification codes were requested. Wait an hour and try again. |
| `DevOpsZon:AccessProtection:EmailVisitorsNotAvailable` | Signing in with an e-mail code without a Komuta account is not available for this service. |
| `DevOpsZon:AccessProtection:CodeInvalid` | The sign-in code is invalid or expired. |
| `DevOpsZon:AccessProtection:CodeAlreadyUsed` | The sign-in code has already been used. |
| `DevOpsZon:AccessProtection:CodeRevoked` | The sign-in code is no longer valid because the service's access settings changed. |
| `DevOpsZon:AccessProtection:CodeTargetNotFound` | The sign-in code does not belong to a protected service. |
| `DevOpsZon:AccessProtection:CodeStoreUnavailable` | Sign-in is temporarily unavailable. Try again shortly. |

---

## Frequently asked questions

**Do visitors need to create a Komuta account?**
If you use Komuta sign-in, yes, unless you send them a [share link](access-protection-sign-in-sharing.md#share-links) or share the service with their email address or domain: then they can [get in with an email code](access-protection-sign-in-sharing.md#without-a-komuta-account). Creating an account is free and takes seconds with Google or GitHub. If you don't want visitors to create accounts, use a share link or the IP allow-list. Signing in with your own identity provider (SSO) is not supported at the moment.

**Can I let in someone without a Komuta account?**
Yes: share the service with their email address or their company's domain, and they get in with a one-time code sent to the mailbox ([Without a Komuta account](access-protection-sign-in-sharing.md#without-a-komuta-account)). Or with a share link (**People → Share links → Create link**). Whoever holds the link gets in without signing in, only on the link's pages, until the link ends (at most 90 days in the console) or you delete it. Remember that link previews in chat apps and mail count as openings, and that anyone the link is forwarded to gets in too.

**Why does my API request get 401 instead of 302?**
`GET` and `HEAD` requests without a session to a path that requires sign-in are redirected to the sign-in page (`302`); other methods get `401`, so that a program doesn't follow the redirect and mistake an HTML page for the answer. For programs, use a [service token](access-protection-machines.md#service-tokens) or the IP list.

**I call my service's API from a browser on another site and get a CORS error.**
The `OPTIONS` check a browser sends first carries no cookies, so it gets `401` on a path that requires sign-in. Turn on **Machines → Methods and CORS → Allow CORS checks without sign-in**: a browser check (one `Origin` and one `Access-Control-Request-Method` header) then passes, while the IP list, block rules and method rules still apply. Only the check passes: the real request still needs sign-in (a session the browser sends, a service token, or an IP list that lets it in), and your application still answers the CORS headers itself. `OPTIONS` still can't be chosen on webhook paths. If you don't see the setting, it isn't enabled on your platform yet; keep such endpoints outside protection, or make the request from your server instead.

**I turned protection on, but the site still opens without sign-in.**
Check the status badge: during **Preparing** the check isn't active yet. In the first seconds of **Applying** the routes may still be refreshing; wait a moment and reload. If it shows **Protected**, your browser may have served the page from its cache; reload the page. If the problem persists, check the card for a warning.

**I locked myself out.**
The Komuta console isn't affected by protection. In the console, go to the **Rules** tab and add your new address to the IP list (you can see your address on the **Access to this service is restricted** page), or remove protection with **Settings → Open to everyone now**.

**What happens to a sleeping service?**
Protection applies while it sleeps too. Only a visitor who passes the checks can wake the service; someone who hasn't signed in or comes from an address that isn't allowed can't.

**I removed a share; is the person out immediately?**
Yes, within about 30 seconds, including their open sessions. Once one-by-one sign-out is active on the service, only the sessions opened through that share end; before that (for 12 hours 10 minutes after session tracking began), the other visitors of the service sign in once more too.

**Can I sign out one person individually?**
Yes: **People → Who is signed in → Sign out** on the person's row. Their open sessions end within about 30 seconds; nobody else is affected. They can sign straight back in while a share still lets them in, so remove that share too if they should stay out. Until one-by-one sign-out is active on the service, signing one person out signs everyone out, and the console says so. Share-link visitors aren't listed; delete the link to end their access.

**Can I export the access log?**
Yes: **Activity → Export**, as CSV or JSON, at most 50,000 rows per export. The log is kept for 30 days by default; an organization can keep it for 90 or 365 days (**Account → Organizations → Keep access logs for**).

**Can I set rules by country, HTTP method or request rate?**
Yes: **Countries** and **Rate limit** on the **Rules** tab, and method rules under **Methods and CORS** on the **Machines** tab. If you don't see these sections, they aren't enabled on your platform yet.

**How does my application learn who the visitor is?**
Turn on **Tell my application who signed in** on the **Settings** tab and verify the `x-komuta-identity` JWT. See [End Date and Visitor Identity](access-protection-settings.md#tell-my-application-who-signed-in).

**Can Komuta check webhook signatures for me?**
Yes: choose a **Signature check at the edge** (GitHub, Stripe or another HMAC-SHA256 header) when you open the webhook path, then add the signing secret under it. Unsigned requests get `401` before they reach your application. Signed paths take only `POST`, `PUT` and `PATCH` with bodies up to 65,535 bytes. If you don't see the choice, it isn't enabled on your platform yet. See [Signature check at the edge](access-protection-machines.md#signature-check-at-the-edge).

**What if a webhook sender such as GitHub changes its IP ranges?**
The sender address list is optional; the real protection is the signature check, at the edge or in your application. If you use the list, keep the ranges the sender publishes up to date, or leave the list empty.

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
| **Share** | Permission for a person, organization, email address or everyone at a domain to sign in to the service. |
| **External sharing** | Sharing outside the organization (a linked organization, an email address, or a domain the organization hasn't verified). Allowed by an organization setting. |
| **Verified domain** | A domain an organization proved it owns with a DNS TXT record; shares under it count as the organization's own. |
| **Suspended** | A share that temporarily doesn't work because external sharing was turned off, the organization link was broken, or its domain is no longer verified. |
| **Page limit (scope)** | A share, token or share link opening only certain paths. |
| **Share link** | A link with an end date that lets whoever holds it in without a Komuta account. |
| **Session length** | How long a sign-in lasts (**Stay signed in for**); 12 hours by default. |
| **One-by-one sign-out** | Ending one person's or one share's sessions without signing everyone out; active 12 hours 10 minutes after session tracking began. |
| **IP allow-list** | The public IP addresses that can reach the service without signing in, or (with "require both") that visitors must come from. |
| **CIDR** | A way of writing an address range, for example `203.0.113.0/24` (256 addresses). |
| **Require both / Either is enough** | How the IP list and sign-in are evaluated together. |
| **Path rule** | Extra protection or a block for a specific path and everything below it. |
| **Only chosen people** | A path rule that opens a path only to chosen shares, optionally within a time window. |
| **Webhook path (open path)** | A path that lets requests with the chosen methods through without sign-in. Komuta (with a signature check) or the application verifies the sender's signature. |
| **Signing secret** | The shared secret a webhook sender signs its requests with; Komuta keeps it encrypted and uses it to check signatures on a signed webhook path. |
| **Service token** | A secret key that programs send as a header instead of signing in. |
| **Private mesh** | A connection between your clusters that doesn't go out to the public internet; it doesn't pass the gateway. |
| **Method rule** | The HTTP methods a path accepts; other methods get `405`. |
| **CORS check (preflight)** | The `OPTIONS` request a browser sends before calling another site; can be let through without sign-in. |
| **Country list** | The countries visitors may come from; checked before sign-in. |
| **Rate limit** | How many requests one address may send in a time window; over it, `429`. |
| **Access preview** | A tool that shows who gets in where with the saved rules, without changing anything. |
| **Access log** | A record of sign-ins, page views and refusals, kept 30, 90 or 365 days. |
| **Protect new services** | An organization setting that starts every new public service behind Komuta sign-in for the organization. |
| **Protection end date** | When protection ends and what happens then. |
| **Visitor identity** | Passing the signed-in visitor's identity to the application in headers. |
| **JWT / JWKS** | A signed identity token / the published list of public keys used to verify the signature. |
| **Cloudflare** | The network in front of Komuta addresses and custom domains; it reports the visitor's address reliably. |
| **DNS only (grey cloud)** | A Cloudflare record working only as DNS, without proxying. The record of a custom domain added to Komuta must be like this in your own Cloudflare account. |
