# Quick Start

Everything you need on Komuta, from shipping your code to spinning up your database, comes down to four services:

- **[Services](https://komuta.io/docs/services)** — Builds and deploys the apps in your Git repo.
- **[Jobs and CronJobs](https://komuta.io/docs/jobs)** — Runs one-off or scheduled work.
- **[Stacks](https://komuta.io/docs/stacks)** — Defines a whole setup of services, resources, and domains in a single file.
- **[Managed services](https://komuta.io/docs/managed-services)** — Serves up PostgreSQL, Valkey, RabbitMQ, and API Gateway with no setup required.

---

## Publishing a Service

This is how you build and deploy an app from your Git repo. The flow starts with **New Project** on the [**Apps**](https://console.komuta.io/services) page and goes through four steps: the project and infrastructure the service will run on (**Komuta PaaS** or your own cluster), the Git account and the registry the image will be pushed to, configuring the services found in the repo, and deploying the service.

> **Note:** In the **Configure** step, Komuta scans the repo and finds services from Dockerfiles; if there's no Dockerfile in the repo, it generates one automatically for the build. **Deploy** builds the code and takes the service live.

![New project wizard's Configure step — services found in the repo, listed](https://cdn.komuta.io/docs/tr/images/services/quick-start-create-service.png)

---

## Jobs and CronJobs

A Job is a one-off task that runs once and finishes; a CronJob ties the same task to a schedule. Both are created from the [**Jobs and CronJobs**](https://console.komuta.io/jobs) page, and their flows only diverge at the scheduling step: Job selects **I'll run it manually**, CronJob selects **Attach to schedule**.

What the job runs is chosen from three sources — a project in the repo (which gets built), a short script written directly (Bash, Python, or Node), or a ready-made container image. A published Job is triggered manually whenever needed; a CronJob runs on its own according to its schedule, and run history is kept on the same page.

![Scheduling step of the CronJob creation flow](https://cdn.komuta.io/docs/tr/images/jobs/create-jobs-page.png)

---

## Setting Up with a Stack

A Stack defines the services, managed resources, and domains that make up an application in a single file, so the whole setup happens in one pass and is repeatable. It's created from the [**Stacks**](https://console.komuta.io/stacks) page.

Once the definition is ready, **Validate** surfaces structural issues, and **Create plan** lists what will be created and the estimated monthly cost before anything is provisioned. Secret values like passwords and keys are entered through a one-time secure link and are never written to the Stack file. Resources are only created once the plan is approved.

![Generated Stack plan — resource list and estimated cost](https://cdn.komuta.io/docs/tr/images/stacks/create-stack-page.png)

---

## Managed Services

Managed services need no repo or build pipeline of their own; Komuta provisions, configures, backs up, and monitors them.

**PostgreSQL, Valkey, and RabbitMQ** are all created the same way: pick the service on the [**Managed Services**](https://console.komuta.io/addons) page and use **New instance** to set the region, plan, version, and the project to associate it with. The service is ready to use within a few minutes, and the plan can be upgraded later.

**API Gateway** is created from **Managed Services > API Gateways**. Once it's set up, your own APIs are added under the gateway, each with its own path, authentication, and rate limit.

> **Warning:** The gateway's API key is shown only once, on the creation screen, and can't be viewed again afterward.

![Creating a new PostgreSQL instance — plan selection screen](https://cdn.komuta.io/docs/tr/images/managed-services/managed-services-page.png)
