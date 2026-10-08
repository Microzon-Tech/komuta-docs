# Runtime Security

Runtime security helps you assess a service while it is running: the behavior recorded for it, the restrictions intended to protect it and the evidence that those restrictions reached the workload. Komuta brings these signals together so you can investigate suspicious activity and make targeted changes without losing sight of normal application behavior.

This guide explains the protection model. For the screen-by-screen workflow, read [Service Security](service-security-guide.md). For priorities and responses across your organization, start with [Security Center](security-center-guide.md).

> **Keep three facts separate:** what is configured, what is reported as applied and what the observed outcome shows. This distinction is useful both when evaluating protection and when recovering an application after a change.

> Screenshots were captured from the Turkish Komuta UI in the test environment on 8 October 2026. They illustrate the interface; check your own service for its current state.

## Protection layers and what each answers

| Layer | Customer question | Evidence to review |
|---|---|---|
| **Public access protection** | Who may open this service's public address? | Access rules, protection status and relevant access activity |
| **Network policies** | Which incoming and outgoing connections are permitted? | Rule scope, application state, flows and dropped connections |
| **Workload hardening** | Which privileges and writable areas does the application need? | Approved settings, deployment result and live posture |
| **Runtime behavior protection** | What process, file or other supported behavior was observed, and which rules apply? | Runtime eligibility, protection action, observations and finding evidence |
| **Build and image evidence** | What did checks report about the image being delivered? | Available scan results for the relevant build and image |

These layers complement one another. A visitor passing an access check does not establish that the application's behavior is safe. A build scan does not inspect everything that a running workload will do. Network isolation can restrict connections without explaining how an application reached its current state.

Use [Access protection](service-access-protection.md) for visitor rules and [Access & ports](services-ports.md) for service entry points.

## Establish scope before interpreting a result

Start with the selected organization, service, runtime and deployment. Confirm that the evidence belongs to the service and period you are assessing. Organization-wide summaries help prioritize; service evidence provides the detail needed for a specific investigation.

Check these conditions together:

- **Applicability:** does this layer support the selected runtime and service?
- **Availability:** can the current view obtain the relevant evidence?
- **Freshness:** is that evidence recent enough for the event or change?
- **Permission:** can your role read the section or perform the proposed action?

### Kata and other applicability limits

For a **Kata** workload, some runtime behavior observations and protection operations are not applicable. Read the service's coverage and eligibility information instead of assuming that every runtime exposes the same capabilities.

Evaluate network rules, workload hardening, build evidence and isolation separately. A runtime restriction on one layer does not decide the support or effectiveness of another. In particular, isolation has its own eligibility check and may be available for an eligible isolated runtime.

An unavailable card is not a finding, and a hidden card is not proof of complete protection. If applicability is unknown, ask the service owner or Komuta support to clarify the supported scope.

![Five supported capabilities in the admin test service, including host runtime detection and enforcement](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/admin-runtime-capabilities.jpg)

*The admin test environment’s komuta-test-app supports five capabilities, including host runtime detection and enforcement. The Turkish UI’s Supported label is not, by itself, a test result proving effective protection.*

### Separate managed PaaS from the admin test environment

**Komuta's managed isolated VM/Kata PaaS does not support host runtime process/file/system-call detection, host runtime blocking or HoneyPath access detection.** The admin test service in this guide runs in a host-observable non-Kata environment. Do not present its successful block as a PaaS result.

![Managed isolated PaaS service capabilities for network, isolation, supply chain and application layer](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/service-capabilities.jpg)

*A separate example from the managed isolated runtime: network detection, workload isolation, supply chain and application layer are shown. This list does not establish host runtime support.*

The network layer can still apply to an isolated runtime. Isolating and reconnecting an eligible customer service is independent of host runtime blocking. Finding decisions remain available for existing records with permission; a stored record does not prove that the environment can produce host runtime events. Public access protection, build/image evidence and hardening have their own applicability conditions.

See [Service Security](service-security-guide.md) for action-specific PaaS availability and real forms, and [Security Center](security-center-guide.md) for HoneyPath setup screenshots.

## Runtime mode and protection action

### Read the runtime mode

The service's **Protection** tab includes a read-only **Runtime protection mode** card for applicable workloads. It shows requested and observed state, including whether application is pending or failed.

| Mode | Intended behavior of this runtime layer |
|---|---|
| **Off** | This runtime baseline is turned off. Other layers retain their own configuration and state. |
| **Shadow** | Learn the workload's behavior while applying the hardening supported by this mode; do not interpret it as active behavior blocking. |
| **Audit** | Observe and record behavior for investigation and review. |
| **Enforce** | Apply blocking protection to the supported, configured scope when application completes. |

Do not assume that every service begins in the same mode or that every mode is available for every runtime. The card reports state; it is not a mode selector. For a required mode change, contact your authorized service owner or Komuta support.

### Read Audit and Block separately

A runtime rule can also have an **Audit** or **Block** action. This action belongs to the relevant rule or protection layer; it is not the same state field as the four runtime modes.

| Action | Meaning |
|---|---|
| **Audit** | The rule is intended to observe and record matching behavior without blocking through that rule. |
| **Block** | The rule is intended to deny matching behavior when it is supported and effectively applied. |

Audit does not remove network restrictions or blocking from another rule. Block does not mean all activity is denied. The matched operation, target, rule scope and runtime support determine what the rule can affect.

A restriction on executing a program is also different from a restriction on writing to its directory. When interpreting path-based rules, confirm what operation and scope they actually describe.

## Configuration, application and observed outcome

Use three checkpoints to explain the state of a control:

| Checkpoint | What it establishes | What remains to verify |
|---|---|---|
| **Configured** | A setting or rule was saved for a target | Whether it reached the running workload |
| **Applied / observed** | The service reports the configuration or mode as applied | Whether the expected behavior is allowed or denied in the relevant circumstances |
| **Observed outcome** | A specific event or connection produced the reported result | Other paths, workloads and conditions outside that evidence |

A preview describes intended content. Approval permits a workflow to continue. A queued deployment indicates pending work. None of these alone demonstrates effective blocking.

When desired and observed modes differ, read the transition and deployment state. If application failed, the saved desired setting may still be visible while the previous observed state remains relevant. If the mode is unavailable or unverified, do not fill the gap with an assumption.

Match evidence to the configuration and deployment being assessed. A successful observation from an older version does not verify a later change. Similarly, a reconnect or rollback request needs a current result and application-health check before recovery can be considered complete.

## Baseline and application requirements

The baseline expresses the expected security configuration. Live posture compares available information from the running service with those expectations and can reveal drift.

Supported customer settings include narrowly allowed **Linux capabilities**, **writable paths** and **permission to run as root**, each with its own management permission. Their purpose is to meet an identified application requirement while keeping the remaining restrictions intact.

### Choose the smallest necessary change

For a startup failure or denied operation, first identify the process, path or connection involved. Compare it with the image's requirements, deployment history and current settings. A general permission error does not by itself establish that the application needs root or broad write access.

For an authorized change, record a clear reason, read its confirmation and check the deployment result. When a root-filesystem restriction applies, a writable-path exception has a different scope from allowing root execution. A writable directory also needs a separate persistence assessment if the application must retain its contents.

![Linux capabilities card for service runtime requirements](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/linux-capabilities.jpg)

*The Linux capabilities card under Service Security → Protection makes current privileges and defaults visible. Compare the image requirements with this list; do not copy the test service settings directly.*

### Verify drift and recovery

If live posture reports a missing expected policy or a mismatch, confirm that the reported evidence is current and the deployment is the one you intended. A posture query failure is a visibility gap rather than a diagnosis of the application.

Use [deployment history](service-deployment-history.md) to correlate changes. After correction or recovery, check the application behavior that failed and the protection condition that motivated the original setting. A successful restart alone does not answer both questions.

## Observations, findings and policy decisions

Runtime observations show recorded behavior and existing review history for applicable services. Findings organize actionable security records, while the service timeline helps place them in context. The underlying sections can have different permissions and data windows.

Pending observation counts describe reviews in their displayed scope. They do not measure the number of attacks or prove the outcome of a posture test. Read the actual operation and evidence before classifying it.

**Allow** and **Mark as threat** are finding decisions with a required reason. They record how the finding was assessed; they do not by themselves install an allow or blocking rule. **Dismiss** concerns the finding's review status, rather than the protection policy.

Where an exception request is offered, review the target, reason and duration. A request awaiting approval is not an applied exception. Where a suggestion or response offers a separate application step, inspect the preview and current policy before continuing. The [Security Center](security-center-guide.md) guide explains these workflows and their permissions.

## Controlled verification

A protection test is a deliberate action with its own authorization and impact. Use a supported, approved scenario for the target service; do not turn an investigation into an unplanned test against a production application.

Before starting, agree on the target, applicable runtime, expected signal, allowed impact and recovery owner. Specify whether the scenario is intended to assess **detection**, **prevention** or **recovery**. These outcomes need different evidence.

1. Capture the current configuration, observed mode, application health and evidence time.
2. Use only the approved scenario and scope, with the permissions it requires.
3. Match the resulting record to the expected source, service, operation and time window.
4. For detection, confirm the expected observation or finding; for prevention, also verify the reported denial and the application outcome.
5. Complete the agreed recovery and check normal behavior and protection state again.

Do not access sensitive or decoy paths simply to generate a finding. A missing signal can indicate a source, permission, filter or applicability problem. Investigate that uncertainty before repeating a test.

A detected scenario demonstrates the behavior observed in that scenario. It does not establish prevention of all attacks or coverage of every service.

### Example: match a test result to Komuta evidence

On 8 October 2026, we ran **Drop and execute a binary** on **komuta-test-app** in the admin test environment. The scenario copies a program into a temporary directory and attempts to execute it. The screenshot shows the test application's actual denial, in the Turkish UI.

![Blocked result of the Drop and execute a binary scenario in komuta-test-app](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-test-result.jpg)

*The test application returned `permission denied` for `/tmp/komuta-dropped`. The “M2M ingest configured” banner does not demonstrate successful authentication; this local test does not rely on direct synthetic ingestion.*

We then opened the matching operation under **My Services → komuta-test-app → Security → Findings**. The finding reports a policy block and a **Last seen** time matching the test. An existing explicit Block rule can deny the operation even while the service reports Audit mode.

![Matching policy-block finding in the Komuta interface](https://raw.githubusercontent.com/Microzon-Tech/komuta-docs/main/img/security/runtime-block-evidence.jpg)

*Matching the test result to the source, target operation and time in Komuta provides the blocking evidence for this example. First-seen time and occurrence count also include earlier runs; do not interpret them as the number of events produced by this run.*

| Observed test result | How to interpret it |
|---|---|
| **Allowed** | The operation could run; behavior within Audit scope can still produce a finding. |
| **Blocked / permission denied** | A denial occurred; verify which layer denied it using matching Komuta evidence. |
| **Connection timeout** | Reachability can also cause this; do not count it alone as a network-policy block. |
| **Not run / invalid_client** | Synthetic ingestion stopped at authentication; detection or prevention was not successfully tested. |

We did not change protection settings for this example. Assess real runtime tests separately from direct synthetic ingestion; neither verifies the other's collection and enforcement chain.

## Common scenarios

### “The mode says Audit, but a request was blocked”

Check which layer produced the denial. Public access rules, network policies, isolation or another applicable rule can restrict a request independently of the runtime mode. Compare the request's path and time with the relevant evidence.

### “We saved a writable path, but the error continues”

Read the save result and deployment state, then compare the running application's actual path with the saved directory. Confirm the error is a write-permission issue rather than a missing dependency or another startup failure. Verify the smallest authorized correction after application completes.

### “We marked a finding as a threat, but it happened again”

The review decision records the assessment. Inspect any separate response or policy action and its application state, then compare fresh evidence. Recurrence is a reason to continue the investigation, not proof that the decision failed to save.

### “We reconnected the service, but it is still unhealthy”

Check that the requested release completed and inspect the application and dependency state. Removing a network restriction does not fix the original compromise, application failure or unrelated access rules.

## Troubleshooting

| Situation | Next check |
|---|---|
| **Loading or unavailable mode** | Wait for a verified result or retry where offered; do not reuse an uncertain state as active protection. |
| **No observations or findings** | Inspect filters, permissions, applicability, record loading and evidence freshness. |
| **Desired and observed state differ** | Read the application status and the latest deployment result. |
| **Configuration drift** | Compare current posture with approved settings and the intended deployment. |
| **Permission denied** | Request the specific read or management permission required for your task. |
| **Kata has fewer runtime controls** | Use the service's applicability information and assess each remaining layer separately. |
| **Application failed or recovery is incomplete** | Preserve the visible error and consult the authorized owner before another change. |

When contacting support, include the service, incident time, visible state, relevant deployment and the outcome you expected. Share only the information needed for that investigation.

## Contextual help and optional AI

The mascot menu contains static Turkish and English help for the current security page or supported service setting. It remains readable when AI or the decorative mascot is disabled. With the mascot enabled and ready, **Explain this screen** can also display the guidance in a help bubble.

Static guidance does not query service data, send it to AI or execute an action. Optional AI chat is separate. Page-context sharing is opt-in and can add a page summary when enabled; keep the conversation within your authorized scope. A recommendation does not replace evidence, page permissions or an action's confirmation and result checks.

## Frequently asked questions

### Is a healthy summary enough to confirm protection?

It helps prioritize, but the conclusion depends on source coverage and freshness. For a specific protection claim, review the target rule, its application and relevant outcome evidence.

### Is Audit the same as having no security?

No. Audit describes observation intent for the relevant runtime mode or rule. Other hardening and access restrictions have their own state.

### Does Enforce guarantee that every unwanted behavior is stopped?

It describes the intended mode for an applicable scope. Effective protection depends on the applied rules and supported behavior; evaluate it through the relevant evidence.

### Can build scanning replace runtime investigation?

No. Build evidence describes the checked image or build. Runtime investigation addresses what happens while the deployed application runs.

### Can a finding decision or AI answer change protection automatically?

A recorded finding decision or explanation is not proof of a protection change. Use the explicit, authorized application or response workflow where offered and verify its result.

## Related guides

- [Service Security](service-security-guide.md) — investigate and manage supported service settings.
- [Security Center](security-center-guide.md) — organization-wide findings, decisions and response.
- [Access protection](service-access-protection.md) — protect public service access.
- [Access & ports](services-ports.md) — inspect entry points and port configuration.
- [Deployment history](service-deployment-history.md) — compare requested changes with deployment results.
