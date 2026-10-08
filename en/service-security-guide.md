# Service Security

Understand what is happening to one service, investigate the evidence and choose a response with a clear view of its impact. **Service detail → Security** brings the service's findings, traffic and protection state into one workspace.

Use [Security Center](security-center-guide.md) to prioritize across your organization, then open a service to investigate its own behavior. For who can open the service's public address, use [Access protection](service-access-protection.md) on **Access & ports**.

> **A useful investigation follows three steps:** check the service and evidence coverage, compare the relevant records, then verify the outcome of any authorized change. A saved setting describes intent; current application and behavior evidence establish what happened.

> Screenshots were captured from the Turkish Komuta UI in the test environment on 8 October 2026. They illustrate the interface; check your own service for its current state.

## Your first visit

1. Select the intended organization and open the service you want to investigate.
2. Open **Security → Overview**. Check the runtime, coverage and freshness information before interpreting the summary.
3. Follow an urgent finding, network incident or dropped-connection summary to the relevant section.
4. Choose a time window that includes the event. Check any active filters and load additional records when available.
5. Review **Protection** before deciding whether the evidence calls for a rule change, a service setting change or an incident response.

A service needs an applicable deployment and available evidence sources for runtime and traffic results. A service that has not been deployed is not ready to produce the same evidence as a running one.

Your role also matters. Reading findings, reading the event timeline, inspecting posture, viewing traffic and making changes can require different permissions. A read-only card can be a valid view for your role. Ask your organization administrator for the specific permission shown if your responsibility requires more access.

## Find the right tab

| Tab | Use it to | Start with |
|---|---|---|
| **Overview** | Prioritize this service's urgent findings, network incidents and evidence gaps | Service identity, coverage and data freshness |
| **Traffic** | Inspect flows, incidents, dropped connections and available DNS detail | Evidence window and connection filters |
| **Protection** | Review network policies, live posture, runtime mode and supported hardening settings | Rule scope and the currently reported state |
| **Findings** | Investigate findings, runtime observations and the evidence timeline | Source, event time and affected behavior |

The incident-response card remains available across the workspace so you can inspect isolation state while moving between tabs.

![Security summary and four security tabs for komuta-test-app](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/service-overview.jpg)

*The Security screen for komuta-test-app keeps Overview (Genel bakış), Traffic (Trafik), Protection (Koruma) and Findings (Bulgular) in the same service context. The top card reports posture; the incident-response card reports isolation status.*

## Overview: decide what needs attention

The overview combines high-priority findings with summaries of network incidents and dropped traffic. Open the relevant summary to continue the investigation in **Findings** or **Traffic**.

Read these summaries alongside coverage. Coverage tells you which evidence is applicable and available for the selected service. An unavailable source, old data or a failed query can leave the result unknown even when a list has no rows.

An urgent view can show only open Critical and High findings. Use **Show all findings** when you need the broader record. Counts describe their displayed scope; pending observation reviews are not an attack counter, and an absence of findings is not a complete security assessment.

## Traffic: connect a symptom to a connection

### Flows and DNS

Select an evidence window and narrow the connection stream by protocol, verdict and connection source or destination as needed. Compare the source, destination, direction and port with what the application should be doing.

The flow view includes security-relevant connections and a representative sample of healthy traffic. Use it for investigation rather than exact totals or a complete record of every connection.

DNS and application-level details help explain a destination when the service and evidence source support them. Their absence does not establish that no name lookup or request occurred. For public address and port configuration, use [Access & ports](services-ports.md).

### Network incidents

Incidents group related network anomalies for investigation. Read the affected service, supporting records and current lifecycle before deciding whether the behavior is expected or needs a response.

Flows and dropped connections use the selected evidence window; incidents show their current lifecycle. Changing the traffic window does not turn an incident list into a historical snapshot of incident status.

### Dropped connections

Use the dropped-traffic section to see which connections were denied and the reported reason. Compare the connection with the rules under **Protection** and with the application's symptoms.

One denied connection is evidence about that connection. It does not prove that every route is blocked, that the whole service is isolated or that every similar attempt will have the same result.

## Protection: understand the current posture

### Network policies

Review the service's incoming and outgoing rules against its legitimate dependencies. A customer request, a health check and an outbound dependency can need different paths through the rules.

Where your role offers a change or policy workflow, check its target and preview before applying it. Afterward, verify the reported application state and the relevant traffic outcome. Public sign-in and IP restrictions have their own [Access protection](service-access-protection.md) settings; a network policy and a visitor access rule answer different questions.

### Baseline and live posture

The baseline is the expected security configuration for the service. **Live posture** reports available checks of the running workload and highlights configuration drift, such as an expected policy being absent or root execution not matching the approved settings.

Compare drift with the latest deployment and approved service settings. A query error means posture could not be established; it is not a failed application test or a clean result. The displayed baseline, a preview and current runtime evidence each answer a different part of the investigation.

### Runtime protection mode

For an applicable service, **Runtime protection mode** is a read-only status card. It separates the requested mode from the observed mode and the deployment state. Mode values are **Off**, **Shadow**, **Audit** and **Enforce**; see [Runtime Security](runtime-security-guide.md) for their meaning.

| State | How to read it |
|---|---|
| **Pending / queued / applying** | The requested change has not been confirmed as applied; inspect the observed mode. |
| **Failed** | A saved request did not complete application. Read the reported failure before taking another action. |
| **Observed mode** | The currently reported mode. Verify relevant behavior separately when you need to establish effective protection. |
| **Unavailable or unverified** | The current mode cannot be established from this view. |

The **Audit / Block** protection action is a separate field from these runtime modes. A Block label states blocking intent for the applicable rules; it is not a blanket result for every operation.

### Linux capabilities

Capabilities grant specific privileges to the application. The card shows the available allowlist, current additions and defaults. Keep the set as small as the image needs rather than adding privileges to address an unrelated startup error.

With capability-management permission, review the required additions or removals, enter a reason and save. Adding privileges requires confirmation. The change affects a subsequent deployment; read the save result and verify application health and live posture after that deployment.

![Linux capability allowlist in the Protection tab](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/linux-capabilities.jpg)

*The Linux capabilities card in Protection lets you compare current additions with platform defaults. The permissions shown belong to the test service; they are not a recommended allowlist for your application.*

### Writable paths

The **Writable paths** card defines directories the application needs to write to when the root filesystem is read-only. It also shows automatically provided paths for a recognized application framework, when available; those paths cannot be edited in this card.

With writable-path management permission, add only the necessary absolute application directories, enter a reason and review the confirmation. The allowed path rules still apply. If current settings cannot be loaded, refresh successfully before editing them.

A writable directory is not automatically persistent storage. Where a persistence option is offered, read its scope and consider what the application must retain across deployments. Verify the result for your service without assuming that every writable path persists.

![Writable paths and reason field in the Protection tab](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/writable-paths.jpg)

*The Writable paths (Yazılabilir yollar) card displays additional directories and the change reason together. The /app/App_Data text is an input example, not a saved path.*

### Permission to run as root

**Allow run as root** is a separate, privileged setting for images that require it. The confirmation explains that this relaxation also affects the read-only root filesystem requirement. Check the image's actual needs before choosing it; a narrow capability or writable directory may address a different requirement with less access.

Changing this setting requires its own permission and a reason. A saved override is not evidence of the user identity of a currently running process. Review the resulting deployment and live posture.

### Preview and change verification

Use **Preview YAML**, when available, to inspect the intended baseline content. A preview does not save or apply a change.

For supported service settings, use this sequence:

1. Record the current setting, relevant symptom and expected improvement.
2. Change the smallest necessary scope and give a meaningful reason.
3. Read the confirmation and save result, including any deployment warning.
4. Check the resulting [deployment history](service-deployment-history.md) and current posture.
5. Verify that the intended application behavior works and that the relevant protection still has supporting evidence.

If settings were saved but application failed or remains queued, retain that distinction in your incident notes. Do not repeatedly save the same change to make an uncertain state appear complete.

## Findings: from evidence to a decision

### Investigate a finding

Open a finding to read its source, severity, affected operation, first and last seen information and available evidence. Compare those details with the service and time window you are investigating. Recurrence helps establish whether you are looking at a one-off event or repeated behavior.

An urgent filter or a partial set of loaded records can narrow the list. Expand the range or load more records when the investigation requires it. A missing row can be a filtering, permission or source-availability issue.

![Service finding showing the policy block, decision provenance and event times](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-block-evidence.jpg)

*Open the relevant record from Findings. This example from the admin test environment shows the policy block for `/tmp/komuta-dropped`, its source and last-seen time in the Turkish UI. We matched the test application's denial to this record; the “Threat” review label alone is not evidence of blocking.*

### Runtime observation review

Applicable services also show recorded file writes, process execution and network attempts with existing review history. Filter **All**, **Pending**, **Allowed**, **Blocked** or **Dismissed** to focus your review.

This section is for reading the observations and their recorded decisions. Respond to an actionable security finding through its authorized Findings workflow. An observation is not automatically an attack, and an Allowed or Blocked review status is not proof that the corresponding runtime rule is effective.

### Evidence timeline

Use the timeline to compare events before and after the behavior in question. Findings and timeline access are independent: you may have permission to see one without the other. Respect the time window and record-loading limits when drawing conclusions.

### Recorded decisions and actual response

Acknowledging, allowing, marking as a threat, dismissing or resolving a finding records a review decision. A decision alone does not apply a traffic rule, prove a denied operation or isolate the service.

For an available response action, inspect the preview, required permission, scope and expected impact. Verify its application and the outcome separately. [Security Center](security-center-guide.md) explains findings, playbooks, suggestions and response workflows together.

## Workload isolation and recovery

Isolation restricts the selected workload's network connectivity and can interrupt normal service and dependency access. It requires an eligible workload and dedicated permission. The preview shows the proposed scope and impact before you confirm.

| Scope | Intended effect |
|---|---|
| **Incoming traffic only** | Remove the workload from serving incoming requests; outgoing connections remain open. |
| **Incoming and outgoing traffic** | Restrict both directions, including access to dependencies. |

Incoming-only isolation is not full containment. Choose scope according to the incident, not merely the shortest interruption.

1. Confirm the service, incident reason, expected impact and recovery owner.
2. Select **Isolate this workload…** and read the preview. An unavailable or rejected preview is not a successful check.
3. Confirm only a supported scope for which you have authorization and enter the required reason.
4. Review the reported result. Pending means the requested restriction is not yet confirmed; failed means isolation must not be assumed.
5. Compare relevant traffic evidence and application behavior with the intended restriction.

When the response is complete, an authorized user can choose **Reconnect this workload**, give a reason and confirm. Verify both the removal of the restriction and the application's recovery. Reconnection is itself a security decision; an accepted request does not prove a healthy recovery.

## Runtime applicability

Available evidence and actions depend on the selected service's runtime. For a **Kata** workload, some behavior observations and runtime-protection operations are not applicable. Networking, build evidence, hardening and isolation must each be assessed according to their own support and reported state.

| Displayed condition | Meaning for your investigation |
|---|---|
| **Applicable with current evidence** | You can assess the records within the stated scope and time window. |
| **Not applicable** | This layer does not apply to the selected runtime; use the other applicable evidence. |
| **Unknown / unavailable** | Support or current state cannot be established; request clarification if needed. |
| **Stale** | The evidence is older than needed to assess the current situation. |

Do not change a runtime solely to make a card appear healthy. Confirm compatibility and application requirements with your service owner or Komuta support.

## Practical investigations

### A service cannot reach a dependency

Open **Traffic** for the incident window. Find the relevant destination and port, then inspect the verdict and any drop reason. Compare the result with outgoing rules in **Protection** and with isolation state. If a change is justified, use the narrowest permitted rule or correction, then verify that the required connection works and unrelated restrictions remain in place.

### A new image fails after deployment

Compare the failure time with deployment history, live posture and relevant observations. Determine whether the application needs a specific writable directory, capability or root permission. Use evidence from the image and failure before relaxing a setting. After an authorized correction, check both application health and the resulting security state.

### A suspicious behavior keeps returning

Check the finding's recurrence and first/last seen times, then compare its timeline and traffic. Read the applicable playbook and identify the permitted response. A resolved review record does not prevent a new event; investigate whether the underlying behavior or exposure changed.

## Troubleshooting

| What you see | What to do next |
|---|---|
| **Loading** | Wait for the section to finish; placeholders are not zero counts. |
| **No records** | Check filters, loaded records, coverage and freshness before concluding there were no observed events. |
| **Stale evidence** | Refresh and compare the latest reported time with the incident window. Escalate a continuing gap to support. |
| **Source unavailable / query failed** | Use the retry control where offered. If it persists, provide service, time and visible error to support. |
| **Permission denied / read-only** | Ask for the specific read or management permission your task needs. |
| **Setting saved, deployment failed** | Inspect the failed deployment and follow the application's normal recovery process with its authorized owner. |
| **Desired and observed modes differ** | Check transition and deployment status; do not report the desired mode as active. |
| **Isolation pending or failed** | Treat containment as unverified and review the action result before deciding the next response. |

## Help in context

Open the mascot menu for guidance matched to the current tab or supported setting. Read the static explanation in the menu; when the mascot is enabled and ready, **Explain this screen** also opens it in a help bubble. Static help is available in Turkish and English, including when AI is disabled. Reading it does not query service records, send data to AI or change protection.

If AI chat is available and enabled, use it as optional assistance within your permitted service or organization scope. Page context is a separate, opt-in setting; enabling it can add a summary of the current page to the conversation. Its explanation does not authorize an action or verify protection. Use the page's own permission and confirmation flow, and assess the actual result afterward.

## Frequently asked questions

### Does an empty Findings tab mean the service is safe?

It means no matching records are displayed. Check source coverage, freshness, filters and permissions to understand what that result covers.

### Why can I read findings but not the timeline or settings?

Those sections have separate permissions. Access to one does not grant access to all evidence or management controls.

### Can I change the runtime protection mode here?

The runtime-mode card reports state. Use the editable controls available to your role for supported service settings; contact your service owner or Komuta support for a required mode change.

### Does Mark as threat stop the behavior immediately?

A finding decision and a deployed protection rule are different records. Inspect any actual response action, its application state and the relevant observed outcome.

### Is Access protection part of the same workflow?

It complements service security by controlling access to a public service address. Configure it under **Access & ports** and use its own activity and status information to assess visitor access.

## Related guides

- [Security Center](security-center-guide.md) — organization-wide prioritization and response.
- [Runtime Security](runtime-security-guide.md) — protection layers, modes and verification.
- [Access protection](service-access-protection.md) — public service access rules.
- [Access & ports](services-ports.md) — service entry points and port configuration.
- [Deployment history](service-deployment-history.md) — compare a change with its deployment result.
