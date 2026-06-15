# Persistent Storage

DevOpsZon services are **stateless** by default — when a pod restarts, all local files are lost. Applications that keep persistent data such as databases, file servers, upload directories or caches need a **disk plan** attached to the service. DevOpsZon meets this need with distributed block storage running on top of Longhorn.

This guide explains how to attach a PersistentVolumeClaim (PVC) to your service from the **Storage** tab in the service management panel, pick a plan, configure the mount path and understand the safety rules.

---

## How Storage Works

DevOpsZon storage has three building blocks:

| Component | Description |
|-----------|-------------|
| **Longhorn** | Distributed block storage engine running on Kubernetes. Keeps volume replicas in sync across worker nodes for high availability. |
| **Disk Plan** | The storage package you purchase. Determines size, replica count, reclaim policy and monthly price. |
| **Mount Preset** | Catalog entry that defines which container directory the volume is mounted at. You pick one of the vetted safe paths maintained by the platform. |

When you attach storage to a service:

1. A PVC is created according to the selected plan
2. Longhorn provisions the PVC with the replica count specified by the plan
3. On the next deploy the platform injects `volumes` and `volumeMounts` into the pod spec
4. Your container can read and write to the volume through the chosen mount path
5. Data survives pod restarts, recreations and node relocations

> **Note:** Longhorn always tries to place replicas on different nodes. Single-node clusters run with 1 replica; multi-node clusters are recommended to run with 3 replicas.

---

## Disk Plans

DevOpsZon offers five disk plans that target different workloads. **Every plan uses the same StorageClass (`longhorn-static-rwx`)** — the only differences are the size quota and the monthly price. This design has two practical consequences:

1. **Every plan supports `ReadWriteMany` (RWX) access mode** → whichever plan you pick, Canary / BlueGreen / AutoPromote rollout strategies, HPA, and multi-replica topologies all work out of the box.
2. **Free → Paid upgrades are lossless** → Kubernetes treats a PVC's `storageClassName` as immutable. Because every plan shares the same class, an upgrade only bumps `SizeGb`, Longhorn online-expands the volume, and your data stays on the same disk.

| Plan | Size | StorageClass | Access Mode | Reclaim | Replicas | Monthly Price | Target Workload |
|------|:----:|:------------:|:-----------:|:-------:|:--------:|:-------------:|-----------------|
| **Free** | 1 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **0.00** | Hobby projects, test environments, prototypes |
| **Small** | 10 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **1.99** | Small production workloads, content storage |
| **Medium** | 50 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **7.99** | Medium-sized databases, file/media storage |
| **Large** | 200 GB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **24.99** | Large databases, file servers, read/write intensive workloads |
| **XLarge** | 1 TB | `longhorn-static-rwx` | **RWX** | Delete | 3 | **89.99** | Enterprise data platforms, log archival |

> 💡 **Why is everything RWX?** DevOpsZon exposes Canary / BlueGreen / AutoPromote rollout strategies to every user. All three keep the old and new pods alive **at the same time** during a rollout, and both pods mount the same PVC. That forces `ReadWriteMany` semantics. The "Always RWX" architecture eliminates the risk of hitting a "Multi-Attach error" when you change tiers, enable HPA, or scale a replica set. See [Storage RWX Architecture](../../storage-rwx-architecture.md) for the full rationale.

> 🔄 **Plan upgrades without data loss**: When you move from Free to any paid plan (or from Small to Medium, etc.), the platform only raises the PVC's size request. Longhorn **expands the volume online** — no pod restart required. Your data stays on the same disk, no migration takes place.

### What Does the Reclaim Policy Mean?

**Every StorageClass DevOpsZon provisions is configured with `reclaimPolicy: Delete`**:

- `longhorn-rwx` → Free plan, 2 replicas, Delete
- `longhorn-static-rwx` → Paid plans (Small/Medium/Large/XLarge), 3 replicas, Delete

This is a deliberate platform-wide decision: **a delete is a delete.** When you remove a storage attachment or tear down a service, the PVC, its PV, the Longhorn volume, all snapshots and the S3 backup archive are all permanently destroyed. Plan tier is irrelevant — free and paid plans behave identically.

As an extra safety net, the platform runs `IServiceStoragePurgeService` as part of every delete. This service:

1. Patches the PV's reclaim policy to `Delete` (already `Delete` in the happy path; this catches externally-imported PVs that may carry `Retain`)
2. Deletes the matching Longhorn Volume, Snapshot, Backup and BackupVolume custom resources from `longhorn-system`
3. Explicitly deletes the PVC from the cluster (in the attachment-delete flow) so there is no race with ArgoCD prune semantics

> ⚠️ **Important:** Deletion is designed to be irreversible. Once an attachment or a service is deleted there is **no admin-side recovery path**. Make sure to take a manual backup of anything you care about before you delete it.

### Free Plan Limits

The Free plan is offered to every user at **no cost**, with a few limits:

- Only 1 GB of space
- **At most 1 storage attachment** per service — the same service cannot hold a second free attachment (billing quota)
- 2 replicas (tolerates 1 node loss, loses availability on 2-node loss)
- Reclaim policy `Delete` — data cannot be recovered if the volume is removed
- Backed by the `longhorn-rwx` StorageClass — RWX is available on the free tier too so hobby workloads can try Canary / BlueGreen / HPA
- Not recommended for production workloads

### Plan Tier Rule

All storage attachments on a single service must belong to **the same pricing tier**: either all free or all paid. Free and paid attachments **cannot be mixed** on the same service.

This is not a technical constraint — every plan is RWX now, so mixing them would technically work. But the platform bills free and paid plans through separate invoice cycles, and mixing them on the same service would break invoice attribution. That is why the platform rejects mixed-tier combinations.

To cross tiers:
1. Edit each existing attachment on the service → move it to the target tier (a direct upgrade works when the service only has one attachment)
2. If the service has multiple attachments: delete the existing ones and re-attach with the new tier

The UI plan picker automatically disables the opposite-tier plan cards based on what is already attached and surfaces the reason in a tooltip.

---

## Mount Preset Catalog

DevOpsZon automatically recommends the best mount path for your container image family. This prevents mounting volumes to sensitive host directories like `/etc` or `/proc`.

### Categories

| Category | Example Preset | Mount Path | Target Image |
|----------|----------------|------------|--------------|
| **Database** | PostgreSQL data | `/var/lib/postgresql/data` | postgres, timescaledb, citus |
| **Database** | MySQL data | `/var/lib/mysql` | mysql, mariadb, percona |
| **Database** | MongoDB data | `/data/db` | mongo |
| **Database** | Elasticsearch data | `/usr/share/elasticsearch/data` | elasticsearch, opensearch |
| **Cache** | Redis data | `/data` | redis |
| **Cache** | Valkey data | `/bitnami/valkey/data` | valkey |
| **Message Broker** | RabbitMQ data | `/var/lib/rabbitmq` | rabbitmq |
| **Message Broker** | Kafka data | `/var/lib/kafka/data` | kafka, confluentinc/cp-kafka |
| **Web** | NGINX web root | `/usr/share/nginx/html` | nginx |
| **Web** | Apache document root | `/var/www/html` | httpd, apache, php, wordpress |
| **Web** | WordPress uploads | `/var/www/html/wp-content` | wordpress |
| **Logs** | Application logs | `/var/log/app` | (generic) |
| **Observability** | Prometheus data | `/prometheus` | prom/prometheus |
| **Observability** | Grafana data | `/var/lib/grafana` | grafana |
| **Generic** | General data | `/data` | (any image) |
| **Generic** | Application data | `/app/data` | (any image) |
| **Generic** | Application storage | `/app/storage` | (any image) |
| **Generic** | Uploads | `/app/uploads` | (any image) |

### Image-Family Recommendations

Based on your service's container image the system ranks family-specific presets at the top with a **"Recommended for image"** tag. For example:

- `postgres:16` → PostgreSQL data preset appears first
- `nginx:alpine` → NGINX web root appears first
- `my-custom-app:1.0` → only Generic presets appear

Generic presets are always present; workloads without a dedicated preset can use them.

---

## The Storage Tab in the Service Management Panel

To add or manage storage attachments:

1. Open your service from the **Projects** menu
2. Pick **Storage** from the left sidebar in the service management panel
3. Existing attachments are listed as cards; the **Add Storage** button at the top right opens the create dialog

### Card Fields

Each storage card displays:

- **Volume name** (the DNS-1123 label used in the PVC resource)
- **Plan** name and size
- **StorageClass** (longhorn or longhorn-static)
- **Container mount path**
- **Sub-path** (optional)
- **Preset** name and category
- **Monthly cost**
- **Read-only** badge
- **Edit** and **Delete** actions

---

## Adding Storage

Clicking **Add Storage** opens a four-section dialog:

### 1. Plan Selection

The five plans are rendered as cards. The **Free** plan is preselected and labeled with a **Default** badge. Click any card to switch the plan. Each card shows plan name, size, replica count, reclaim policy and monthly price.

### 2. Mount Path Selection

Mount presets are grouped into categories and shown as pill buttons. Presets that match your container image family appear at the top with a green border and the **"Recommended for image"** tag. Other presets are listed below.

Clicking a preset:

- Auto-fills the mount path
- Copies `DefaultSubPath` into the sub-path field (if defined)

> **Security note:** Users cannot type a free-form mount path. The platform only accepts catalog-vetted paths, which prevents mounting volumes to sensitive host directories like `/etc`, `/proc` or `/sys`.

### 3. Volume Name and Sub-Path

- **Volume name:** Enter a unique DNS-1123 label (lowercase letters, digits and dashes, e.g. `db-data`, `uploads`, `redis-persist`). This is part of the generated PVC resource name.
- **Sub-path:** Optional. If you want to mount a sub-directory of the PVC (e.g. `db/prod`), provide the relative path here. Useful when multiple services share a PVC and you want to isolate them into sub-folders.

### 4. Read-Only Mount

Tick the checkbox if you want the volume mounted as `readOnly: true` inside the container. This prevents the container from writing to the volume — useful for workloads that only read existing data.

After you press **Save**:

1. The backend persists the attachment
2. **An auto-deploy is queued immediately** using the service's current image tag (no new build)
3. The pipeline commits `base/pvc.yaml` to Git
4. ArgoCD syncs the manifest into the cluster
5. Longhorn provisions the volume with the plan's parameters
6. The pod is recreated with the volume mounted

A **green toast** appears in the top right: _"Storage saved — Deployment automatically queued, PVC ready in ~30 seconds"_. You can watch the progress live from the **Pipeline Summary** tab.

> 💡 See the [Automatic Deployment](#automatic-deployment) section below for details.

---

## Editing an Existing Attachment

Use the **Edit** button on a card to update an attachment. Editable fields:

- **Plan:** You can upgrade to a larger plan within the same StorageClass and reclaim policy (e.g. Small → Medium → Large)
- **Sub-path**
- **Read-only flag**

Pressing **Save** **automatically triggers a deploy** just like the create flow. A plan upgrade (e.g. 10 GB → 50 GB) is applied via Longhorn volume expansion — a pod restart may be required.

### Non-Editable Fields

- **Mount path:** Kubernetes treats the mount path as part of the immutable PVC binding. To change it, delete the attachment and create a new one.
- **Volume name:** Part of the PVC resource name — cannot be changed.
- **StorageClass transitions** (e.g. Free → Medium): Kubernetes does not allow StorageClass changes on an existing PVC. The platform blocks such plan changes. To migrate, **delete the old attachment first, then create a new one with the target plan**.
- **Shrinking:** Longhorn can expand volumes but not shrink them. Attempting to pick a smaller plan returns an error.

---

## Deleting an Attachment

The **Delete** button on the card removes an attachment. **Deletion is irreversible.** Plan tier, StorageClass and reclaim policy all stop mattering at this point — every storage delete performed through the DevOpsZon UI is a full wipe.

### What happens under the hood

As soon as you confirm, the backend runs the following steps in order:

1. **PVC ↔ PV resolution** — the Longhorn volume name (CSI volume handle) is read from the live cluster.
2. **Reclaim policy override** — the PV's `persistentVolumeReclaimPolicy` is merge-patched to `Delete`. Even Retain-policy PVs from paid plans will be destroyed after this step.
3. **Longhorn snapshot cleanup** — entries in `snapshots.longhorn.io` matching the volume are deleted.
4. **Longhorn backup cleanup** — entries in `backups.longhorn.io` and `backupvolumes.longhorn.io` matching the volume are deleted. Deleting a BackupVolume also removes the underlying S3 archive directory.
5. **Longhorn volume delete** — the `volumes.longhorn.io` resource is explicitly removed.
6. **PVC delete** — the PVC is removed from the service namespace; the PV, now on Delete policy, is garbage-collected along with the underlying disk.
7. **DB row removal** — the attachment record is deleted from the DevOpsZon database.
8. **Auto-deploy** — the service manifest is regenerated (the deleted PVC is no longer included) and committed to Git. ArgoCD performs a no-op on the already-cleaned resources.

These steps are not atomic but they are **best effort**: if the Longhorn API is momentarily unavailable the backend appends a warning to the purge report and continues with the remaining steps. A transient cluster hiccup cannot indefinitely block a delete.

### Confirmation dialog

When you click Delete you will see the following warning:

> **"Storage attachment '<volume-name>' will be permanently deleted. The PVC, all data on disk, Longhorn snapshots and S3 backups will be irreversibly destroyed. Continue?"**

> **Important:** The delete operation runs only after you click **Delete** in the dialog. The platform refuses to delete without acknowledgment. Back up anything you care about before confirming — **there is no way back**.

---

## Automatic Deployment

DevOpsZon turns every storage attachment change (create / update / delete) into an **automatic deployment** for the affected service. You no longer need to navigate to **Dashboard → Deploy** after saving storage — a single **Storage → Save** click is enough.

### How it works

1. When you press Save the backend writes the DB record
2. As soon as the write succeeds the backend calls `IDeploymentTriggerService.EnqueueDeploymentAsync` **with the service's currently active image tag**
3. An `IDeploymentQueueMessage` is published to MassTransit / RabbitMQ
4. `DeploymentQueueConsumer` picks up the message → pipeline runs → PVC manifest is generated → committed to Git → ArgoCD syncs → Longhorn provisions the PV

> **Note:** Auto-deploy **reuses the service's current image**. No new build is triggered — only the YAML manifests are regenerated to include your new storage attachment.

### Three toast outcomes

| State | Toast color | Message |
|---|---|---|
| ✅ Save succeeded + deploy queued | **Green (success)** | "Deployment automatically queued — PVC will be provisioned in ~30 seconds" |
| ⚠️ Save succeeded + enqueue failed (RabbitMQ down, etc.) | **Yellow (warning)** | "Save succeeded but auto-deploy could not be queued: {reason}. Trigger a deploy manually from the Dashboard" |
| ℹ️ Platform opted out of auto-deploy | **Blue (info)** | "Save succeeded. Auto-deploy is disabled by platform configuration — trigger a deploy from the Dashboard" |

### Conflict with an active rollout

If you change storage while a Canary / BlueGreen rollout is in progress:

- The platform uses a **per-service deployment lock** (Redis-backed)
- The storage change deploy message is queued but **waits** until the active rollout completes
- When the active rollout finishes (or aborts) the new deploy takes over
- **Your storage changes are never lost** — they just apply slightly later

### For platform administrators

Platform administrators can opt out of auto-deploy via `appsettings.json`:

```json
{
  "Storage": {
    "autoDeployOnAttachmentChange": false
  }
}
```

When `false` every storage change requires the user to manually trigger a deploy from the Dashboard. Useful for dev/test environments or organisations that require a manual deploy gate. **Defaults to `true`** and should stay on in production.

### Tracking progress

Once auto-deploy is triggered, you can track its progress in two places:

1. **Pipeline Summary tab**: Live step-by-step progress (via SignalR), green/red tick per step
2. **Dashboard tab**: Active rollout card + pod status

Progress is streamed in real time via the `RolloutStatusHub` SignalR hub — no need to refresh the page.

---

## Backup and Restore

DevOpsZon provides two automatic backup mechanisms via Longhorn:

### Daily Snapshot

Every day at **02:00** all volumes are snapshotted. The most recent **7 snapshots** are retained. Snapshots are kept inside the cluster — useful for fast recovery from node or pod failures.

### Weekly S3 Backup

Every **Sunday at 03:00** all volumes are fully backed up to the platform-configured S3-compatible storage target. The last **4 weekly backups** are retained. While snapshots live inside the cluster, weekly backups live outside it — so you can recover data even if the entire cluster is lost.

> **Note:** The S3 backup target is active only when the platform administrator has configured it. For bring-your-own-cluster deployments, an admin needs to add a ManagedObjectStorage record with the "longhorn" purpose.

### Restoring from a Snapshot

UI-driven snapshot restore is not yet supported. If you experience data loss, contact platform support — admins can restore from a snapshot manually.

---

## Pipeline Integration

When you add, update or delete a storage attachment, the **auto-triggered** deploy performs the following steps:

0. **AppService auto-deploy enqueue** — As soon as the DB row is persisted, `IDeploymentTriggerService.EnqueueDeploymentAsync` is called with the service's current image tag
1. **AcquireDeploymentLockStep** — Takes a per-service Redis lock (waits if another rollout is active)
2. **DeploymentContextFactory** — Loads attachments + plan info into the snapshot
3. **PvcContributor** — Scans active attachments and generates `base/pvc.yaml` (one `ReadWriteMany` PersistentVolumeClaim per attachment)
4. **RolloutContributor** — Adds `volumes` (`persistentVolumeClaim`) to the pod template and `volumeMounts` to the container spec
5. **KustomizationContributor** — Includes `pvc.yaml` in the resource list of `base/kustomization.yaml`
6. **ValidateDeploymentStep** — Compares the live cluster's PVC `accessModes` and `storageClassName` against the target manifest; if there is a mismatch (e.g. a legacy RWO PVC) the deploy is halted with a "delete and recreate the service" message
7. **PushToGitStep** — Commits manifests to the service repo
8. **SyncArgoCdStep** — Triggers ArgoCD sync (or ArgoCD auto-sync picks it up via polling)
9. **Longhorn Provisioner** — Dynamically creates a PV to satisfy the PVC
10. **Pod Restart** — The new pod comes up with the volume mounted

Manifests are committed to your service repo; ArgoCD applies them declaratively. The answer to "what is running?" is always the manifest in Git.

---

## Frequently Asked Questions

### Can I attach multiple storage volumes to a single service?

Yes. For example, a web application can have separate attachments for `uploads`, `cache` and `logs`. Each attachment is provisioned as an independent PVC. You cannot reuse the same volume name or mount path within a service — the platform enforces uniqueness at the database level.

### Can two services share the same disk?

No — a PVC belongs to a single service. But **multiple pods of the same service** (canary + stable, replica 1 + replica 2, etc.) can mount the same PVC. The platform handles this uniformly: the DevOpsZon pipeline **provisions every PVC as `ReadWriteMany` (RWX)** regardless of tier, strategy, or replica count.

This is a deliberate platform-wide decision under the "Always RWX" architecture. All three rollout strategies (Canary / BlueGreen / AutoPromote) keep both the old and new pod alive at the same time during a rollout, and both mount the same PVC — so RWX is unavoidable. Even single-replica single-strategy services are provisioned as RWX so they are not affected by future HPA activation or replica scaling.

> ℹ️ **Performance note**: Longhorn RWX volumes run through a share-manager (NFS) pod per volume, so they are ~2-3x slower than pure block-device RWO on latency and IOPS. For very random-I/O-heavy databases you may feel the hit; in that case keep the replica count at 1 and tune `fsync` behaviour to reduce the bottleneck. For hot-path databases it is usually better to pick a managed addon (PostgreSQL, Valkey, RabbitMQ) — those use their own optimised block-device storage.

### My service was provisioned with RWO volumes — what happens now?

Services created before the "Always RWX" refactor carry PVCs in `ReadWriteOnce` (RWO) mode. Kubernetes does not allow mutating a PVC's `spec.accessModes` field, so on the next deploy of such a service the platform will surface a "StorageAccessMode mismatch" error and abort the rollout.

Resolution: **delete and recreate the service.** The new service is automatically provisioned with RWX PVCs and every rollout strategy / HPA / multi-replica feature will work. If you need to migrate data, snapshot the old PVC before deleting the old service and restore it into the new service manually (the support team can help).

### What happens when my volume fills up?

Your application receives a disk-full error (`ENOSPC`). To grow the volume, edit the existing attachment from the service management panel and pick a larger plan — the platform uses Longhorn's volume expansion feature (a pod restart may be required).

### Can I recover an attachment I deleted by mistake?

- **Retain plans:** The underlying PV still exists in the cluster. A platform admin can manually create a new PVC and bind it to the PV. Contact support.
- **Delete plans (Free):** Recovery is not possible. The data is permanently gone.

### Is the Free plan enough for production?

No. The Free plan only offers 1 GB of space, 2 replicas and a `Delete` reclaim policy. For production workloads we recommend **at least Small**, and **Medium or higher** for critical databases.

### I saved storage but auto-deploy did not trigger — what should I do?

If you see a yellow warning toast "Save succeeded but auto-deploy could not be queued", the RabbitMQ / message queue is temporarily unreachable. The attachment is safe in the DB — **no data loss** — simply navigate to **Dashboard → Deploy** and trigger a manual deploy. The pipeline will automatically include your new attachment through `DeploymentContextFactory`.

### When should I use sub-path?

Sub-paths are useful in two scenarios:

1. **Mounting different sub-directories of the same PVC into different containers** (rare multi-container pod setups)
2. **Logically partitioning a large shared directory** — for example mounting just `wp-content` (instead of the full `wp-content/uploads`) in a WordPress site

Simple single-container services typically do not need sub-paths.

---

## Security and Isolation

- **Multi-tenant isolation:** Every attachment is linked to `TenantId`. Services outside your tenant cannot access your PVCs.
- **Mount path whitelist:** The platform only accepts catalog-vetted mount paths. Free-form path entry is restricted to platform administrators and limited to the `/data`, `/app`, `/var/lib`, `/var/log`, `/home`, `/mnt`, `/opt`, `/usr/share`, `/prometheus`, `/bitnami` prefixes.
- **Encryption at rest:** Longhorn volumes are encrypted on disk according to the platform administrator's encryption settings.
- **Backup credential isolation:** S3 backup credentials are stored in a dedicated Secret. Your container cannot access this Secret.

---

## Related Guides

- [Service Management](service-guide.md) — The service management panel that hosts the Storage tab
- [Billing & Plans](billing-plans.md) — Where disk plans fit into the overall pricing model
- [Managed Services](managed-addons.md) — Managed services like PostgreSQL, RabbitMQ and Valkey bring their own storage (a separate paradigm)
