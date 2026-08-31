# Auto Deploy

When auto deploy is on, every push to the branch the service tracks triggers Komuta's pipeline itself and releases the new version. No need to trigger a deploy manually.

Not every push turns into a deploy directly. Komuta first checks the rules below; if the push passes them the pipeline starts, and if it doesn't, it's skipped and the reason is logged.

![alt text](img/service-auto-deploy-page.png)
---

## Include Path Patterns

A push only triggers a deployment when at least one changed file matches a pattern here. This is the setting that lets a single service in a monorepo deploy only for changes in its own folder.

When no pattern is defined, the service's Dockerfile context is used — if the service builds from the repo root, every push to the tracked branch deploys.

Example patterns:

```text
src/**/*
services/api/**
services/gateway-service/**
```

---

## Exclude Path Patterns

Even if a file matches an include pattern, files that also match a pattern here are ignored. This prevents non-code changes like a documentation update or a README fix from triggering an unnecessary deploy.

Example patterns:

```text
**/*.md
docs/**
```

> **Note:** If a push has both included and excluded files, the exclusion only filters out the files it matches; if an included file remains, the deployment still fires.

---

## Debounce Interval

The minimum time that must pass between two consecutive automatic triggers. Default is 30 seconds, and it can be set up to 3600 seconds (one hour).

For a burst of consecutive pushes, this prevents a separate pipeline from starting for every commit; pushes that arrive within the interval are skipped.

---

## Skip Directives

If the commit message contains one of the phrases below, the push is ignored and no deployment is triggered. This lets you skip a single commit you don't want released, without touching any settings.

Example directives:

```text
[skip deploy]
[no deploy]
[ci skip]
```

---

## Trigger Decisions

The decision made for every incoming push is logged along with its commit. If an expected deploy didn't start, you can see why here.

![alt text](img/service-auto-deploy-recent-trigger-decisions.png)
---

## Related Documents

- [Pipelines](pipeline-guide.md) — The build and release process for a triggered deployment.
- [Deployment History](deployment-history.md) — Released versions and rollback.
- [Service Dashboard](service-dashboard.md) — Changing the branch the service tracks.
