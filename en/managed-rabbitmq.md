# RabbitMQ

RabbitMQ is a message queue service that Komuta provisions and operates. Server setup, TLS, the management UI, daily backups, and monitoring rules are all in place the moment the instance is created; the only thing left for the application to do is use the connection address it's given.

---

## What Is RabbitMQ?

RabbitMQ is a broker that lets two pieces of an application communicate without waiting on each other. A publisher drops a message on the broker and moves on; the message waits in a queue; a consumer picks it up and processes it when it's ready. This way the publisher's speed isn't tied to the consumer's speed, and no message is lost even if the consumer is down for a while.

### The Path a Message Takes

A message isn't published directly to a queue — it's published to an **exchange**. The exchange decides which queue (or queues) to place the message in based on **binding** rules and the message's routing key. A queue holds messages in order, consumers read from the queue, and an acknowledged message is removed; an unacknowledged message goes back to the queue if the consumer drops.

This separation makes it possible for the same message to be distributed to a single consumer as one job, or copied to multiple queues at once — the publisher's code is the same either way; only the binding rule changes.

### What It's Used For

- **Background jobs.** Long-running work like video transcoding, report generation, or sending email is processed from a queue instead of inside the request; the user isn't kept waiting.
- **Decoupling services.** A service publishes an event without knowing who's listening. Adding a new consumer requires no change on the publisher's side.
- **Load balancing.** When multiple consumers read the same queue, work is split between them; adding more consumers increases processing capacity.
- **Smoothing traffic spikes.** A sudden burst of requests piles up in the queue and the backend consumes it at its own pace, instead of hitting the database or an external API with a flood of requests.
- **Retries.** A failed job stays in the queue because it was never acknowledged; when the consumer comes back up, the job isn't lost.

RabbitMQ is a transport layer, not a data store. This distinction becomes concrete in backups: a backup covers configuration only, not the messages sitting in queues (see [Backups](#backups)).

---

## How It Works

When an instance is created, Komuta provisions a RabbitMQ cluster in the selected region, enables TLS, generates application and management credentials, publishes the management UI, and sets up daily backups and monitoring rules. None of these steps require any additional configuration.

### Instance, Plan, and Region

An **instance** is an independent RabbitMQ broker with its own credentials and its own backup chain. CPU, memory, disk, and node count all come from the **plan** it's on; the plan's memory also determines the threshold at which the broker starts throttling publishers (see [Broker Limits](#broker-limits)).

The **region** is where the instance physically runs, and it can't be changed after creation. Moving to another region means setting up a new instance.

The **version** is the RabbitMQ version the instance will run, chosen from the versions supported by the selected plan. It can't be changed later.

The **instance name** starts with a lowercase letter, contains only lowercase letters, digits, and hyphens, and is at most 40 characters; it's used as the resource name within the cluster. An instance can optionally be attached to a **project**; if it isn't, it's treated as shared.

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

---

## Connecting

### Connection Address and TLS

Every instance is given a hostname, and connections must use that name **as-is**. Resolving the address to an IP, or pinning it to an IP in a connection pool, will prevent the connection from being established.

Connections are made over TLS: the scheme is `amqps`, the port is `5671`.

### Virtual Host

Queues, exchanges, and bindings live inside a **virtual host** (vhost); the one that ships with the instance is `/`. This part comes through correctly when the connection address is copied from the UI, but if the address is typed by hand, the `/` character needs to be URI-encoded as `%2F`:

```text
amqps://app:<password>@<host>:5671/%2F
```

> **Note:** Skipping this encoding is the most common RabbitMQ connection mistake. Even with correct credentials, the connection is rejected with an unauthorized virtual host error.

### Users and Permissions

Every instance ships with two users, and their jobs are separate:

| User | Purpose | Permissions |
|---|---|---|
| **app** | Application connections | Full read, write, and configure permissions on the `/` virtual host. Can't log in to the management UI. |
| **admin** | Management UI and operations | Full permissions; can log in to the management UI. |

Applications connect with the `app` user. Putting the `admin` account into application configuration means distributing a credential that can delete queues and policies to application servers; it shouldn't be used outside the management UI.

### Viewing Connection Info

Passwords are stored encrypted on the platform side and are only decrypted when the connection info is viewed. **Every view is written to the audit log with the instance, user, and time.** Rather than sharing credentials, it's recommended to give team members access from their own accounts.

RabbitMQ has no credential rotation operation; if the password needs to change, a support ticket should be opened.

![Overview tab — connection info and management UI section](https://cdn.komuta.io/docs/tr/images/rabbitmq/dashboard.png)

---

## Management UI

RabbitMQ's own management UI is published alongside every instance, and it sits at a **different address** from the data connection: `.dash` is inserted right after the first segment of the connection address. If the data address is `rmq-a1b2c3d4.example.com`, the management UI is at:

```text
https://rmq-a1b2c3d4.dash.example.com
```

Login is with the `admin` user. From the UI you can monitor queues, connections, and consumers, create queue and exchange definitions, and export existing definitions as a file.

Exporting definitions isn't part of the backup process, but keeping your own copy of the configuration on hand makes it possible to rebuild without waiting on support if a restore is ever needed (see [Backups](#backups)).

---

## Queue Durability

Choosing a multi-node plan doesn't make queues durable on its own. A queue that isn't declared as **quorum** type lives on a single node only; if that node is lost, the queue and its messages are lost with it. The plan provides high availability — the application chooses the queue type.

### Quorum Queues

Every queue that needs to be durable is declared with the `x-queue-type=quorum` argument. A quorum queue replicates its messages across multiple nodes; the queue survives a node loss.

- Quorum queues must be `durable`; they can't be `exclusive` or `auto-delete`.
- An existing queue's type can't be changed later. Moving a classic queue to quorum means creating a new queue and migrating consumers to it.
- Short-lived reply (RPC reply) queues are left classic; they aren't expected to be durable.

### Why the Default Type Isn't Changed

The cluster-wide default queue type isn't made quorum, because doing so would break clients that open temporary (`exclusive` / `auto-delete`) queues — the reply queues used by libraries like MassTransit and Celery are an example. This is why durability is chosen per queue by the application, not cluster-wide.

---

## Broker Limits

The broker enforces memory and disk thresholds to protect itself. When a threshold is hit, the broker holds back the publisher instead of crashing — so a blocked publish is a normal operating condition, and the application's timeout and retry behavior needs to account for it.

| Limit | Value | When exceeded |
|---|---|---|
| **Memory threshold** | 80% of plan memory | The broker throttles publishers (flow control); publishing stops until consumption catches up. |
| **Free disk threshold** | 1.5x memory size | The disk alarm triggers and publishing is blocked. |
| **Concurrent connections** | 5,000 | New connections are rejected. |

The usual cause of hitting the memory threshold is queue buildup: when consumers fall behind the publish rate, the waiting messages take up memory. That's why queue buildup and consumer-less queue alerts are early warning signs that come before the memory alert.

In multi-node setups, if a network partition occurs, the minority-side nodes pause themselves (`pause_minority`); the majority side keeps serving. Clients connected to a paused node get a connection error and need to reconnect.

---

## Network Access

On RabbitMQ, all connections — including application traffic — go through the public connection address; the only way to narrow access is an IP list. Two modes are available.

### Public

The public connection address is reachable from the internet with no source IP restriction; anyone with the credentials can connect. A new instance starts in this mode. A mode change takes effect immediately; the instance isn't restarted and existing connections aren't dropped.

### Restricted by IP List

The public address stays open, but only connections from listed sources are accepted; any source not on the list is rejected at the gateway — the connection never reaches the broker.

List rules:

- One IP or CIDR per line.
- A single address (`203.0.113.4`) is automatically expanded to a single host (`/32`).
- An invalid entry isn't silently ignored — it errors out.
- The list can't be left empty. An empty list would reject everyone, so the request is turned away.

This mode isn't available on every cluster: on some clusters the gateway can't see the real source address, so the list wouldn't actually restrict anything — the mode is kept disabled there rather than leaving behind a "restriction" that accepts everyone. The request is rejected outright rather than leaving a half-applied restriction in place.

> **Note:** RabbitMQ doesn't have the private-network-only mode available on PostgreSQL and Valkey. RabbitMQ's private address only resolves inside the broker cluster, not wherever the applications run; closing off the public path would make the instance unreachable from anywhere.

![Network access card — IP list mode selected](https://cdn.komuta.io/docs/tr/images/rabbitmq/network-access.png)

---

## Backups

A backup is taken once a day and kept in object storage. The backup covers **configuration only**: exchanges, queue definitions, bindings, users, and policies.

> **Warning:** Messages sitting in queues aren't part of the backup and can't be recovered in a disaster. A backup lets the broker be rebuilt; it doesn't bring back the messages inside it.

### Not Losing Messages

Message durability is the broker's and the application's job, not the backup's. Four things need to be true together for messages that can't afford to be lost:

- Queues are declared as quorum type (see [Queue Durability](#queue-durability)).
- Messages are published as `persistent`.
- Publisher confirms are used; a message without a confirmation isn't considered published.
- Messages whose processing is critical are also recorded in the application's own database.

### Schedule and Retention Period

- The backup time is given as a five-field cron expression — `0 3 * * *` means every day at 03:00 UTC.
- The retention period is given in days and can be between 1 and 365 days; the default is 7 days.
- A manual backup can be taken outside the scheduled one. This doesn't replace the daily backup; it produces an extra copy alongside it. It's the right move when a protection point is needed right before a major change to queue and policy definitions.

Changing the backup time doesn't affect the retention period; the retention period only updates when it's explicitly changed. The last backup being older than 26 hours is flagged separately — that means the daily schedule has fallen behind.

> **Warning:** Shortening the retention period is irreversible. Once the period is lowered, backups that fall outside the new window are permanently deleted at the next maintenance run. Extending the period back out doesn't bring the deleted ones back — it takes that many days for the protection window to refill.

### Backup Destruction and Restore

Backups are only permanently deleted in two cases:

- When a scheduled deletion's grace period expires,
- Through an admin's explicit permanent-deletion request.

No other path touches backups. If an instance goes into an error state and gets auto-cleaned, or setup stalls halfway, backups remain in place for the retention period.

> **Note:** Backups staying in place doesn't mean the restore can be done from the UI. Restoring definitions requires opening a support ticket. The definition file exported from the management UI is the copy that lets you rebuild without that wait.

![Backups tab — schedule and retention period](https://cdn.komuta.io/docs/tr/images/rabbitmq/backup-page.png)

---

## Plan Change

A plan change moves CPU, memory, disk, and node count to the target plan's values. Since memory changes, the threshold at which the broker throttles publishers changes proportionally too (see [Broker Limits](#broker-limits)).

Once a target plan is selected, an impact preview is calculated: the type of change (upgrade, downgrade, topology change), estimated downtime, resource changes, and any warnings. When the preview reports a change that can't be applied, the operation can't be started.

### Constraints

- **Disk can't be shrunk.** This isn't a platform restriction — it's how block storage works.
- **Downgrading is possible.** The tradeoff is that the memory threshold drops, and the broker enters flow control sooner.
- **Capacity is measured per node.** A node's disk has to grow on a single physical machine; even if the cluster's total free space looks sufficient, the change can't happen if that particular machine is out of room.
- **The preview isn't a guarantee.** Capacity is re-checked at the moment of confirmation; if there's a wait after the preview is calculated, the cluster may fill up and the same change could be rejected.

### Downtime and High Availability

For a resource (CPU/RAM/disk) change, nodes are restarted one at a time; expected downtime is roughly 30 seconds to 2 minutes, and at least one node stays up in multi-node setups. A change that alters node count takes 2–5 minutes and causes brief connection interruptions.

On high-availability plans, nodes are forcibly spread across different physical machines; losing a single machine doesn't stop the service. For this protection to carry over to queues, the queues need to be quorum type. Single-node plans have no such protection, and brief downtime can occur during maintenance or node changes.

In both cases, the application needs to be resilient to dropped connections: reconnecting, reopening channels, and republishing unacknowledged messages.

### If the Upgrade Fails

If a plan change stalls halfway, the instance keeps running on its old plan and moves to **Upgrade Failed** state. An instance in this state is live, its data is intact, and the same change can be retried. If the problem keeps happening, a support ticket should be opened.

![Change plan panel — impact preview](https://cdn.komuta.io/docs/tr/images/rabbitmq/change-plan.png)

---

## Maintenance and Lifecycle

### Restarting

Nodes are restarted one at a time; in multi-node plans the broker keeps serving, but clients connected to the node being restarted drop and reconnect. On a single-node plan, the instance briefly stops accepting connections.

### Suspending and Resuming

There are two forms of suspension, and the difference between them is recovery time:

- **Soft suspend** drains client connections but keeps the nodes running. Resuming takes seconds.
- **Hard suspend** stops the nodes entirely. Resuming takes a few minutes because the cluster has to reschedule.

In both cases, a suspended instance doesn't accept connections; the application gets a connection error. Publishers can't drop messages during this time — suspension closes access, it doesn't drain the queue.

### Deletion

There are two paths, with different outcomes:

- **Schedule deletion** flags the instance for deletion at the end of a 7-day grace period. It can be canceled before the period ends; the instance keeps running and being billed throughout.
- **Permanent deletion** instantly destroys the instance, the messages in its queues, and **all of its configuration, including backups.**

> **Warning:** Permanent deletion can't be undone, and since backups are deleted too, definitions can't be recovered through support either. If the configuration needs to stick around a bit longer, scheduled deletion is the right choice.

---

## Metrics and Alerts

Monitoring rules are set up automatically when the instance is created; no additional configuration is needed. Topics covered for RabbitMQ: memory usage, queue buildup, consumer-less queues, disk alarm, node loss, and connection count. Triggered alerts are listed in the alert history.

The metrics that matter most for decision-making on the graphs:

| Metric | What it shows |
|---|---|
| **Queue depth** | Number of waiting messages. A steady rise means consumption capacity is falling behind the publish rate; it's also the path toward the memory threshold. |
| **Publish and delivery rate** | Messages published and delivered to consumers per second. A delivery rate below the publish rate is the cause of buildup. |
| **Connection and channel count** | Number of open clients. An unexpected rise points to a client that isn't closing connections, or that opens a new one per request, and heads toward the connection limit. |
| **Memory usage** | How close usage is to the memory threshold. Once the threshold is hit, publishing stops. |

![Monitoring tab — queue depth and publish rate graphs](https://cdn.komuta.io/docs/tr/images/rabbitmq/monitoring.png)

---

## Frequently Asked Questions

### I got a multi-node plan — are my queues durable?

Not unless the queue is declared as quorum type. A classic queue lives on a single node, and both the queue and its messages are lost if that node is lost. The plan provides high availability — the application chooses the queue type.

### Can I get my messages back from a backup?

No. A backup only covers exchanges, queue definitions, bindings, users, and policies; messages sitting in queues aren't part of it.

### Why did publishing suddenly slow down?

When the memory threshold (80% of plan memory) or the free disk threshold is hit, the broker holds back publishers. This isn't a failure — it's the broker protecting itself; publishing returns to normal on its own once consumption speeds up and queues drain.

### I'm getting an unauthorized virtual host error — are my credentials wrong?

Usually not. The virtual host is `/`, and that character needs to be encoded as `%2F` in the connection address. It comes through correctly when the address is copied from the UI; this is the most common mistake in hand-typed addresses.

### Can I connect my application with the admin user?

It'll connect, but it shouldn't. `admin` is the fully-privileged account for the management UI and operations; putting it in application configuration distributes queue- and policy-deletion permissions to application servers. Applications should connect with the `app` user.

### Do backups get deleted along with the instance?

**Permanent deletion** destroys the backups too, and it can't be undone. **Schedule deletion**, on the other hand, leaves both the instance and its backups in place for the 7-day grace period, and can be canceled.
