# Stacks

A Stack defines the services, managed services (PostgreSQL, Valkey, RabbitMQ) and domains that make up an application in a single `komuta.yaml` file. Komuta compares the file with what already exists, shows what will be created and the estimated monthly cost in a plan, and applies the plan once you approve it.

Use a Stack when you want to set up several services and a database together, in a repeatable way. The file lives next to your source code and is versioned with it. Every field of the file is described in the [komuta.yaml Manifest Reference](stack-manifest-reference.md).

---

## How It Works

1. Prepare a `komuta.yaml` file: with your AI assistant, by hand, or from your repository.
2. Use **Create plan** to validate the file and see the plan.
3. Enter secret values such as passwords and keys through a one-time secure link.
4. Start the plan with **Approve and apply plan** and follow the run.

A Stack is identified by the `name` field in the file. When you plan a file with the same name, Komuta updates the same Stack; a different name means a new Stack. The resources a Stack creates or adopts belong to that Stack, and a resource can belong to only one Stack.

A Stack shows up in the [**Stacks**](https://console.komuta.io/stacks) list, labeled **Not applied**, as soon as its first plan is created.

### Required permissions

| Permission | What it allows |
|---|---|
| **View stacks and runs** | The Stack list and the Stack page |
| **Author and validate stack manifests** | The **New Stack** page |
| **Create stack plans** | **Create plan** and viewing the latest plan |
| **Apply stack plans** | Applying a plan and the secure secret form |
| **Resume failed stack runs** | **Resume failed steps** |
| **Export stack manifests** | **Export** |

The plan also requires the matching permission for every resource it touches, for example the permission to create services, or to edit services to adopt or update one. A missing permission is listed as an issue in the plan.

---

## Creating a Stack

**Stacks → New Stack** opens the [Start a Stack](https://console.komuta.io/stacks/new) page. There are two ways to start.

### Create with AI

This is the **Recommended** path. Your project must be open in an AI client connected to Komuta.

1. **Open the local project in your AI client.**
2. **Confirm Komuta MCP is connected.** If it isn't set up yet, **Set up Komuta MCP** takes you to [Installing the Komuta MCP Server](mcp-setup.md).
3. **Paste the publishing prompt.** Get the prompt with **Copy prompt**.

The assistant inspects the code, asks the remaining business decisions in one message, prepares `komuta.yaml` next to your source code, validates it and shows the final cost and risk summary. It waits for your approval before publishing. Secret values are never asked for in the chat; they are entered through the one-time secure link.

> **Note:** Komuta can't attach your local folder from the browser. Start the conversation in the AI client that already has your project open.

**Profile.** Your assistant may ask how you want to proceed and save your choice: guided (`guided`, the default), balanced (`balanced`) or expert (`expert`). The profile only decides how many decisions the assistant explains and asks about; cost, public exposure, persistent data and security relaxations need your approval in every profile. In the console, the expert profile also shows each resource's **Managed fields** in the plan review.

### Review existing YAML

This is the **Advanced** path, for reviewing a `komuta.yaml` file you already have.

- Upload the file with **Choose komuta.yaml**. Only files with a `.yaml` or `.yml` extension and up to 1 MiB are accepted.
- Open an empty editor with **Paste existing YAML instead** and paste the content. In the empty editor, **Start from a skeleton** loads a starter file with placeholders to fill in; replace the values with your own resources and compare the fields with the [reference](stack-manifest-reference.md).

The editor shows the line count and a copy button. **Download YAML** downloads the draft as `komuta.yaml`; keep it in your repository so the repository stays the source of truth. The file doesn't leave the browser until you click **Create plan**.

If you try to leave the page before the draft is applied, **Leave this YAML review?** appears; the draft and its plan are discarded. **Choose another source** also discards the draft and returns to the start screen.

### From a komuta.yaml in your repository

In the [**Apps → New App**](new-service-guide.md) wizard, if the selected repository has a `komuta.yaml` file at its **root**, the wizard doesn't look for Dockerfiles. The file opens on the Stack page under the title **Review deployment proposal**, and a plan is created automatically.

Services in the file without `source.repository` get the repository you selected in the wizard, and those without `source.branch` get the selected branch. Services that point to a different repository are left as they are.

> **Note:** Only a file named exactly `komuta.yaml` at the repository root is detected.

### .NET Aspire and multi-service projects

The old Aspire import address now opens the **New Stack** page; there is no separate Aspire import screen. To publish a multi-service project such as an Aspire or docker-compose project, use the **Create with AI** path or write `komuta.yaml` yourself.

---

## Validation and Plan

There is no separate validate button: **Create plan** validates the file first. If the file has errors, "Manifest needs attention." appears and the **Validation** card lists each issue with its code, its path in the file (for example `$.services[0].compute.plan`) and its message. Warnings don't block the plan, but they are listed too. All messages are in the [reference](stack-manifest-reference.md#validation-messages).

If the file is valid, Komuta creates the plan. Planning also checks:

- resources with the same name that already exist in the target project (whether adoption is needed),
- each service's branch and Dockerfile: the Dockerfile must be in the **pushed** branch; the plan pins the current commit and Dockerfile,
- the names and prices of packages, managed service plans, storage plans and bandwidth plans,
- your permissions and the free package rules.

> **Warning:** The plan doesn't generate a missing Dockerfile for you. For a service that doesn't use a prebuilt image (`build.strategy` other than `image`), the plan is blocked until the Dockerfile is pushed to the branch: `Dockerfile '…' does not exist in pushed branch '…'. Create, test, commit and push it before planning.`

### What the plan shows

| Tile | Meaning |
|---|---|
| **Planned actions** | The number of resource actions in the plan. |
| **Estimated monthly cost** | The monthly estimate for the planned state; below it, the last applied version's estimate and the difference (**Current … · change …**). |
| **Missing secrets** | The number of required secret values not entered yet. |
| **Immutable plan validity** | The time left on the plan. A plan is valid for 30 minutes. |

When the time runs out, **Plan expired** appears and the plan can no longer be applied. **Create fresh plan** makes Komuta re-check permissions, prices and live resource state.

If the plan contains anything risky, three cards appear: **Public resources** (services with a public address, managed services that aren't private-only, and domains), **Persistent data** (persistent storage) and **Security relaxations** (running as root, extra Linux capabilities, writable paths).

The **Plan review** card is labeled **Ready to apply** or **Action required**. You can search actions by name and filter them with **All actions**, **Changes only**, **Needs attention** and **No change**. Each row shows the resource name, Komuta's reason, **Changed fields** when there are any, and, when the action also affects other Stacks, a "… more stacks are affected" notice with their names. Issues in the plan are listed below the card as `CODE · path — message`. Reasons and issue messages are shown in English.

| Action | Meaning |
|---|---|
| **Create** | The resource doesn't exist and will be created. |
| **Update** | A resource the Stack owns; fields changed in the file. |
| **Adopt** | A resource with the same name already exists; with your confirmation it moves to the Stack. |
| **No change** | The file matches the last applied version. |
| **Orphan** | The resource is no longer in the file; it isn't deleted and keeps running. |
| **Use** | An existing managed service is used without being managed (`uses.addons`). |

| Impact | Where it appears |
|---|---|
| **Rollout** | Services that are created or updated: a new version is rolled out. |
| **Recreate** | Managed services; new ones are listed with this label too. |
| **No impact** | The project, domains, and resources that are adopted, unchanged, orphaned or used. |

The **Needs attention** filter shows actions awaiting adoption confirmation, actions with the **Recreate** impact, **Orphan** actions and actions that affect other Stacks.

### Estimated cost

The estimate adds up:

- each service's package price × replica count (`count` in `fixed` mode, `min` with autoscaling; 1 if not set),
- persistent storage plans,
- bandwidth plans,
- the monthly plan price of the managed services the Stack creates.

Not included: existing managed services used through `uses.addons`, domains, build minutes beyond the included allowance, observability storage and tax. **Current** is the estimate for the last applied version (0 for a new Stack); **change** is the difference between the two. Prices are in USD. Current unit prices are on the [Pricing](https://www.komuta.io/pricing) page.

If a price can't be found, the plan is blocked and "Price unavailable. Refresh the plan before approval." appears.

> **Note:** The spending limit isn't checked in the plan; Komuta checks it when it creates each service and managed service. If the limit would be exceeded, that step fails. See [Billing](billing.md) for the limit.

### Adopting existing resources

If a service, managed service or domain with the same name already exists in the target project, the plan shows the **Adopt** action and the **Confirm adoption of existing resources** card.

1. Tick each resource you want to adopt.
2. Click **Re-plan with confirmed adoptions**.

An adopted resource isn't changed; the Stack only takes over its ownership. To adopt a managed service, its type, plan and network access in the file must match how it is now. A shared managed service that isn't attached to any project is moved into the target project when it is adopted; if services outside the project use it, adoption is blocked, so use `uses.addons` instead. A resource that belongs to another Stack can't be adopted.

---

## Secrets

Values such as passwords, keys and tokens can't be written into `komuta.yaml`. You declare a name under `secrets` in the file, and an environment variable refers to that name. A value can come from three sources:

| Source | Where the value comes from |
|---|---|
| `prompt` (default) | You enter it through the one-time secure link. |
| `generate` | Komuta generates a random value while the plan is applied. |
| `existing` | A value already stored in this Stack under another name is used (`existingName`). |

Values are kept in an encrypted store that belongs to the Stack and are never shown again. Later plans of the same Stack use the stored value and don't ask for it again.

### The one-time secure link

1. Click **Open secure secret form** in the plan. The form opens in a new tab. If it doesn't open, use the link in the "If the secure form did not open, use this one-time link" notice on the plan page.
2. The **Secure secret handoff** page shows which service created by the plan, and which environment variable, each value will be written to.
3. Enter the values one by one, or use **Paste from .env or JSON** and then **Fill fields**. Lines are matched by environment variable name or by secret name; anything else is skipped. Values that contain line breaks (such as PEM keys) aren't filled this way.
4. Click **Save securely**. When **Secrets saved securely** appears, close the tab.

The plan page refreshes by itself and the "Secrets received securely. The plan is ready for review." notification appears.

| Rule | Detail |
|---|---|
| Duration | The link is valid for 15 minutes. |
| Use | It works once; it becomes invalid after saving. |
| Person | Only the person who opened it can use it, with the **Apply stack plans** permission. |
| Scope | It asks only for that plan's missing `prompt` values and expects all of them at once. |

An expired or used link shows "The secure link is invalid, expired or already used."; open a new link from the plan.

> **Note:** A `prompt` value with `required: false` isn't asked for in the form. If an environment variable refers to it, that step stops with `SECRET_NOT_RESOLVED` during the run.

---

## Applying a Plan

1. When the plan is **Ready to apply**, click **Approve and apply plan**. On a long page, the **Go to approval** button takes you to it.
2. In the **Apply this exact plan?** dialog, read the number of actions and the estimated cost, then click **Apply approved plan**.

The button is disabled while the plan has errors, missing secrets, unconfirmed adoptions, an unknown price, or has expired. Only the person who created a plan can apply it. When you apply, Komuta checks some conditions again:

| Message | What to do |
|---|---|
| This plan has expired; create a new plan. | Create a new plan. |
| Permissions changed after planning; create a new plan. | Create a new plan. |
| Required secrets have not been completed. | Complete the secure form. |
| Resource state changed after planning. Create a new plan. | Create a new plan. |
| The remote source revision changed after planning; create a new plan. | A new commit reached the branch; create a new plan. |
| The remote source revision could not be verified; retry apply without changing the plan. | Apply again a little later. |
| Resource pricing changed or is unavailable. Create a new plan before approval. | Create a new plan. |
| Another Stack run is already active for this project. | A project runs one Stack run at a time; wait for it to finish. |

---

## Following the Run

Once the run starts, the **Live apply timeline** shows one step per resource. **No change**, **Orphan** and **Use** actions don't create steps.

Steps run in dependency order. If the project is to be created, it is created before anything else. A service runs after the managed services it reads values from and the services it refers to; a domain runs after the service it is attached to. At most `apply.maxParallel` steps run at once (default 3).

| Run status | Meaning |
|---|---|
| **Queued** | The run is waiting to start. |
| **Running** | Steps are being carried out. |
| **Waiting on input** | Komuta is waiting for running steps to finish or for a resource to become ready (for example a database being set up or a service's first deployment). Nothing is expected from you. |
| **Succeeded** | All steps finished; this version became the Stack's active version. |
| **Partially failed** | Some steps finished, others failed. |
| **Failed** | No step could finish. |

| Step status | Meaning |
|---|---|
| **Pending** | Waiting for its turn or for a retry. |
| **Running** | Being carried out. |
| **Waiting** | Waiting for the resource to become ready. |
| **Succeeded** | Finished. |
| **Failed** | Stopped; the error code and message appear under the step. |
| **Blocked** | Not run because a step it depends on failed. |

Temporary errors are retried automatically, with growing intervals, for up to 5 attempts. Permanent errors stop the step at once; the steps that depend on it are blocked and the others carry on. Common step errors:

| Code | Meaning |
|---|---|
| `REGION_NOT_FOUND` | `target.region` is missing or isn't an active region. |
| `GIT_CONNECTION_NOT_FOUND` / `REGISTRY_CONNECTION_NOT_FOUND` | A connection name under `target.connections` wasn't found. |
| `COMPUTE_PLAN_NOT_FOUND` / `ADDON_PLAN_NOT_FOUND` / `ADDON_PLAN_INACTIVE` | The package or managed service plan doesn't exist or isn't on sale. |
| `ROUTE_HOST_NOT_OWNED` | A route uses a domain that isn't attached to that service under `domains`. |
| `SECRET_NOT_RESOLVED` | The referenced secret value isn't stored. |
| `OWNERSHIP_CONFLICT` | The resource belongs to another Stack. |
| `OWNED_RESOURCE_NOT_FOUND` | A resource of the Stack was deleted in the console. |

### Resuming failed steps

A **Failed** or **Partially failed** run shows the **Resume failed steps** button.

1. Click **Resume failed steps**.
2. In the **Resume this Stack run?** dialog, click **Resume run**.

Succeeded steps are left untouched; failed and blocked steps are evaluated again. Only the person who created the plan can resume, and only while their permissions are unchanged. Because the plan pins the branch's commit, a service step fails again if the branch has moved on; in that case, create a new plan.

---

## The Stack Page

In the [**Stacks**](https://console.komuta.io/stacks) list, each card shows the Stack's name, its project, the latest run status (**Not applied** if it was never applied), **Export** and **Open operations**.

The top of the Stack page has the **Target project**, **Latest run**, **Owned resources** and **Drifted resources** tiles. Below them are the **Ownership and drift** panel, the plan of the latest run (read-only; needs the **Create stack plans** permission) and the **Live apply timeline**.

### Ownership and drift

The panel lists every resource the Stack owns with its **Managed fields** and an **In sync** or **Changed outside Stack** status. The header label reads **Desired state matches** or **Drift detected**.

> **Note:** Today this comparison records only the changes Komuta makes through the Stack. A change made in the console doesn't appear as drift in this panel. Such a change surfaces the next time the Stack changes the same setting: an environment variable update stops with `ENV_UPDATE_DRIFT`, an access protection update with `ACCESS_DRIFT`.

---

## Exporting

**Export** downloads the file applied by the last **successful** run as `komuta.yaml`. The file comes back as it was uploaded, comments included. Secret values are never in the file, so they aren't in the export either.

A Stack with no successful run can't be exported: "This Stack has no successfully applied revision to export."

Export doesn't generate a file from resources you set up in the console. To bring existing resources into a Stack, declare them in the file with the same names and adopt them during planning.

---

## Editing and Re-applying

The Stack page has no edit button. Changes go through the file:

1. Edit `komuta.yaml` in your repository or in a copy you got with **Export**. Don't change the `name` field.
2. Load the file with **New Stack → Review existing YAML**.
3. Click **Create plan**. The plan compares the file with the last applied version; each resource is listed as **No change**, **Update**, **Create** or **Orphan**.

Changes that can be applied in place today:

- Adding new services, managed services and domains.
- On a service whose rollout strategy is `auto-promote`, adding, changing or removing plain environment variables. The variable must not be secret, must not be used at build time, must not use `valueFrom`, and its value must not be empty. Komuta writes the values, redeploys the service with the same image and waits until it is healthy. An updated variable no longer follows an environment group.
- Changing the `access` block on its own ([Access Protection in a Stack Manifest](stack-manifest-access.md)).

Other changes (such as the package, ports, image, managed service plan or domain) block the plan: `In-place updates for Stack-managed … resources are not enabled on this target.` Make those changes on the resource's own page.

> **Note:** The plan compares the file with the last applied file, not with the live resource. If you also write a setting you changed in the console into the file, the plan sees it as a change and blocks.

---

## Deleting

There is no console action to delete a Stack. Removing a resource from the file doesn't delete it: the resource is listed as **Orphan** and keeps running and being billed.

To remove a resource, delete it from its own page: **Apps** for services, **Managed Services** for databases and queues, **Domains** for domains. A deleted resource stays in the Stack's ownership list; if the Stack later tries to change it, the step stops with `OWNED_RESOURCE_NOT_FOUND`.

---

## Related Documents

- [komuta.yaml Manifest Reference](stack-manifest-reference.md) — every field of the file, defaults and error messages.
- [Access Protection in a Stack Manifest](stack-manifest-access.md) — the `access` block.
- [Installing the Komuta MCP Server](mcp-setup.md) — connecting your AI client to Komuta.
- [Creating a New Service](new-service-guide.md) — the app wizard and a `komuta.yaml` in your repository.
- [Environment Variables](services-environment-variables.md) — secret and build-time variables.
- [Custom Domains](custom-domains.md) — verifying the domains a Stack adds.
- [Billing](billing.md) — the spending limit and charges.
