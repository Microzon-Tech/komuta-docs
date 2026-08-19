# Service Management

Service management is DevOpsZon's most comprehensive section. Once you select a service, you manage all operations belonging to that service — from deploy to monitoring, from logs to alerts — from a single panel.

---

## Service Management Panel

To access the service management panel, click a service's name from the project list or select a service from the **Services** section in the left menu. The panel lets you switch between different management areas using the tab bar at the top.

### Tab Structure

| Tab | Purpose |
|-------|------|
| **Dashboard** | The service's live status, traffic flow, pod information |
| **Environment Variable** | Managing environment variables |
| **Ingress Management** | Ingress rules, hostname and traffic configuration |
| **HPA** | Automatic scaling (Horizontal Pod Autoscaler) settings |
| **Resources** | CPU and memory resource limits |
| **Ports** | Service port configuration |
| **Storage** | Longhorn-backed persistent disk (PVC) management |
| **Health Probes** | Health check (Liveness/Readiness) settings |
| **Alert Management** | Service-specific alert rules |
| **Pipeline Summary** | CI/CD pipeline run history |
| **Logs** | Real-time log viewing |
| **History & Rollback** | Deployment history and rollback |

> **Note:** In SaaS mode, the **Resource Plan** and **Invoice** tabs additionally appear; some operational tabs may vary.

---

## Dashboard

The Dashboard is the main view showing your service's live status. All information updates in real time via **SignalR** — no page refresh is needed.

### Live Traffic Visualization

At the center of the Dashboard is the **Live Traffic Analysis** component. This interactive visualization shows, in real time, the flow of traffic coming into your service from ingress down to the pods:

- **HTTPRoute / Ingress point:** The entry point for incoming traffic (at the top)
- **Service cards:** Active and (if present) Preview/Canary service groups, each showing a traffic percentage and progress bar
- **Pod nodes:** Each pod's phase status (Running, Pending, Failed), image tag and health information (at the bottom)
- **Traffic arrows:** The direction and intensity of traffic flowing from ingress to services and from services to pods

The visualization presents different information depending on the **rollout strategy** you use. A detailed description of the strategies is below.

### Status Information

The Dashboard displays summary information about your service:

- **Pod count and status:** Running, Pending, Failed
- **Latest deployment revision:** Active version information
- **HPA status:** If automatic scaling is active, the current/minimum/maximum replica count
- **Resource consumption:** CPU and memory usage
- **Alert count:** Number of active alerts
- **Pipeline status:** Step-by-step status of the latest build via task dots (with special emphasis for the rollout-status task)

---

## Rollout Strategies

DevOpsZon supports three different deployment strategies using **Argo Rollouts** infrastructure. Each strategy lets you take your new version live with a different approach.

### Strategy Comparison

| Feature | Canary | Blue/Green | Auto-Promote |
|---------|:------:|:----------:|:------------:|
| **Traffic control** | Gradual (by percentage) | Instant cutover | Automatic cutover |
| **Preview environment** | None (single hostname) | Yes (separate hostname) | Yes (separate hostname) |
| **Manual approval required** | Yes (at each step) | Yes (promote) | No (automatic) |
| **Traffic percentage adjustment** | Yes (step-by-step increase) | No (100% cutover) | No |
| **Number of hostnames** | 1 (shared) | 2 (active + preview) | 2 (active + preview) |
| **Instant rollback** | Yes (abort) | Yes (abort) | Yes (abort) |
| **Recommended use** | High-traffic services | Critical services | Safe automatic deployment |

---

### Canary Strategy

The Canary strategy lets you test your new version with real users by routing a small percentage of live traffic to it. If there are no issues, you gradually increase the traffic percentage.

#### How Does Canary Work?

```
Step 1: 5% traffic to new version → Monitor → Any issues?
Step 2: 25% traffic to new version → Monitor → Any issues?
Step 3: 50% traffic to new version → Monitor → Any issues?
Step 4: 100% traffic to new version → Completed ✓
```

1. When the new version is deployed, only a small percentage of traffic (e.g. 5%) is routed to the new pods
2. The old (stable) pods continue to receive the remaining traffic
3. At each step, the rollout **pauses (Paused)** and waits for your approval
4. After checking the metrics, logs, and alerts, you move on to the next step
5. In the final step, all traffic is shifted to the new version and the old pods are cleaned up

#### Canary View in the Dashboard

When the Canary strategy is active, the dashboard looks like this:

**Top Banner:**
- **"New version being prepared — Full Promote required"** info message
- **Paused** badge: Canary is paused, waiting for approval for the next step
- Strategy badge: **Canary** icon

**Live Traffic Analysis:**
- **A single hostname** is shown (Canary has no separate preview hostname)
- **Stable service card:** Old version, e.g. **75% traffic** — green progress bar
- **Canary service card:** New version, e.g. **25% traffic** — blue progress bar
- **Pod groups:** Stable pods and Canary pods are shown as separate groups
- Image tag and status information on each pod

**Traffic percentages** are determined by weight rules on the Gateway API and update in real time.

#### Canary Actions

| Action | Where | Description |
|---------|--------|----------|
| **Resume (Next Step)** | Traffic Distribution dialog | Advances to the next traffic step (e.g. 5% → 25%) |
| **Full Promote** | Banner + Traffic Distribution dialog | Instantly routes all traffic to the new version (100%) |
| **Abort** | Banner | Cancels the Canary, returns all traffic to the old version |

**Traffic Distribution Dialog:**

In the dialog opened by clicking the **"Full Promote"** link in the banner or the traffic cards:
- The **current traffic distribution** is shown (e.g. Stable 75%, Canary 25%)
- Information about the **next step** is shown (e.g. 50%)
- You can advance to the next step with the **Resume** button
- You can shift all traffic to the new version with the **Full Promote** button

> **Important:** In Canary, the traffic percentage can only be **increased**. If you want to lower the traffic percentage, you need to cancel the rollout with **Abort** and return to the previous version.

#### Canary Traffic Steps

Traffic steps are defined in your service's rollout manifest. DevOpsZon automatically runs as many `promote` commands as needed to reach the target weight. An example step structure:

| Step | Canary Weight | Stable Weight | Status |
|:----:|:---------------:|:---------------:|-------|
| 1 | 5% | 95% | Paused — awaiting approval |
| 2 | 25% | 75% | Paused — awaiting approval |
| 3 | 50% | 50% | Paused — awaiting approval |
| 4 | 100% | 0% | Completed — old pods cleaned up |

#### When Should You Use Canary?

- When you want to minimize risk on high-traffic services
- When you want to measure the new version's performance with real traffic
- In A/B-testing-like scenarios
- When you want a gradual and controlled transition

---

### Blue/Green Strategy

The Blue/Green strategy lets you prepare and test the new version completely in a separate environment (Preview) before taking it live in a single step. Two environments run in parallel — one receives live traffic while the other is being tested.

#### How Does Blue/Green Work?

```
┌─────────────────────────────────────────────────┐
│  Active (Blue)        Preview (Green)            │
│  ┌──────────┐         ┌──────────┐              │
│  │ Old      │ ←traffic│ New      │ ←test        │
│  │ Version  │  100%   │ Version  │  (preview)   │
│  └──────────┘         └──────────┘              │
│                                                  │
│  ────── Promote ──────→                          │
│                                                  │
│  ┌──────────┐         ┌──────────┐              │
│  │ (cleaned │         │ New      │ ←traffic     │
│  │  up)     │         │ Version  │  100%        │
│  └──────────┘         └──────────┘              │
└─────────────────────────────────────────────────┘
```

1. When the new version is deployed, separate pods are created in the **Preview** environment
2. Live traffic continues to flow entirely from the **Active** (old version) pods
3. You can test the new version via the Preview hostname
4. The rollout **pauses (Paused)** and waits for your approval
5. With **Promote**, all traffic is routed to Preview (the new version) — the cutover is instant
6. The old Active pods are cleaned up

#### Blue/Green View in the Dashboard

When the Blue/Green strategy is active, the dashboard looks like this:

**Top Banner:**
- **"New version being prepared in the preview environment"** info message
- New and old version information (revision numbers)
- **Full Promote** link (if preview is healthy)
- **Preview Link** button: Direct access to the preview hostname
- **Paused** badge
- Strategy badge: **BlueGreen** icon

**Live Traffic Analysis:**
- **Two hostnames** are shown:
  - **Active:** `my-api-a1b2c3d4.devopszon.com` — live traffic
  - **Preview:** `my-api-a1b2c3d4-preview.devopszon.com` — test traffic
- **Active service card:** Old version pods — 100% live traffic
- **Preview service card:** New version pods — preview traffic only
- Separate pod nodes for each group

**Unhealthy Preview Status:**

If the pods in the preview environment are unhealthy (crash, failed readiness probe, etc.), the dashboard shows:
- **Warning message:** "Preview environment unhealthy — Full Promote not recommended"
- The **Abort** button is highlighted
- The Full Promote link is hidden or shown with a warning

#### Blue/Green Actions

| Action | Where | Description |
|---------|--------|----------|
| **Full Promote** | Banner + Traffic Distribution dialog | Routes all traffic to the new version (Preview → Active) |
| **Abort** | Banner | Cancels the Preview, the old version keeps running |
| **Preview Link** | Banner + Hostname list | Direct access to the preview hostname |

**Traffic Distribution Dialog:**

In Blue/Green, the traffic percentage cannot be adjusted — the cutover happens **100% in a single step**. In the dialog:
- Current status information is shown
- The transition is confirmed with the **Full Promote** button
- Description: "All traffic will be routed to the new version in a single step"

#### Blue/Green Hostnames

In the Blue/Green strategy, two hostnames are assigned to each service:

| Type | Format | Usage |
|-----|--------|------|
| **Active** | `{service-name}-{id}.devopszon.com` | Live traffic — your users use this address |
| **Preview** | `{service-name}-{id}-preview.devopszon.com` | Test traffic — use it to verify the new version |

After Promote, the Preview hostname becomes invalid and is removed from view once the rollout completes.

#### When Should You Use Blue/Green?

- When you want a zero-risk transition for critical services
- When you need to run comprehensive tests before taking the new version live
- When instant rollback capability is essential
- When gradual traffic management is unnecessary

---

### Auto-Promote Strategy

Auto-Promote is an automated version of the Blue/Green strategy. After the preview environment is prepared, the traffic cutover happens **automatically** once certain conditions are met — no manual approval is required.

#### How Does Auto-Promote Work?

1. The new version is created in the Preview environment (same as Blue/Green)
2. Preview pods become healthy
3. The system monitors preview for a defined waiting period
4. If no issues are detected, **automatic promote** occurs
5. All traffic is routed to the new version

#### Auto-Promote View in the Dashboard

When the Auto-Promote strategy is active, the dashboard looks like this:

**Waiting Status:**
- A **"New version being prepared"** overlay is shown on screen
- A visual waiting indicator with a rocket animation
- Manual promote buttons are **hidden** — the process is automatic
- No additional action is expected from the user

**Completion:**
- Once automatic promote occurs, the overlay disappears
- A **"Deployment successful"** notification is shown
- The dashboard returns to its normal view

**Error Status:**
- If the preview pods are unhealthy, an **Abort** banner is shown
- Warning message: "Preview environment unhealthy — automatic cutover cannot proceed"
- The user can only take the **Abort** action

#### Auto-Promote Actions

| Action | Where | Description |
|---------|--------|----------|
| **Abort** | Banner (only in error state) | Cancels the rollout if preview is unhealthy |

> **Note:** In the Auto-Promote strategy, user intervention is kept to a minimum. Promote happens automatically; the user can only intervene with Abort in the event of an issue.

#### When Should You Use Auto-Promote?

- When you fully trust your CI/CD process
- When you want fast and automatic deployment
- When you want to eliminate the manual-approval bottleneck
- In development and staging environments

---

### Stale Rollout Policy

In the Canary or Blue/Green strategy, a rollout can remain **Paused** for a long time (if the user forgot to promote or postponed it). DevOpsZon provides a platform-level policy to automatically manage these "stale" rollouts.

#### How Does It Work?

- The platform administrator defines a **maximum wait time** (e.g. 60 minutes)
- If a rollout remains Paused beyond this time:
  1. The health status of the preview pods is checked
  2. If healthy → **automatic Full Promote** occurs
  3. If unhealthy → **automatic Abort** occurs
- Certain services can be **excluded** from this policy

> **Note:** The stale rollout policy applies to the Canary and Blue/Green strategies. Since the Auto-Promote strategy already manages its own automatic cutover, it is outside the scope of this policy.

---

### Rollout Statuses

During the deploy process, your service goes through the following statuses. These statuses are shown live in the dashboard's banner, badges, and traffic visualization:

| Status | Visual | Meaning |
|-------|--------|--------|
| **Progressing** | Animated progress | Deploy is in progress, new pods are being created |
| **Paused** | Yellow badge | User intervention is awaited (Canary step or Blue/Green promote) |
| **Healthy / Completed** | Green badge | Deploy completed successfully, all pods are healthy |
| **Degraded** | Red badge | Some pods are unhealthy; intervention may be required |

All these status transitions are reported in real time via **SignalR**. The pipeline's **rollout-status** task dot (the small status indicators in the header bar) is also colored according to the rollout's status.

### Deploy Lock During Rollout

While a rollout is **Paused** (awaiting a Canary step or a Blue/Green promote), a **new deploy cannot be triggered** for the same service. You must first Promote or Abort the current rollout.

This mechanism prevents conflicting deploys, preserving consistency

---

## Environment Variables

Manage the configuration values your services need at runtime.

### Adding a Variable

1. Go to the **Environment Variable** tab
2. Click the **Ekle** (Add) button
3. Enter a key and value
4. Check the **Secret** option for sensitive values
5. Click the **Kaydet** (Save) button

> **Important:** Environment variable changes take effect on the next deploy. They are not applied to existing pods instantly.

### Secret Variables

Variables marked as Secret:
- Are stored as a Kubernetes Secret
- Are shown masked in the panel (`••••••••`)
- Can only be viewed by authorized users

---

## Ingress Management

Configure how your service is accessed from outside.

### Hostname

An automatic hostname is assigned when each service is created:

```
{service-name}-{unique-id}.devopszon.com
```

In the Blue/Green strategy, a preview hostname is additionally created:

```
{service-name}-{unique-id}-preview.devopszon.com
```

### Custom Domain

Use the **Domains** page to attach your own domain name to a service. See the **Ingress and Domain Management** guide for details.

### Traffic Configuration

In the Ingress Management tab, you can configure the following settings:

| Setting | Description |
|------|----------|
| **Host rules** | Which hostnames route to this service |
| **Path rules** | URL path-based routing |
| **TLS** | SSL/TLS certificate configuration |
| **Backend port** | The port the service listens on |

---

## Automatic Scaling (HPA)

The Horizontal Pod Autoscaler enables your service to scale automatically under load.

### HPA Configuration

| Parameter | Description |
|-----------|----------|
| **Minimum Replica** | The minimum number of pods to run |
| **Maksimum Replica** | The maximum number of pods to scale to |
| **CPU Target (%)** | Target CPU usage percentage |
| **Memory Target (%)** | Target memory usage percentage |

Kubernetes automatically increases or decreases the pod count based on the defined metrics.

---

## Resource Management (Resources)

Set CPU and memory limits for each service.

| Parameter | Description |
|-----------|----------|
| **CPU Request** | The minimum amount of CPU guaranteed to the service |
| **CPU Limit** | The maximum amount of CPU the service can use |
| **Memory Request** | The minimum amount of memory guaranteed to the service |
| **Memory Limit** | The maximum amount of memory the service can use |

> **Tip:** Set Request values to match your application's consumption under normal load, and set Limit values to accommodate peak load.

---

## Port Configuration

Configure the ports your service listens on. The default port is sufficient for most web applications, but you can define additional ports for services that listen on multiple ports.

---

## Storage (Persistent Storage)

The **Storage** tab lets you attach Longhorn-backed persistent disks to your service. For data that needs to survive pod restarts — such as a database, file server, upload folder, cache, or log archive — you can create a PersistentVolumeClaim (PVC) by choosing a **disk plan**.

### Quick Summary

- **5 disk plans:** Free (1 GB, free), Small (10 GB), Medium (50 GB), Large (200 GB), XLarge (1 TB). Paid plans provide production safety with a `Retain` reclaim policy.
- **25+ mount path presets:** Predefined, safe mount paths for image families such as PostgreSQL, MySQL, MongoDB, Redis, Valkey, RabbitMQ, NGINX, Apache, Prometheus, Grafana.
- **Image-family highlighting:** Presets suitable for your container image are highlighted with a "Recommended for Image" label.
- **Safe path whitelist:** There is no free-text path entry — the platform only accepts safe paths from the catalog. Volumes cannot be attached to sensitive host directories such as `/etc`, `/proc`, `/sys`.
- **Automatic backup:** Daily snapshot + weekly S3 backup (if configured by the platform administrator).
- **Two-step deletion confirmation:** Both a UI confirmation and a backend acknowledgment flag guard against data-loss risk.
- **Automatic deploy:** Every storage change (add/update/delete) automatically triggers a deploy — no need to click Deploy manually.

### Basic Usage Flow

1. Open the **Storage** tab from the service management panel
2. Click the **Depolama Ekle** (Add Storage) button
3. Choose a plan from the dialog (default: Free)
4. Choose the mount path preset suitable for the image family (e.g. PostgreSQL data for `postgres:16`)
5. Enter a unique volume name (e.g. `db-data`)
6. Optionally set the sub-path and read-only flag
7. **Kaydet** (Save) → **automatic deploy** is triggered, the PVC is ready on the cluster within ~30 seconds
8. Green toast: _"Deploy automatically triggered"_ → you can watch the progress live from the **Pipeline Summary** tab

### Detailed Guide

A separate guide has been prepared with comprehensive information on storage management, plan selection criteria, the mount path catalog, backup/restore, and pipeline integration:

➡️ **[Persistent Storage Guide](storage-management.md)**

You can also find the following in the guide:

- Detailed usage scenarios for each plan
- Comparison of Retain vs. Delete reclaim policy
- Plan upgrade/change rules (what can and cannot be changed)
- Warnings about the irreversibility of deletion and data-loss risk
- Multi-volume services and sub-path usage scenarios
- Security and multi-tenant isolation details
- Frequently asked questions

---

## Health Probes

Define probes for Kubernetes to check your service's health status.

### Liveness Probe

Checks whether the service is running. If it fails, the pod is restarted.

| Parameter | Description |
|-----------|----------|
| **HTTP Path** | The health check endpoint (e.g. `/health`) |
| **Port** | The port being checked |
| **Initial Delay** | Wait time before the first check (seconds) |
| **Period** | Check interval (seconds) |
| **Failure Threshold** | The number of failed checks after which it is considered unhealthy |

### Readiness Probe

Checks whether the service is ready to receive traffic. If it fails, the pod is removed from the traffic pool but is not restarted.

> **Tip:** Check prerequisites such as database connectivity or cache warm-up in the readiness probe.

---

## Pipeline Summary

In this tab, you can see all CI/CD pipeline runs for your service.

- **Live pipeline:** Watch the step-by-step progress of an ongoing build
- **History:** A list of completed pipelines with duration and status information
- **Task details:** Access the logs for each pipeline step

See the **CI/CD Pipeline** guide for detailed information.

---

## Log Viewing

View your service's application logs in real time from the **Logs** tab.

- Query a specific time range using the date range filter
- Search for keywords within logs using text search
- Watch a specific pod's logs using per-pod filtering

---

## Deployment History and Rollback

In the **History & Rollback** tab, you can see your service's entire deployment history.

### Revision List

The following information is shown for each revision:
- Revision number
- Deploy date and time
- Image tag
- Status (Active, Stable, Revision)

### Rollback

To return to a previous version after a problematic deploy:

1. Find the revision you want to return to
2. Click the **Rollback** button
3. Confirm

The rollback operation reactivates the selected version by creating a new revision.

> **Warning:** Rollback reverts the application's code to the older version but does not revert environment variable and configuration changes.
