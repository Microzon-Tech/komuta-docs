# Persistent Storage

The disk inside your service's container is temporary. Files the app writes while it runs belong to that pod: they are lost when the pod restarts or a new version is deployed, and a newly started replica does not see them. For files that must survive restarts and deploys, attach a persistent **volume** to the service. A volume is mounted at a folder you choose inside the container. The app finds the files it wrote there again in every later pod.

Volumes are managed from the **Storage** tab on the service page: **Apps** → service → **Storage**.

---

## When You Need a Persistent Volume

| Case | Recommended |
|---|---|
| Files your users upload, reports the app generates, a file-based index or cache | Persistent volume |
| Relational database | [PostgreSQL](managed-postgresql.md). Backups and restore come built in. |
| Session data, key-value cache | [Valkey](managed-valkey.md) |
| Queues and messaging | [RabbitMQ](managed-rabbitmq.md) |
| Temporary files needed only during a request | No volume needed, the container's own disk is enough |

The writable paths described in [Service Security](service-security-guide.md) are not persistent either. Files written to them are deleted when the pod is recreated.

Limits to know:

- Each volume belongs to a single service. It cannot be shared with another service.
- Volumes can only be added to long-running services. Jobs and CronJobs have no **Storage** tab.
- The Storage page has no backup or restore action. Keep your own copy of files you cannot afford to lose.

---

## The Storage Tab

When the service has no volumes, the tab shows **No Volumes Attached**. Otherwise the **Storage Volumes** list shows the following for each volume:

| Item | Meaning |
|---|---|
| Volume name | The name you gave when you created the volume |
| **Disabled** | The volume is inactive and not mounted into the service (see Editing a Volume below) |
| **Read-only** | The app cannot write to this volume |
| **Free** | The volume is on the free plan |
| Plan and size | The volume's plan, for example `Small` |
| Mount path | The folder inside the container where the volume is mounted, with the sub path if there is one |
| Monthly price | The plan's monthly price, for paid plans |

The **Add Volume** button and the edit and delete icons are shown only to users who can edit the service. Without that permission you can see the list but not change it.

---

## Adding a Volume

1. On the **Storage** tab, click **Add Volume**. The window shows three steps at the top: **Plan**, **Volume**, **Path**.
2. Pick a plan from the **Storage Plan** table. The table has **Plan**, **Size**, **Replicas** and **Price** columns. The **Replicas** column shows how many copies of your data are kept. It has nothing to do with your service's replica count.
3. Check the **Volume Name** field. A name is suggested from the service name, for example `orders-api-vol-1` for the `orders-api` service. Change it if you like.
4. In **Mount Path**, choose the folder the volume is mounted at (details below).
5. If needed, enter a **Sub Path (optional)** and turn on **Read-only**.
6. Click **Create**.

When the save succeeds you see **Volume added**. If a field is missing or invalid, the **Create** button stays disabled and the bottom of the window tells you what to do, for example **Select a storage plan** or **Fix volume name format**.

### Plans

| Plan | Size |
|---|---|
| Free | 1 GB |
| Small | 10 GB |
| Medium | 50 GB |
| Large | 200 GB |
| XLarge | 1024 GB |

In the plan table the free plan is marked **FREE**. The monthly price of the other plans is shown in the **Price** column.

All volumes on a service must be either free or paid:

- With the free plan, a service can have only one volume, and it must be the service's only volume.
- If the service has a paid volume, you cannot add a free one.
- If the service has a free volume, upgrade it to a paid plan before adding a new paid volume (see Editing a Volume below).

Plans that break these rules are dimmed in the table and cannot be selected. Hover over one to see why.

### Mount path

If the service has a running pod, Komuta lists the folders directly under the app's working directory (WORKDIR). The first folder in the list is selected automatically. Check that the right folder is selected before you click **Create**.

If the folder you want is not in the list, click **Enter manually** and type the path yourself, for example `/app/data`. Click **Browse directories** to go back to the list. The path does not have to exist in the image. You can also enter a folder the app creates while it runs.

If the service has not been deployed yet, is asleep, or its image has no shell (`sh` or `bash`), the folder list cannot open. You then see **Directory browse unavailable — enter path manually** and type the path yourself.

### Sub path and read-only

- **Sub Path (optional)**: mounts a folder inside the volume at the mount path, instead of the volume's root. Enter a relative path inside the volume, for example `data/cache`.
- **Read-only**: the app can read the volume but cannot write to it.

### Name and path rules

| Field | Rule | Message in the console |
|---|---|---|
| **Volume Name** | Up to 63 characters. Only lowercase letters, digits and hyphens; starts and ends with a letter or digit. Uppercase letters you type are lowercased automatically. Cannot start with `fwk-`. Must be unique within the service. | Lowercase letters, numbers, hyphens only (DNS-1123) |
| **Mount Path** | An absolute path starting with `/`, up to 256 characters. Cannot contain `..` and cannot be the root directory (`/`). Must be unique within the service. | Enter a valid absolute container path (must start with '/', without '..' segments). |
| **Sub Path (optional)** | A relative path inside the volume, up to 256 characters | — |

Some paths cannot take a volume, because the volume would hide the container's own files at that path and break the app:

- The `/app` folder itself. Its sub folders, for example `/app/uploads` or `/app/data`, are fine.
- `/app/wwwroot` and everything under it.
- System folders and everything under them: `/bin`, `/sbin`, `/usr`, `/lib`, `/lib64`, `/etc`, `/boot`, `/root`, `/proc`, `/sys`, `/dev`, `/var/run`.
- A few paths the platform keeps for itself.

If you pick such a path, a message appears under the field, for example for `/etc`: **'/etc' is a reserved system path. Mounting a volume here would shadow the container's built-in files and break the app. Pick a dedicated data directory such as /data or /app/uploads.** Use a separate folder such as `/data` or `/app/uploads` for your data.

---

## How Changes Are Applied

When you add, edit or remove a volume, Komuta redeploys the service automatically. No new build runs; the service is redeployed with the image that is running now. The notification says **Storage updated — deploy will start in ~30s**.

The deploy follows the service's deployment strategy. With Blue-Green, the new version takes no traffic until you promote it, so the volume only shows up in the pods serving live traffic at that point. See [Deployment Strategies](deployment-strategies.md) for details.

If the automatic deploy cannot be started, the notification says **Storage updated — auto-deploy skipped:** followed by the reason. Your change is saved. To apply it, start a deploy with the **Deploy** button on the [Service Dashboard](service-dashboard.md).

If the service has never been deployed, the volume is mounted on its first deploy.

> **Note:** If your account is suspended, or the free plan's awake hours for this month are used up, volume changes are blocked like any other change to the service. The error message tells you the reason and the fix. See [Sleep and wake](service-sleep.md).

---

## Files in the Image and the First Mount

The mount path may be a folder that already holds files in your image. In that case, while the volume is empty, those files are copied into the volume once, on the first deploy. The app then finds the files it expects from the image on the volume too.

As long as the volume holds files, this copy never runs again. On later deploys, even if the image's files in that folder change, they do not overwrite the files on the volume. The volume's content always wins.

- Read-only volumes are not filled from the image. If you mount an empty read-only volume over a folder that has files in the image, the app sees that folder as empty.
- With a sub path, the copy goes into that sub folder of the volume.
- You may see a hidden file named `.komuta-seed.lock` in the mount path folder. Komuta uses it to stop several replicas from doing the first copy at the same time. Do not delete it. A new volume may also contain a `lost+found` folder.

---

## Restarts, Deploys, Scaling and Sleep

| Event | Data on the volume |
|---|---|
| Pods recreated with **Restart** | Kept. The new pods mount the same volume. |
| Deploying a new version or rolling back to an earlier one | Kept. Files from the image do not overwrite the volume. |
| More replicas, or [autoscaling](services-autoscaling.md) | All replicas read and write the same volume at the same time. |
| A Blue-Green, Auto Promote or Canary rollout | The old and the new version share the volume during the rollout. |
| The service goes to sleep | Kept. The same volume is mounted again when the service wakes. |
| Moving to a larger plan | Kept. |
| Removing the volume | Deleted permanently. |
| Deleting the service | All of the service's volumes are deleted with their data. |

A volume is shared by all replicas of the service. There is no single-writer limit, so your app has to coordinate replicas that write the same file at the same time. Do not run a file-based database such as SQLite on a volume with more than one replica; use [PostgreSQL](managed-postgresql.md) for databases.

During a deploy, the old and the new version see the same files. If the new version changes the format of files on the volume, expect the old version to keep reading them for a while.

---

## Editing a Volume

Click the pencil icon on the volume's row. The **Edit Volume** window opens.

| Field | Can it change? |
|---|---|
| **Storage Plan** | Yes. Pick a plan of the same or a larger size. |
| **Sub Path (optional)** | Yes |
| **Read-only** | Yes |
| **Active** | Yes |
| **Volume** (name) | No |
| **Mount Path** | No |

Save with **Update**. You see **Volume updated** and the service is redeployed.

### Growing a volume

Pick a larger plan and click **Update**. The data on the volume is kept. You can upgrade a free volume to a paid plan the same way.

Volumes cannot shrink. A smaller plan may look selectable in the table, but when you save you see **Volume operation failed** with a message that the storage cannot be downgraded. To move to a smaller volume, add a new volume at a different mount path, move the files yourself, then remove the old volume.

A volume's name and mount path cannot be changed later. If you need another path, copy your files, add a new volume and remove the old one.

### Changing the sub path

When you change the sub path, the app sees a different folder inside the volume. Files in the previous sub folder stay on the volume but no longer appear at the mount path.

### The Active switch

When you turn **Active** off, the volume shows as **Disabled** in the list. It is not mounted into the service and is left out of the service's next deploy. An inactive volume still holds its name and mount path, so you cannot add a new volume with the same name or path.

> **Warning:** Do not use the **Active** switch to pause a volume while keeping its data. Assume that turning it off carries the same risk of data loss as removing the volume, and copy important files first.

---

## Removing a Volume

1. Click the trash icon on the volume's row.
2. Confirm the browser prompt **Remove {name}? Data on this volume will be deleted.**

You see **Volume removed** and the service is redeployed. After that deploy, the app no longer sees the volume at that path.

> **Warning:** Removal is permanent. All files on the volume are deleted together with the volume's snapshots and backups. Komuta cannot bring a removed volume back. Copy the files you need before you remove it.

Deleting the service deletes all of its volumes in the same way.

---

## Watching How Full a Volume Is

In the topology on the [Service Dashboard](service-dashboard.md), each active volume appears as a **Storage** node. When usage can be measured, the node shows used and total space above it, for example `1.2 GiB / 10 GiB`. The bar under that text shows how full the volume is: it turns orange at 75% and red at 90%. When usage cannot be measured, only the volume's size is shown.

Move to a larger plan before the volume fills up.

---

## Errors and Fixes

When the server refuses an action, you see **Volume operation failed:** followed by one of the messages below.

| Message | Cause | What to do |
|---|---|---|
| The free plan only supports a single storage attachment. Upgrade to a paid plan to add more storage. | The service already has a free volume | Pick a paid plan |
| This service already has a free-plan storage attachment. Upgrade that attachment to a paid plan before adding another paid attachment. | A paid volume is being added to a service with a free volume | First edit the free volume and move it to a paid plan |
| This service already has paid-plan storage attachments. Free-plan storage cannot be mixed with paid storage on the same service. | A free volume is being added to a service with paid volumes | Pick a paid plan |
| A message that another volume with the same name or mount path already exists | The name or mount path is already used on this service, including by an inactive volume | Enter a different name or path |
| A message that the volume name cannot start with `fwk-` | The name starts with `fwk-`, which is reserved | Pick another name |
| Persistent disks can only be attached to a long-running service. A one-off or scheduled job does not mount storage. | A volume is being added to a Job or CronJob (for example through MCP) | Use a managed service for persistent data |
| A message that the storage cannot be downgraded | A smaller plan was selected | Pick a plan of the same or a larger size |

---

## Managing Volumes with MCP and Stacks

With [Komuta MCP](mcp-tools.md) you can ask your AI assistant to list volumes and plans, and to add, edit and remove volumes. The same rules as in the console apply.

For services you create with a [Stack](stacks.md), you can declare volumes in the manifest file. See the [Stack Manifest Reference](stack-manifest-reference.md) for the fields.

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — Volume usage in the topology, and manual deploys.
- [Autoscaling](services-autoscaling.md) — How the replica count follows load.
- [Deployment Strategies](deployment-strategies.md) — How the deploy after a volume change goes live.
- [Sleep and wake](service-sleep.md) — What is kept while a service sleeps.
- [PostgreSQL](managed-postgresql.md) — The managed, backed-up option for databases.
- [Service Security](service-security-guide.md) — Writable paths and the read-only root filesystem.
- [Komuta MCP](mcp-tools.md) — Managing volumes from your assistant.
