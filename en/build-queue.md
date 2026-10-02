# Build Queue

Every deploy in Komuta starts with a build. Builds run on shared build capacity; a build that cannot start right away is not lost. It enters the **build queue** and starts on its own when its turn comes. Nothing in the queue is ever cancelled for you.

---

## How the Order Is Decided

These rules decide whether a build starts now or waits:

- **Builds of the same service run one after another.** A new build does not start until the service's previous build has finished.
- **Your account tier has a concurrent build limit.** How many builds your account can run at once depends on its tier; when the limit is reached, a new build waits for a slot. Your tier, and how to reach the next one, is shown under **Account → Wallet**.
- **The queue is fair across accounts.** While one account has many builds queued, another account's single build can still move ahead; no account can hold the whole queue.
- **Build capacity is measured, not guessed.** A build starts only when there is room for it.

| Account tier | Concurrent builds | Queued at most |
|------|-------------------|----------------|
| Starter | 1 | 5 |
| Verified | 1 | 10 |
| Basic | 2 | 20 |
| Pro | 4 | 30 |
| Scale | 8 | 50 |

---

## Why a Build Is Waiting

Every queued build shows why it is waiting:

| Reason | What it means |
|--------|---------------|
| Waiting for this service's previous build to finish | Another build of the same service is still running. This one starts when it finishes. |
| Waiting for a build slot of your plan | Your account's concurrent build limit is reached. It starts when one of your builds finishes; reach a higher account tier for more concurrent builds. |
| Waiting for build capacity to free up | The platform's build capacity is full right now. It starts when capacity frees up. |
| Waiting for a free build slot | The platform-wide concurrent build limit is reached. |
| Build capacity cannot be read right now, builds start one at a time | Capacity cannot be measured for a moment; builds start one at a time meanwhile. |

A previous build that is only cleaning up, or that has passed the time limit, no longer holds the service; the new build can start without waiting for it.

---

## Pushes While a Build Waits

If new pushes arrive on the same branch while a build is waiting, no extra builds are created. The pushes are folded into the waiting build, which builds the **latest commit** when its turn comes. The queue card shows how many pushes were folded in as `+N`.

---

## Estimated Start

For queued builds Komuta estimates when they will start ("Expected to start in about 5 min"). The estimate is based on:

- The typical duration of the service's recent successful builds.
- The remaining time of your account's running builds.
- Your account tier's concurrent build limit and your position in the queue.

No estimate is shown while a build waits for platform capacity; its start then depends on other accounts' builds and no reliable time can be given.

---

## Where You See the Queue

- **Pipelines tab:** the "Queued builds" lane shows the build running now and the queued ones in order. New builds join at the right end. Click a card for the wait reason, wait time, estimate and the cancel action.
- **Services list:** a service with a queued build shows a **Queued #N** chip under its status. If the service is already building, the chip reads **Next build queued #N**.
- **Service dashboard:** the status badge shows the queue position, the wait reason and the estimated start.

The position is your place in your own account's queue; other accounts' builds are never shown.

---

## Cancelling a Queued Build

Users who can edit the service can cancel a queued build: on the **Pipelines** tab click the card in the queue lane and choose **Cancel**. The cancellation is recorded in the audit log.

---

## Long Wait Notification

When a build waits in the queue for more than 15 minutes, the **Build waiting long in the queue** event is raised. Choose where it is delivered under **Account → Notifications → Event routing**. It is sent once per build.

---

## Billing

Time spent in the queue is not billed. Build minutes are counted from the moment the build actually starts until it finishes.

---

## Related Documents

- [Pipelines](service-pipeline-guide.md) — Build stages and logs.
- [Auto-deploy](service-auto-deploy.md) — Rules for triggering builds on push.
- [Deploying from CI](ci-deploy.md) — Calling Komuta from your own CI pipeline.
