# Security Center

Komuta Security Center brings security findings, service protection, access activity and investigation evidence into one place. Use it to identify what needs attention across your organization, investigate the affected service and follow an authorized response through to its outcome.

You can move from a risk summary to the underlying finding, compare it with service traffic and protection state, review policy suggestions and track the records behind a decision. The same workspace connects these investigations with audit history, build and image evidence, controlled security scenarios and notification workflows.

> Screenshots were captured from the Turkish Komuta UI in the test environment on 8 October 2026. They illustrate the interface; check your own service for its current state.

## Runtime Security: understand and protect running services

Runtime Security helps you investigate what your application does while it is running and manage supported protection controls. Komuta connects available evidence about process execution, file access and network connections with findings, rules and service posture. You can move from suspicious behavior to the affected service, then from investigation to a targeted response.

| Capability | What it helps you do |
|---|---|
| **Behavior observations and findings** | Review recorded program execution, file writes and network attempts; assess severity, recurrence and the event timeline together. |
| **Traffic visibility** | Inspect connection flows, network incidents and denied connections against the service's expected communication. |
| **Behavior rules and suggestions** | Define Audit or Block rules for supported process and file behavior; evaluate suggestions against their evidence and follow application status. |
| **Baseline protection and live posture** | Compare expected security settings with the running service; narrowly manage allowed privileges and writable areas around application needs. |
| **Service isolation and recovery** | Restrict connections for an eligible service through an authorized response; assess the impact in the preview and follow the reconnection outcome. |
| **Controlled verification** | Use supported security scenarios to inspect whether the expected finding appears and how long detection takes. |

**Security Center** provides the organization-wide risk and findings view of these capabilities. For a particular service's traffic, protection settings and detailed evidence, open **Services → the relevant service → Security**. Available controls depend on the service environment and your permissions; assess effective protection using application status and relevant outcome evidence.

Detailed usage guides are under **Services → Service Security**:

- [Runtime Security](runtime-security-guide.md): protection layers, runtime modes, rule actions, applicability and verification steps.
- [Service Security](service-security-guide.md): step-by-step investigation and authorized response across the Overview, Traffic, Protection and Findings tabs.

> **Start with the right context:** confirm the organization, service, time window and freshness of the evidence. A saved protection setting describes intent; application status and observed outcomes tell you what happened.

### A real runtime finding

In this example, **komuta-test-app** in the admin test environment attempted to execute a program it had placed in a temporary directory. The Komuta finding reports that a policy blocked the operation and shows when it was last seen. Match the test result to its source, operation and time when investigating.

![Komuta finding details showing a runtime policy block and last-seen time](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-block-evidence.jpg)

*Real test on 8 October 2026, shown in the Turkish UI: execution of `/tmp/komuta-dropped` was denied. A deduplicated finding can have an older first-seen date; check last-seen time for this run. This result verifies only the operation and rule scope shown.*

Continue to [Runtime Security](runtime-security-guide.md) to compare the test screen with its evidence, or [Service Security](service-security-guide.md) for the steps to open finding details.

## Choose your starting point

| Your goal | Start here |
|---|---|
| Decide what to investigate first | **Security → Overview**, then **Findings** |
| Investigate one application's behavior | [Service Security](service-security-guide.md) |
| Control who can open a public application | **Security → Access protection** and [Access & ports](service-access-protection.md) |
| Understand a protection change | **Policies**, **Blocks** and [Runtime Security](runtime-security-guide.md) |
| Review actions or sign-ins | **Audit Log** or **Login Activity** |
| Assess an image or retained audit evidence | **Supply chain** or **Audit record protection** |

## Your first investigation

1. **Confirm your organization.** Security Center works with records available to your account in the current organization. Open the intended service when you need a narrower view.
2. **Read Overview.** Compare risk, open findings, affected services, coverage and data freshness. Investigate missing evidence before interpreting a quiet dashboard as a healthy result.
3. **Open Findings.** Select the service and time window, then narrow source, severity, type or status as needed.
4. **Read the detail.** Identify the affected operation, source, first and last seen, recurrence and available evidence. Follow the service context to compare traffic and protection.
5. **Choose a next step.** Record an investigation decision, consult a playbook or prepare a response if you have the required permission.
6. **Verify the result.** Check the saved record, application state and relevant behavior separately. Leave unresolved evidence gaps visible when handing the investigation to a teammate.

Reading a page or opening its guide does not change service protection. Applying a policy, creating an exception, running a scenario and removing a block are separate actions.

## Scope and access

**Security Center** provides an organization view of the services and records your account can access. **Service → Security** focuses on the open service. **Account security** covers your own account, sessions and personal security history.

Read permission does not automatically include permission to respond, edit a policy, manage an exception, run a security scenario, export records or inspect archive assurance. Some features also require support in the service's environment. A missing action can reflect permissions or applicability; changing a URL does not grant access.

After switching organization, service or filters, check the displayed context again and wait for the current view to finish loading. A link, count or old browser tab is not evidence that you can access all organization records.

## Find your way around

| Group | Page | What you can review |
|---|---|---|
| Watch | **Overview** | Risks, affected services, recent findings and evidence coverage |
| Watch | **Findings** | Searchable security records, detail, investigation decisions and permitted response |
| Record | **Audit Log** | Recorded actions, actor, time and result |
| Record | **Login Activity** | Organization sign-in activity and unexpected outcomes |
| Record | **Audit record protection** | Retention, legal hold and archive verification for your organization |
| Protect | **Access protection** | Public access exposure and links to service access settings |
| Protect | **Policies** | Applicable protection rules and policy suggestions |
| Protect | **Blocks** | Block records, application state and permitted rollback |
| Protect | **Honey paths** | Decoy path configuration and detection records for supported services |
| Protect | **IR Playbooks** | Investigation and response instructions |
| Verify | **Synthetic attacks** | Available security scenarios, run history and detection results |
| Verify | **Supply chain** | Build and image scans, artifact evidence and exceptions |

The visible pages and controls depend on your permissions and feature availability. Refer to the state and explanation shown on the page when a capability is unavailable.

## Overview: prioritize with context

Overview helps answer **which services need attention and why**. Use the risk summary, severity distribution, recent findings and network threat information to choose an investigation. Read coverage and freshness alongside those summaries.

The main risk card reflects the highest-risk service snapshot available to the summary; read its service name and calculation time. A risk score is a prioritization aid, not a compliance certificate. A posture indicator describes assessed settings; an observation count describes recorded behavior. Neither is an attack counter or proof that every protection layer is effective.

When a summary opens a filtered list, check which filters were carried over. Compare summaries and detail within the same scope and time window; different sources can update at different times.

![Security Center overview with prioritized work, risk and critical findings](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/security-center-overview.jpg)

*The overview combines pending investigations with risk and evidence freshness. In this example, the low risk score appears alongside stale and degraded evidence.*

## Findings: turn signals into an investigation

Findings connect a security signal with its service, severity, status and evidence. Use the available filters to narrow a busy list before making decisions. Search and filters can exclude records even when the visible list is empty. The time window filters findings by their last-seen time and starts at the last seven days by default.

### Understand the record

| Field or concept | How to use it |
|---|---|
| **Source** | Understand which kind of evidence produced the record: runtime behavior, network activity, posture or build security, for example. |
| **Severity and confidence** | Prioritize attention. Confirm the affected service and evidence before treating the record as a confirmed incident. |
| **First and last seen** | Establish when the behavior appeared and whether it recurred. |
| **Recurrence** | See repeated instances of the same logical finding. A total does not identify every individual event. |
| **Affected operation** | Compare the relevant program, path, connection or artifact with expected application behavior. |
| **Evidence and timeline** | Connect source, target, time and outcome. Use related events or process context where the record provides them. |

An **observation** records behavior or state. A **finding** adds an investigation lifecycle. An **observation summary** aggregates observations awaiting review at calculation time; closing that summary does not review the underlying records or alter protection.

Unavailable detail, incomplete source data or an old timestamp should remain part of your assessment. If two records look related, match their service and timing as well as their description.

### Decisions and responses are separate

| Decision | What it records |
|---|---|
| **Acknowledge** | The finding has been seen and triaged; it leaves the Open queue. |
| **Allow** | You consider the behavior legitimate, with the required reason. |
| **Mark as threat** | You classify the behavior as a threat, with the required reason. |
| **Dismiss** | You consider the finding invalid or outside the investigation. |
| **Resolve** | You conclude the investigation with an appropriate explanation. |

These decisions help your team track investigation progress. Mark as threat does not, by itself, apply a blocking rule; Allow does not automatically apply a policy exception.

When the record offers a response, review its target, scope, proposed change and reason in the confirmation flow. Actual blocking, exceptions and workload isolation require their own permissions and support. Isolation can interrupt application connectivity. After a response or its reversal, verify the application state and relevant service behavior before closing the incident.

## Access protection: who can reach the application?

**Security → Access protection** brings the public access state of available services into one organization view. Compare their status, protection end dates and permitted or refused requests over the last 24 hours. Use it to spot services that are open, protected, changing state or need attention, then open the service's **Configuration → Access & ports** workspace.

That workspace separates **Overview, Rules, People, Machines, Activity, Network and Settings**. Depending on support and your permissions, you can configure sign-in and IP requirements, path-specific access, sharing, machine access and protection end dates. **Activity** helps investigate permitted or refused access and requires the relevant management permission.

Access protection concerns entry to the application. Runtime findings concern behavior around a running service, and network policies govern permitted connections. Compare these layers when investigating an access problem; their statuses answer different questions.

Follow the [Access Protection guide](service-access-protection.md) for setup and the [Access Log guide](access-protection-activity.md) for event interpretation. Check application progress and the intended visitor's result after a permitted change; a saved rule or preview alone is not an access test.

## Policies, suggestions and exceptions

### Prepare a protection policy

On **Policies**, review existing rules and their service targets before creating another. The policy wizard supports applicable service rules for program execution, file access and related behavior. Review the selected service, rule details, Audit or Block action and preview before saving.

**Audit** expresses observation intent for matching behavior. **Block** expresses prevention intent for matching behavior in a supported protection layer. Rules are limited by their match conditions and runtime support; choosing Block does not block all behavior. See [Runtime Security](runtime-security-guide.md) for applicability and verification.

Creating a policy in the wizard saves an enabled policy and may lead to automatic application. Treat creation as a protection change, not an inactive draft. If a separate publication control is offered, follow its result as well. A saved record alone is not proof of effective protection; check the displayed application state and errors. Check a service's normal startup, health checks and required connections when evaluating a restrictive change.

### Evaluate a suggestion

Suggestions connect proposed policy changes with their supporting evidence. Review the target, confidence, observed traffic or behavior, and proposed rule before accepting. Follow the available lifecycle actions with the dedicated permission.

Acceptance, successful application and effective protection are separate outcomes. If application is pending or failed, retain that status in the investigation. When rollback is available, review its preview and verify the resulting state afterwards.

### Keep exceptions narrow

From an eligible finding, the exception response can request a policy allowlist exception with a reason and duration. Review its target and preview before confirming. This opens a request that may need approval; it does not itself redeploy the service or prove that protection changed. If the flow instead offers finding suppression, that records a dismissal and does not create a policy exception.

Use the smallest scope needed for the legitimate behavior. Confirm both the intended access and the remaining protection after application or expiry. A risk exception is not a fix for the underlying weakness.

## Blocks and recovery

**Blocks** helps review active and rolled-back records, their affected services, supporting evidence and application status. An applying, failed or unverified record requires further investigation even if the block request was accepted.

For an authorized rollback, read the preview before confirming. A rollback may involve a new deployment; an accepted request is not completed recovery. Check that the intended restriction changed and that normal application behavior returned. Do not remove unrelated protection to resolve one blocked operation.

## Honey paths

Honey paths are decoy paths intended to attract attention when accessed unexpectedly. For a supported service, the page shows saved configuration and detection history. Check the latest deployment separately when assessing whether that configuration is in use.

With the required management permission, select an eligible service to add a path, edit its configuration, change its enabled state or remove it. Review the saved entry and detection history after the action. Confirm that the path is suitable for the application and will not overlap legitimate activity. An enabled setting is not a detected event. Investigate a recorded hit using its service, time and evidence; it still needs context before being classified as an incident.

Do not open or modify a decoy path merely to check whether it works. Use a separately approved scenario with an explicit target and expected outcome.

## Audit Log, sign-ins and personal security

**Audit Log** helps reconstruct recorded activity by source, actor, time and outcome. Grouped views summarize repeated records; raw views help inspect individual events. Select the view appropriate to the question and check how much data is loaded.

**Login Activity** concerns organization sign-ins. Review the affected account, timestamp, outcome and available context when an unexpected login appears. Its filters, counters and CSV describe **loaded events**; use the loaded/total indicator and available older-event loading to understand coverage.

Your personal account security log, password settings and active sessions belong to **Account security**. Service visitor access belongs to **Access & ports → Activity**. A sign-in to Komuta and a request to a protected application are different events.

## Playbooks: make response repeatable

**IR Playbooks** provides ordered investigation and response guidance. Read built-in playbooks and, with the appropriate permission, manage a copy or custom playbook for your team's process.

For each step, identify the target, prerequisite, expected result and responsible person. Opening a playbook does not execute its instructions. Carry out supported actions through the authorized page controls and record the outcome. Use the same sequence during handover so that another teammate can see what remains unresolved.

## Controlled security scenarios

**Synthetic attacks** lets authorized users review supported scenarios, start permitted runs and inspect their results. Use it to evaluate whether the selected scenario produces the expected finding within its detection window.

Before a run, confirm the target service, scenario availability, runtime support, expected signal, possible application impact and recovery plan. A listed scenario or a successful historical run does not establish current readiness.

Afterwards, compare the run state, matching finding and detection timing for that target. A completed action alone is insufficient if expected evidence is missing. Results apply to the tested scenario and window; they do not establish prevention of every attack or effective blocking by every protection layer.

![Synthetic attacks screen showing scenario applicability and drill history](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/scenario-applicability.jpg)

*The Synthetic attacks screen brings together the scenario catalog and drill history. Automated scenarios are not applicable to the runtime shown here; this is not a successful attack-test result.*

## Supply chain: connect evidence to the image

**Supply chain** brings build and image security evidence into the investigation. Review scan status, scan time, severity, the affected artifact and available detail. Where provided, inspect the software bill of materials (SBOM), vulnerability and signature information for that exact build or image.

Match the evidence to the artifact actually used by the service. A newer build, changed image or stale scan may require a fresh assessment. Missing or failed evidence is not a clean result; a build scan also does not inspect every behavior of a running service.

With the required permission, you can accept risk for a selected service and a specific image version for a limited time, or revoke an existing exception. Review its reason, image, expiry and impact on the next deployment attempt. Check the separate deployment admission status: it may be unevaluated, verified, blocked, covered by an exception or audit-only. Accepted risk does not remove a vulnerability. A signature indicator does not establish that every artifact is signed or that every signature has been independently verified.

## Audit record protection

This read-only page helps you assess the protection of your organization's audit records. Review **archive protection**, **retention**, **legal hold**, inherited defaults where shown, and **verification of the current configuration** together.

| What the page shows | How to interpret it |
|---|---|
| **Archive protection configured** | Protection is configured; check the verification result separately. |
| **Verified** with a recent proof | The current external copy has reported verification against the audit records. Interpret it within the displayed scope and time. |
| **Verification pending** | Current protection has not yet been proven. |
| **Protection not verified / unavailable** | Do not rely on external archive protection until the issue is resolved. |
| **Legal hold** | Shows whether a hold applies; review it alongside the applicable retention period. |

A protected audit history and verified external archive protection are distinct assurances. An old successful verification does not validate a changed configuration. If no profile is available, loading fails or verification remains unresolved, contact support with the displayed status and time. This page is not a compliance certification or a control for changing archive retention.

## Alerts, channels and notification preferences

Use **Alerts → Rules, Channels, History, Silences and Templates** to manage supported alert workflows. Rules determine when an alert matches, channels determine destinations, and history helps investigate recorded results. Account **Notifications & Alerts** contains available event-to-channel preferences and channel settings.

Confirm the event, destination and permissions for the workflow you intend to use. If preferences are shown as a preview or unavailable, do not assume that editing is active. A saved channel, matched rule or configured preference does not prove message delivery; check the available delivery result separately.

Silencing an alert suppresses the relevant notification workflow; it does not resolve a finding or remove the condition that caused it. Reading or dismissing an inbox item likewise does not confirm remediation. Test delivery only through an authorized test action to an agreed destination.

## Exports and investigation handover

Where authorized, Findings offers CSV or JSON exports using server-side filters and a time window, beyond the visible table page. One export is limited to **50,000 rows**, and processing limits can further restrict results. Narrow service scope or time window when you need to review a large result set.

Audit Log and Login Activity CSV can have a different scope tied to the selected view or loaded records. Check the page's export description, loaded/total indicator and resulting file before calling an export complete.

For handover, include the organization and service, filters and time zone, finding references, evidence timestamps, decisions made and outcomes still unverified. Share records only with the intended recipients. An exported file does not establish automated delivery to another security system.

## Help and optional AI

Open the Komuta mascot menu to read Turkish or English guidance on the current page. When the mascot is available, **Explain this screen** also opens the explanation in its speech bubble. Relevant service security tabs, access protection and authorized access activity also have contextual help.

This static guide works without enabling an AI provider. When the decorative mascot is disabled, guidance remains readable inside the menu; the speech-bubble action is unavailable. It explains purpose, next steps and limits; it does not inspect current records, send them to AI or execute actions. Loading and permission checks can affect which explanation is available.

If AI assistance is available and enabled, use it as optional help for the authorized context. Page-context sharing starts off in a new session; turning it on can include the supported page summary with your question. Avoid including sensitive information in a prompt and recheck the current organization, service and evidence before acting on a recommendation. An AI explanation is not an approval, verified protection result or guarantee of a correct diagnosis. Changes still follow their own permission and confirmation flows.

## Common investigation scenarios

### A critical finding appears

Confirm the service and evidence time, inspect the affected operation, and compare traffic and protection in Service Security. Use the relevant playbook, record your assessment and perform only the response appropriate to the evidence. Close the finding after checking the outcome, or record why further investigation is needed.

### A customer cannot open a protected application

Start with central Access protection, then the service's Rules, People and Activity tabs. Compare the intended visitor with the applicable rule and reported access outcome. Check application progress before changing the rule; use the [Access Protection guide](service-access-protection.md) to investigate the specific access method.

### A protection change disrupts an application

Compare the last deployment with the relevant finding, desired setting and observed state. Identify the smallest rule, capability, path or connection involved. Use a permitted correction or rollback with a recovery plan, then verify application health and the remaining protection.

### The dashboard looks quiet

Check the organization, time range and filters first. Then inspect source freshness, runtime applicability and errors. If coverage is incomplete, record an evidence gap; an empty list cannot answer whether activity was absent or unobserved.

## Troubleshooting

| What you see | Next step |
|---|---|
| **Loading** | Wait for current scope and permissions to resolve before interpreting counts. |
| **Empty result** | Check filters, time range, access and evidence availability. |
| **Permission denied or missing action** | Ask an organization administrator for the specific access your task needs. |
| **Stale or unknown evidence** | Compare timestamps and source coverage; seek support if freshness does not recover. |
| **Unavailable / not applicable** | Review environment support and assess the other applicable protection layers. |
| **Application failed or still pending** | Inspect the current request and error before submitting a duplicate. |
| **No notification arrived** | Compare rule, channel, preferences, silences and recorded delivery result. |
| **Archive protection cannot be verified** | Treat external protection as unverified and contact support with status and time. |

When contacting [support](support-tickets.md), include the affected page and service, time window, visible status and safe record references. Exclude passwords, authentication material and unnecessary personal data.

## Frequently asked questions

### Does a low risk score mean my service is secure?

It helps prioritize the findings and assessments available in the selected scope. Review coverage, freshness and effective protection evidence as well.

### Does marking a finding as a threat activate a rule?

The decision records your assessment. A supported response or policy action has its own permission, confirmation, application status and outcome verification.

### Why do two services show different protection options?

Options depend on permissions, enabled capabilities and runtime support. Managed isolated runtimes such as Kata do not offer identical behavior visibility. See [Runtime Security](runtime-security-guide.md).

### Is every empty result a successful check?

No. Filters, incomplete loading, missing permissions and unavailable sources can all limit what is visible. Resolve those conditions before drawing a conclusion.

### Can I use a successful scenario or archive badge as a certification?

A scenario result concerns a particular test; archive assurance concerns the displayed record protection. Neither independently certifies your application or organization.

### Do I need AI to use Security Center?

No. Investigation pages, authorized controls and static page help work independently of optional AI assistance, subject to their own availability and permissions.

## Continue learning

- [Service Security](service-security-guide.md) — investigate traffic, protection and findings for one application.
- [Runtime Security](runtime-security-guide.md) — understand applicability and verify protection outcomes.
- [Access Protection](service-access-protection.md) — configure who can open your application.
- [Access Log](access-protection-activity.md) — investigate visitor access.
