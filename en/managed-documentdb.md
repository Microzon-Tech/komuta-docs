# DocumentDB

DocumentDB is a MongoDB-compatible document database that Komuta provisions and operates. Encrypted connections, daily backups, continuous change archiving, and monitoring rules are all in place the moment the instance is created; the only thing left for the application to do is hand the `mongodb://` address it's given to a MongoDB driver.

---

## What Is DocumentDB?

DocumentDB keeps data in **documents** rather than table rows. A document is a JSON-like record (stored as BSON); nested objects and arrays stay inside the same document instead of being split into separate tables, and they can be queried directly. Documents are grouped into **collections**, and collections into **databases**. Documents in a collection don't need to share the same fields; the schema can be tightened later with validation rules if needed.

### What It's Used For

- **Records with a varying shape.** Product catalogs, user profiles, form responses — data where each record carries slightly different fields.
- **Content and event records.** Keeping data that's read together in one document, such as an order and its line items, or an article and its tags.
- **Existing MongoDB applications.** If the application is written against a MongoDB driver, in most cases the only thing that changes is the connection address (see [Compatibility and Limits](#compatibility-and-limits)).
- **Vector search.** Embedding vectors are stored alongside the documents and queried with similarity search.

For relational data, complex joins, and anything that needs SQL, [PostgreSQL](managed-postgresql.md) is the better fit.

### The Open-Source DocumentDB Project

The service runs the open-source **DocumentDB** project, developed under the Linux Foundation and released under the MIT license. The project speaks MongoDB's wire protocol: MongoDB drivers, `mongosh`, and libraries built on top of a driver (Mongoose, Spring Data MongoDB, and so on) connect without a separate client. DocumentDB is not MongoDB itself; it supports most commands, but not all of them. The ones that aren't supported are listed under [Compatibility and Limits](#compatibility-and-limits).

> **Note:** AWS's **Amazon DocumentDB** is a separate product and has no relationship to the service on this page.

---

## How It Works

When an instance is created, Komuta sets up a DocumentDB server in the selected region, enables TLS, generates the connection user and password, starts daily backups and continuous change archiving, and installs monitoring rules. None of these steps require any additional configuration.

### Instance, Plan, and Region

An **instance** is an independent database server with its own disk, its own user, and its own backup chain. CPU, memory, and disk all come from the **plan** it's on (see [Plans](#plans)).

The **region** is where the instance physically runs, and it can't be changed after creation. Moving to another region means creating a new instance and migrating the data.

The **version** is the DocumentDB version the instance will run; today `1.0` is offered. It can't be changed later.

The **instance name** starts with a lowercase letter, contains only lowercase letters, digits, and hyphens, and is at most 40 characters. An instance can optionally be attached to a **project**.

No database name is asked for at creation: databases and collections are created on their own the first time the application writes to them.

### Creating an Instance

A new instance is created with the **New DocumentDB** button on the DocumentDB tab of the **Managed Services** page. Region, plan, name, and version are chosen, and the final step shows a summary of the selections and the estimated monthly cost. **Create** queues the setup; the instance appears in the list in **Provisioning** state, and no connection info is generated until setup finishes.

### Instance States

| State | Meaning |
|---|---|
| **Provisioning** | Setup is in progress; connection info isn't ready yet. |
| **Active** | Running and accepting connections. |
| **Upgrading** | A plan change is being applied. |
| **Upgrade Failed** | The plan change couldn't complete; the instance is **live** on its old plan and its data is intact. |
| **Restoring** | A restore operation is in progress. |
| **Pending Deletion** | Deletion has been scheduled; the connection address is closed and only the disk is billed. It can still be canceled before the window expires. |
| **Error** | An instance whose setup couldn't complete. Instances in this state are automatically cleaned up by the platform; their backups remain in place for the retention period. |

DocumentDB has no suspension and no sleep; an instance runs for as long as it exists (see [Maintenance and Lifecycle](#maintenance-and-lifecycle)).

---

## Plans

| Plan | vCPU | Memory | Disk |
|---|---|---|---|
| 1 GB | 0.5 | 1 GB | 10 GB |
| 2 GB | 0.5 | 2 GB | 40 GB |
| 4 GB (default) | 1 | 4 GB | 80 GB |
| 8 GB | 2 | 8 GB | 160 GB |
| 16 GB | 4 | 16 GB | 320 GB |
| 32 GB | 8 | 32 GB | 640 GB |

The current plans and their prices are listed in the creation wizard.

- **Every plan is a single server.** There's no second server (replica) for high availability. Brief downtime can occur during maintenance, plan changes, and node changes; the application needs to reconnect after a dropped connection.
- **At most 5 instances per organization on the 1 GB plan.** Instances in every state count, pending deletion included, and they're counted separately from PostgreSQL's small plans. Larger plans have no such limit.

---

## Connecting

### Connection Address

**Show connection info** on the **Overview** tab reveals the host, port, username, password, and a ready-made `mongodb://` connection string. The string works as-is in every MongoDB driver and in `mongosh`:

```text
mongodb://<user>:<password>@<host>:10260/?tls=true&authMechanism=SCRAM-SHA-256&retryWrites=false&maxIdleTimeMS=1500000
```

- **The port is `10260`**, not MongoDB's usual `27017`.
- **The connection always uses TLS.** The certificate is managed by Komuta and is publicly trusted; no CA file is needed. Authentication uses SCRAM-SHA-256.
- **The hostname must be used as-is.** The infrastructure that routes traffic to the correct instance decides based on the server name inside the TLS handshake. Resolving the address to an IP and connecting to that won't work — the connection appears to succeed and then drops immediately. Older clients that don't send the server name (SNI) fail for the same reason.
- No `mongodb+srv://` address or replica set name (`replicaSet`) is provided, and none is needed.

Copy the string the UI provides rather than building one by hand; two of its parameters in particular must stay in place:

- **Don't remove `retryWrites=false`.** By default, drivers send a write once more when the connection drops. The server can apply a write resent this way a second time: a counter goes up twice, a single-document delete removes two documents. This parameter turns the driver's automatic retry off. The tradeoff is that a write on a dropped connection comes back to the application as an error; the application should retry it itself, after checking whether the operation was applied.
- **Don't remove `maxIdleTimeMS=1500000`.** On the public address, a connection that sees no traffic for 30 minutes is closed. This parameter makes the driver close and replace an idle connection itself after 25 minutes; without it, the first request on a connection that sat idle for a long time fails.

Example usage (with the string kept in an environment variable):

```bash
mongosh "$MONGODB_URI"
```

```js
// Node.js (mongodb driver)
const { MongoClient } = require("mongodb");
const client = new MongoClient(process.env.MONGODB_URI);
```

```python
# Python (pymongo)
import os
from pymongo import MongoClient
client = MongoClient(os.environ["MONGODB_URI"])
```

### The User You're Given

The user in the connection info has **read and write access to every database on the instance** (the `readWriteAnyDatabase` and `clusterAdmin` roles):

- Create, drop, and rename databases and collections
- Perform any document operation, aggregation, and transaction
- Create indexes, views, and schema validation rules

User management isn't open to this user: commands like `createUser` return a permission error, so additional users can't be created inside the instance. For a tool or a person who only needs to read, a read-only user is created from the console (see [Read-Only Users](#read-only-users)).

### Viewing Connection Information

Passwords are stored encrypted on the platform side and are only decrypted when **Show connection info** is opened. **Every view is recorded with the instance, user, and time.** Rather than sharing credentials, it's recommended to give team members access from their own accounts.

If the instance has a private network path, a **Connection String (private network)** also appears at the end of the connection strings (see [Private Address](#private-address)).

### Read-Only Users

The **Read-only access** card on the **Overview** tab creates a user to hand an AI tool, or anyone who only needs to read, instead of the main password.

- **Create user** generates a new user and its connection strings. The password is shown **only once, at that moment**; Komuta doesn't store it. If it's lost, revoke the user and create a new one.
- This user reads every database on the instance and can't change anything: write, delete, and user-management commands are rejected.
- At most **3** read-only users can be open at the same time. A user stays open until it's revoked.
- **Revoke** closes the user right away: connections opened with it are refused at their next request, and its password stops working.
- If the instance is restricted by an IP allow-list, the address the tool connects from has to be on the list; in **Private only** mode, a tool running outside the private network can't connect.

> **Note:** When an AI tool connects with this user, the data it reads goes to the company that provides the tool.

---

## Network Access

Access mode is changed from the **Network access** card on the **Overview** tab. The mode change takes effect immediately; the instance isn't restarted.

### Access Modes

| Mode | Meaning |
|---|---|
| **Public** | Reachable from the internet via the public address, no source IP restriction. Default. |
| **Public, IP allow-list** | The public address is only open to the specified IP/CIDR ranges. |
| **Private only** | The public address is removed entirely; the instance is only reachable from the private network. |

When a mode is marked **Not available yet**, the reason is written inside the card. The IP allow-list stays disabled when the cluster can't see a connection's real source address (otherwise the list would silently accept everyone). **Private only** can only be selected once one of the organization's application clusters has been joined to the private network; otherwise the instance would become unreachable from anywhere, so the request is rejected.

> **Warning:** Switching to **Private only** mode deletes the public connection address. Every client connecting from outside the clusters — a laptop, a GUI client, a CI job, anything not running on Komuta — is disconnected and can't reconnect. Services using the private address aren't affected.

### IP Allow-List

The allow-list is filled in with one CIDR or IP per line:

```text
203.0.113.0/24
198.51.100.7
```

- A single address entered on its own is automatically expanded to a single host (`/32`).
- Any source not on the list is rejected before it reaches the instance.
- The list can't be left empty — an empty list would cut off all access, so the request is rejected. If the goal is to close off all external access, the right choice is **Private only** mode.
- An invalid entry isn't silently ignored — it errors out.

### Private Address

Services in the organization's clusters that have been joined to the private network connect to the instance through its private address. The address appears among the connection strings as **Connection String (private network)** and **works in every mode** — it's valid in public mode too, and removing the public address doesn't affect it. In **Private only** mode, the main connection string is the private address itself.

```text
mongodb://<user>:<password>@<private-host>:10260/?authMechanism=SCRAM-SHA-256&retryWrites=false
```

The private address doesn't use TLS, so the string has no `tls=true` and no certificate setting is needed. Authentication is still verified with SCRAM-SHA-256 and the password never crosses the network in plain text; documents and queries, however, travel unencrypted inside the private network. Keep `retryWrites=false` here as well, for the same reason.

---

## Backups

### Backup Layout and Retention Period

Protection has two parts, working together:

- A **full backup** is taken once a day. The backup time is set independently per instance.
- **Changes are archived continuously.** This is what makes point-in-time recovery possible; protection also covers the changes between two full backups.

The default retention period is **7 days** (it can be set anywhere from 2 to 365 days), covering both full backups and the change archive. That period is also what determines the earliest point you can recover to.

If an extra copy is needed before a specific operation, one can be taken manually with **Backup now** on the **Operations** tab. It doesn't replace the daily backup; it produces an extra copy alongside it.

### Backup Schedule

The time of the daily backup and the retention period are changed from the **Backup schedule** card on the **Backups** tab. The time is given as a five-field cron expression (`0 3 * * *` → every day at 03:00 UTC), and the retention period is given in days. Changing the backup time doesn't affect the retention period.

> **Warning:** Shortening the retention period is irreversible. Once the period is lowered, backups that fall outside the new window are permanently deleted at the next maintenance run. Extending the period back out doesn't bring the deleted ones back — it takes that many days for the protection window to refill.

### Backup Health

Backup health is checked once a day, and one of three states is shown on the **Backups** tab:

| State | Meaning | What to do |
|---|---|---|
| **Healthy** | The last backup and the change archive are newer than 26 hours. | — |
| **Problem** | The check ran, but the last backup or the archive is older than expected. | Check disk usage; open a support ticket if it persists. |
| **Unknown** | The check hasn't run for 48 hours; nothing can be said about backup status. | Open a support ticket. "Unknown" doesn't mean healthy. |

Since the indicator is based on the daily check, a backup taken just now may not show up here right away.

### Backup Destruction

Backups are only permanently deleted in two cases:

- When a scheduled deletion's grace period expires,
- Through **Hard delete**.

No other path touches backups. If an instance goes into an error state and gets auto-cleaned, if setup stalls halfway, or if a restore is canceled, backups remain in place for the retention period. Reaching the backup of an auto-cleaned instance requires opening a support ticket.

---

## Restoring

A restore is started with **Open restore** on the **Backups** or **Operations** tab. There are two options: restoring from the latest backup, or going back to a specific moment within the retention period.

### Recoverable Range

- **Going back:** at most as far as the retention period, but never before the instance's first backup. On a newly created instance, this range may still only be a few hours.
- **Going forward:** the upper bound isn't "now" — it's the **most recently archived change**. On an instance with little write activity, the last few minutes may not be recoverable yet.

The current range is shown as **Oldest** and **Newest** on the **Restore points** card. Picking a moment outside the range errors out before the operation starts; **Validate target** lets a timestamp be tested beforehand.

### Restoring Creates a New Instance

A restore doesn't overwrite the existing instance:

1. A **new instance** is created with the data as of the selected moment.
2. The existing instance keeps running, serving traffic, and **being billed** — the two instances sit side by side for a while, and both are charged.
3. It's up to the user to point the application's connection settings at the new instance and delete the old one.

> **Warning:** The restored copy is a separate instance and gets its own connection address — its public and private addresses differ from the source's. Take the connection string from the new instance's **Overview** tab; services still pointed at the source keep connecting to it until their settings are changed by hand.

A restore in progress can be stopped with **Cancel restore** on the **Operations** tab. The half-provisioned copy is removed; the source instance and its backups aren't affected.

---

## Plan Change

The plan is changed with **Change plan** in the instance header. An impact preview runs automatically once a target plan is selected: the type of change, the estimated downtime, resource changes, and any warnings. When the preview reports a change that can't be applied, **Apply plan** stays disabled.

- **Plans can't be downgraded.** Lower plans may appear in the list, but the change won't be applied.
- **Disk can't be shrunk.** This isn't a platform restriction — it's how block storage works.
- **Capacity is measured per node.** The disk has to grow on a single physical node; even if the cluster's total free space looks sufficient, the change can't happen if that particular node is out of room.
- **The preview isn't a guarantee.** Capacity is re-checked at the moment of confirmation; if there's a wait after the preview opens, the same change could be rejected.

Since plans are single-server, the server restarts during a plan change and there's a brief outage; the estimated duration is shown in the preview. If a plan change stalls halfway, the instance keeps running on its old plan and moves to **Upgrade Failed** state; its data is intact, and the same change can be retried. If the problem keeps happening, a support ticket should be opened.

---

## Maintenance and Lifecycle

Restarting, password rotation, and deletion are all gathered on the **Operations** tab.

### Restarting

**Restart** restarts the server. Since it's a single server, the instance briefly stops accepting connections; the application needs to reconnect.

### No Suspension

DocumentDB has no suspension and no sleep. An instance runs, and is billed at its plan's full price, for as long as it exists; the only time just the disk is billed is the week a deletion is pending.

### Password Rotation

**Rotate database password** generates a new password for the connection user and **drops every open client connection.** The application gets errors until it reconnects with the new password.

- Can only be done on a running instance.
- The old password becomes invalid the moment the operation completes.
- The next rotation date is flagged as 90 days out; this is a reminder, not a requirement.
- Read-only users' passwords don't change.

This operation should be done during a maintenance window, with the new password ready to roll out.

### Deletion

There are two paths, with different outcomes:

- **Schedule deletion** takes the instance offline right away: the public address closes and the instance is removed from the private network too. It's deleted at the end of a 7-day grace period, and only the disk is billed meanwhile. **Cancel scheduled deletion** before the period ends undoes it: an instance that was serving when the deletion was scheduled serves again on the address it had. The cancel needs the spending limit to cover the plan's full price; otherwise it's refused. An instance that was in an error state stays in error after the cancel.
- **Hard delete** instantly destroys the instance and **all of its data, including backups.**

A deletion can't be scheduled while the instance is being set up, having its plan changed, or being restored. If services use the instance, the deletion is rejected and the error names those services; remove the instance's environment variables from them, or delete the services, first.

> **Warning:** Hard delete can't be undone, and since backups are deleted too, the data can't be recovered through support either. If the data needs to stick around a bit longer, scheduled deletion is the right choice.

---

## Compatibility and Limits

The lists below are based on checks run over the MongoDB protocol against a live instance.

### Supported

- Inserting, reading, updating, and deleting documents (CRUD); bulk writes
- Update and query operators
- The aggregation pipeline, including `$lookup`, `$facet`, `$graphLookup`, and `$setWindowFields`
- Every index type; TTL indexes (expired documents are deleted in the background, after a short delay)
- Vector search. The vector index isn't created with MongoDB Atlas's search index commands but in the format the open-source DocumentDB project documents: `"cosmosSearch"` as the index key, with `cosmosSearchOptions` next to it (index kind, number of dimensions, similarity metric).
- Views and schema validation
- Multi-document transactions
- 17 BSON data types

### Not Supported

| Feature | Note |
|---|---|
| Change streams (`watch()`, `$changeStream`) | Changes can't be listened to in real time. Polling on an indexed last-updated field can be used instead. |
| Capped collections | Fixed-size collections can't be created; a TTL index can be used to remove old records. |
| Time series collections | Time series data can be kept in a regular collection with the time field indexed. |
| `$where` | JavaScript can't be run inside a query. |
| Collation | There's no language-aware comparison or sorting. For case-insensitive search, a normalized copy of the field can be stored. |
| Administrative and diagnostic commands | `serverStatus`, `top`, `rolesInfo`, `replSetGetStatus`, `listShards`, `fsync`, `logRotate`, `dbHash`, `reIndex`, `getParameter`. Monitoring tools that rely on these commands won't work; the instance's state is followed on the **Monitoring** tab. |
| User management | There's no permission for `createUser` and similar commands (see [The User You're Given](#the-user-youre-given)). |

### Sorted Cursors Close After 60 Seconds of Silence

A cursor that returns a sorted result — an aggregate with `$sort`, or a find with `sort()` — is closed on the server if the driver **doesn't ask for the next batch for 60 seconds**, and the next request gets a `CursorNotFound` error. The `noCursorTimeout` option is ignored. Unsorted cursors aren't affected.

Short paged queries (for example, an API page with `sort` + `limit`) don't leave the cursor open, so they aren't affected. The pattern that is affected is a background job that walks a sorted result row by row and does slow work per record. Jobs like that should read the result page by page: queries that continue from the last value read (for example, `{ _id: { $gt: lastId } }` with `limit` on `_id` or another indexed field).

### High Concurrency

When a very large number of requests is sent at the same time, requests can fail; the situation recovers on its own within a few minutes. Keep the number of requests in flight across all of the application's copies reasonable, and don't set the driver's connection pool (`maxPoolSize`) larger than needed.

### Large Responses

Every response is held in the server's memory until the application reads it. Many large responses at once (for example, batches of thousands of documents, or very large documents) can fill that memory; when it happens, open connections drop and the application has to reconnect, and data isn't affected. Asking only for the fields you need with a projection, keeping batch sizes small, and queuing large reads prevents this. If it keeps happening, move to a larger plan.

### Read-Only When the Disk Fills

The instance switches to read-only mode when the disk fills up to its threshold: **about 85% on a 10 GB disk, 87% on 40 GB, 93% on 80 GB, and 95% from 160 GB up.** Reads keep working, but write requests fail. Writes only come back once space is freed up or the instance moves to a plan with a bigger disk. Disk alerts fire before the threshold, which is why they shouldn't be left unattended.

---

## Monitoring and Alerts

- The **Monitoring** tab shows graphs such as connection count, transaction rate, and disk usage over a selected time range.
- The **Logs** tab searches the server logs within the last 24 hours at most; level, time window, and row limit are selected separately. This screen isn't a long-term log archive; if longer retention is needed, ship the logs to your own side.
- Monitoring rules are set up automatically when the instance is created. Topics covered for DocumentDB: disk fill, connection saturation, the instance becoming unreachable, the layer that accepts connections stopping or running out of memory, the server restarting, and stale or failed backups. Triggered alerts are listed on the **Alerts** tab.

---

## Frequently Asked Questions

### Will my existing MongoDB application work without changes?

In most cases, the only thing that changes is the connection string. If the application uses change streams, capped or time series collections, `$where`, or collation, those parts need to change; details are under [Compatibility and Limits](#compatibility-and-limits).

### Why is `retryWrites=false` there, and can I remove it?

It shouldn't be removed. A write that the driver resends after a dropped connection can be applied a second time on the server. The parameter turns that automatic retry off; if a retry is needed, the application checks the outcome and retries itself.

### Is there high availability?

No. Every plan is a single server; brief downtime can occur during maintenance and node changes. The application should be written to reconnect after a dropped connection.

### Can I suspend it while I'm not using it?

No, DocumentDB has no suspension and no sleep. An instance is billed at its plan's full price for as long as it exists.

### Do backups get deleted along with the instance?

**Hard delete** destroys the backups too, and it can't be undone. **Schedule deletion**, on the other hand, leaves the backups in place for the 7-day grace period, and can be canceled.
