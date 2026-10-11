# Jobs and CronJobs

A Job is work that runs once and finishes: a data migration, a one-off fix or a report you trigger by hand. A CronJob ties the same kind of work to a schedule; use it for recurring work such as a nightly cleanup or an hourly sync. Both are created and managed from the [**Jobs & CronJobs**](https://console.komuta.io/jobs) page.

Jobs and CronJobs differ from long-running services: they do not serve requests, so they have no public address. Ports, domains, autoscaling, health probes and persistent disks are not used by these workloads.

---

## Job or CronJob?

| | Job | CronJob |
|---|---|---|
| When it runs | Only when you choose **Run now** | On your schedule, automatically; also on demand with **Run now** |
| Choice in the wizard | **I'll run it myself** | **On a schedule** |
| Menu shortcut | **New Job** | **New CronJob** |
| Suspending the schedule | Not applicable | **Suspend schedule** |
| Typical use | Data migration, one-off fix, report on request | Nightly cleanup, periodic sync, weekly report |

The type is chosen before the job is published. A published Job cannot be turned into a CronJob later (or the other way round); publish a new job for a different type.

> **Note:** Publishing does not start a run. A Job waits until you choose **Run now**; a CronJob runs at its first schedule slot.

---

## Opening the Wizard

1. Open **Jobs & CronJobs** from the left menu.
2. Choose **New Job** for one-off work or **New CronJob** for scheduled work. The wizard opens with that choice selected; you can still change the timing in the **Job setup** step.
3. Complete the steps in order. You cannot move to the next step until the required fields of the current one are filled in.

The wizard opens only for users who have permission to create services.

The wizard's steps depend on what the job runs:

| Source | Steps |
|---|---|
| **A project in your repository** | Target → Source → Connection → Repository → Job setup → Summary |
| **Write a script** | Target → Source → Job setup → Summary |
| **An image you already have** | Target → Source → Registry → Job setup → Summary |

---

## 1. Target

Pick the **Project** the job belongs to, or create one with **Create new project…**. Then pick a **Region**. The **Deployment target** is always **Komuta PaaS**.

## 2. Source

| Option | What it does | Tag |
|---|---|---|
| **A project in your repository** | Your repository code is built into an image with a Dockerfile. Suited to work that has dependencies. | **needs a build** |
| **Write a script** | You type a few lines of work straight into the wizard; it runs on Bash, Python 3.12 or Node 22. No repository needed. | **no build** |
| **An image you already have** | Runs a container image from your registry as it is. | **no build** |

## 3. Connection and Repository (repository source)

In the **Connection** step, choose the **Git account** the code is read from and the **Container registry** the built image is pushed to. Komuta's own registry is available out of the box (**Managed by platform**). You connect your Git account and your own registry on the [Integrations](integrations.md) page.

In the **Repository** step, choose the repository and the **Branch**.

## 3. Registry (existing image source)

The wizard asks "Which connection should Komuta use to pull this image?". Choose the connection of the registry that holds the image. If you use a private registry, add its connection on the [Integrations](integrations.md) page first.

## 4. Job Setup

### What runs

| Field | Source | Description |
|---|---|---|
| **Job name** | All | Required. Use lowercase letters, numbers and hyphens; the name must start and end with a letter or number. Example: `nightly-cleanup`. |
| **Resource Package** | All | Required. Sets the capacity each run gets. Each package's price is shown next to it in the list. |
| **Configuration file** | Repository | Required. Path from the repository root to the Dockerfile; defaults to `Dockerfile`. |
| **Image reference** | Existing image | Required. Use an immutable tag or digest, for example `ghcr.io/acme/cleanup:1.4`. |
| **Runtime** | Script | **Bash**, **Python 3.12** or **Node 22**. Defaults to Python 3.12. |
| **Script** | Script | Required. Up to 64 KB; the counter under the field shows the size used. |
| **Command** | Repository, existing image | Optional. Leave it empty to use the image's own entrypoint. Example: `python -m tasks.cleanup`. Text you put in quotes is passed as a single argument. |

### When it runs

Under **When should this run?**, choose **I'll run it myself** (Job) or **On a schedule** (CronJob).

With **On a schedule**, you set the **Frequency**:

| Frequency | What you enter |
|---|---|
| **Hourly** | **Minute past the hour** (0–59) |
| **Daily** | **Time** |
| **Weekly** | **Day** and **Time** |
| **Monthly** | **Day of month** and **Time**. The day is limited to 1–28 so the job runs every month. |
| **Advanced** | **Schedule expression**: a five-field cron expression (minute, hour, day of month, month, day of week) |

The **Time zone** sets the local clock the schedule is evaluated in. It defaults to the time zone you use in the console; the list offers common zones such as UTC, Europe/Istanbul and Europe/Berlin.

Below your choices, the result is summarized: a readable description (for example "Every day at 03:00 (Europe/Istanbul)"), the **Generated schedule** expression, and the next run in the selected time zone and in UTC. The **Next runs** list shows the next three runs. A new CronJob starts with every day at 03:00 (`0 3 * * *`).

> **Note:** A schedule runs approximately once per slot; rarely a run can be skipped or repeated. Write your job so that running it again on the same data does no harm.

### Advanced settings

The **Advanced settings** section is collapsed and safe defaults are already selected. Open it only when you need different behavior.

| Field | Default | Description |
|---|---|---|
| **If the run fails** | **Don't retry — report the first failure** | **Retry once** or **Retry up to 3 times** are also available. |
| **Maximum run time** | 3000 (50 minutes) | Entered in seconds. The run is stopped after this duration; leave it empty for no time limit. |
| **If the next run overlaps** (CronJob) | **Skip the new run — safest for most jobs** | The other option is **Cancel the active run and start the new one**; the active run can then be stopped before it finishes. |
| **Missed-run recovery period** (CronJob) | 1800 (30 minutes) | In seconds. After an interruption, a missed run is started only within this period; leave it empty for no limit. |
| **Successful runs to keep** (CronJob) | 3 | Number of completed runs kept for later review. |
| **Failed runs to keep** (CronJob) | 5 | Raise it when troubleshooting matters. |
| **When should completed runs be deleted?** | 604800 (7 days) | In seconds. Leave it empty to rely only on the counts above. |
| **Restart behavior within a run** | **Recommended — finish the failed attempt and use the retry rule** | The other option is **On failure — restart the program within the same attempt**. |
| **Publish without starting the schedule** (CronJob) | Off | When on, the CronJob is published suspended; use it to review the settings before the first automatic run. |

In the **Work security** section, the **Recommended secure profile** is selected: the job runs without administrator permissions, with application files protected from changes and unnecessary system permissions removed. A size-limited area under `/tmp` is provided for temporary files (**Temporary file space limit**, default `64Mi`). If your image cannot run with these restrictions, choose **Custom profile** and turn off **Run without administrator permissions**, **Protect application files from changes** and **Remove unnecessary system permissions** one by one. Leave **Does this work need special platform access?** off for most jobs.

At the end of the step you see "Configuration is complete. You can continue to the review step." when everything is in order; otherwise the fields to fix are listed under "Complete {count} required item(s) to continue".

## 5. Summary and Publishing

The **Summary** step shows the job's name, source, schedule and the time of the first run, the **Initial state** of a CronJob (**Active** or **Suspended**), the security profile and the execution settings. Expand **Configuration to publish** to see all settings that will be applied.

**Publish job** saves the job, shows **Job published** and takes you back to the **Jobs & CronJobs** list. With the repository source the image is built first; follow progress on the job's **Pipelines** tab. Script and existing-image sources have no build step.

---

## The Jobs & CronJobs List

The list shows the Jobs and CronJobs of the selected project (or of all projects when none is selected). The counters at the top count jobs whose **Last run succeeded** and **Last run failed**.

| Column | Shows |
|---|---|
| **Workload** | The job's name |
| **Trigger** | **On demand** for a Job, **Scheduled** for a CronJob |
| **Schedule** | The CronJob's expression, its readable description and time zone |
| **Last run** | How long ago the last run happened; **Never run** if there has been none |

A status label appears next to each job's name:

| Status | Meaning |
|---|---|
| **Provisioning** | The job is being published. |
| **Failed** | Publishing failed, or the last run failed. |
| **Active** / **Suspended** | The CronJob's schedule is running or suspended. |
| **Succeeded** / **Canceled** | The result of the Job's last run. |
| **Ready to run** | The Job has not run yet. |

The row buttons open the **Logs** and **Runs** pages. The **…** menu holds **Open workload** and **Delete**.

---

## The Runs Page

When you open a job, these tabs appear: **Overview**, **Runs**, **Pipelines**, **Logs**, **Environment variables** and **Invoice**. **Overview** shows the **Runs** page as well.

The top of the page shows the job type (**Job** or **CronJob**). For a CronJob you also see the schedule expression, the time zone, the overlap rule, a "Next run: … UTC" line and a state badge:

| Badge | Meaning |
|---|---|
| **Active** | The schedule is running. |
| **Suspended** | The schedule is stopped; no new scheduled runs start. |
| **Not deployed** | The CronJob has not reached the place it runs yet. **Run now** and the suspend button are disabled meanwhile; check the **Pipelines** tab. |
| **Verifying cluster** | The state is being read. |
| **Cluster state unknown** | The state could not be read right now. Use **Retry** to try again. |

**Run now**, **Suspend schedule**, **Build image** and editing the settings are available only to users who can edit services. Other users can view the run history and the saved settings.

### Running a Job Manually

1. Click **Run now**.
2. You see "Job run started in the target cluster." and, briefly, the **Job created; waiting for its pod** card.
3. Once the run starts, the **Run active in cluster** card shows the trigger, the number of active pods and the start time.
4. When the run finishes, it is added to the **Run history** table.

Every manual run uses the job's most recently published image. For a CronJob, **Run now** is independent of the schedule and also works while the schedule is suspended.

While a run is active, a new run cannot start and **Run now** stays disabled. The one exception is a CronJob whose overlap rule is **Cancel the active run and start the new one**: the active run is stopped first and recorded as **Canceled**, then the new one starts.

If you see "The job run could not be started.", the two most common reasons are that the first build of a repository-based job has not finished yet (check the **Pipelines** tab) or that another run is still in progress.

### Canceling a Run

1. Click **Cancel run** on the active run's card.
2. Confirm with **Cancel run** in the **Cancel active Job run?** dialog.

You see "Job run canceled." and the run is recorded as **Canceled** in the history. Only runs that are still in progress can be canceled.

### Suspending and Resuming a Schedule

To pause a CronJob's automatic runs, click **Suspend schedule**. You see "CronJob suspended." and the badge changes to **Suspended**. Suspending does not stop a run that is already in progress; use **Cancel run** for that.

To start the schedule again, click **Resume schedule**; you see "CronJob resumed.". If **Publish without starting the schedule** was on in the wizard, the CronJob starts suspended and you turn it on with the same button.

### Run History and Logs

The **Run history** table lists the last 50 runs, newest first, and refreshes itself every few seconds.

| Column | Shows |
|---|---|
| **Status** | **Succeeded**, **Failed** or **Canceled** |
| **Trigger** | **Manual** (Run now) or **Scheduled** |
| **Started** / **Duration** | When the run started and how long it took |
| **Exit code** | The exit code the program returned when it finished |
| **Cluster pod** / **Reason** | The run's pod and the finish reason, if any |
| **Job & logs** | The run's name and an **Open logs** link |

If there are no runs yet, the table shows **No completed runs yet**. **Open logs** opens the job's **Logs** tab, where you can search your program's output ([Logs](service-logs.md)).

### Changing the Settings

The saved settings live in the **Saved scheduled task settings** panel (**Saved one-time task settings** for a Job) on the **Runs** page. Open it with **Review settings**. Sections:

- **Schedule** (CronJob only): frequency, time and **Time zone**; below them you see whether the schedule is valid and the next three runs.
- **Execution behavior**: retry count, maximum run time, overlap rule, missed-run recovery period, the numbers of runs to keep, when completed runs are deleted, and **Publish without starting the schedule**. Here you can enter durations in seconds, minutes, hours or days.
- **Resource profile**: the CPU, memory and temporary storage reserved for each run and the maximum it may use.
- **Work security**: the same security settings as in the wizard.

When you change something, an **Unsaved changes** badge appears. Save with **Save & publish**; you see "Configuration saved; publishing was queued." and the new settings are published. If a field is invalid, the button stays disabled and the fields to fix are listed.

The job's type (Job or CronJob) and its source cannot be changed from this panel.

### Environment Variables

The wizard has no environment variable step. After publishing, add them on the job's **Environment variables** tab, just like for a service, and apply them with **Apply Changes**. Details: [Environment Variables](services-environment-variables.md).

### Building a New Image

Repository-based jobs have a **Build image** button on the **Runs** page. After changing your code, build a new image from the **Build job image** dialog; once the build finishes, later runs use the new image. Script and existing-image jobs do not have this button.

---

## Deleting

In the **Jobs & CronJobs** list, choose **Delete** from the **…** menu on the job's row. In the dialog, type the job's name to confirm.

To delete several jobs, tick their rows and click **Delete selected**. The **Delete workloads** dialog states that the selected workloads are permanently deleted together with their run history.

> **Warning:** Deleting cannot be undone. The job's settings and run history are deleted too.

---

## Limits and Validation Messages

| Topic | Limit |
|---|---|
| Job name | Lowercase letters, numbers and hyphens; starts and ends with a letter or number |
| Script size | Up to 64 KB |
| Script runtimes | Bash, Python 3.12, Node 22 |
| Schedule expression | Five fields; `*`, `,`, `-` and `/` are allowed. Month and day names (`JAN`, `MON`) and `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly` are accepted. A seconds field and `@every` are not. Day of week is 0–6 (0 = Sunday); 7 is not accepted. |
| Shortest interval | There is no separate lower limit; `* * * * *` runs every minute. |
| Overlapping runs | Only "skip the new run" or "cancel the active run" can be chosen; there is no option that lets runs overlap. |
| Run history | The console shows the last 50 runs |

Common messages:

| Message | What to do |
|---|---|
| "Give the job a name" / "Use lowercase letters, numbers and dashes, e.g. nightly-cleanup" | Fix the **Job name** to follow the rules. |
| "Pick a resource package." | Choose a package from the **Resource Package** list. |
| "The script is too large — keep it under 64 KB" | Shorten the script, or move the code into a repository and use the **A project in your repository** source. |
| "That isn't a valid cron expression" | Write a five-field expression, for example `0 3 * * *`. |
| "This schedule never comes around" | Check the day and month fields; for example `0 0 30 2 *` never runs. |
| "Choose a valid IANA time zone, for example Europe/Istanbul or UTC" | Pick a time zone from the list. |
| "Timeout must be greater than zero" | Enter a positive number in **Maximum run time**, or leave it empty. |
| "Choose a container registry that can pull the image" | Choose a connection in the **Registry** step. |
| "A service with this name already exists with a different type. …" | A workload of a different type already uses this name; give the job a different name. |

---

## Resource Package and Price

A job runs on the resource package you choose in the wizard. Package prices are shown in the **Resource Package** list while you choose. For billing details, see [Billing](billing.md) and the [Pricing](https://www.komuta.io/pricing) page.

---

## Managing Jobs with MCP

You can also manage your jobs from an AI assistant through Komuta MCP: view the run history and active runs, read the saved settings, use **Run now**, cancel an active run, and suspend or resume a CronJob. For setup, see [Installing the Komuta MCP Server](mcp-setup.md); for the list of tools, see [Komuta MCP](mcp-tools.md).

---

## Related Documents

- [Quick Start](quick-start.md) — An overview of the four Komuta offerings.
- [Pipelines](service-pipeline-guide.md) — How the image of a repository-based job is built.
- [Logs](service-logs.md) — Reviewing the output of runs.
- [Environment Variables](services-environment-variables.md) — Configuration values your job reads.
- [Integrations](integrations.md) — Git account and registry connections.
- [Stacks](stacks.md) — Defining services, jobs and resources in one file.
- [Billing](billing.md) — Spending, wallet and statements.
