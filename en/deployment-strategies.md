# Deployment Strategies

DevOpsZon provides advanced deployment strategies on top of **Komuta Rollout**, enabling you to safely release new versions of your applications. This guide walks you through each strategy step by step, supported by dashboard screenshots.

---

## Overview

DevOpsZon offers three deployment strategies:

| Feature | Canary | Blue/Green | Auto-Promote |
|---------|:------:|:----------:|:------------:|
| **Traffic control** | Gradual (percentage) | Instant switch (100%) | Automatic switch |
| **Preview environment** | No (single hostname) | Yes (separate hostname) | Yes (separate hostname) |
| **Manual approval** | Yes (each step) | Yes (promote) | No |
| **Traffic percentage control** | Yes (step-by-step increase) | No | No |
| **Instant rollback** | Yes (abort) | Yes (abort) | Yes (abort) |
| **Number of hostnames** | 1 | 2 (active + preview) | 2 (active + preview) |
| **Zero downtime** | Yes | Yes | Yes |
| **Recommended for** | High-traffic services | Critical services | Safe automated deployments |

Each service's active strategy is shown as a badge on the service card:

![Strategy badges on the service list](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/services-overview.png)

---

## Starting a New Deploy

To start a deploy with any strategy:

1. Go to the **Service Dashboard**
2. Click the **Deploy New Version** button above the topology area
3. In the dialog:
   - **Current Version:** Shows the currently running image tag
   - **New Version Tag:** Enter the tag you want to deploy (e.g., `v1.2.3`, `latest`)
4. Click **Deploy Now**

![Deploy dialog — new version tag selection](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/deploy-dialog.png)

### Deploy Lock

When a rollout is in **Paused** state, a new deploy cannot be triggered for the same service. You must first complete the current rollout with **Promote** or **Abort**.

---

## Canary Deployment

The Canary strategy routes a small percentage of live traffic to the new version, allowing you to test it with real users. You progressively increase the traffic percentage at each step.

### How It Works

```
Deploy → 5% traffic → Pause → Monitor → 25% → Pause → Monitor → 50% → ... → 100% → Completed
```

1. New version pods are created
2. A small percentage of traffic is routed to new pods via **Komuta Gateway**
3. The rollout **pauses** and waits for your approval at each step
4. Use **Resume** to advance to the next step, or **Full Promote** to go straight to 100%
5. At any step, use **Abort** to route all traffic back to the old version

### Traffic Steps

Steps are defined in the rollout manifest. The platform automatically executes the required promote commands to reach target weights:

| Step | Canary | Stable | Status |
|:----:|:------:|:------:|--------|
| 1 | 5% | 95% | Paused |
| 2 | 25% | 75% | Paused |
| 3 | 50% | 50% | Paused |
| 4 | 100% | 0% | Completed |

> **Important:** Traffic percentage can only be increased. To decrease, use **Abort**.

### Canary on the Dashboard

When a Canary deploy is active, you'll see:

- **Canary Progress:** A segmented progress bar at the top showing each step's percentage, with fill animation on the active step
- **Topology diagram:** HTTPRoute → Stable Service (80%) + Canary Service (20%), pods and traffic percentages
- **Action buttons:** Resume (next step percentage) and Full Promote buttons
- **Badges:** Canary strategy and Paused status

![Canary Paused — traffic split and progress bar](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/canary-paused.png)

### Actions

| Action | Description |
|--------|-------------|
| **Resume** | Advance to the next traffic step |
| **Full Promote** | Route all traffic to the new version instantly (100%) |
| **Abort** | Cancel the canary, route all traffic back to old version |
| **Pause** | Temporarily halt progression |

### When to Use

- When you want to minimize risk on high-traffic services
- When you need to measure performance of the new version with real traffic
- When you want gradual, controlled rollouts

---

## Blue/Green Deployment

The Blue/Green strategy prepares the new version in a completely separate environment (**Preview**) and switches all traffic in a single step after testing.

### How It Works

```
Deploy → Create Preview → Test → Promote → Clean Old → Completed
```

1. **Active (Blue):** The current live version continues receiving traffic
2. **Preview (Green):** The new version is prepared in separate pods
3. Test the new version via the Preview hostname
4. **Promote** switches all traffic instantly to Preview
5. Old Active pods are cleaned up

### Hostnames

Blue/Green strategy creates two separate hostnames:

| Type | Format | Usage |
|------|--------|-------|
| **Active** | `{service}-{id}.devopszon.com` | Live traffic |
| **Preview** | `{service}-{id}-preview.devopszon.com` | Test traffic |

### Blue/Green on the Dashboard

When a Blue/Green deploy is active:

- **Topology diagram:** HTTPRoute → Active Service (live, 100% traffic) + Preview Service (test, dashed line) → Pods
- **Progress bar:** 4-phase stepper — Creating → Ready/Paused → Promoting → Active
- **Status badge:** "Blue-Green" strategy and "Paused" / "Completed" status

![Blue/Green dashboard — topology and traffic flow](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/bluegreen-dashboard-healthy.png)

When a new deploy is triggered, the rollout enters **Paused** state:

- **Blue-Green Progress:** 4-phase stepper — Creating ✓ → Paused ⏸ → Promoting → Active
- Preview hostname is shown as a separate URL
- **Full Promote** button becomes active

![Blue/Green Paused — Preview ready, awaiting promotion](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/bluegreen-paused.png)

### Actions

| Action | Description |
|--------|-------------|
| **Full Promote** | Route all traffic to the new version |
| **Abort** | Cancel Preview, keep old version running |
| **Rollback** | Revert to previous version |

### Unhealthy Preview

If Preview pods become unhealthy (crash, readiness failure):

- **Full Promote** is not recommended and a warning is displayed
- **Abort** action is highlighted
- Can be re-evaluated after the issue is resolved

### When to Use

- When you need zero-risk switchover for critical services
- When you need comprehensive testing before going live
- When you need to test database migrations

---

## Auto-Promote

Auto-Promote is an automated version of Blue/Green. After the Preview environment is ready and health checks pass, the traffic switch happens **automatically** — no manual approval needed.

### How It Works

```
Deploy → Create Preview → Health Check → Wait → Auto Promote → Completed
```

1. The new version is created in the Preview environment
2. Preview pods become healthy
3. The system monitors for a defined period
4. If no issues are detected, **automatic promotion** occurs
5. No manual intervention **required**

### Auto-Promote on the Dashboard

When Auto-Promote is active:

- **Topology diagram:** Same structure as Blue/Green — Active Service + Preview Service → Pods
- **Progress bar:** 3-phase stepper — Creating → Ready → Active
- **Auto badge:** "Auto Promote" and "Completed" status

![Auto-Promote dashboard — automatic promotion completed](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/autopromote-dashboard-healthy.png)

### Actions

| Action | Description |
|--------|-------------|
| **Abort** | Only in case of error — cancels automatic promotion |

> In Auto-Promote, user intervention is minimal. Promotion happens automatically; you can only intervene with Abort if the preview is unhealthy.

### When to Use

- When you fully trust your CI/CD pipeline
- When you want to eliminate manual approval bottlenecks
- For fast iteration in development and staging environments

---

## Changing Strategy

You can change a service's rollout strategy at any time:

1. Go to the **Deployment History & Rollback** page
2. Click the **Strategy** button at the top right
3. Select the new strategy from the dropdown
4. Confirm with **Change Strategy**

![Strategy change dialog](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/strategy-change-dialog.png)

> **Note:** Strategy changes take effect with the next deploy. Currently running pods are not affected; new deployment YAMLs are generated with the selected strategy.

---

## Rollout Management & History

On the **Deployment History & Rollback** page, you can manage your rollouts:

- **Status card:** Current rollout status, strategy, replica info
- **Operations:** Promote, Rollback, Abort, Retry, Pause, Resume buttons
- **History table:** All revisions with image info, traffic percentage, status, and timestamps

![Rollout history and operations](https://raw.githubusercontent.com/Microzon-Tech/docs/main/user-guide/img/deployment-strategies/rollout-history.png)

---

## Rollout States

During the deploy process, your service goes through the following states:

| State | Visual | Meaning |
|-------|--------|---------|
| **Progressing** | Animated progress | Deploy in progress, new pods being created |
| **Paused** | Yellow badge | Awaiting user action (Canary step or Blue/Green promote) |
| **Healthy / Completed** | Green badge | Deploy completed successfully, all pods healthy |
| **Degraded** | Red badge | Some pods unhealthy; intervention may be needed |

All states are updated in **real-time** — no need to refresh the page.

---

## Stale Rollout Policy

In Canary or Blue/Green strategy, a rollout can remain in **Paused** state for an extended period. DevOpsZon provides a platform-level policy to automatically manage stale rollouts:

- Platform administrator defines a **maximum wait time** (e.g., 60 minutes)
- When the time is exceeded:
  - If Preview is healthy → **automatic Full Promote**
  - If Preview is unhealthy → **automatic Abort**
- Specific services can be **exempted** from the policy

> **Note:** Auto-Promote strategy manages its own automatic promotion, so it is outside the scope of the stale rollout policy.

---

## Strategy Selection Guide

| Question | Recommended Strategy |
|----------|---------------------|
| I want to test the new version with live traffic | **Canary** |
| I want to test in an isolated environment and switch in one step | **Blue/Green** |
| I want automatic promotion without manual approval | **Auto-Promote** |
| Rollback must happen instantly | **Blue/Green** or **Auto-Promote** |
| I need to control traffic percentage step by step | **Canary** |
| I need to test with a Preview URL for QA | **Blue/Green** or **Auto-Promote** |

---

## Storage Compatibility

Canary, Blue/Green and Auto-Promote all keep the **stable pod and the new preview/canary pod alive at the same time** during a rollout. If your service has a `ProjectRepoServiceStorageAttachment` (a PVC) bound to it, both pods have to mount that volume **concurrently** — otherwise Kubernetes rejects the new pod with a "Multi-Attach error" and the rollout gets stuck in the "Progressing" state forever.

That's why the DevOpsZon pipeline **always provisions PVCs as `ReadWriteMany` (RWX)**, regardless of tier, strategy, or replica count. Consequences:

- **Every plan (including Free) supports every strategy.** You no longer need a Small plan or higher to run Canary — the Free plan also works with Canary / Blue-Green / Auto-Promote.
- **HPA and multi-replica** can be enabled on any plan. The "Always RWX" rule makes the platform behave uniformly across tiers.
- **Plan changes and strategy switches** no longer risk Multi-Attach errors, because every volume is already RWX — stable ↔ preview transitions just work.

### Architecture note

For the full rationale behind the "Always RWX" decision and the underlying Longhorn StorageClass layout, see [Storage RWX Architecture](../../storage-rwx-architecture.md). RWX volumes are served through Longhorn's share-manager over NFS; performance notes for random-I/O-heavy workloads (databases, for example) are documented there as well.

### Legacy RWO services

Services created before the "Always RWX" refactor still hold `ReadWriteOnce` (RWO) volumes. Kubernetes treats a PVC's access mode as immutable, so the next deploy of those services will be stopped by the platform with a "StorageAccessMode mismatch" error.

Resolution: **delete and recreate the service.** The new service automatically gets an RWX PVC. If you need to migrate data, snapshot the old PVC before deleting the service and restore it into the new one manually.
