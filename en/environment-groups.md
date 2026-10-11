# Environment Groups

An environment group keeps variables that several services share in one place: a shared database connection, an API key, a log setting that is the same everywhere. You write the value into the group once, then import it into any service.

A group is a source, not a live binding. Importing copies the values into the service, and from then on the service runs with its own copy. Changing a value in the group does not change any service by itself; you push the update to the services.

Groups live in the left menu under **Configuration → Environment Groups**, at [console.komuta.io/env-groups](https://console.komuta.io/env-groups). Groups are shared across the whole organization; a service in any project can import from the same group.

---

## Group or Service Variable?

| Situation | Use |
|---|---|
| The value belongs to one service only | The service's own [environment variables](services-environment-variables.md) |
| The same value is used by several services and should be updated together | An environment group |
| The value is the same in most services but must differ in one | Import from the group, then edit the value in the service that needs a different one. The edited variable is detached from the group. |

---

## Creating a Group

1. Open the **Environment Groups** page. If there are no groups yet, click **Create your first group**; otherwise click the **+** (**New group**) button in the list header.
2. In the **New environment group** dialog, fill in **Name** (for example `shared-postgres`). You can describe what the group is for in **Description (optional)**.
3. Click **Create group**.

The new group opens with two tabs: **Variables** and **Services**. The list on the left shows, for each group, the number of variables, how many of them are secrets (key icon) and how many services use it (server icon). The **Search groups...** box searches names and descriptions.

To change the name or description later, use **Edit** in the group header and click **Save**. Renaming a group does not detach the copies in services.

---

## Adding Variables

1. On the **Variables** tab, click **Add variable**.
2. In the **Add group variable** dialog, fill in **Key** and **Value**.
3. Turn on the options below if needed and click **Add**.

| Option | What it does |
|---|---|
| **Secret** | The value is stored encrypted and never shown again. When imported into a service it is written among the service's secret values and listed under the **Secret** tab on the service page. |
| **Available at build time** | The value is passed not only to the running app but also to the build stage. It turns on by itself when you type a key with a prefix such as `NEXT_PUBLIC_`, `VITE_` or `REACT_APP_`. |

Keys that clearly carry credentials are always stored as secrets, even with **Secret** off. Ignoring case and the characters `_`, `-` and `.`, this covers keys ending in `PASSWORD`, `SECRET`, `SECRET_KEY`, `API_KEY`, `PRIVATE_KEY`, `ACCESS_TOKEN`, `REFRESH_TOKEN`, `AUTH_TOKEN`, `CREDENTIAL`, `CREDENTIALS`, `CONNECTION_STRING`, `APPLICATION_KEY`, `SIGNING_KEY`, `ENCRYPTION_KEY`, `WEBHOOK_KEY`, `DATABASE_URL`, `REDIS_URL`, `RABBITMQ_URL` or `AMQP_URL`, and keys starting with `ConnectionStrings`.

In the list, plain values are shown as they are and secret values as dots with a lock icon. Secrets have an amber dot next to the key; build-time variables have a violet dot and a **Build** label.

> **Warning:** Values not marked as secret can be read by everyone who is allowed to view environment groups. A value marked **Available at build time** is baked into the image, and with a public prefix into the browser bundle too; don't define information that must stay secret this way.

### Editing a variable

**Edit** and **Delete** appear when you hover over a row. In the **Edit group variable** dialog:

- To keep a secret's current value, leave **Value** empty; the stored value is kept.
- To turn a secret back into a plain variable, you must type the value again; the stored value is never shown.
- If you rename the key, copies under the old name in services are detached from the group. They stay on the services as their own variables.

---

## Bulk Adding by Pasting a .env File

1. On the **Variables** tab, click **Paste .env**.
2. Paste the contents of a `.env` file or JSON into the dialog and click **Preview**.
3. Check the status of each row. Turn the row's **Secret** switch on or off if needed.
4. Click **Save … variables**. The result is reported as "*X added, Y replaced.*"

| Label | Meaning |
|---|---|
| **New** | The key is not in the group; it is added. |
| **Replace** | The key already exists in the group; its value is replaced with the pasted one. |
| **Skipped** | The value is empty; it is not saved. |

Paste rules:

- In a `.env` file, lines starting with `#` are skipped, an `export ` prefix is accepted and quotes around a value are removed.
- Nested JSON is flattened; levels are joined with `__` (`{"Logging": {"Level": "info"}}` becomes `Logging__Level`).
- If the same key appears more than once, the last value wins.
- Lines with an invalid key name do not appear in the preview at all.
- **Secret** starts switched on for keys containing words such as `SECRET`, `PASSWORD`, `TOKEN`, `CREDENTIAL`, `PRIVATE`, `API_KEY`, `PWD` or `AUTH`, and for keys that are already secrets in the group.
- This dialog has no build-time option: new keys get it from their prefix, existing keys keep theirs. To change it, edit the variable on its own.

If the total would go over the limit, the warning "*A group can hold at most 500 variables.*" appears and saving is disabled. Click **Back** to change the text.

---

## Importing a Group into a Service

Importing is done from the service's page:

1. Open the service and go to its **Environment variables** page.
2. Click **From group** in the toolbar (box icon; its name appears on hover). You can reach the same screen from the **From group** tab of the **Bulk Import** dialog. If the service has no variables yet, the **From group** button is in the middle of the empty page.
3. Pick the group in the **Environment group** list.
4. Every variable in the group is listed and compared with the service's value. Untick the ones you don't want.
5. Click **Import …**. If a value in the service will change, the button reads "*Import … (replaces …)*".
6. On the service page, click **Apply Changes**. Imported values are saved, but they don't reach the running service until you apply them.

| Label | Meaning |
|---|---|
| **New** | The key is not in the service; it is added. |
| **Replaces** | The service has a different value, or a different secret or build-time setting; the group's value is written. For plain values the old and new value are shown side by side. |
| **Same value** | The service already has the same value and settings; the value is not written again, only linked to the group. |

When the import finishes, the message "*X added, Y replaced, Z already matched. Apply to roll them out.*" appears. Each imported variable carries a label with the group's name; clicking the label opens the group.

> **Note:** If the service keeps a variable as a secret and the group's value is plain, the row shows "*(the service keeps this as a secret; importing makes it plain)*" and starts unticked. Select such a row only on purpose.

A variable can follow only one group at a time. A key that came from another group is shown with "*(now from …)*"; if you import it, it follows this group from then on.

---

## What Happens to Services When the Group Changes?

Nothing, until you push. The group header says so too: "*Services get a copy of these values when they import them. Changing a value here does not change any service until you push it from the Services tab.*"

The **Services** tab lists the services holding copies, with their project names. Each row shows how many variables the service has from this group and a status label:

| Label | Meaning |
|---|---|
| **Up to date** | Every copy in the service matches the group. |
| **Out of date** | At least one copy's value, secret setting or build-time setting differs from the group. The differing keys are listed on the row. |

When there are out-of-date copies, an amber dot appears next to the tab name.

### Pushing to services

1. Open the **Services** tab. Services that have out-of-date copies and that you can edit are selected; untick the ones you don't want.
2. Click **Push to … services**.
3. In the **Push group values** dialog, check which keys will be written in which service.
4. Decide on **Roll the pods now** (on by default) and click **Push and roll** or **Push values**.
5. Read the results and click **Close**.

| Result | Meaning |
|---|---|
| **… variables updated · rolling out** | The values were written to the service; the service is restarting with the new values. |
| **… variables updated · waits for the next deploy** | The values were written to the service; the running service keeps the old values until the next deploy or until you click **Apply Changes** on the service page. |
| **Already up to date** | There was nothing to write. |
| Red error text | The values could not be written; the reason is shown on the row. |

A push only overwrites copies that still follow the group. Values that were edited on a service are detached from the group and stay as they are.

> **Warning:** Apps read their variables when they start. If you push a secret with **Roll the pods now** off, pods that restart on their own in the meantime use the new value while the others keep the old one. Leave the option on for values that must change together (for example a user name and a password).

- With **Roll the pods now** on, if some of a service's values cannot be written, that service is not restarted and the message "*Not rolled out: some values could not be written. Push again once they succeed.*" appears. The keys that failed stay **Out of date**; pushing again completes them and restarts the service.
- If the values were written but the restart failed, the values are saved on the service; you can roll them out with **Apply Changes** on the service page.
- Values marked **Available at build time** enter the image during the build; restarting the pods does not rebuild the image. For such a value to reach your app, build and deploy the service again ([Pipelines](service-pipeline-guide.md)).

---

## Which Value Wins?

A running service only uses its own variables; the group is not merged with the service's values at deploy time. If the same key exists in both the group and the service, the service's value applies until you import or push.

| Situation | Value that applies |
|---|---|
| The key exists only in the group | The service doesn't have it until you import it. |
| The key exists in both, and the service did not take it from the group | The service's value. If you import, the group's value is written (**Replaces**). |
| The service's copy follows the group, and the value changed in the group | The service's old value until you push (**Out of date**). |
| The service's copy was edited on the service | The service's value. The link is broken; a push does not touch it. |

A copy on a service is detached from the group when:

- You change the variable's value, key or **Sensitive** setting on the service page.
- You paste a different value with **Bulk Import**.
- A [Stack](stacks.md) writes a value to the same variable.
- You delete or rename the variable in the group, or delete the group.

The group label next to a detached variable disappears. To make it follow the group again, import it from the group again.

> **Note:** Changing only the build-time setting on the service does not break the link. The copy then shows as **Out of date** and goes back to the group's setting on the next push.

---

## Deleting Variables and Groups

**Deleting a variable:** click **Delete** on its row and confirm in the **Delete variable** dialog. **Deleting a group:** click the trash icon (**Delete group**) in the group header and confirm.

Neither can be undone, but neither touches the services. If services hold copies, the confirmation says so: "*… services hold a copy. The copies stay on the services as their own variables; nothing is removed or redeployed.*" After a group is deleted, the message "*Group deleted. Services keep their copies.*" appears.

To remove a variable from a service completely, delete it on the service's **Environment variables** page and click **Apply Changes**. There is no separate button to detach a service from a group; editing or deleting the variable on the service is enough.

---

## Who Can Manage Groups?

Organization admins decide what members can do here on the **Access Control** page, by selecting the member and using the **Environment Groups** card.

| Permission | What it allows |
|---|---|
| **View environment groups** | Seeing the page and the groups, reading plain values, seeing the **Services** tab. Together with edit access to a service: importing into that service with **From group** and pushing the group to it. |
| **Create environment groups** | Creating new groups. |
| **Edit groups and their variables** | Changing the name and description; adding, editing and deleting variables; **Paste .env**. |
| **Delete environment groups** | Deleting groups. |

Importing and pushing also require edit access to the target service, so a member can only write a group's values into services they could already edit. The **Services** tab only lists services you can see. A service you cannot edit shows "*You cannot edit this service, so it cannot be updated from here.*" and cannot be selected.

---

## Limits and Validation

| Field | Rule |
|---|---|
| Group name | Required, up to 128 characters. Must be unique in the organization; case does not matter. |
| Description | Optional, up to 512 characters. |
| Number of variables | Up to 500 per group. |
| Key | Up to 256 characters; letters, digits, `-`, `_` and `.` only. Must be unique in the group and is case-sensitive (`db_url` and `DB_URL` are different keys). |
| Value | Required. Leading and trailing spaces are trimmed from plain values; secret values are stored exactly as entered. |

Messages you may see:

| Message | When |
|---|---|
| "*An environment group named '…' already exists.*" | The organization already has a group with that name. |
| "*A variable with this key already exists*" / "*This group already has a variable named '…'.*" | The group already has that key. |
| "*'…' is not a valid variable name. Use letters, digits, '-', '_' and '.' only.*" | The key contains a character that is not allowed. |
| "*Enter a value for '…'.*" | The value is empty, or you turned a secret into a plain variable without typing the value again. |
| "*A group can hold at most 500 variables.*" | The addition would go over the limit. |
| "*The group no longer has these variables: …. Reload the page and try again.*" | A variable you selected for import was deleted or renamed in the group meanwhile. |
| "*Not rolled out: some values could not be written. Push again once they succeed.*" | During a push, some of a service's values could not be written. |

---

## With Your AI Assistant

If Komuta MCP is connected, your assistant can list and show groups, create a group, add and update variables in it, show the comparison before importing into a service, import, and push the group to services. It asks for your confirmation before any change, and it can never read secret values. See [MCP Setup](mcp-setup.md) to set it up and [Komuta MCP](mcp-tools.md) for the general list of tools.

---

## Related Documents

- [Environment Variables](services-environment-variables.md) — A service's own variables, their types and **Apply Changes**.
- [Pipelines](service-pipeline-guide.md) — The build stage where build-time variables are used.
- [Stacks](stacks.md) — Managing service variables with a Stack file.
- [Organizations and Team](organizations-team.md) — Members, roles and permissions.
- [Komuta MCP](mcp-tools.md) — What you can do with the assistant.
