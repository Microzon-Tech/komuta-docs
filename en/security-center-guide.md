# Security Center

Komuta Security Center brings together your organization's workload security findings, protection state and event evidence. Start from the **Security** menu; move to a service's **Security** workbench when investigating one workload.

The pages and actions available to you depend on your account's permissions, the current organization and the workload runtime. Check scope, source and timestamps first. An empty result or a low risk score does not establish that a workload is safe.

## Which scope does each screen use?

| Screen | Scope | Purpose |
|---|---|---|
| Customer Console → Security | Workloads and records available to your account in the current organization | Organization-wide prioritization, investigation and permitted response |
| Service → Security | The service currently open | Investigate traffic, protection, findings and event timeline in one service context |
| AdminUI → Security | The displayed scope available to an authorized platform operator | Platform operations, cross-organization investigation and infrastructure security administration |

AdminUI is the platform operator console. Being an administrator within a customer organization does not provide platform operator access. In AdminUI, **platform host scope** differs from **selected organization scope**; selecting a target on one page does not automatically select the target on another.

## Customer Console pages

The **Security** menu has four groups. Permissions or runtime applicability can hide some pages.

| Group | Page | Where to start |
|---|---|---|
| Watch | **Overview** (`/security`) | Check scope, risk, open findings and evidence availability. |
| Watch | **Findings** (`/security/findings`) | Investigate using service, source, severity and time filters. |
| Record | **Audit Log** (`/security/audit`) | Compare recorded actions, actor and outcome. |
| Record | **Login Activity** (`/security/login-activity`) | Investigate organization sign-in activity and unexpected outcomes. |
| Record | **Audit record protection** (`/security/audit-storage`) | Read archive assurance and current verification for your own scope. |
| Protect | **Policies** (`/security/policies`) | Review workload policies and suggestions with their targets. |
| Protect | **Blocks** (`/security/blocks`) | Inspect existing blocks, affected services and evidence. |
| Protect | **Honey paths** (`/security/honey-paths`) | Review decoy paths and detection records for supported services. |
| Protect | **IR Playbooks** (`/security/playbooks`) | Follow investigation and response instructions. |
| Verify | **Synthetic attacks** (`/security/synthetic-attacks`) | Review eligible scenarios, permitted drills and detection results. |
| Verify | **Supply chain** (`/security/supply-chain`) | Inspect build and image scans, artifact evidence and exceptions. |

The former standalone **Violations** address redirects to Findings with the relevant filters. Notification channels, rules, history and silences belong to **Alerts**.

## Observations, findings and evidence

| Term | Meaning |
|---|---|
| **Observation** | Behavior or state recorded by a source. It does not yet establish an attack or a persistent blocking decision. |
| **Violation** | Recorded behavior associated with a protection rule. Audit indicates observation; Block indicates the reported blocking outcome. |
| **Finding** | A security record with an investigation lifecycle. Assess its source, severity, service context and evidence. |
| **Recurrence** | Another occurrence of the same logical finding. Count and last-seen time do not replace an individual event identifier. |
| **Observation summary** | An aggregate count of observations awaiting review at calculation time. It is not a new attack, failed posture check or current queue size. |
| **Evidence** | Source, time, target and outcome information that explains a finding or action. It must be current and match the relevant scope. |

Posture score, pending observation count and event recurrence count are separate measurements. Closing an observation summary does not review its underlying observations or change protection.

## How to interpret protection state

| Displayed state | Meaning | Next check |
|---|---|---|
| **Configured / desired mode** | A rule or setting was saved. | Does it target the intended service and scope? |
| **Pending / queued** | Application has not finished. | What is the deployment outcome and observed mode? |
| **Applied / observed mode** | The platform reports application state at the target. | Is the data current, and does the relevant behavior have the expected outcome? |
| **Effect verified** | Outcome evidence exists for a particular target, behavior and time window. | Does the evidence match current configuration and runtime? |
| **Unknown / unavailable / stale** | The result cannot be verified. | Investigate source access, applicability and freshness. |

A saved Block or Enforce setting, a queued change or a healthy sensor heartbeat does not by itself prove effective workload enforcement. Missing findings do not establish that sources observe every behavior.

Host sensors and runtimes with a separate guest kernel, such as Kata, have different visibility. Host runtime protection can be **not applicable** or **unavailable** in unsupported environments; assess network, posture and supply-chain features using their own eligibility. See [Runtime Security](https://komuta.io/docs/services/runtime-security-guide).

## Overview

Overview helps prioritize risks and workloads for investigation in your organization. Read the risk card alongside open findings; compare source coverage, time window, evidence freshness and the service's protection state.

Check filters when moving from a summary card to Findings or a service. Risk scores and hardening indicators support operational prioritization; they are not compliance certification or a guarantee that all attacks are prevented.

## Findings

1. Narrow the time window, service, source, severity and status.
2. Read first and last seen, recurrences, affected operation and evidence in the detail view.
3. Compare traffic, protection state and event timeline in the service workbench.
4. Select only a decision or response permitted for your account; confirm its target and required reason.
5. Check the recorded decision separately from any deployment or response outcome.

### Finding decisions and actual response

| Decision | Meaning |
|---|---|
| **Acknowledge** | The finding is under review. |
| **Allow** | The behavior is considered legitimate; record the required reason. |
| **Block** | A decision that the behavior should be blocked is recorded. |
| **Dismiss** | The finding is considered invalid or out of scope. |
| **Resolve** | Investigation is concluded; confirm the closure reason. |

These decisions are separate from **creating a runtime block policy**, **applying a policy exception** and **isolating a workload**. A Block decision does not prove kernel enforcement; Allow does not prove an automatically applied allow rule. Actual response requires additional permissions, runtime eligibility, explicit confirmation and outcome checks.

Isolation can affect application network access. Before releasing it, confirm the intended target, recorded isolation state and recovery outcome.

## Policies

Assess workload protection rules and **Suggestions** separately on the Policies page. Host runtime rules and network policies depend on different sources and eligibility. Platform-wide and cluster-wide administration belongs to AdminUI.

The protection policy wizard helps select an eligible service and prepare rules for shell execution, sensitive file access, specific programs or execution from temporary directories. Cluster and namespace come from the selected service. Review target, paths, Audit/Block behavior and YAML preview before creating a policy.

A service appearing in the wizard is not a live sensor health or effective protection check. The platform can apply a created record; check application state and evidence from an authorized test afterwards. Block can disrupt startup, health checks or maintenance.

### Suggested Policies

Review the target, supporting observations, confidence and rule changes. Track acceptance, application and rollback outcomes separately. Failed application or a pending deployment must not be interpreted as active protection. The actions available to your account require their own suggestion-management permission.

### Policy Exceptions

An exception can relax protection for a defined reason and duration. Review scope, approval state, expiry and impact on the current policy. Filing a request, approving it or recording Allow on a finding does not establish that the exception is effectively applied to the workload.

## Blocks and honey paths

On **Blocks**, check the affected service and supporting evidence. Removing a block is a separately permitted action; distinguish an accepted request from a healthy recovered workload.

**Honey paths** monitor service paths that legitimate application behavior should not touch. Verify the target service, runtime support and installation state. An enabled setting is not detection evidence. Do not read, write or probe a decoy file to test it; use only an explicitly approved drill and target.

## Audit Log and Login Activity

**Audit Log** supports investigation of authorized security activity by source, time, actor and outcome. Grouped view summarizes repeated events; raw view helps inspect individual records. Check CSV scope against the selected raw or grouped view and loaded records.

**Login Activity** shows organization sign-in activity. Your personal account security log and active sessions are separate account screens. Verify the affected account, timestamp, result and evidence when investigating an unexpected login.

Login Activity filters, counters and CSV cover **loaded events**. Check the loaded/total indicator; loading older events extends the investigation range. A loading error does not mean no activity occurred.

## IR Playbooks

Playbooks provide ordered investigation and response instructions. Review built-in guidance and, when permitted, manage a copy or a custom playbook for your team's process.

Check each step's prerequisites and expected evidence. Opening a guide does not automatically isolate workloads, rotate credentials or send notifications. Execute supported response only through separate authorized page controls and verify the result.

## Synthetic attacks

Drills help evaluate the detection pipeline for a selected target and scenario. A visible or enabled scenario has not necessarily run. Execution requires a separate permission and an explicit action.

Confirm the target service, runtime, expected signal source, scenario effects and recovery plan first. Compare the run record, expected finding and detection timing afterwards. Detecting one scenario does not prove that every attack class is prevented or that blocking is effective. Platform-wide scenario availability is administered in the AdminUI catalogue.

## Supply chain

Review the scanned artifact, scan time, severity and available evidence when assessing build and image security. Match any SBOM, vulnerability scan or signature information to the relevant build and image; do not rely on evidence for another artifact.

Missing, stale or failed scans are not clean results. Review the reason, scope and expiry of any exception. Accepting a risk does not fix a vulnerability; a signature record does not prove that every build is signed or that its signature was independently verified.

## Audit record protection

The customer view provides **archive protection assurance for your own scope**. Review retention, legal hold, inherited platform settings, external immutability status and current verification together.

A configured archive is separate from **verification of the current configuration**. Pending, catching-up, failed, unknown or unavailable verification is not current immutability evidence. Check the last successful verification timestamp.

Append-only audit records and a cryptographic chain are different guarantees from external **WORM / Object Lock** storage. Storage immutability and any signature verification depend on actual configuration and relevant proof. Operational controls such as bucket, archive destination, legal hold administration and cross-organization selection belong to AdminUI.

## Alerts, notifications and export

Follow security notifications through **Alerts → Rules, Channels, History, Silences and Templates**. Rule matching, channel configuration and actual message delivery are distinct outcomes. Silencing an alert does not resolve a finding or remove its underlying security risk.

Permitted **CSV/JSON finding exports** are prepared server-side using filters and the time window, beyond the current on-screen page. A file contains at most **50,000 rows**; scan limits can also narrow results. Do not treat a capped export as a complete set of all matches; narrow the time window or service scope. Check loaded-record scope separately for Audit Log and Login Activity CSV. Verify file scope, timestamps and records against your investigation purpose.

An exported file is not proof of automated SIEM transfer or notification delivery. If your organization uses an integration, verify its destination, schema, access and delivery evidence separately. This guide does not promise automatic SIEM delivery for every organization or a fixed delivery interval.

## Operator pages in AdminUI

Platform operators use these pages under **Security**, subject to their permissions and the displayed scope. The Customer Console organization view does not replace these administration screens.

| Page | Operator responsibility |
|---|---|
| **Security Findings** | Review permitted cross-organization finding, forensic and telemetry tabs |
| **Observations** | Inspect baseline observations across organizations |
| **Network Incidents** | Inspect permitted cross-organization network incidents |
| **Audit Logs** | Review API audit summaries across organizations |
| **Security signal coverage** | Review source mappings to framework controls in the displayed scope |
| **Service baseline administration** | Manage baseline observations, protection mode and operations for an explicitly selected service |
| **Infrastructure health findings** | Inspect infrastructure findings in platform host scope |
| **Security retention policy** | Manage retention windows and legal hold policy for the displayed scope |
| **Runtime protection controls** | Manage configured enforcement, eligibility and platform controls |
| **Audit storage administration** | Manage archive configuration and verification for the selected scope |
| **Security scenario catalogue** | Manage platform-wide scenario availability and source readiness |

### What does security signal coverage measure?

The screen formerly presented as **Compliance Coverage** is now **Security signal coverage** in AdminUI. It maps recorded security sources to framework controls in the displayed platform host or organization scope.

It is **not a service or cluster assessment**. Source mapping does not establish that a control is operating, protection is effective or an organization is certified. Confirm framework, scope and assessment time; an audit also needs evidence of the control's actual operation. Evaluation and evidence export are separately permitted actions from viewing.

## Page help with the Komuta mascot

In the Customer Console, select **Explain this screen** from the mascot menu. Guidance explains the current security page or service tab, where to start and its limits in Turkish or English. Relevant alert and account security screens also have contextual explanations.

The AI provider does not need to be enabled for this explanation. Page guidance is also available when the decorative mascot is disabled. Static help does not read security records, send data to AI or perform an action. It explains when access is still loading or your permission for the page has not been verified.

In AdminUI, the mascot icon opens **Komuta page guide**, which describes the supported security screen's purpose, prerequisites, next step and limits. This operator guide is not an automatically acting chat or an animated response system.

Keep secrets out of the separate AI chat. AI explanations and recommendations do not replace recorded action results, current authorization, effective protection or compliance evidence. Security changes still require the page's own permissions, confirmations and outcome checks.

## A practical investigation flow

1. Confirm the current organization and service under investigation.
2. Check source coverage and data freshness on Overview.
3. Narrow Findings by service, source and time.
4. Compare traffic, protection and evidence in Service Security.
5. Use a playbook and **Explain this screen** when needed.
6. Explicitly execute only an authorized response; verify its record and actual outcome.
7. Escalate infrastructure, source health, retention or archive issues to a platform operator.

## Related documents

- [Service Security Workbench](https://komuta.io/docs/services/service-security-guide)
- [Runtime Security](https://komuta.io/docs/services/runtime-security-guide)
- [Service access protection](https://komuta.io/docs/services/service-access-protection)
