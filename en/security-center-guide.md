# Security Center

Komuta Security Center lets you monitor and manage the security posture across all your clusters and services **from a single console**. It is built on runtime telemetry: it observes at the kernel level what actually happens inside your workloads (which process ran, which file was accessed, which network connection was established), turns that into findings, and lets you take action.

Access it by clicking **Security** in the left menu.

---

## How Does Security Center Work?

```
Runtime Sensors  →  Telemetry Collection  →  Finding Generation  →  Action
(Runtime Protection,  (Komuta Telemetry     (deduplication,        (decision, notification,
 Runtime Monitoring,   Engine)               risk scoring)          SIEM, compliance)
 Network Flow Observation, Posture)
```

1. **Runtime sensors** deployed to your clusters collect in-container behavior (process, file, network) and posture information.
2. Events are **deduplicated**: when the same logical event recurs, no new record is opened — the counter and "last seen" time of the existing finding are updated instead.
3. A **risk score between 0–100** is calculated for each service and updated every 5 minutes.
4. Findings automatically flow to notification channels, your SIEM system, and the compliance dashboard.

### Core Concepts

You'll see four distinct concepts in Security Center. The difference between them matters:

| Concept | What does it mean? | Where does it appear? |
|--------|------------------|-----------------|
| **Observation** | The raw behavior *seen* by the sensor on a service in Audit mode (not yet decided upon) | Service detail → Security → Observations |
| **Violation** | The live stream of behaviors that trip a protection policy | Security → Violations |
| **Finding** | A deduplicated, permanent, actionable security record | Security → Findings |
| **Evidence** | The timestamped event history associated with a finding or decision | Service detail → Security → Findings (timeline) |

In short: **observations** are the raw data of the learning phase, **violations** are the "what's being detected right now" stream, and **findings** are the work list you manage.

---

## Overview

The landing page of Security Center summarizes the security posture across the tenant.

### Risk Score (0–100)

The risk card at the top left shows the combined risk level of all your services. The score is **explainable** — it's not a black box; the contribution of each component can be tracked individually:

- **Finding severity**: Critical findings carry the highest weight, Info the lowest.
- **Source reliability**: Kernel-level observations are evaluated with a higher confidence coefficient than posture checks.
- **Freshness**: The impact of older findings decreases over time (a 24-hour half-life curve).
- **Blast radius**: The service's tier (criticality level), internet-facing endpoints, and use of privileged identities increase the score.
- **Recurrence factor**: The number of times the same finding recurs is added to the score logarithmically.

A service with no findings is not considered "risky" just because it's internet-facing — the score is designed not to produce noise.

### Other Overview Cards

| Card | Content |
|------|------|
| **Open Findings** | Total number of open findings + critical/high breakdown |
| **Clusters / Services** | Number of connected clusters and sync status |
| **Critical Findings** | Live list of open findings with Critical and High severity |
| **Riskiest Services** | List of services sorted by risk score; clicking goes to the service's security page |
| **Cluster Security Matrix** | Per-cluster version, node status, platform protection layers (Baseline badges), and online status |
| **Service Hardening** | The percentage of the 8 protection layers applied for each service |

**Service hardening layers**: Network Policy, Rate Limit, Runtime Protection, TLS, Resource Quota, Limit Range, Service Account hardening, and Non-root execution. Hover over each point to see which layer is missing.

The **"Near real-time · telemetry-based"** badge at the top right of the page indicates that the data comes from a live telemetry stream.

---

## Findings

The main screen where all security findings across the tenant are managed.

### Severity Levels

| Level | Meaning |
|--------|--------|
| **Critical** | A finding requiring immediate response, which may indicate active exploitation |
| **High** | An important risk that needs to be addressed quickly |
| **Medium** | A risk to be remediated in a planned manner |
| **Low / Info** | A low-priority record for awareness purposes |

### Finding Lifecycle

```
Open ──► Acknowledged ──► Allowed / Blocked / Dismissed ──► Resolved (closure)
```

| Decision | What does it do? |
|-------|-----------|
| **Acknowledge** | Marks "seen, investigating" — the finding stays open |
| **Allow** | The behavior is deemed legitimate; **a justification is required** |
| **Block** | Decides the behavior must be blocked; **a justification is required** |
| **Dismiss** | Closes the finding as invalid/unimportant |
| **Resolve** | Permanent closure (cannot be reopened; if the same behavior recurs, a new finding is created) |

Key points:

- Decisions can be made **individually or in bulk** (up to 200 findings at a time).
- Allow/Block decisions are available only to authorized roles; the **developer role** can only perform Acknowledge and Dismiss.
- Every decision is recorded, along with who made it and the justification, in an **immutable audit chain** (see [Audit Storage](#audit-storage)).
- When the same logical finding is detected again, no new row is opened; the **Recurrence count** increases and **Last seen** is updated. This way, an event that repeats 400,000 times is managed in a single row.

### Filtering

You can narrow the findings list using the search box, the source filter (Runtime Protection, Runtime Monitoring, Network Flow Observation, Posture, Identity, Cluster Health), and the severity filter. The statistic chips at the top (Total / Open / Critical / High) show the current status at a glance.

---

## Violations

Shows the **live stream** of behaviors that trip protection policies: which pod, which process, and with which action (Blocked / Audited) it was caught.

- Difference from the Findings screen: violations are raw and instantaneous; findings are deduplicated, actionable records.
- The **bell badge** at the top right (visible on all pages) shows a live counter of Critical + High violations; an instant notification drops on a critical spike.

---

## Policies

A unified view of security policies across all your clusters.

### Policy Types

| Type | Scope | Status |
|-----|--------|-------|
| **Runtime Protection** | In-container process/file/network behavior rules | Active |
| **Network Policy** | Rules for traffic between services | Coming soon to this screen |
| **Runtime Monitoring** | Advanced behavior observation rules | Coming soon to this screen |

Each policy row shows the service/cluster information, application status, and action (Block/Audit); you can inspect the rule content from the detail panel. For details on policy concepts, see [Runtime Security](runtime-security-guide.md).

### Suggested Policies

The platform analyzes the actual behavior of your services and generates **policy suggestions**. Example justification: *"Sensitive file read attempts on this service were blocked 47 times in the last 24 hours — a permanent rule is suggested."*

Suggestion lifecycle: **Pending → Accepted → Applied**. Pending suggestions can be rejected or automatically expire after 30 days. An applied suggestion can be rolled back when needed. All transitions are written to the audit chain.

### Policy Exceptions

You can define a **time-limited exception** for a legitimate workflow that trips a policy (e.g., a maintenance window, a known behavior of a third-party tool):

- Exceptions go through an approval flow: **Pending → Approved → Active → Expired / Cancelled**.
- The maximum exception duration is **90 days** — a permanent hole cannot be opened.
- Every exception is justified and recorded in the audit chain.

---

## Audit Log

A unified timeline of events on the platform management plane: who did what, when, and what the result was (success / error).

- Filter using source chips: Identity, Runtime Protection, Network Flow Observation, Posture, Audit, etc.
- A minimum severity and a date window can be selected.
- API calls and authentication server requests (including 4xx/5xx errors) are tracked on this screen.

---

## Login Activity

A record of identity events on your account:

- Successful / failed logins, logouts, account lockouts
- Each record includes the IP address, browser/client information, and timestamp
- You can report on records via **CSV export**
- From a suspicious login, you can jump via a **forensic investigation link** to other security events in the same time window

This screen is the first place to check for "who logged in at this time?" and "were there any failed login attempts?"

---

## Incident Response Playbooks (IR Playbooks)

These are ready-made response guides that describe **step by step what you need to do** when you encounter a security finding. They are automatically matched to the finding's severity, type, and source.

Ready-made playbooks that ship with the platform:

| Playbook | Trigger | Recommended steps |
|----------|------------|------------------|
| **Sensitive file read** | Critical runtime finding | Open incident → Isolate pod → Page service owner → Prepare forensic package |
| **Service account token + external traffic** | Critical runtime finding | Open incident → Isolate pod → Rotate credentials → Suggest network block → Page owner |
| **Post-deploy shell execution** | Medium finding | Verify health probe → Flag for review |
| **Package manager at runtime** | High finding | Open incident → Suggest block rule → Flag for review |
| **Cloud metadata access** | High network finding | Open incident → Suggest network block → Rotate credentials → Page owner |

- Ready-made playbooks cannot be edited; however, you **can clone them and customize them for your own process**.
- Playbook steps are currently **guidance only** — your team executes the steps; automated execution is on the roadmap.

---

## Compliance

Your security findings and protection layers **automatically provide evidence** for industry-standard compliance controls. You don't need to set up a separate scanning tool.

### Supported Frameworks

| Framework | Example controls |
|---------|------------------|
| **CIS Kubernetes** | Existence of network policy, privileged pod audit |
| **NIST 800-53** | Access control (AC-3), audit events (AU-2, AU-9), system monitoring (SI-4) |
| **SOC 2** | Logical access (CC6.1), system monitoring (CC7.2) |
| **ISO 27001** | Monitoring activities (A.8.16), network security (A.8.34) |
| **PCI-DSS** | Audit logging (10.2.1), change detection (11.5.1) |

### Coverage Statuses

The status for each control is calculated based on the most recent quarterly evidence window:

- **Met** — sufficient evidence exists for the control
- **Partially Met** — evidence exists but is incomplete
- **Missing Signal** — no data source has been connected for this control yet
- **Not Applicable / Not Assessed**

### Actions

- **Export Evidence**: Download an evidence package in JSON or CSV format to present to your auditor.
- **Reassess**: Reassess a framework instantly, outside of schedule (e.g., before an audit).

---

## Notifications and Routing Rules

Here you configure how security findings reach your team.

### Notification Channels

| Channel | Description |
|-------|----------|
| **In-app** | Instant notification within the console |
| **Email** | Email to defined addresses |
| **Slack** | Slack webhook integration |
| **Microsoft Teams** | Teams channel card |
| **PagerDuty** | On-call triggering |
| **Webhook** | HTTPS POST to your own system |

Channel configurations (webhook address, token, etc.) are stored encrypted and are not shown to read-only users.

### Routing Rules

You determine which finding goes to which channel using rules:

- **Severity threshold**: e.g., High and above only
- **Source / cluster / service / owning team** filters
- **Business hours window**: limit notifications to your working hours (default 09:00–18:00, local time)
- **Escalation**: If a finding is not resolved within a defined period (up to 24 hours), a second notification goes to the escalation channels
- **Deduplication**: The same finding does not repeatedly trigger notifications

---

## SIEM Export

Automatically export your security findings to your own SIEM / log platform. Designed for integration with your existing security operations center (SOC).

| Schema | Example compatible destinations |
|------|------------------------|
| **OCSF** | Splunk, AWS Security Hub, Snowflake |
| **Elastic ECS** | Elasticsearch / Kibana |
| **OpenTelemetry** | OTel Collector, Honeycomb |

How it works and security guarantees:

- Findings are sent to your destination **incrementally every 5 minutes** (resuming where it left off).
- The destination address **requires HTTPS**; the authentication token is stored encrypted and is never shown back in the interface.
- After **5 consecutive failures**, the export is automatically disabled and a notification is generated.
- All configuration changes are recorded in the audit chain.

---

## Synthetic Attacks

**Prove that your security monitoring actually works.** The platform triggers a controlled, harmless "attack signal" on your own workload and measures whether the detection pipeline catches it, and **how long it takes to catch it**.

### What is it for?

In classic security tools, "no alarm" can mean two things: either everything is fine, or the monitoring is broken. Synthetic attacks eliminate this ambiguity — you find **detection coverage gaps** before an attacker does.

### Scenario Catalog

Ready-made scenarios include running a suspicious process, writing to a sensitive directory, external traffic from a restricted workload, and a privileged pod attempt. Each scenario has a **detection SLA** (e.g., 60 seconds).

### Running an Exercise

1. Select a scenario from the **Synthetic Attacks** page and specify your target service.
2. Click **Trigger Exercise**.
3. Watch the results dashboard: **Detected** (within SLA) / **Not Detected** (coverage gap!).

The metric cards at the top show the total number of exercises, detection rate, SLA compliance rate, and average detection time.

> Exercise records are kept **isolated** from real findings: they do not pollute your risk score, your SIEM stream, or your compliance evidence.

---

## Honey Paths

Define a **decoy file path** on one of your services: a path your legitimate code should never touch (e.g., a fake credentials file). **Any access** to this path immediately generates a Critical finding and suggests the relevant incident response playbook.

- Service selector → add a decoy path (e.g., `/app/config/.fake-credentials`).
- Track which decoys have caught traffic via the **Last triggered** and **Trigger count** columns.
- Honey paths are a low-cost, high-signal method for catching internal lateral movement and automated scanning behavior.

---

## Audit Storage

Every security-related operator decision (finding decisions, policy suggestion transitions, exceptions, SIEM configurations, exercise triggers, capability/writable directory changes) is written to an **immutable audit chain**.

- **Append-only**: Records cannot be updated or deleted.
- **Cryptographic chain**: Each record contains the hash of the previous record; if a single past record is altered, the rest of the chain becomes invalid and the verifier detects it. You give your auditor the guarantee that "these records have not been tampered with."
- **Legal Hold**: Prevent records from being deleted during a legal process, even if their retention period has expired (toggle on/off with justification).
- **Archive destination**: The storage destination and retention period for a long-term archive copy of the records can be configured.

This structure is designed to directly satisfy the "audit log integrity" requirements in SOC 2, ISO 27001, and PCI-DSS audits.

---

## Other Pages

| Page | Content |
|-------|------|
| **Runtime Enforcement** | Platform-wide Monitor ↔ Enforce mode; changing modes requires a justification and passes a readiness check. For administrators. |
| **Cluster Health** | Infrastructure-layer health findings (storage, node components). |
| **Retention Policy** | Retention period and data residency settings for security data. Requires administrator privileges. |

---

## Permissions

Security Center capabilities are authorized individually via **Access Control → Roles**:

| Capability | Who should use it? |
|---------|-----------------|
| Security Center read access (overview, findings, violations) | Entire development team |
| Finding decisions (Allow/Block/Resolve) | Security operator / team lead |
| Finding decisions (Acknowledge/Dismiss) | Developer |
| Notification channel and routing rule management | Security operator |
| Policy exception and suggested policy management | Security operator |
| SIEM export configuration | Security operator |
| Compliance viewing / evidence export | Auditor role |
| Triggering synthetic exercises, honey path management | Security operator |
| Audit storage viewing | Auditor / security operator |
| Retention policy, enforcement mode, archive management | Platform administrator |

Best practice: give developers read + acknowledge access; limit permanent decisions like Allow/Block, and channel/exception management, to security officers.

---

## Frequently Asked Questions

**What's the difference between "Findings" and "Violations"?**
Violations are the instantaneous raw stream; findings are the deduplicated, actionable state of the same event. Run your daily operations from the Findings screen; use Violations for live observation.

**Why did my risk score change?**
The score is recalculated every 5 minutes. A new Critical finding raises the score quickly; as findings are resolved and time passes (the freshness curve), the score decreases.

**I accidentally Resolved a finding — can I undo it?**
No — Resolve is permanent. If the same behavior is detected again, a new finding is created. Use Acknowledge when you're not sure.

**I'm not receiving notifications.**
Check in order: (1) Does the routing rule's severity threshold and filters match the finding? (2) Are you outside the business hours window? (3) Did the channel test succeed? (4) If a notification was already sent for the same finding, deduplication is in effect.

---

## Related Documents

- [Service Security Workspace](service-security-guide.md) — security management for a single service
- [Runtime Security](runtime-security-guide.md) — protection modes and policy concepts
- [Access Control](access-control-guide.md) — role and permission management
- [Notification Management](notification-guide.md) — platform-wide notification channels
