# Pipelines

A pipeline is the process of building your code, running it through a security scan, and deploying it to the service. All pipeline runs for your service are listed here.

![Pipelines tab](img/piplines-piplines-page.png)

---

## Pipeline Runs

The list shows each run's status, which branch and commit it came from, the image tag it produced, and its duration.

From a run's row you can **cancel** a running pipeline, go to its logs, or delete the pipeline.

> **Warning:** Deleting a pipeline also removes its run history and can't be undone. A running pipeline can't be deleted; cancel it first.

---

## Run Details

In a run's details, each task's logs can be inspected separately. For a failed run, the fastest way to find the error is to open the red task and check the last lines of its log. Logs stream live while the run is in progress; you can filter, copy, or download them.

![Pipeline detail page](img/piplines-failed-warning.png)

---

## Automatic Dockerfile Repair

When the build stage fails because of a Dockerfile issue, Komuta steps in: it analyzes the error and the Dockerfile, produces a fix, opens it as a **pull request**, and triggers the new pipeline. The run details show the decisions behind this process step by step.
 
Repair doesn't run for every error; when it doesn't apply, it states the source of the error and what you need to do. For example, if the service is crashing because it exceeded its memory limit or was evicted, that isn't a Dockerfile problem — you need to upgrade the service's resource package or reduce the memory the application consumes.
 
> **Note:** The number of repair attempts is limited. The same fix isn't tried over and over; if no result is reached, the process stops and the reasoning behind the decision stays in the run details.

---

## Fixing with AI or MCP

On a failed run, **Fix with AI** collects the error logs and generates a ready-made fix prompt. You can open it directly in Claude or ChatGPT, or copy it and paste it into the assistant of your choice.

If Komuta MCP is set up, **Fix with MCP** goes further: your agent connects directly to the service, reads the logs, and can apply the fix. See [MCP Setup](mcp-setup.md) for setup instructions.

![Fix with AI menu](img/piplines-mcp-ai-solution.png)

---

## Rollout Attempts

Runs where the build succeeded but the run got stuck at the deployment stage are listed here. Komuta waits a certain amount of time for a rollout to complete; once that time is up, it either cancels the rollout automatically or just notifies you.

If automatic cancellation is enabled, the new version is withdrawn and **the stable version keeps serving traffic** — a stuck deployment doesn't cause an outage for your service. This list shows what happened in each attempt, how long it took, and which action was applied.

---

## Related Documents

- [Deploy Strategies](deployment-strategies.md) — How the deployment stage works under Blue-Green, Canary, and Auto Promote.
- [Service Dashboard](service-dashboard.md) — Deploying a new version and monitoring rollout status.
- [MCP Setup](mcp-setup.md) — Connecting your coding agent to Komuta.
- [Monitoring and Logs](monitoring-logs.md) — Service logs and alerts.
