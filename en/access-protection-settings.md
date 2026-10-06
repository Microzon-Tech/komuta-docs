# End Date and Visitor Identity

The **Settings** tab of the **Access & ports** page has three sections:

- **Protection ends** — when protection ends and what happens then.
- **Tell my application who signed in** — passing who the signed-in visitor is to your application.
- **Remove protection now** — the **Open to everyone now** button.

These sections appear once protection has been turned on and saved (also while it is **Preparing**). While protection is off, the tab shows the **Access protection is off** notice. Changing the settings needs the **Manage service access protection** permission; people without it only see the summary sentence.

---

## Protection ends

If you want to protect an application for a limited time (for example until a launch, or during a customer demo), you can give protection an end date.

The sentence at the top of the section summarises the current situation:

| Summary | Meaning |
|---|---|
| "No end date — protection stays on until you turn it off." | No end date. |
| "Ends {date}, then the service stays locked." | There is an end date; protection continues when it is reached. |
| "Ends {date}, then the service opens to everyone." | There is an end date; protection is removed when it is reached. |
| "The end date {date} has passed; protection stayed on." | The end date passed; protection continues with "keep it locked". |
| "The end date {date} has passed; the service will be opened to everyone shortly." | The end date passed; protection is being removed. |

### Setting or changing the end date

1. Enter the date and time in **End date and time**. Times are in the time zone chosen in your account (**Account → General**). If your device is in another time zone, a warning appears under the field and shows what the chosen time is on your device.
2. Choose what happens in **When it ends**:
   - **Then keep it locked** (default) — when the end time comes, protection continues as it is; nothing opens until you decide.
   - **Then open it to everyone** — when the end time comes, protection is removed and the service opens to everyone (the same result as **Open to everyone now**).
3. Save with **Save end date**.

Rules:

- The end must be in the future and at most **365 days** away.
- An end date can be set while protection is on (once it has been saved). Saving on the **Rules** tab doesn't change the existing end date.
- **Make permanent** removes the end date (after the **Make protection permanent?** confirmation). Clearing the field and saving opens the same confirmation.
- If the service's public URL is off, or it has no public address yet, you can only move the end date later, keep the service locked, or make protection permanent; changes that would widen access (moving the end earlier, adding an end where there was none, choosing "open it to everyone") aren't possible.

### When the end time comes

- With **Then keep it locked**, nothing changes; protection continues and you get a "Protection ended, the service is still locked" email.
- With **Then open it to everyone**, protection starts being removed and you get a "Protection ended, the service is open" email. Shares aren't deleted. Until protection is fully removed, nobody new can sign in; sessions opened under this option never outlive the end time anyway.
- The end is usually processed right on time, and at the latest within a few minutes.
- If the organization isn't active (suspended, deleted or banned), the end isn't processed and protection stays as it is.

### Reminder emails

For protection with an end date, Komuta sends three emails:

| When | Subject |
|---|---|
| 24 hours before the end | "Access protection for {service} ends on {date}" |
| 1 hour before the end | The same |
| At the end | "Access protection for {service} has ended; the service is still locked" or "Access protection for {service} has ended; the service is now open to everyone" |

- **Recipients:** active users with a verified email who have been given edit access to the service itself and the **Manage service access protection** permission. If there are none, the organization admins with that permission. At most 50 people.
- **Language:** Turkish if the organization's default language is Turkish, otherwise English. Dates in the emails are written in UTC.
- If you set the end closer than 24 hours (or 1 hour) away, reminders whose time has already passed aren't sent. If you change the end date, the reminders are set up again for the new time; changing only the **When it ends** choice doesn't re-send reminders already sent (later ones go out with the new choice).
- The end email is sent only within 7 days after the end.
- The upcoming-end and "still locked" emails contain three links: **Open to everyone now**, **Extend** and **Make permanent**. The links open the **Settings** tab in the console; the action is taken there with your confirmation (**Open to everyone now** and **Make permanent** open the confirmation dialog, **Extend** puts the cursor in the date field). A link acts once, and only for people with the **Manage service access protection** permission.

Each share's own **Access ends** date applies separately; the protection end date doesn't change share end dates (see [Sign-in and Sharing](access-protection-sign-in-sharing.md#choices-when-adding-a-share)).

---

## Tell my application who signed in

While this setting is on, Komuta passes who the signed-in visitor is to your application with every request. Your application can then use "who is connected" without building its own sign-in screen: to keep records, show content per person or grant permissions.

- **It is off by default.**
- It needs Komuta sign-in to be required somewhere (on the site or in a path rule). If sign-in isn't required, the switch stays off and the section says "Turn on Komuta sign-in under Rules first; without sign-in nobody is known."
- If the section isn't visible, the feature isn't turned on on your platform yet.

### Turning it on

Turn the switch on; the setting is saved immediately ("The visitor will be named to your application"). Komuta updates the service's routing; meanwhile the section says "Getting ready: the service's routes are being updated. Until then your application receives the headers empty." This usually takes a few minutes. If it takes long, the section says "Deploying the service again updates them."; deploying the service again is enough.

### Headers your application receives

| Header | Contents |
|---|---|
| `x-komuta-user-email` | The visitor's verified email address. |
| `x-komuta-user-id` | The visitor's Komuta user id (a GUID). |
| `x-komuta-identity` | A proof of the visitor's identity signed by Komuta (a JWT). |

Things to know:

- **The headers are filled only on requests that need sign-in.** On the open parts of the site, where the IP list lets visitors in without sign-in, and on webhook paths, the headers are empty even if the visitor has signed in.
- **On requests with a service token** only `x-komuta-identity` is filled; its `kind` is `service_token` and its `sub` is the token's id: the 32 hex characters after `kst_` in the token value, written as a dashed GUID. The other two headers are empty.
- **Visitors who signed in before you turned this on** are named without their email until they sign in again (at most 12 hours): `x-komuta-user-email` is empty and the JWT has no `email` claim.
- Unless the section says "Getting ready" or that the routes haven't been updated yet, headers with the same names that a visitor sends themselves are removed while passing through Komuta and rewritten with the right value (or empty); nobody on the internet can fake these headers. While it is getting ready, don't trust the plain headers; the signed `x-komuta-identity` can always be verified.

### Which header to trust

Your other services on the same cluster can reach your pods directly, without passing Komuta, so they could send these headers themselves. If that matters to you, trust only the **signed `x-komuta-identity` header**, not the plain headers, and verify it on every request. The plain headers are a convenience.

### Proof of identity (JWT)

`x-komuta-identity` is a JWT signed with ES256.

Header: `{"alg": "ES256", "typ": "JWT", "kid": "<key id>"}`

| Claim | Value |
|---|---|
| `iss` | Always `komuta-access`. |
| `aud` | The address (host) the visitor opened, for example `panel.example.com`. Each address of the service (custom domain, `*.komuta.app` address, blue-green preview address) is a separate `aud` value. |
| `sub` | The Komuta user id; for a service token, the token's id. |
| `kind` | `user` or `service_token`. |
| `email` | The visitor's email. Present only for `user` and when the email is known. |
| `sid` | The service id. In the console it is the `/services/<id>` part of the service's address. |
| `tid` | The organization (tenant) id. |
| `iat` | When it was signed (Unix seconds). |
| `nbf` | `iat` − 30 seconds. |
| `exp` | `iat` + 5 minutes. |

**Public keys:** `https://api.komuta.io/api/devopszon/access-protection/identity-keys`

This address returns a standard JWKS (`{"keys":[{"kty":"EC","crv":"P-256","alg":"ES256","use":"sig","kid":"…","x":"…","y":"…"}]}`). It needs no sign-in and may be cached for 5 minutes. The list fills once the first service on a cluster turns this setting on; until then it may be empty. If you see a `kid` in a JWT that you don't know, fetch the key list again.

### Verification rules

The same key signs for the services of every organization on a cluster. So verifying the signature alone is not enough: another organization's application could replay a valid token it received to your application. Your application must check **all** of these:

1. `alg` must be `ES256` (accept no other algorithm).
2. `kid` must be in the published key list, and the signature must verify with that key.
3. `iss` must be `komuta-access`.
4. `aud` must be one of **your own** addresses. Compare against an address list configured in your application, not the `Host` header the visitor sends.
5. `sid` must be **your own** service's id.
6. `exp` must not have passed and `nbf` must have been reached (you can allow 30 seconds for clock skew).

### Example: Node.js (jose)

```javascript copy
import { createRemoteJWKSet, jwtVerify } from "jose";

const KOMUTA_KEYS = createRemoteJWKSet(
  new URL("https://api.komuta.io/api/devopszon/access-protection/identity-keys"),
);

const MY_HOSTS = ["panel.example.com"];
const MY_SERVICE_ID = "3a2331e6-0000-0000-0000-000000000002";

export async function komutaVisitor(headers) {
  const token = headers["x-komuta-identity"];
  if (!token) return null;

  const { payload } = await jwtVerify(token, KOMUTA_KEYS, {
    algorithms: ["ES256"],
    issuer: "komuta-access",
    audience: MY_HOSTS,
    clockTolerance: 30,
  });

  if (payload.sid !== MY_SERVICE_ID) {
    throw new Error("identity token belongs to another service");
  }

  return {
    kind: payload.kind,
    userId: payload.sub,
    email: payload.email ?? null,
  };
}
```

`jwtVerify` checks the signature, that the `kid` is in the list, and the `alg`, `iss`, `aud`, `exp` and `nbf` claims, and throws on a token that doesn't match. `createRemoteJWKSet` caches the keys and fetches the list again when it sees an unknown `kid`. In Express, call it as `komutaVisitor(req.headers)`.

### Example: Python (PyJWT)

```python copy
import jwt

KOMUTA_KEYS = jwt.PyJWKClient(
    "https://api.komuta.io/api/devopszon/access-protection/identity-keys",
    headers={"User-Agent": "my-app/1.0"},
)

MY_HOSTS = ["panel.example.com"]
MY_SERVICE_ID = "3a2331e6-0000-0000-0000-000000000002"


def komuta_visitor(headers):
    token = headers.get("x-komuta-identity")
    if not token:
        return None

    signing_key = KOMUTA_KEYS.get_signing_key_from_jwt(token)
    claims = jwt.decode(
        token,
        signing_key.key,
        algorithms=["ES256"],
        issuer="komuta-access",
        audience=MY_HOSTS,
        leeway=30,
        options={"require": ["exp", "nbf", "iat", "sub", "sid"]},
    )

    if claims["sid"] != MY_SERVICE_ID:
        raise PermissionError("identity token belongs to another service")

    return {"kind": claims["kind"], "user_id": claims["sub"], "email": claims.get("email")}
```

Install with `pip install "pyjwt[crypto]"`. Keep the `headers={"User-Agent": …}` line: the Komuta API refuses requests that come with Python's default client identification.

### Turning it off

When you turn the switch off, the headers become empty within a few seconds ("The visitor will no longer be named to your application").

---

## Remove protection now

The **Open to everyone now** button at the bottom of the **Settings** tab removes protection right away: sign-in, the IP allow-list and path rules stop applying, and anyone with the URL can open the service. It asks **Open this service to everyone?** first. Shares and service tokens are kept and apply again if you turn protection back on. For details see [Access Protection](service-access-protection.md#turning-protection-off).

---

## Related Documents

- [Access Protection](service-access-protection.md) — turning on and off, status.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — visitor sessions.
- [Machines and Private Mesh](access-protection-machines.md) — service tokens.
- [Reference](access-protection-reference.md) — headers and limits.
