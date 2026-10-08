# Access Protection in a Stack Manifest

A service in a [Stack](quick-start.md#setting-up-with-a-stack) can declare its access protection in the Stack file, next to its source, compute and network settings. When the plan is applied, Komuta turns protection on with those settings. This page describes the `access` block: what it can express, how it is checked, and what happens when a service is created, updated or adopted.

What access protection does, and everything you can set in the console, is explained in [Access Protection](service-access-protection.md).

---

## The access block

```yaml copy
version: 1
name: main
services:
  - name: api
    source: { repository: 'https://github.com/acme/api.git', branch: main }
    compute: { plan: dz-dev }
    network: { public: true }
    access:
      signIn: true
      allowIps: ['203.0.113.0/24']
```

| Field | Type | Default | Meaning |
|---|---|---|---|
| `signIn` | `true` / `false` | `false` | **Require Komuta sign-in** for the whole site. |
| `organization` | `true` / `false` | `true` when `signIn` is `true` | Share the service with **Your organization** (every active member). Set it to `false` to turn sign-in on without sharing it with the organization; then add people in the console. |
| `allowIps` | list of addresses | empty | The **IP allow-list**: one IPv4 or IPv6 address or CIDR range per entry. |

When both `signIn` and `allowIps` are set on a new service, they combine as **Require both**: visitors must come from a listed address and sign in. On a service that is already protected, the combination set in the console is kept.

Field names are case-sensitive. A field that isn't one of these three is refused ("Unknown field '…'. Expected fields: signIn, organization, allowIps.").

An **open block** — `access: {}`, or `signIn: false` with no `allowIps` — means "no protection" (see below for what that does on create and on update).

---

## Requirements

- The service must be a `service` workload; jobs and cron jobs have no public address.
- The service must be public: `network: { public: true }`. A port alone isn't enough.
- The service must run on Komuta hosting.
- Access protection must be available to your organization.
- The person who plans the Stack needs the **Manage service access protection** permission when a new service has an `access` block (even an open one) or when the plan changes `access` on an existing service. If permissions change between planning and applying, the apply is refused ("Permissions changed after planning; create a new plan.").

---

## Validation

**Validate** and **Create plan** check the block. The `allowIps` entries follow the same rules as the console's [IP allow-list](access-protection-rules.md#ip-allow-list): at most 100 entries, public internet addresses only, no `/0`, host bits zero, no leading zeros.

| Code | Message |
|---|---|
| `ACCESS_WORKLOAD_UNSUPPORTED` | Access protection is supported only for service workloads. |
| `ACCESS_REQUIRES_PUBLIC_ADDRESS` | Access protection needs a public address; set network.public to true. |
| `ACCESS_ORGANIZATION_NEEDS_SIGN_IN` | Sharing with the organization needs signIn: true. |
| `ACCESS_ALLOW_IPS_TOO_LONG` | allowIps accepts at most 100 entries. |
| `ACCESS_ALLOW_IP_NOT_PUBLIC` | allowIps entries must be public internet addresses; private, loopback and documentation ranges never reach the gateway. |
| `ACCESS_ALLOW_IP_EVERYTHING` | A /0 range admits everyone; leave allowIps empty instead. |
| `ACCESS_ALLOW_IP_INVALID` | Every allowIps entry must be an IPv4/IPv6 address or CIDR range with no host bits set. |

The plan also refuses a block it can't apply (code `CAPABILITY_NOT_AVAILABLE`), for example "Access protection is not available to this organization.", "Access protection is supported only on Komuta hosting." or "Access protection cannot be combined with private network exposure on this platform."

---

## Creating a service

When the plan creates a new service:

- **With a block that asks for protection** (`signIn: true`, or a non-empty `allowIps`), protection is set up together with the service: the declared sign-in, IP allow-list and organization share, with no path rules and no end date. It takes effect a few minutes after the service's first deploy, like protection turned on in the console. If the protection can't be set up, the service isn't created and the run step fails with one of the codes under [Run errors](#run-errors).
- **With an open block** (`access: {}`), the service starts open, even if the organization's **Protect new services** setting is on.
- **Without a block**, the organization's **Protect new services** setting decides, as for a service created in the console.

---

## Updating a service

When `access` changes on a service the Stack already manages:

- **Change `access` on its own.** A plan that changes `access` together with other fields of the same service can't be applied in place; apply the `access` change in a separate revision.
- **Console changes are never overwritten silently.** Before applying, Komuta compares the service's current sign-in, IP allow-list and organization share with what the previous revision declared. If they were changed in the console, the apply stops with `ACCESS_DRIFT` ("Access protection changed outside the approved baseline; inspect it in the console before replanning."). If the previous revision had no `access` block and protection was set up in the console, declare the current settings first, then change them in a later revision.
- **What isn't in the manifest is kept.** Path rules, the combination, the end date, telling your application who signed in, and every share other than the organization share stay as they were set in the console.
- **An open block turns protection off.** The apply waits until protection is off.
- **Removing the block leaves protection as it is.** To turn protection off from the Stack, declare an open block.
- The apply waits until the change is in force (**Protected**). If someone changes or turns off protection in the console meanwhile, it stops with `ACCESS_DRIFT`.

Stack drift checks don't look at access protection; a console change shows up only when the next `access` update is applied.

---

## Adopting an existing service

A Stack can't change access protection while it adopts a service. A plan that adopts a service and also has an `access` block for it is refused: "Access protection is not applied while a service is adopted; adopt it first, then declare access in a later revision." Adopting without a block doesn't touch the service's existing protection.

---

## What the manifest can't express

These are set only in the console (**Service Detail → Configuration → Access & ports**), and a Stack update keeps them:

- path rules and webhook (open) paths, method rules and the CORS setting, countries and the rate limit;
- the **Either is enough** combination;
- shares other than **Your organization** (members, linked organizations, email addresses), share links and service tokens;
- the session length, the protection end date and telling your application who signed in;
- services that may come in over the private mesh.

**Export** of a Stack writes `access` only if the applied manifest declared it; protection set up in the console isn't written into the export.

---

## Related Documents

- [Access Protection](service-access-protection.md) — what protection does, turning it on and off.
- [Rules](access-protection-rules.md) — Komuta sign-in and the IP allow-list in detail.
- [Sign-in and Sharing](access-protection-sign-in-sharing.md) — the organization share and other shares.
- [Reference](access-protection-reference.md) — limits and error messages.

---

## Run errors

When a plan is applied, a step that can't set up or change the declared protection stops with a code such as these. The message is shown as Komuta returns it.

| Code | Message |
|---|---|
| `ACCESS_PROTECTION_NOT_AVAILABLE` | Access protection is not available to this organization. (or: Access protection is not enabled on this platform.) |
| `ACCESS_REQUIRES_PUBLIC_ADDRESS` | Access protection needs a public service on Komuta hosting. (On an update: Access protection needs a service with a public address.) |
| `ACCESS_MESH_EXPOSED` | Access protection cannot be combined with private network exposure on this platform. |
| `ACCESS_UNSUPPORTED_CLUSTER` | Access protection is supported only on Komuta hosting clusters. |
| `ACCESS_RULES_NOT_AVAILABLE` | This service uses access rules that cannot be changed on this platform right now. |
| `ACCESS_DECLARATION_INVALID` | The declared access protection needs sign-in or an allow-list, and sharing with the organization needs sign-in; provisioning was not started. |
| `ACCESS_PROTECTION_NOT_PREPARED` | The declared access protection could not be prepared; provisioning was not started. |
| `ACCESS_DRIFT` | Access protection changed outside the approved baseline; inspect it in the console before replanning. (Also: protection was configured in the console and differs from the declaration, or was turned off or changed while the Stack change was being enforced.) |
| `ACCESS_ALLOW_IP_INVALID` | The declared allowIps could not be normalized. |
