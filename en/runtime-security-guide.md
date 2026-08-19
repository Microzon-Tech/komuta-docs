# Runtime Protection

Komuta provides a kernel-level protection layer to protect your applications against **runtime attacks**: **Runtime Protection**. This layer observes behaviors *inside* your containers — which process is running, which file is being accessed, which network connection is being opened — and blocks out-of-policy behaviors.

This document explains the concepts of the protection layer. For screen usage, see [Service Security Workspace](service-security-guide.md) and [Security Center](security-center-guide.md).

---

## Protection Layers

Security in Komuta is enforced across multiple layers; each layer closes off a different attack surface:

| Layer | Scope |
|--------|--------|
| **Network Policy** | Traffic between services, DNS — restricts communication *between pods* |
| **L7 Rate Limiting** | HTTP request rate limiting |
| **Runtime Protection** | Process / file / capability behaviors *inside the container* |
| **Host Protection** | Critical files and processes on the node itself |
| **Runtime Monitoring** | Advanced behavior telemetry (process lineage tree, system calls) |

Runtime Protection can block the following behaviors:

- **Process execution**: running binaries like `apt-get`, `curl`, `nmap` inside the container
- **File access**: reading/writing sensitive files, making the service account credential file read-only
- **Network access**: which processes can open network connections
- **Linux capabilities**: use of dangerous kernel privileges (raw socket, system administration, etc.)

---

## Automatic Setup and Platform Baselines

The protection layer is installed **automatically** on clusters managed by Komuta — you don't need to do anything:

1. During cluster setup, the protection components are installed as the **first add-on** (security is prioritized).
2. The **host baseline policy** protects the node's critical directories (cluster certificates, credential files, user management tools).
3. The **cluster baseline policy** blocks known attack tools (package managers, network scanning tools) on user workloads.
4. Areas where **platform system components** such as CNI, storage, observability, and GitOps run are **automatically excluded** — conflicts between critical infrastructure and the protection layer have been prevented based on field experience.

### Service Baseline Policy

The deployment pipeline automatically generates a **runtime baseline policy** for each service. This policy:

- Blocks package managers and attack tools (`apt`, `yum`, `wget`, `nc`, `nmap`, etc.)
- Handles `curl` / `wget` **intelligently**: allows them if used in your health probes, blocks them if not
- Blocks dangerous Linux capabilities (raw socket, system administration, kernel module, process tracing)
- Makes the service account credential file **read-only**

Baseline policies are **managed by the platform** and are protected against manual editing. For per-service fine-tuning (adding capabilities, writable directories, additional restrictions), use the service's [Security → Policies](service-security-guide.md#politikalar) tab.

---

## Policy Actions

### Block (Recommended — Production Target)

An out-of-policy operation is **denied at the kernel level**: the system call fails, the process cannot execute the binary. Even if an attacker gets inside the container, the operation never actually happens.

```
$ apt-get update
bash: /usr/bin/apt-get: Permission denied
```

### Audit (Observation Mode)

The violation is **logged but not blocked** — the application continues to run normally. Used to learn the actual behavior before putting a new service into production. Observations accumulated in Audit mode are processed in the [Observations tab](service-security-guide.md#g%C3%B6zlemler) and converted into a baseline.

### Allow (Advanced — Whitelist)

Only explicitly permitted operations run; everything else is denied. Provides maximum security but requires extensive testing.

> Recommended path: **Start with Audit → process observations → upgrade to Block.** For the detailed flow, see [Service Security → Practical Flows](service-security-guide.md#pratik-ak%C4%B1%C5%9Flar).

---

## Violation Tracking

Every event captured by the protection layer flows to the following surfaces:

- **Security Center → Violations**: the live violation feed across the tenant
- **Security Center → Findings**: deduplicated, actionable records
- **Service detail → Security**: service-specific findings and live statistics (Blocked / Critical counts for the last 1 hour in the top strip)
- **Top bar badge**: the Critical + High violation counter is visible on every screen; an instant notification drops on a critical spike
- **Alert rules**: ready-made alert rules for high violation rate, critical block, host violation, and sensor health are defined together with cluster setup; these are routed to your [notification channels](notification-guide.md)

---

## Policy Recommendations and Exceptions

- **Recommended Policies**: The platform automatically generates policy recommendations from observed behaviors ("this access was blocked 47 times on this service — a permanent rule is recommended"). You review the recommendations and apply them with a single click.
- **Policy Exceptions**: For a legitimate workflow that gets caught by policy, you can define a **justified, approval-flow** exception for **up to 90 days**.

Both are managed under [Security Center → Policies](security-center-guide.md#politikalar) and every transition is recorded in an immutable audit trail.

---

## Troubleshooting

### The pod went into "CrashLoopBackOff" and there's a "Permission denied" error

A legitimate behavior of your application was likely blocked in Block mode:

1. Review the recent Blocked records in the service's **Security → Findings** tab — which process, which path?
2. If it's a write block, add an **extra writable directory**; if it's a permission block, add a **Linux capability** (Policies tab).
3. In a complex case, temporarily switch the service to **Audit** mode, collect observations, then **upgrade back to Block**.

### A "high violation rate" alert came in

1. Filter the relevant service in **Security Center → Violations**.
2. Find which process/policy triggered it.
3. If it's legitimate usage: make an **Allow** decision on the finding or define a policy exception.
4. If it's suspicious: follow the matching [incident response playbook](security-center-guide.md#olay-m%C3%BCdahale-playbooklar%C4%B1-ir-playbooks).

### The new policy was not applied to the cluster

- Make sure the policy is **active** (disabled policies are not deployed).
- Check the deployment status field in the policy list; if there's an error, the details appear there.
- If there appears to be a cluster connectivity issue, verify the cluster status from **Security Center → Overview → Cluster Security Matrix**.

---

## Related Documents

- [Security Center](security-center-guide.md) — findings, policies, compliance, SIEM, drills
- [Service Security Workspace](service-security-guide.md) — modes, observations, capabilities, writable directories
- [Notification Management](notification-guide.md) — routing alerts to channels
- [Access Control](access-control-guide.md) — security permissions
