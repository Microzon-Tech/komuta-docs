# Valkey

Valkey is an in-memory key-value store that Komuta provisions and operates. Server setup, TLS, daily backups, and monitoring rules are all in place the moment the instance is created; the only thing left for the application to do is use the connection address it's given.

---

## What Is Valkey?

Valkey is a key-value store that keeps data in memory rather than on disk. Every record is accessed by a key, and because the operation happens directly in memory, response times stay under a millisecond — where a relational database has to read an index and go to disk, Valkey answers in a single step. The tradeoff is capacity: data is bounded by memory, not disk (see [Available Memory](#available-memory)).

### What It Can Hold

A key's value isn't limited to plain text. Valkey natively holds strings, numbers, hashes (field-value pairs), lists, sets, sorted sets, bitmaps, HyperLogLog, and streams.

Operations on these structures happen server-side: adding an element to a list or incrementing a counter doesn't require pulling the data out and writing it back — a single command does it, and the command is atomic, meaning two concurrent requests can't corrupt each other's result. Every key can also be given a lifetime (TTL); a key past its TTL is deleted on its own.

### What It's Used For

- **Caching.** The result of an expensive query or an external API call is written to a key with a TTL; subsequent requests never hit the database. This is the most common use.
- **Session store.** Session data lives in a shared location instead of the application's own memory, so a request lands on the same session no matter which app instance it hits, and sessions aren't lost during a rollout.
- **Queues and job lists.** Lists and streams are used to queue up work to be processed in the background.
- **Counters and rate limiting.** Atomic increment combined with a TTL lets counters like "how many requests from this IP in the last minute" be kept with a single command.
- **Pub/sub.** A message published to a channel is delivered to every client listening on it; used for things like real-time notifications and cache invalidation.

Whichever of these uses is chosen, what happens once memory fills up depends on the [eviction policy](#eviction-policy) — and the right policy depends on the use case.

### Relationship to Redis

Valkey is a fork that split off from Redis 7.2.4 in March 2024. When Redis's license moved to a non-open-source model, the project's long-time contributors founded Valkey under the Linux Foundation to keep the codebase going under the BSD 3-Clause license; AWS, Google Cloud, Oracle, Ericsson, and Snap are among the organizations backing the project.

In practice, this means the protocol and command set are preserved. Existing Redis clients, drivers, and tools — `redis-cli`, ioredis, `redis-py`, Spring Data Redis — need no changes to connect to Valkey; the only things that change are the connection address and credentials (see [Connecting](#connecting)).

---

## How It Works

When an instance is created, Komuta provisions a Valkey cluster in the selected region, enables TLS, generates connection credentials, and sets up daily backups and monitoring rules. None of these steps require any additional configuration.

### Instance, Plan, and Region

An **instance** is an independent Valkey server with its own memory, its own credentials, and its own backup chain. CPU, memory, disk, and server (replica) count all come from the **plan** it's on. The plan's memory isn't just a price difference — it's the value that determines how much room your data has (see [Available Memory](#available-memory)).

The **region** is where the instance physically runs, and it can't be changed after creation. Moving to another region means setting up a new instance.

The **version** is the Valkey version the instance will run, chosen from the versions supported by the selected plan. It can't be changed later.

The **instance name** starts with a lowercase letter, contains only lowercase letters, digits, and hyphens, and is at most 40 characters; it's used as the resource name within the cluster. An instance can optionally be attached to a **project**; if it isn't, it's treated as shared.

### Where the Data Lives

Data is kept in memory and written to disk as a snapshot. Because of this, part of the plan's memory isn't available to your data — taking a snapshot creates a temporary copy, and room for that copy is reserved in advance.

### Instance States

| State | Meaning |
|---|---|
| **Provisioning** | Setup is in progress; connection info isn't ready yet. |
| **Active** | Running and accepting connections. |
| **Suspended** / **Fully Suspended** | Manually suspended, not accepting connections. |
| **Upgrading** | A plan change is being applied. |
| **Upgrade Failed** | The plan change couldn't complete; the instance is **live** on its old plan and its data is intact. |
| **Pending Deletion** | Deletion has been scheduled; it can still be canceled before the window expires. |
| **Error** | An instance whose setup couldn't complete. Instances in this state are automatically cleaned up by the platform; their backups remain in place for the retention period. |

![Valkey list](https://cdn.komuta.io/docs/tr/images/valkey/home-page.png)

---

## Available Memory

The memory available to data is **75% of the plan's total memory.** The rest is reserved for the server's own needs: memory fragmentation, the replication buffer, and the temporary copy made while taking a snapshot. Without that reserve, the server would blow past its memory limit while snapshotting and crash.

| Plan memory | Available for data |
|---|---|
| 1 GB | ~768 MB |
| 2 GB | ~1.5 GB |
| 4 GB | ~3 GB |
| 8 GB | ~6 GB |

What happens once the limit is reached is determined by the instance's eviction policy; one of two behaviors applies (see [Eviction Policy](#eviction-policy)).

---

## Eviction Policy

The eviction policy determines what Valkey does with new writes once the memory limit is reached. The policy is set when the instance is created and isn't a setting that can be changed afterward. There are two behaviors, and here's what each does once the limit is hit.

### No Eviction (default)

New write commands return an error and **no key is deleted.** The data in memory stays exactly as it is, reads keep working, and the server doesn't crash since the limit itself isn't being exceeded in a way that breaks anything. This is the right behavior for uses that can't afford to lose data — queues, session stores, job lists. The tradeoff is that the application has to handle this write error: a failed write doesn't silently vanish, but it also isn't retried automatically.

### Eviction Enabled

The least-used or soon-to-expire keys are deleted and **writes keep going without interruption.** The application never notices the memory limit; the tradeoff is that every new write at the limit takes the place of an older key. This suits caching, because a deleted key can just be regenerated from its source.

> **Tip:** If it's going to be used as a cache, the policy should be set to eviction from the start. Starting with "no eviction" and letting memory fill up just means the application hits a write error at a moment it didn't expect.

---

## Network Access

Every instance runs in one of three access modes. A mode change takes effect immediately; the instance isn't restarted. Here's what each of the three modes does.

### Public

The public connection address is reachable from the internet with no source IP restriction; anyone with the credentials can connect. A new instance starts in this mode.

### Restricted by IP List

The public address stays open, but only connections from listed sources are accepted; any source not on the list is rejected at the gateway — the connection never reaches the instance.

List rules:

- One IP or CIDR per line.
- A single address (`203.0.113.4`) is automatically expanded to a single host (`/32`).
- An invalid entry isn't silently ignored — it errors out.
- The list can't be left empty. An empty list would reject everyone, so the request is turned away; the way to close off access entirely is private network mode.

This mode isn't available on every cluster: on some clusters the gateway can't see the real source address, so the list wouldn't actually restrict anything — the mode is kept disabled there rather than leaving behind a "restriction" that accepts everyone. In that case, the way to narrow access is private network mode.

### Private Network Only

The public connection address is removed entirely; the instance is only reachable from your own services on the private network. This mode can only be selected once your application cluster has been joined to the private network — otherwise the instance would become unreachable from anywhere, so the request is rejected.

> **Warning:** The moment you switch to private network mode, the public address is deleted. Every client connecting from outside your clusters — a GUI client on your machine, a CI job, anything not running on Komuta — drops immediately and can't reconnect. Services using the private address aren't affected.

![Network access card](https://cdn.komuta.io/docs/tr/images/valkey/network-access.png)

---

## Connecting

### Connection Address and TLS

Every instance is given a hostname, and connections must use that name **as-is**: the infrastructure that routes traffic to the correct instance decides based on the server name inside the TLS handshake.

- Resolving the address to an IP and connecting to that won't work. A connection made by IP doesn't reach the right instance; it typically appears to succeed and then drops immediately.
- Older clients that don't send the server name (SNI), and connection pools that pin the address to an IP, fail for the same reason.

With TLS enabled, the connection uses the `rediss://` scheme; the default port is `6379`. Tools like `redis-cli` need TLS turned on explicitly (`--tls`).

### Credentials

Connection info consists of host, port, username, and password; Redis-compatible clients authenticate with that password.

Passwords are stored encrypted on the platform side and are only decrypted when the connection info is viewed. **Every view is written to the audit log with the instance, user, and time.** Rather than sharing credentials, it's recommended to give team members access from their own accounts.

Valkey has no credential rotation operation; if the password needs to change, a support ticket should be opened.

### Private Network Address

Instances with a private network path get a private address alongside the public one. This address is reachable from services in the cluster and **is independent of the access mode** — it's the path that stays up even when the public address is removed. The private address presents the instance's own certificate and doesn't expect server verification from the client.

![Overview tab](https://cdn.komuta.io/docs/tr/images/valkey/dashboard.png)

---

## Backups

A full backup is taken once a day and kept in object storage. The backup is a copy of the in-memory contents at the moment it was taken; changes between two backups aren't captured.

### Schedule and Retention Period

- The backup time is given as a five-field cron expression — `0 3 * * *` means every day at 03:00 UTC.
- The retention period is given in days and can be between 1 and 365 days; the default is 7 days.
- A manual backup can be taken outside the scheduled one. This doesn't replace the daily backup; it produces an extra copy alongside it. It's the right move when a protection point is needed right before a risky operation.

Changing the backup time doesn't affect the retention period; the retention period only updates when it's explicitly changed. The last backup being older than 26 hours is flagged separately — that means the daily schedule has fallen behind.

> **Warning:** Shortening the retention period is irreversible. Once the period is lowered, backups that fall outside the new window are permanently deleted at the next maintenance run. Extending the period back out doesn't bring the deleted ones back — it takes that many days for the protection window to refill.

### Backup Destruction

Backups are only permanently deleted in two cases:

- When a scheduled deletion's grace period expires,
- Through an admin's explicit permanent-deletion request.

No other path touches backups. If an instance goes into an error state and gets auto-cleaned, or setup stalls halfway, backups remain in place for the retention period.

> **Note:** Backups staying in place doesn't mean the restore can be done from the UI. Reaching a Valkey backup requires opening a support ticket.

---

## Plan Change

A plan change moves CPU, memory, disk, and server count to the target plan's values. Since memory changes, the space available to data changes proportionally too (see [Available Memory](#available-memory)).

Once a target plan is selected, an impact preview is calculated: the type of change (upgrade, downgrade, topology change), estimated downtime, resource changes, and any warnings. When the preview reports a change that can't be applied, the operation can't be started.

### Constraints

- **Disk can't be shrunk.** On Valkey, a request to move to a plan with a smaller disk is rejected. This isn't a platform restriction — it's how block storage works.
- **Downgrading is possible, but subject to a memory check.** If the data in memory doesn't fit the target plan's data share, the change won't be applied.
- **Capacity is measured per node.** A replica's disk has to grow on a single physical node; even if the cluster's total free space looks sufficient, the change can't happen if that particular node is out of room.
- **The preview isn't a guarantee.** Capacity is re-checked at the moment of confirmation; if there's a wait after the preview is calculated, the cluster may fill up and the same change could be rejected.

### Downtime and High Availability

For a resource (CPU/RAM/disk) change, servers are restarted one at a time; expected downtime is roughly 30 seconds to 2 minutes, and at least one server stays up in multi-server setups. A change that alters the server count takes 2–5 minutes and causes brief connection interruptions.

On high-availability plans, replicas are forcibly spread across different physical nodes; losing a single node doesn't stop the service. Single-server plans have no such protection, and brief downtime can occur during maintenance or node changes. In both cases, the application needs to be resilient to dropped connections — reconnecting and retrying the request.

### If the Upgrade Fails

If a plan change stalls halfway, the instance keeps running on its old plan and moves to **Upgrade Failed** state. An instance in this state is live, its data is intact, and the same change can be retried. If the problem keeps happening, a support ticket should be opened.

![Change plan panel](https://cdn.komuta.io/docs/tr/images/valkey/change-plan.png)

---

## Maintenance and Lifecycle

### Restarting

Servers are restarted one at a time; plans with a replica see no downtime. On a single-server plan, the instance briefly stops accepting connections.

### Suspending and Resuming

There are two forms of suspension, and the difference between them is recovery time:

- **Soft suspend** drains client connections but keeps the servers running. Resuming takes seconds.
- **Hard suspend** stops the servers entirely. Resuming takes a few minutes because the cluster has to reschedule.

In both cases, a suspended instance doesn't accept connections; the application gets a connection error.

### Deletion

There are two paths, with different outcomes:

- **Schedule deletion** flags the instance for deletion at the end of a 7-day grace period. It can be canceled before the period ends; the instance keeps running and being billed throughout.
- **Permanent deletion** instantly destroys the instance and **all of its data, including backups.**

> **Warning:** Permanent deletion can't be undone, and since backups are deleted too, the data can't be recovered through support either. If the data needs to stick around a bit longer, scheduled deletion is the right choice.

---

## Metrics and Alerts

Monitoring rules are set up automatically when the instance is created; no additional configuration is needed. Topics covered for Valkey: memory usage, persistence (snapshot) failures, client count, CPU, and instance reachability. Triggered alerts are listed in the alert history.

Three metrics on the graphs matter most for decision-making:

| Metric | What it shows |
|---|---|
| **Memory usage** | How close usage is to the data share. Once the limit is reached, the eviction policy kicks in. |
| **Eviction rate** | Above zero means the instance is running at its memory limit and keys are being deleted. Under a "no eviction" policy, this value never rises — instead, the application gets write errors. |
| **Hit rate** | What fraction of queried keys are found in memory. A drop points to data being evicted, or to key lifetimes being too short. |

---

## Frequently Asked Questions

### Will my Redis clients work with Valkey?

Yes. Valkey speaks a Redis-compatible protocol; clients like `redis-cli`, ioredis, and `redis-py` need no changes. Since TLS is enabled, the connection scheme is `rediss://`, not `redis://`.

### Can I use all of the plan's memory for data?

No. 75% of the plan's memory is available to data; the rest is for memory fragmentation, the replication buffer, and the snapshot copy. On a 4 GB plan, there's roughly 3 GB available for data.

### Will I lose data when memory fills up?

Depends on the eviction policy. Under the default "no eviction" behavior, no key is deleted and write commands return an error. On an instance with eviction enabled, the least-used keys are deleted and writes keep going.

### Does changing the access mode drop connections?

Switching between public and IP-list modes doesn't restart the instance. Switching to private network mode removes the public address, so it immediately drops every client connecting from outside; services using the private address aren't affected.

### Do backups get deleted along with the instance?

**Permanent deletion** destroys the backups too, and it can't be undone. **Schedule deletion**, on the other hand, leaves both the instance and its backups in place for the 7-day grace period, and can be canceled.
