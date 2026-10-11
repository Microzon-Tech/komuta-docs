# Quick Start

Everything you need on Komuta, from shipping your code to spinning up your database, comes down to four services:

- **[Services](https://www.komuta.io/docs/services/new-service-guide)** — Builds and deploys the apps in your Git repo.
- **[Jobs and CronJobs](jobs-cronjobs.md)** — Runs one-off or scheduled work.
- **[Stacks](stacks.md)** — Defines a whole setup of services, resources, and domains in a single file.
- **[Managed services](https://www.komuta.io/docs/managed-services/postgresql)** — Serves up PostgreSQL, Valkey, RabbitMQ, and API Gateway with no setup required.

---

## Before You Start

**Account.** Sign up at [console.komuta.io](https://console.komuta.io) with an email address and password, or with your Google or GitHub account. If you signed up with email, confirm your address with the link we send you. No credit card is needed to sign up.

**Organization.** Signing up creates an organization that belongs to you, with you as its admin. Services, managed services, the wallet and the spending limit all belong to the organization. Invite teammates as **Member** or **Admin** from the **Access Control** page; an invitation is valid for 72 hours. If you belong to more than one organization, switch between them from the user menu in the top right. Details: [Organizations and Team](organizations-team.md).

**Git account.** Services are built from a Git repo. Before your first service, connect your GitHub, GitLab or Azure DevOps account on the **Integrations** page; public repos are deployed through a connected account too. Details: [Integrations](integrations.md).

**Starting for free.** The organization created at sign-up gets a USD 10 welcome credit, valid for 30 days. Each organization can run one service on the free package; a free service sleeps after a while without traffic and wakes on the first request ([Service Sleep](service-sleep.md)). Every billing cycle includes 2,000 build minutes at no cost. PostgreSQL, Valkey, RabbitMQ and API Gateway are paid. The free plan rules and all prices are on the [Pricing](https://www.komuta.io/pricing) page.

**Cost.** Prices are in USD and exclude tax; usage is metered by the hour. You see the price before you create anything paid: the service wizard shows each package's price, and the managed service wizard and the Stack plan show the **Estimated monthly cost**. Organization admins see spending so far under **Account → Current Spend** and issued statements under **Account → Statements**; a statement is issued on the 6th of every month.

**Spending limit.** A new organization starts with a USD 20 monthly resource limit. If creating something, or moving it to a bigger plan, would take the monthly estimate over the limit, the action is refused; usage that is already running is not stopped. The limit goes up as you verify a card, add funds and pay statements; your current level is under **Account → Wallet**. Details: [Billing and Wallet](billing.md).

---

## Publishing a Service

This is how you build and deploy an app from your Git repo. The flow starts with **New App** on the [**Apps**](https://console.komuta.io/services) page and has three steps:

1. **Project** — The project the service is added to and the region it runs in.
2. **Connect** — The Git account, repo and branch, and the container registry the built image is pushed to. Komuta's own registry is ready to use; you can add your own on the **Integrations** page.
3. **Configure** — Settings for the services found in the repo. The final button builds the code and takes the service live.

> **Note:** Komuta scans the repo and finds services from their Dockerfiles. If the repo has no Dockerfile, AI generates one and opens it as a pull request on your repo; merge that pull request into your branch, or later builds will fail. If there is a `komuta.yaml` at the repo root, the flow switches to creating a Stack.

![New app wizard's Configure step — services found in the repo, listed](https://cdn.komuta.io/docs/tr/images/services/quick-start-create-service.png)

---

## Jobs and CronJobs

A Job is a one-off task that runs once and finishes; a CronJob ties the same task to a schedule. Both are created from the [**Jobs & CronJobs**](https://console.komuta.io/jobs) page, and their flows only diverge at the scheduling step: Job selects **I'll run it myself**, CronJob selects **On a schedule**.

What the job runs is chosen from three sources — a project in the repo (which gets built), a short script written directly (Bash, Python, or Node), or a ready-made container image. A published Job is triggered manually whenever needed; a CronJob runs on its own according to its schedule, and run history is kept on the same page.

![Scheduling step of the CronJob creation flow](https://cdn.komuta.io/docs/tr/images/jobs/create-jobs-page.png)

---

## Setting Up with a Stack

A Stack defines the services, managed resources, and domains that make up an application in a single file, so the whole setup happens in one pass and is repeatable. It's created from the [**Stacks**](https://console.komuta.io/stacks) page.

The file format is described in the [komuta.yaml Reference](stack-manifest-reference.md). Once the definition is ready, **Create plan** validates the file and, if it is valid, lists what will be created and the estimated monthly cost before anything is provisioned. Secret values like passwords and keys are entered through a one-time secure link and are never written to the Stack file. Resources are only created once the plan is approved.

![Generated Stack plan — resource list and estimated cost](https://cdn.komuta.io/docs/tr/images/stacks/create-stack-page.png)

---

## Managed Services

Managed services need no repo or build pipeline of their own; Komuta provisions, configures, backs up, and monitors them.

**PostgreSQL, Valkey, and RabbitMQ** are all created the same way: pick the service on the [**Managed Services**](https://console.komuta.io/addons) page and use **New instance** to set the region, plan, version, and the project to associate it with. The service is ready to use within a few minutes, and the plan can be upgraded later.

**API Gateway** is created from **Managed Services > API Gateways**. Once it's set up, your own APIs are added under the gateway, each with its own path, authentication, and rate limit. Details: [API Gateway](api-gateway.md).

> **Warning:** The gateway's API key is shown only once, on the creation screen, and can't be viewed again afterward.

![Creating a new PostgreSQL instance — plan selection screen](https://cdn.komuta.io/docs/tr/images/managed-services/managed-services-page.png)
