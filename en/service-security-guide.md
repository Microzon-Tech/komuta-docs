# Service Security Workspace (Service Security)

The **Security** tab on each service's detail page is that service's dedicated security workspace: you manage the protection mode, review runtime observations, catch drifts, and track service-specific findings all in one place.

For the tenant-wide view, see [Security Center](security-center-guide.md).

---

## Protection Modes

Every service runs in a **runtime protection mode**:

| Mode | Behavior | When to use |
|-----|----------|----------------------|
| **Off** | Protection layer is inactive | Not recommended; special cases only |
| **Shadow** | Policy is prepared but not enforced | Transition preparation |
| **Audit** | Behavior is **observed and logged**, nothing is blocked | Onboarding a new service, learning the baseline |
| **Block** | Out-of-policy behavior is **rejected at the kernel level** | Target mode for production |

Recommended lifecycle:

```
New service → Observe in Audit mode → Process observations → Upgrade to Block
```

In Audit mode, your service's real behavior is learned; once you approve the legitimate behaviors and select **Upgrade to Block**, protection kicks in without breaking your application.

---

## Top Info Bar (Security Summary)

In every view of the Security tab, a summary bar fixed at the top shows:

- **Mode badge** — e.g. *"Runtime Protection: Audit"*. Hover over it to see the full description of the mode (Audit: "observes, logs, does not block"; Block: "out-of-policy operations are rejected at the kernel level").
- **Drift badge** — a red badge appears if the running pod's security posture has diverged from the defined baseline. Clicking it takes you to the details.
- **Three live stats** — Number of pending observations · Number of blocked operations · Number of Critical findings in the last hour.
- **Next-step suggestion** — a one-click prompt based on your current situation: *"Review observations"*, *"Review posture"*, or *"Upgrade to Block"*.

### What does the "Live posture has drifted from baseline" warning mean?

The platform continuously compares your service's **running state** (live pods) against the defined security baseline. Types of drift it detects:

| Drift | Meaning | Action |
|-------|--------|-----------|
| Baseline signature mismatch | The running configuration differs from the expected baseline version | Redeploy the service |
| Running as root | The service is running as the root user but no exemption is defined for it | Make the image non-root or define an exemption |
| Missing network policy | The expected network policy was not found in the cluster | Redeploy / support |
| Missing protection policy | The runtime protection policy was not found in the cluster | Redeploy / support |
| Still in Audit mode | The baseline requires Block but the policy remains in Audit | Complete the "Upgrade to Block" flow |
| Cluster query failed | The cluster could not be reached momentarily (may be temporary) | Check the cluster connection |

---

## Tabs

### Overview

A summary of the service's security status:

- **Critical findings** feed (Critical + High)
- **Most dropped traffic** — a summary of traffic blocked by network policies in the last hour (clicking it goes to the Network tab)
- **Active network incidents** — ongoing network security incidents
- If everything is fine, a mode-aware "clean" status line is shown

> If the service is not connected to a cluster, this tab first directs you to deploy it.

### Network

The service's network security view, with sub-tabs:

- **Flows** — the service's actual network traffic flow
- **Drops** — connections blocked by the network policy (who, where to, which port)
- **Incidents** — incident records generated from suspicious network activity
- **Policies** — network policy rules applied to the service (inbound/outbound rule counts)

> Detailed application-layer (L7 — HTTP path/method level) traffic visibility is part of the **Service Insights** subscription; see [Billing and Plans](billing-plans.md).

### Policies

The tab where you manage the service's security posture. It contains three main cards:

**1. Baseline Posture** — live posture + drift summary. If there is drift, a red dot appears in the tab title.

**2. Linux Capabilities** — By default, your services run with the strictest permission set (all capabilities dropped). The platform defines a small, safe default set so that common applications start up without issues:

| Capability | Typical need |
|---------|---------------|
| `CHOWN` | Adjusting file ownership at startup (e.g. web servers, cache directory preparation) |
| `NET_BIND_SERVICE` | Binding to privileged ports such as 80/443 |
| `FSETID` / `FOWNER` | File permission management |
| `KILL` | Process signaling |

- You can **narrow** this set (tightening is always allowed) or make controlled additions from the list.
- Confirmation is required when adding, **a justification is mandatory**, and the change is recorded in the immutable audit chain.
- A capability that is not on the list (one the platform does not consider safe) cannot be added.

**3. Baseline Overrides** — fine-tuning under a read-only root filesystem:

- **Extra writable directories**: Define the paths your application needs to write to when the root filesystem is read-only (e.g. `/app/uploads`). Sensitive system paths (`/etc`, `/proc`, `/sys`, the service account credential directory) are **rejected** by the platform.
- **Framework automatic paths**: Based on the detected framework, the platform automatically adds the required write paths (e.g. `/app/App_Data` for .NET) — these appear with a read-only badge, so you don't need to add them yourself.
- **Framework key persistence**: By default, the framework write area is ephemeral (it is erased when the pod restarts). If you enable the persistence toggle, this area is moved to persistent, shared storage across replicas — e.g. so that .NET session/anti-forgery keys are preserved across restarts.
- **Additional blocked paths / blocked execution paths**: Add your own restrictions on top of the baseline.
- **YAML preview**: View the full content of the policy that will be applied before making the change.

### Observations

A queue of all behaviors captured by the sensor while the service is **in Audit mode**: which process, which file/network operation, how many times.

Workflow:

1. Run the service in Audit mode under normal load (recommended: a few days, so all business scenarios run).
2. Process the observations one by one: **Allow** (legitimate behavior) / **Block** (unwanted) / **Ignore**.
3. Once no pending observations remain, enable protection with the **Upgrade to Block** step in the top bar.

The badge in the tab title shows the number of pending observations.

### Findings

Security findings belonging to this service (the service-filtered view of the tenant-wide list) + an **evidence timeline**: the event history from the last 7 days, related to findings and decisions. For details on finding decisions, see [Security Center → Findings](security-center-guide.md#bulgular-findings).

---

## Practical Workflows

### Safely onboarding a new service

1. Deploy the service — protection starts in **Audit** mode, nothing is blocked.
2. Run it with normal traffic for a few days; make sure all workflows (cron, reporting, backup, etc.) run at least once.
3. Process the backlog in the **Observations** tab: Allow the legitimate ones.
4. If needed, add writable directories/capabilities from the **Policies** tab.
5. **Upgrade to Block** from the top bar — from now on, every out-of-policy behavior is blocked and generates a finding.

### The pod is giving a "Permission denied" error / went into CrashLoopBackOff

A legitimate behavior was likely blocked in Block mode:

1. Check the latest Blocked records in the **Findings** tab — which process, which path?
2. If it's a write error: add the relevant path to **Policies → Extra writable directories**.
3. If it's a permission error: add the required capability from **Policies → Linux Capabilities**.
4. If the behavior is caught by the policy: give an **Allow** decision on the finding, or define a time-limited [policy exception](security-center-guide.md#politika-istisnalar%C4%B1-policy-exceptions).
5. If you cannot isolate the problem, temporarily switch the service to **Audit** mode, collect observations, and upgrade to Block again.

### I saw a suspicious finding

1. Open the finding's details — process, path, repeat count, first/last seen.
2. Follow the matching [incident response playbook](security-center-guide.md#olay-m%C3%BCdahale-playbooklar%C4%B1-ir-playbooks).
3. Review the context of the event (before/after) from the evidence timeline.
4. Make your decision (Block/Acknowledge) — the decision is recorded in the audit chain along with your justification.

---

## Permissions

| Action | Required permission |
|-------|---------------|
| Viewing the Security tab | View security posture |
| Processing observations, upgrading to Block | Manage security baseline |
| Adding/removing Linux capabilities | Manage capabilities |
| Managing writable directories | Manage writable directories |
| Deciding on service findings | Manage findings |

Permissions are assigned from the **Access Control → Roles** screen; see [Access Control](access-control-guide.md).

---

## Related Documents

- [Security Center](security-center-guide.md) — tenant-wide security console
- [Runtime Protection](runtime-security-guide.md) — protection modes and policy concepts
- [Service Management](service-guide.md) — service lifecycle
