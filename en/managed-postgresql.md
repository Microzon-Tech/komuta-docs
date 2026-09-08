# PostgreSQL

PostgreSQL is a relational database service that Komuta provisions and operates. Server setup, encrypted connections, continuous WAL archiving, daily backups, and monitoring rules are all in place the moment the instance is created; the only thing left for the application to do is use the connection address it's given.

---

## How It Works

When an instance is created, Komuta provisions a PostgreSQL cluster in the selected region, creates the application database and a fully privileged user on it, enables write-ahead log (WAL) archiving and daily backups, and installs monitoring rules. None of these steps require any additional configuration.

### Instance, Plan, and Region

An **instance** is an independent database server with its own disk, its own user, and its own backup chain. CPU, memory, disk, and server (replica) count all come from the **plan** it's on; server parameters are also computed from the plan and can't be tuned by hand (see [Automatically applied limits](#automatically-applied-limits)).

The **region** is where the instance physically runs, and it can't be changed after creation. Moving to another region means creating a new instance and migrating the data.

An instance can optionally be attached to a **project**.

### Instance States

| State | Meaning |
|---|---|
| **Provisioning** | Setup is in progress; connection info isn't ready yet. |
| **Active** | Running and accepting connections. |
| **Suspended** / **Fully Suspended** | Manually suspended; data stays in place. |
| **Upgrading** | A plan change is being applied. |
| **Upgrade Failed** | The plan change couldn't complete; the instance is **live** on its old plan and its data is intact. |
| **Restoring** | A restore operation is in progress. |
| **Pending Deletion** | Deletion has been scheduled; it can still be canceled before the window expires. |
| **Error** | An instance whose setup couldn't complete. Instances in this state are automatically cleaned up by the platform; their backups remain in place for the retention period. |

![PostgreSQL list](https://cdn.komuta.io/docs/tr/images/postgresql/postgresql-home-page.png)

---

## Creating an Instance

A new instance is created in three steps from the **New PostgreSQL instance** button on the PostgreSQL list.

### 1. Region and Plan

Region and plan are chosen in this step. The plan choice determines not just the amount of resources, but also the automatically applied connection limits and query timeouts — on smaller plans these are fairly short. For bulk data loading or reporting workloads, the plan should be chosen based on the values in [Automatically applied limits](#automatically-applied-limits).

### 2. Name, Version, and Database

The **instance name** starts with a lowercase letter, contains only lowercase letters, digits, and hyphens, and is at most 40 characters.

The **initial database name** is a PostgreSQL identifier and follows a different rule: it starts with a lowercase letter, contains only lowercase letters, digits, and underscores (hyphens aren't allowed), and is at most 63 characters — `^[a-z][a-z0-9_]{0,62}$`.

The **version** is the PostgreSQL version the instance will run; only the versions supported by the selected plan are listed. The major version can't be changed later — moving to another version means creating a new instance and migrating the data.

### 3. Review and Create

The final step shows a summary of the selections and the estimated monthly cost. **Create** queues the setup; the instance appears in the list in **Provisioning** state, and no connection info is generated until setup finishes.

![New PostgreSQL instance summary](https://cdn.komuta.io/docs/tr/images/postgresql/create-postgresql-summary.png)

---

## Connecting

### Connection Address

The connection is established using the hostname given to the instance, and that name must be used **as-is**. Resolving the address to an IP and connecting to that won't work — the connection appears to succeed and then drops immediately. Connection pools that pin the address to an IP, and older clients that don't send the server name, fail for the same reason.

The connection is always encrypted; the certificate is managed by Komuta and needs no extra configuration. The connection strings the UI provides already carry the correct encryption setting and should be used as-is — changing that setting by hand can break the connection.

### The User You're Given

The user in the connection info is a **fully privileged application user** on that database — not the server's superuser account. Administrative access stays on the platform side and isn't shared with the customer.

What this user can do:

- Create, alter, and drop schemas, tables, indexes, views, functions, and triggers
- Perform any data operation and transaction
- Create their own roles and grants
- Use any pre-installed extension

Operations that require superuser privileges aren't supported: permanently changing server-level parameters, installing an extension that isn't on the list, or running commands that touch the file system. Server parameters are set automatically based on the plan; if a different value is needed, it's evaluated through support.

### Viewing Connection Information

**Show connection info** on the **Overview** tab reveals the host, port, database name, username, and password; below it are ready-made connection strings (plain URI, .NET, JDBC, Python psycopg, Go).

Passwords are stored encrypted on the platform side and are only decrypted when this screen is opened. **Every view is written to the audit log with the instance, user, and time.** Rather than sharing credentials, it's recommended to give team members access from their own accounts.

If the instance has a private network path, a **Connection String (private network)** also appears at the end of the connection string list. This address is not the same as the public one; which one to use depends on the mode described in [Network access](#network-access).

![Overview tab](https://cdn.komuta.io/docs/tr/images/postgresql/dashboard.png)

---

## Automatically Applied Limits

### Connection and Memory Limits

Server parameters are computed from the plan's resources; they can't be tuned by hand.

| Parameter | Calculation | Bounds |
|---|---|---|
| Database connections | vCPU × 100 | min 100, max 1,000 |
| Client connections | 5× database connections | 500 – 5,000 |
| `shared_buffers` | RAM × 0.25 | — |
| `effective_cache_size` | RAM × 0.75 | — |

Because a connection pool sits in front, the number of client-side connections can be higher than the actual number of database-side connections. Even so, the application's pool size should stay under the client limit in the table.

When more concurrent connections are needed, the only way is to move to a plan with a higher vCPU count; this limit can't be raised separately.

### Query Timeouts

Automatic timeouts are applied based on the plan to keep a query or an open transaction from locking up the database. These values aren't user-configurable.

| Plan RAM | Query duration | Idle transaction | Lock wait | Temp file |
|---|---|---|---|---|
| ≤ 4 GB | 30s | 30s | 10s | 512 MB – 1 GB |
| ≤ 8 GB | 60s | 60s | 15s | 2 GB |
| ≤ 16 GB | 120s | 120s | 30s | unlimited |
| > 16 GB | unlimited | 300s | 60s | unlimited |

A query that exceeds its duration is canceled and the application gets an error. In addition, a check that runs every 5 minutes on every instance terminates queries and idle transactions that have exceeded roughly 5x these limits.

> **Warning:** Bulk data loads, large index builds, and reporting queries hit timeouts on smaller plans. Work like this should be split into chunks, run during off-peak hours, or run on a higher plan.

### Disk Layout and Fill State

Data files and the WAL share the same disk; roughly 10% of the disk is reserved for WAL, and the rest goes to data.

**When disk usage passes 80%, the database switches to read-only mode**: queries keep running, but write attempts fail. Writes only come back once space is freed up or the instance moves to a plan with a bigger disk — which is why a disk-fill alert shouldn't be left unattended.

---

## Pre-Installed Extensions

### Extensions Installed by Default

Every new database comes with the following extensions already installed; no extra action is needed:

`pgcrypto` · `citext` · `pg_trgm` · `unaccent` · `pgaudit` · `hstore` · `uuid-ossp` · `dblink` · `postgres_fdw`

If the server image supports them, vector search extensions (`vector` / `pgvector` and `vectorscale`) are also installed. `pg_stat_statements`, used for query statistics, kicks in the first time the **Insights** tab is opened.

### Logical Replication and CDC

The server runs with `wal_level = replica`. This means logical replication and change data capture (CDC) tools — like Debezium — aren't available by default. If a setup like this is needed, a support ticket should be opened; the setting is evaluated on a per-instance basis.

---

## Network Access

Access mode is changed from the **Network access** card on the **Overview** tab. The mode change takes effect immediately; the instance isn't restarted and existing sessions aren't dropped.

### Access Modes

| Mode | Meaning |
|---|---|
| **Public** | Reachable from the internet via the public endpoint, no source IP restriction. Default. |
| **Public, IP allowlist** | The public endpoint is only open to the specified IP/CIDR ranges. |
| **Private only** | The public endpoint is removed entirely; the instance is only reachable from the private cluster network. |

When a mode is marked **Not yet available**, the reason is written inside the card. There are two distinct reasons: the IP allowlist stays disabled when the cluster can't see the real source address (otherwise the list would silently accept everyone); **Private only** can only be selected once the application cluster has been joined to the private network — otherwise the instance would become unreachable from anywhere.

> **Warning:** Switching to **Private only** mode deletes the public connection address. Every client connecting from outside the clusters — a laptop, a GUI client, a CI job, anything not running on Komuta — is disconnected immediately and can't reconnect. Services using the private address aren't affected.

### IP Allowlist

The allowlist is filled in with one CIDR or IP per line:

```text
203.0.113.0/24
198.51.100.7
```

- A single address entered on its own is automatically expanded to a single host (`/32`).
- Any source not on the list is rejected at the gateway.
- The list can't be left empty — an empty list would cut off all access, so the request is rejected. If the goal is to close off all external access, the right choice is **Private only** mode.
- An invalid entry isn't silently ignored — it errors out, and no half-applied restriction is left behind.

### Private Address

If the instance has a private network path, its private address appears below the **Network access** card. This address is reachable from services in this cluster, and from other clusters once the private mesh is set up, and **keeps working in every mode** — it's valid even in public mode. Removing the public address doesn't affect the private one.

![Network access card](https://cdn.komuta.io/docs/tr/images/postgresql/network-access.png)

---

## Backups

### Backup Layout and Retention Period

Protection has two parts, working together:

- A **full backup** is taken once a day. The backup time is set independently per instance; not all instances are backed up at the same time.
- **WAL is archived continuously.** This is what makes point-in-time recovery (PITR) possible; protection also covers the changes between two full backups.

The default retention period is **7 days**, covering both full backups and the WAL archive. That period is also what determines the earliest point you can recover to.

Taking more than one full backup a day doesn't add any protection: every full backup is a copy of the entire dataset, and every point in between is already recoverable via WAL. If an extra copy is needed before a specific operation, one can be taken manually with **Back up now** on the **Operations** tab.

### Backup Schedule

The time of the daily backup and the retention period are changed from the **Backup schedule** card on the **Backups** tab. The time is given as a five-field cron expression (`0 3 * * *` → every day at 03:00 UTC), and the retention period is given in days.

Changing the backup time doesn't affect the retention period; the retention period only updates when it's explicitly changed.

> **Warning:** Shortening the retention period is irreversible. Once the period is lowered, backups that fall outside the new window are permanently deleted at the next maintenance run. Extending the period back out doesn't bring the deleted ones back — it takes that many days for the protection window to refill, and the earliest recoverable point moves forward at the same rate.

### Backup Health

Backup health is checked once a day, and one of three states is shown on the **Backups** tab. The card also shows the last base backup, the start of the PITR window, the last time WAL archiving was verified, and the archive size.

| State | Meaning | What to do |
|---|---|---|
| **Healthy** | The last backup and WAL stream are newer than 26 hours. | — |
| **Problem** | The check ran, but the last backup or WAL stream is older than expected. | Check disk usage; open a support ticket if it persists. |
| **Unknown** | The check hasn't run for 48 hours; nothing can be said about backup status. | Open a support ticket. "Unknown" doesn't mean healthy. |

Since the indicator is based on the daily check, a backup taken just now may not show up here right away.

### Backup Destruction

Backups are only permanently deleted in two cases:

- When a scheduled deletion's grace period expires,
- Through an admin's explicit permanent-deletion request.

No other path touches backups. If an instance goes into an error state and gets auto-cleaned, if setup stalls halfway, or if a restore is canceled, backups remain in place for the retention period.

> **Note:** Backups staying in place doesn't mean the restore can be done from the UI. Reaching the backup of an auto-cleaned instance requires opening a support ticket.

![Backups tab](https://cdn.komuta.io/docs/tr/images/postgresql/backup-page.png)

---

## Restoring

A restore is started with **Open restore** on the **Backups** or **Operations** tab. There are two options: restoring from the latest backup, or point-in-time recovery (PITR) to a specific moment within the retention period.

### Recoverable Range

Both ends of the range are bounded:

- **Going back:** at most as far as the retention period, but never before the instance's first backup. On a newly created instance, this range may still only be a few hours.
- **Going forward:** the upper bound isn't "now" — it's the **most recently archived WAL**. On a database with little write activity, the last few minutes may not be recoverable yet.

The current range is shown as **Oldest** and **Newest** on the **Restore points** card. Picking a moment outside the range errors out before the operation starts; **Validate target** lets a timestamp be tested beforehand.

### Restoring Creates a New Instance

A restore doesn't overwrite the existing instance:

1. A **new instance** is created with the data as of the selected moment.
2. The existing instance keeps running, serving traffic, and **being billed** — the two instances sit side by side for a while, and both are charged.
3. It's up to the user to point the application's connection settings at the new instance and delete the old one.

> **Warning:** The restored copy is a separate instance and gets its own connection address — its public and private addresses differ from the source's. Services still pointed at the source keep working unchanged; moving them to the copy requires manually updating their connection settings.

### Canceling an In-Progress Restore

A restore in progress can be stopped with **Cancel restore** on the **Operations** tab. The half-provisioned sibling instance is removed; the source instance and its backups aren't affected.

---

## Plan Change

The plan is changed with **Change plan** in the instance header. An impact preview runs automatically once a target plan is selected.

### Preview

The preview shows the type of change (upgrade, downgrade, topology change), the estimated downtime, resource changes, and any warnings. When the preview reports a change that can't be applied, **Apply plan** stays disabled.

> **Warning:** The preview isn't a guarantee. Capacity is re-checked at the moment of confirmation; if there's a wait after the preview opens, the cluster may fill up and the same change could be rejected.

### Constraints

- **Disk can't be shrunk.** Even when moving to a plan with a smaller disk, the current disk size is kept. This isn't a platform restriction — it's how block storage works.
- **PostgreSQL plans can't be downgraded.** Lower plans may appear in the list, but the change won't be applied.
- **Capacity is measured per node.** A replica's disk has to grow on a single physical node; even if the cluster's total free space looks sufficient, the change can't happen if that particular node is out of room.

### Downtime and High Availability

For a resource (CPU/RAM/disk) change, servers are restarted one at a time; expected downtime is roughly 30 seconds to 2 minutes, and at least one server stays up in multi-server setups. A change that alters the server count takes 2–5 minutes and causes brief connection interruptions.

On high-availability plans, replicas are forcibly spread across different physical nodes; losing a single node doesn't stop the service. Single-server plans have no such protection, and brief downtime can occur during maintenance or node changes. In both cases, the application needs to be resilient to dropped connections (reconnecting and retrying transactions).

### If the Upgrade Fails

If a plan change stalls halfway, the instance keeps running on its old plan and moves to **Upgrade Failed** state. An instance in this state is live, its data is intact, and the same change can be retried. If the problem keeps happening, a support ticket should be opened.

![Change plan panel](https://cdn.komuta.io/docs/tr/images/postgresql/change-plan.png)

---

## Maintenance and Lifecycle

Restarting, credential rotation, and deletion are all gathered on the **Operations** tab.

### Restarting

**Restart** restarts servers one at a time; plans with a replica see no downtime. It's only available on instances that are running or whose upgrade has failed.

### Password Rotation

**Rotate database password** generates a new password for the application user and **drops all existing connections.** The application gets errors until it reconnects with the new password.

- Can only be done on a running instance.
- The old password becomes invalid the moment the operation completes.
- The next rotation date is flagged as 90 days out; this is a reminder, not a requirement.

This operation should be done during a maintenance window, with the new password ready to roll out.

### Deletion

There are two paths, with different outcomes:

- **Schedule deletion** flags the instance for deletion at the end of a 7-day grace period. It can be reversed with **Cancel scheduled deletion** before the period ends; the instance keeps running and being billed throughout.
- **Permanent deletion** instantly destroys the instance and **all of its data, including backups.**

> **Warning:** Permanent deletion can't be undone, and since backups are deleted too, the data can't be recovered through support either. If the data needs to stick around a bit longer, scheduled deletion is the right choice.

---

## SQL Console

The SQL console on the **Query** tab is for running queries against this database without setting up a separate client. Table or column names can be inserted into the editor from the schema list; the editor offers completions from the live schema.

The console is **read-only by default.** Once **Write mode** is turned on, `INSERT`, `UPDATE`, `DELETE`, and DDL statements run as the database owner and can't be undone — in this mode, the console has the same privileges as a client operating on production data.

Results are capped at 1,000 rows; a result that hits the cap is marked **Truncated**. If the full result set is needed, the query should be split with `LIMIT`/`OFFSET`, or the result should be downloaded as CSV.

![Query tab](https://cdn.komuta.io/docs/tr/images/postgresql/select-query-response.png)

---

## Logs, Insights, and Monitoring

### Logs

The **Logs** tab searches across the primary, replicas, and `pgaudit` records. The search pattern is a case-insensitive substring; level (`LOG`, `WARNING`, `ERROR`, `FATAL`, `AUDIT`), time window, and row limit are selected separately. The default window is the last hour, and at most 5,000 rows are fetched at a time.

When a result is cut off by the limit, a note says so; in that case, narrow the window or raise the limit. If longer-term retention is needed, logs should be shipped to your own side — this screen isn't a long-term log archive.

### Insights

The **Insights** tab does two things:

- The **Slow queries** list comes from `pg_stat_statements` counters and can be sorted by total time, average time, call count, or row count. **Reset statistics** zeroes the counters out — useful for measuring the effect of adding an index or changing a query on a clean baseline. Resetting doesn't bring back historical records.
- The **Active queries** list shows statements still running past the threshold, and a query can be stopped from here with **Cancel**. The canceled query's client gets a query-cancellation error; if the query has already finished by the time the cancel request arrives, it has no effect.

The health snapshot includes database size, cache hit ratio, connection count, replication lag, and vacuum debt. These values are a point-in-time snapshot; for trends over time, see the **Monitoring** tab.

### Monitoring and Alerts

The **Monitoring** tab shows graphs of connection count, transactions per second, replication lag, cache hit ratio, and disk usage over a selected time range.

Monitoring rules are set up automatically when the instance is created; no additional configuration is needed. Topics covered for PostgreSQL: disk fill, connection saturation, replication lag, WAL archiving failures, stale backups, and lock contention rate. Triggered alerts are listed on the **Alerts** tab.

---

## Importing from a Dump File

**Import from URL** on the **Operations** tab restores a `pg_dump` output into this instance. This is not a merge operation.

> **Warning:** Once the import is confirmed, the target application database is dropped and rebuilt from the dump. All existing data is permanently deleted, and this can't be undone. If the target database isn't empty, confirmation requires typing the instance name.

### Source

The source is an HTTPS URL pointing to the dump file. Only HTTPS is accepted, and private network addresses are rejected. Compatible sources:

- Heroku Postgres backup URLs (`heroku pg:backups:url`)
- Pre-signed S3 / GCS / B2 URLs
- `pg_dump` custom format (`-Fc`) output

### Compatibility Report

Once a URL is given, a pre-check runs first: the dump is downloaded and inspected, and the source is compared against the target — versions, schemas, extensions, table counts, and any data already sitting in the target. The report produces two kinds of findings:

- **Blocking** findings reject the import; confirmation doesn't open.
- **Warnings** allow the operation to proceed, but leave the decision to the user.

Some extensions referenced in the dump but missing from the target can be excluded from the restore. The report lists these with a checkbox; left checked, the restore completes by skipping that extension — unchecked, the import fails explicitly because of that extension.

### Restore and Afterward

After confirmation, the restore runs and its progress can be tracked; an in-progress operation can be canceled, and a failed one can be retried with the same source. Previous imports into the same instance sit in the history list on the **Operations** tab and can be reopened from there.

Once the import completes, the new state of the database is protected by the next scheduled backup. If a protection point is needed the same day, a manual backup should be taken with **Back up now**.

---

## Frequently Asked Questions

### Can superuser access be granted?

No. The user provided has full privileges on the database, but the server's admin account stays on the platform side. Server parameter changes or extensions not on the list are evaluated through support.

### Can the connection limit be raised separately?

No. The connection count is computed from the plan's vCPU; the only way to raise it is to move to a plan with a higher vCPU count.

### Does restoring break the existing instance?

No. The restore doesn't touch the source — it creates a new instance alongside it. Switching traffic to the copy and deleting the instance that's no longer needed is up to the user; as long as both instances sit side by side, both are billed.

### Do backups get deleted along with the instance?

**Permanent deletion** destroys the backups too, and it can't be undone. **Schedule deletion**, on the other hand, leaves both the instance and its backups in place for the 7-day grace period, and can be canceled.
