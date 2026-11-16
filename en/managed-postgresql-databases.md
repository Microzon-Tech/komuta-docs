# Multiple Databases in PostgreSQL

Inside a PostgreSQL instance, more databases can be opened next to the main database the instance was created with. Each extra database has its own user and password, and that user can reach only its own database. Extra databases share the instance's plan, backups and connections, so applications that should be kept apart can each get a database of their own without a new instance for each.

> **Note:** Extra databases are a feature turned on per organization. If the **Databases** card doesn't appear on the **Overview** tab while the instance is running, the feature isn't on for your organization; open a [support ticket](support-tickets.md) to have it turned on. If the feature is turned off later, the databases already opened keep working; only new ones can't be opened. The card then says "Creating more databases is not available for your organization. The existing ones keep working."

---

## How It Works

Each extra database is opened together with a user of the same name, and that user owns the database: a database opened as `reports` has the username `reports`. Komuta generates the password.

The databases are closed to each other:

- An extra database's user can connect only to its own database; it can connect neither to the main database nor to the other extra databases.
- The main database's user can't connect to the extra databases.

Like the main database's user, an extra database's user is a fully privileged application user in its own database, not the server's superuser (see [The User You're Given](managed-postgresql.md#the-user-youre-given)). The [pre-installed extensions](managed-postgresql.md#pre-installed-extensions) come installed in extra databases too.

### What's separate, what's shared

- **Each database's own:** its data, its username and password, its read-only users, and its query statistics on the **Insights** tab.
- **Shared by the instance:** the plan (CPU, memory and disk), the connection address and port, network access (access mode and IP allowlist), server connections, backups and restores, query limits, monitoring and alerts.

Because the disk is shared, all databases switch to read-only mode together when disk usage reaches the threshold (see [Disk Layout and Fill State](managed-postgresql.md#disk-layout-and-fill-state)).

Extra databases aren't billed separately; the instance's plan and price stay the same.

### How many databases can be opened?

An instance holds at most **5** databases, the main one included. The counter in the header of the **Databases** card (for example `2 / 5`) shows how many are open and the limit. The limit is worked out from the plan and is 5 on every plan today. Once the limit is reached, one has to be deleted before a new one can be opened.

### How connections are shared

While there's only one database, the instance's server connections belong entirely to the main database (see [Connection and Memory Limits](managed-postgresql.md#connection-and-memory-limits)). When a second database is opened, Komuta splits these connections evenly between the databases and gives every database's connection pool the same upper limit. A busy database can't use up the others' connections this way: once a database reaches its limit, its new transactions wait in the pool until one of its own connections frees up.

Each database's share gets smaller as databases are opened and larger as they're deleted. The card shows each database's current share, and the create dialog shows what the share drops to with the new database. On single-server plans, the most server connections one database can use at once:

| Plan RAM | 2 databases | 3 databases | 4 databases | 5 databases |
|---|---|---|---|---|
| 512 MB | 18 | 12 | 9 | 7 |
| 1 GB | 23 | 15 | 11 | 9 |
| 2 GB and above | 43 | 28 | 21 | 17 |

On high-availability plans the connection pool is split across two servers, so the values can be up to two connections lower; the card shows the exact number. The numbers in the table are connections on the database side. The limit on client connections that applications can open isn't split; the clients of all databases share it. When the plan changes, the shares are worked out again from the new plan's connections.

> **Warning:** The moment the first extra database is opened, the main database's share gets smaller too; on a 2 GB single-server plan, for example, the main database can then use at most 43 server connections at once. If the main database runs many transactions at the same time, take this into account before opening the first extra database.

---

## Opening a Database

1. On the instance's **Overview** tab, click **Create database** on the **Databases** card.
2. Type the database's name in the **Name** field. The database's user gets the same name, and it can't be changed later.
3. The note at the bottom of the dialog shows the most connections each database can use at once with this database added. Click **Create**.

Once the database is open, its connection details are shown in a dialog (see [Connecting](#connecting)).

To open a database:

- The instance must be **Active**. Otherwise the card says "A new database can be created while the service is running."
- No dump import may be running on the instance.
- You need **Edit** permission on the instance. Everyone with **Read** permission sees the list and the databases' sizes; showing connection details, resetting the password and deleting also need **Edit** (see [Organizations and Team Members](organizations-team.md)).

### Naming rules

- 2–31 characters; only lowercase letters, digits and underscores (`_`).
- The first character must be a letter, the last a letter or a digit.
- A name can't be used if a database or user with that name already exists on the instance, the main database's name and username included: "A database or user with this name already exists on this instance. Choose another name."
- PostgreSQL's and the platform's own names can't be used: `postgres`, `template0`, `template1`, `app`, `public`, `all`, `user`, `none`, `replication`, `samerole`, `samegroup`, `sameuser`, `current_user`, `current_role`, `session_user`, `streaming_replica`, nor names that start with a platform prefix such as `pg_`, `cnpg_`, `komuta_` and `documentdb`. The dialog then says "This name is reserved by the platform. Choose another one."

These rules are narrower than the main database's naming rule (see [Name, Version, and Database](managed-postgresql.md#2-name-version-and-database)).

---

## Connecting

When the database is opened, or when its row's **Connection** button is clicked, the host, port, database name, username and password are shown along with ready-made connection strings. The host and port are the same as the main database's; the database name and the username are the extra database's name. If the instance has a private network path, the private network address is among the connection strings too.

An extra database's password is stored, so its connection details can be shown again from the list at any time. The connection address and encryption rules are the same as for the main database (see [Connection Address](managed-postgresql.md#connection-address)).

---

## Resetting the Password

**Reset password** on the database's row generates a new password for its user and shows the new connection details.

- Applications that connect with the old password can't open new connections until they get the new one. Connections that are already open stay open.
- The reset is refused if a Stack's services use this database (see [Connecting from a Stack](#connecting-from-a-stack)).
- The password can't be reset while a dump import is running on the instance.

The main database's **Rotate database password** doesn't change the extra databases' passwords; each extra database's password is reset from its own row. That operation restarts the connection pool, though, so the extra databases' open connections drop as well; since their passwords haven't changed, the applications connected to them can reconnect with their own passwords (see [Password Rotation](managed-postgresql.md#password-rotation)).

---

## Deleting

Click **Delete** on the database's row, type the database's name in the dialog and click **Delete database**. The database, its data, its user and its read-only users are deleted, and every client connected to this database is cut off at once. The query statistics collected for this database on the **Insights** tab are deleted too, and the database's connection share goes back to the remaining databases.

To delete:

- The instance must have a backup completed within the last 26 hours. The only way to bring a deleted database back is restoring from a backup taken before the deletion. Without one, the message "No backup has completed in the last day, so this database could not be brought back after deletion. Take a backup and try again." appears: take a backup with **Back up now** on the **Operations** tab and try again once it completes.
- No Stack's services may be using this database (see [Connecting from a Stack](#connecting-from-a-stack)).
- The instance must be **Active**, **Suspended** or **Upgrade Failed**. A database can't be deleted while a plan change, restore or setup is in progress, or while a dump import is running.

The main database can't be deleted on its own; it's deleted only together with the instance (see [Deletion](managed-postgresql.md#deletion)).

> **Warning:** Deletion can't be undone. A deleted database comes back only by restoring the instance to a point before the deletion, and a restore creates a new instance (see [Backups and Restoring](#backups-and-restoring)).

---

## Backups and Restoring

Backups are per instance: the daily full backup and the continuous WAL archive cover all databases together (see [Backups](managed-postgresql.md#backups)). A single database isn't backed up separately and isn't restored on its own.

A [restore](managed-postgresql.md#restoring) creates a new instance and brings back all databases as they were at the chosen point, with their data and users. The copy's address differs from the source's; take the extra databases' connection details from the copy's own **Databases** card.

The extra databases' passwords are carried over to the copy from the source. The copy doesn't know the password of a database that no longer exists on the source, because it was deleted later, or of one whose password was reset after the point you go back to. That database's connection details fail with "The password of this database is not stored, for example after a restore. Reset the password to get a new one."; get a new password on the copy with **Reset password**.

---

## Choosing the Database

While the instance has an extra database, a **Database** picker appears on the screens below. The picker shows the main database with "(main)" after its name, and it's selected to begin with.

| Screen | What the choice does |
|---|---|
| **Query** tab | Queries and the schema list run in the chosen database. The console is still read-only by default; while **Write mode** is on, statements run as the chosen database's own user (see [SQL Console](managed-postgresql.md#sql-console)). |
| **Insights** tab | The health snapshot, slow queries, **Reset statistics** and the query plan belong to the chosen database. The **Active queries** list shows the whole instance; its **Database** column says which database each query runs in. |
| **Overview** → **Read-only access** | A user created with **Create user** reads only the chosen database. At most 3 read-only users can be open on an instance at once; this count is the total across all databases. |
| **Operations** → **Import from URL** | The dump is imported into the chosen database (see [Importing into an extra database](#importing-into-an-extra-database)). |

### Importing into an extra database

[Importing from a dump file](managed-postgresql.md#importing-from-a-dump-file) works on extra databases too:

- The chosen database is dropped and rebuilt from the dump; its existing data is permanently deleted. The other databases aren't touched.
- The imported objects are owned by that database's own user.

On every import while the instance has an extra database, even when the target is the main database:

- WAL archiving stops for the whole instance while the import runs; no database can be restored to a point inside that window. When the import ends, even if it fails, Komuta starts a new full backup; every point after that backup completes can be restored again.
- While the import runs, no database can be opened or deleted, and no password can be reset.

---

## Connecting from a Stack

A service in a Stack manifest connects to an extra database with the `database` field under `valueFrom.addon`:

```yaml
services:
  - name: reports-api
    env:
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: shop-db, property: connectionString, database: reports }
```

- Without `database`, or with the main database's name, the main database is bound.
- With the name of an extra database, `database`, `user`, `password` and `connectionString` come from that database and its user; `host` and `port` are the instance's.
- Only for PostgreSQL. The managed service can be under `addons` or `uses.addons`.
- A Stack doesn't open databases: the database must already be open in the console. An extra database of a managed service created in the same plan can't be bound; the plan refuses it with `ADDON_DATABASE_NOT_OPENED`. Apply the managed service first, open the database in the console, then add the service.
- On a service the Stack already runs, adding, changing or removing this binding can't be applied in place and blocks the plan; the database is chosen when the service is first created.

Once a Stack service is bound to an extra database, that database can't be deleted and its password can't be reset until no Stack service uses it; the console says which services use it: "Services read this database through a Stack: … Deleting it or giving it a new password would disconnect them; remove those variables from the services, or delete the services, first. Nothing was changed." This protection starts when the service's step finishes; don't delete a database or reset its password while an apply that binds it is running.

During the apply, the service's step can stop with these errors:

| Code | Meaning and what to do |
|---|---|
| `ADDON_DATABASE_NOT_FOUND` | The managed service has no database with this name. Open the database in the console, then apply again. |
| `ADDON_DATABASE_INVALID` | The name is neither the main database's name nor a valid extra database name. Fix the name in the manifest. |
| `ADDON_DATABASE_PASSWORD_UNKNOWN` | The database's password isn't stored (for example after a restore). Reset the password in the console, then apply again. The reset is refused while other Stack services use this database. |
| `ADDON_DATABASE_CHANGED` | The database was deleted or its password was reset while the service was being created; the service wasn't started and the values it holds don't work. Delete the service, then apply again. |
| `ADDON_DATABASE_NOT_READABLE` | The databases couldn't be read for a moment. The step is retried a few times on its own. |
| `ADDON_DATABASE_NOT_AVAILABLE` | The managed service's databases can't be read. Open a support ticket. |

For the manifest fields and validation messages, see [komuta.yaml Manifest Reference](stack-manifest-reference.md).

---

## Frequently Asked Questions

### Are extra databases billed separately?

No. Extra databases share the instance's plan, and the instance's price stays the same. If more CPU, memory or disk is needed, the instance's plan is upgraded (see [Plan Change](managed-postgresql.md#plan-change)).

### Can an extra database's user read the main database?

No. An extra database's user can connect only to its own database, and the main database's user can't connect to the extra databases. If a tool or a person only needs to read, create a read-only user for that database on the **Read-only access** card.

### Can I restore a single database?

Not on its own. Backups and restores are per instance: a restore creates a new instance that holds all databases as they were at the chosen point.

### Can a database be renamed?

No; the name is also the database's username. If another name is needed, a database is opened with the new name, the data is moved into it, and the old one is deleted.

### Does opening a second database affect the main database?

Opening a database doesn't restart PostgreSQL and doesn't touch the main database's data. What changes is the connection share: the server connections the main database can use at once shrink with the split (see [How connections are shared](#how-connections-are-shared)). Because the connection pool's setting changes, the pool may be renewed once at that moment; applications that can reopen a dropped connection aren't affected.
