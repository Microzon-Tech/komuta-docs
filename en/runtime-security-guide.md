# Runtime Security

Runtime security examines a running workload's behavior and protection state. Komuta combines network policies, workload hardening and behavior sensors in applicable environments. Each layer has its own scope; the presence of one does not establish that the others are effective.

For screen usage, read [Security Center](https://komuta.io/docs/services/security-center-guide) and [Service Security](https://komuta.io/docs/services/service-security-guide).

## Protection layers

| Layer | Observed or restricted area | What to check |
|---|---|---|
| **Network policies** | Traffic between workloads and to external destinations | Target, direction, application state and relevant traffic outcome |
| **Workload hardening** | Root permission, Linux capabilities and writable paths | Desired settings, applied deployment and live posture |
| **Host runtime protection** | Process, file and capability behavior in supported workloads | Runtime eligibility, sensor state, rule and event evidence |
| **Runtime monitoring** | Behavior telemetry and related evidence in supported environments | Source access, freshness and target match |
| **Platform host protection** | Platform node and infrastructure security | Operator scope, source health and platform evidence |

Build and image scans provide separate **supply-chain** evidence. A clean build scan does not inspect every behavior of a running workload.

## Host runtime and Kata eligibility

Host sensors can observe eligible container behavior that shares the host kernel. Workloads using a separate guest kernel, such as Kata, do not offer the same internal process and file visibility to host sensors.

Host behavior protection can therefore be **not applicable** or **unavailable**. Assess network policies, build scans and workload hardening separately. A listed sensor name does not establish that the sensor observes internal behavior of the selected service.

When runtime or source support is unknown, do not conclude that the workload is supported, unprotected or safe. Confirm the target service and source scope with a platform operator.

## Policy action versus runtime mode

Do not interpret settings of different protection engines as one mode.

| Setting | Meaning |
|---|---|
| **Audit policy action** | Intended to observe and record matching behavior; this rule has no blocking intent. |
| **Block policy action** | Intended to reject matching behavior in a supported, applied protection layer. |
| **Off / Shadow / Audit / Enforce runtime mode** | Desired or observed mode of the relevant runtime mechanism. It is not equivalent to a similarly named policy action. |
| **Monitor / Enforce administration setting** | Configuration of the relevant operator protection control. Target application and effective behavior outcomes need separate verification. |

Selecting Audit does not remove blocking from other layers. Selecting Block does not reject every process, file or connection; results depend on rule matching and engine support.

## Configuration, application and outcome

Answer three separate questions for protection:

1. **What was requested?** Inspect the saved rule for the intended service, paths, programs, direction and behavior.
2. **What was applied?** Check deployment outcome, observed mode and any application errors.
3. **What happened?** Inspect event, blocking or allowed-outcome evidence for the relevant target and time window.

A queued deployment, successful API request or healthy heartbeat does not answer the third question. Evidence for an older configuration does not verify a new rule. Missing or failed measurements are not safe results.

## Baselines and service policies

Platform baselines and runtime administration are the responsibility of authorized operators. In the Customer Console, review eligible policies for your workloads and service protection state, and manage supported service settings with their separate permissions.

Review service target, paths and YAML before selecting Audit or Block in the policy wizard. Accepted file paths do not prove that those paths exist in the container or that the rule is active. Restricting execution from temporary directories is different from blocking all writes there or every script invoked through an interpreter.

Block can disrupt startup, health checks, maintenance or scheduled work. Evaluate observations and normal workflows in a suitable environment first; apply a permitted change with limited scope and a recovery plan. Do not assume every service has the same initial mode.

## Observation and finding review

The Service Security **Findings** tab includes applicable runtime observations and existing decision history. Organization-wide Findings helps prioritize records by service, source, severity and time.

An observation summary counts pending reviews at calculation time. Do not present it as an attack counter, current queue size or posture test result. Review underlying records and source evidence separately.

Allow or Block on a finding does not establish actual policy application. Baseline observation administration, protection promotion and platform controls belong to AdminUI. Actual response has separate permissions and outcome checks.

## Exceptions and recovery

Confirm target, reason, duration and policy impact when accepting an exception or suggestion. Pending approval is different from an applied exception. A rollback request does not prove that recovery is complete.

If a protection change disrupts the application, compare the last deployment, observed mode and behavior evidence. Choose the narrowest correction with the authorized owner; evaluate the required behavior instead of disabling broad protection or allowing every finding.

## Controlled verification

A drill or protection test is a separate operational action. Require an approved target, applicable runtime, expected signal, possible impact and recovery plan before running one. Do not probe protected or decoy files indiscriminately.

Assess success by matching the test record to expected source evidence for the same target and time window. A detected scenario does not establish prevention of every attack. Keep detection evidence separate from blocking evidence.

## Troubleshooting

| Situation | Investigation step |
|---|---|
| **Permission denied or startup failure** | Compare the relevant process/path, last protection change and deployment; select an authorized narrow correction. |
| **Mode changed but no result** | Check desired/observed mode, deployment and event evidence separately. |
| **No findings are visible** | Verify time filters, permissions, runtime eligibility and source freshness. |
| **Policy application failed** | Inspect the reported error and target; check the existing policy before creating duplicates. |
| **A Kata workload has no host card** | Check layer applicability and assess other layers through their own evidence. |

Source health, platform host and cluster-wide settings require the operator console. An empty customer Security Center result does not establish infrastructure health.

## Mascot guidance

**Explain this screen** provides static Turkish/English help for the current security page or service tab. It remains available when the AI provider is disabled and does not query data or execute protection actions.

Guidance and separate AI recommendations do not replace current application and behavior evidence. Use the page's own authorization and confirmation flow for changes, then verify the outcomes.

## Related documents

- [Security Center](https://komuta.io/docs/services/security-center-guide)
- [Service Security Workbench](https://komuta.io/docs/services/service-security-guide)
- [Service access protection](https://komuta.io/docs/services/service-access-protection)
