# Service Security Workbench

A service's **Security** section brings together traffic, protection, findings and evidence for the workload currently open. Use [Security Center](https://komuta.io/docs/services/security-center-guide) for organization-wide prioritization; return to the relevant service when investigating a record.

## Before you start

Confirm the service name and current organization. Check the displayed runtime, source eligibility and data timestamps. An undeployed service, unsupported source, loading error or stale data is not a clean security result.

Each section has separate read and management permissions. Reading a section does not grant finding decisions, isolation, capability changes or protection mode administration.

## Tabs and scope

| Tab | Content | First check |
|---|---|---|
| **Overview** | Service posture, source coverage, highlighted findings and suggested investigation steps | Intended service and current evidence |
| **Traffic** | Flows, network incidents, drops, DNS and available traffic detail | Time window and source availability |
| **Protection** | Network policies, posture and drift, runtime mode and workload hardening | Rule target and application state |
| **Findings** | Service findings, applicable runtime observations and event timeline | Source, event time and affected operation |

Advanced links can open capabilities, writable paths, root permission or runtime observation review in the same service context. Legacy Observations links lead to the appropriate subsection of Findings.

## Overview

Read summaries and next-step guidance alongside source coverage. A count of observations awaiting review is not a count of failed posture checks or new attacks. Low risk and an empty findings list do not establish sensor health or effective protection.

When following a suggestion to another tab, check that the service and time window still match your investigation. Investigate applicability and data flow with a platform operator when a source is unknown, stale or missing.

## Traffic

Select the relevant subsection and time window. Compare connection source, destination, direction, port and reported outcome with the service's network policies.

A dropped-traffic record reports the outcome of that connection; it does not mean all connections are blocked. DNS or application-layer detail is available only when the relevant source and service scope support it. Missing telemetry is not zero traffic.

## Protection

### Network policies and live posture

Compare ingress and egress rules with the application's needs. Use the posture card to investigate differences between expected baseline and reported state of the running workload.

Assess application state, current deployment and source timestamps together when a drift or query error appears. A queued redeployment does not prove that the baseline has been restored.

### Runtime mode

In supported environments, the **Runtime protection mode** card displays **desired mode**, **observed mode** and deployment state separately. Off, Shadow, Audit and Enforce are values of a runtime mode; they are not the same state field as another protection engine's Audit/Block action.

- **Pending / queued**: the change may not have reached the target.
- **Failed**: application is unverified even if the desired setting was saved.
- **Observed mode**: the reported current state; effective blocking of the relevant behavior still needs evidence.
- **Unknown / unavailable**: active enforcement cannot be established.

Read protection state in the customer view. Baseline evaluation, reset, Block/Enforce promotion and platform-wide administration belong to **AdminUI → Service baseline administration / Runtime protection controls**, with their respective permissions. [Runtime Security](https://komuta.io/docs/services/runtime-security-guide) explains the layers.

### Linux capabilities

Review default capability hardening and the permitted narrow allowlist. Adding or removing a capability needed by an image requires the dedicated management permission. Use only the capabilities the application needs.

A saved value does not by itself prove that a running pod changed. Verify the applied deployment and application health.

### Writable paths

Define directories the application actually needs to write to on top of the read-only root filesystem. Check path and storage rules, framework needs and persistence expectations. Server-side restrictions apply to sensitive system paths.

Adding a path requires its own management permission. Verify deployment after saving. Do not test by reading or writing protected files; use an appropriate verification plan.

### Permission to run as root

Consider this override only if the image requires root permission. Granting it does not prove which user a live process runs as. Check the dedicated permission, applied deployment and runtime posture.

### Preview

Review target, rules and changes in any policy or baseline preview. A preview describes intended content; it is not a save, application or effective protection result.

## Findings and evidence

Service findings and event timeline can require different read permissions. One being unavailable does not mean the other is empty or clean.

Review source, severity, first and last seen, recurrence and affected operation in finding detail. Compare the event timeline before and after the event under investigation. Account for loaded records and time filters.

### Runtime observation review

For applicable services, the runtime observation section inside Findings shows recorded behavior and existing decision history. Filter pending, allowed, blocked or closed observations.

An observation is not automatically an attack or a blocking policy. The customer view supports record review; baseline observation administration belongs to the operator screen. Respond to actionable security findings through the authorized Findings response flow.

### Decisions and response

Acknowledge, Allow, Block, Dismiss and Resolve decisions are separate from actual policy application or isolation. An Allow record is not a directly applied allow rule; Block is not direct kernel enforcement. See [Security Center](https://komuta.io/docs/services/security-center-guide) for details.

## Workload isolation

Isolation is a response that restricts workload network access. Availability depends on runtime eligibility and a dedicated isolation permission. Confirm target, reason, expected impact and recovery plan before acting.

An accepted isolation request does not establish measured network containment. A release request also does not prove a healthy application recovery. Check reported state and relevant network and application evidence afterwards.

## Runtime eligibility

A host sensor cannot observe every behavior inside a workload with a separate guest kernel, such as Kata. Some host runtime cards can therefore be not applicable or hidden. That alone does not establish that the workload is unsafe or fully protected.

Assess networking, workload hardening, build scans and isolation using their own eligibility. Do not treat an unknown source as supported or healthy.

## Help with the mascot

Select **Explain this screen** from the mascot menu on the relevant tab. Guidance provides contextual next steps for Overview, Traffic, Protection, Findings and supported advanced sections.

Static help is available in Turkish and English without enabling an AI provider or the decorative mascot. Reading help does not query service records, send data to AI or make changes. Actual actions still use the page's own authorization and confirmation flow.

## Practical flows

### Investigate a suspicious finding

1. Confirm the intended service and organization.
2. Read the finding's source, time and evidence.
3. Compare traffic and event timeline over the same window.
4. Read the relevant playbook and execute only a permitted response.
5. Verify the recorded decision separately from the actual response outcome.

### An application stops working after a protection change

1. Compare the last deployment, desired/observed mode and error record.
2. Inspect evidence for the relevant process, path or connection.
3. Consider required capability or writable path changes with the narrowest scope.
4. Involve the authorized owner for baseline mode or exceptions; blanket Allow decisions are not a remedy.
5. Verify application health and protection outcome after the permitted change.

### Protection evidence is missing

Check runtime support, source access, timestamps and page permissions. Do not report an empty list as a safe result. Escalate source health or platform configuration problems to an operator.

## Related documents

- [Security Center](https://komuta.io/docs/services/security-center-guide)
- [Runtime Security](https://komuta.io/docs/services/runtime-security-guide)
- [Service access protection](https://komuta.io/docs/services/service-access-protection)
