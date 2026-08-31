# Deployment History

This page shows which version the service is currently running and which versions have been released before. When a deployment causes problems, this is where you switch versions.

![alt text](img/deployment-history-page.png)
---

## Current Version

At the top of the page is the service's live version: which image, which revision number, how many replicas, and which rollout strategy it's running with. The rollout strategy can also be changed from here; the change takes effect on the next deploy.

![alt text](img/deployment-history-current-version.png)
---

## Deployment Strategies

The strategy determines how the new version is rolled out to live. Which strategy the service runs with is shown in the current version section and changed from there.

| Strategy | Suited for | How it works |
|---|---|---|
| **Blue-Green** | Most services | Two identical environments. Zero downtime, easy rollback; switching to live requires manual approval. |
| **Auto Promote** | Hands-off flows | Same as Blue-Green, but the switch happens automatically once the new version is ready. |
| **Canary** | Advanced use | Gradually shifts traffic to the new version. Promoting the new version requires manual management. |

See the [Deploy Strategies](deployment-strategies.md) page for details.

---

## Rollout Actions

The actions used to intervene in an ongoing rollout are gathered here.

| Action | What it does |
|---|---|
| **Promote** | Shifts full traffic to the new version |
| **Resume** | Continues a paused rollout from the next step |
| **Pause** | Holds the rollout where it is |
| **Abort** | Cancels the rollout; the stable version stays live |
| **Retry** | Retries a failed rollout from the start |
| **Restart** | Recreates the pods of the running version |

---

## Rolling Back to a Previous Version

Every deployment record has a **Rollback** action. Once confirmed, the service returns to that revision.

> **Warning:** Rolling back cancels an ongoing rollout. If you roll back while a Canary deployment is in progress, the gradual rollout is left halfway and traffic returns to the old version.

![alt text](img/deployment-history-rollback.png)
---

## Related Documents

- [Deploy Strategies](deployment-strategies.md) — How Blue-Green, Canary, and Auto Promote work.
- [Pipelines](pipeline-guide.md) — The build and release process that produces a deployment.
- [Service Dashboard](service-dashboard.md) — Deploying a new version and live rollout status.
