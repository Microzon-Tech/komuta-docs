# What Is Komuta? Core Concepts

This page explains what Komuta is and how the pieces of the console fit together. It ends with a glossary of the terms you will meet in the console. Start here if you are opening the console for the first time, or if you are not sure what a term means.

The steps for taking your first service live are on the [Quick Start](quick-start.md) page; this page does not repeat them.

---

## Komuta in Brief

Komuta is a platform as a service (PaaS). You push your code to Git; Komuta builds it, runs it, scales it, protects it and watches it. You do not manage servers, operating systems or infrastructure.

- **Builds.** It finds services from the Dockerfiles in your repo. If there is no Dockerfile, AI generates one and opens it as a pull request on your repo. Every build goes through a security and vulnerability scan.
- **Runs.** Your app runs on the resource package you choose, as one or more replicas. A new version goes live with the rollout strategy you choose.
- **Scales.** Autoscaling raises and lowers the replica count based on CPU and memory usage.
- **Protects.** Access protection decides who can open your service's address. The Security Center brings findings and the behavior of running services together in one place.
- **Watches.** Logs, metrics and traces are on the **Observability** screen. Alert rules send notifications to email, Slack or Microsoft Teams.

It also offers **managed services** that need no code and no build: PostgreSQL, Valkey, RabbitMQ and API Gateway. Komuta handles their setup, backups and monitoring.

You can also run Komuta from your AI assistant. An assistant connected to the Komuta MCP server (Claude, Codex) can create projects, deploy, set up databases and read logs for you.

Services currently run in one region, **Germany-East** (`de-east`).

---

## How the Pieces Fit Together

Everything in Komuta belongs to an **organization**. An organization holds **projects**, and projects hold the workloads that run:

```text
Your user account
└── Organization ("Acme")
    │   Members and roles · Wallet, statements, spending limit
    │   Integrations (Git accounts, container registries)
    │   Domains · Environment groups · Alert channels
    │
    ├── Project: shop
    │   ├── App: api
    │   ├── App: web ─── private connection ──► api
    │   ├── CronJob: nightly-report
    │   ├── API Gateway: public-api
    │   ├── PostgreSQL instance: shop-db
    │   └── Stack: shop   (defines the items above in one file)
    │
    ├── Project: internal-tools
    │   └── App: admin-panel
    │
    └── Shared: Valkey instance   (not attached to any project)
```

### Organization

An organization is your team's workspace. Signing up creates an organization that belongs to you, with you as its admin. Services, managed services, members, Git connections, the wallet and the spending limit all belong to the organization; each organization's services and billing are separate.

You can belong to more than one organization. Switch between them from the **Organizations** section of the user menu in the top right, or open a new one with **Create organization**. An account can have at most 5 organizations.

### Project

A project is a folder that keeps related repos and apps together. Open one with **Apps → New Project**: enter a **Project Name** (3 to 100 characters) and a **Description**. You can also open one while creating an app or a Job, with the **Create new project…** option in the project list.

A project holds:

- The Git repos you connect and the apps that come from them,
- Jobs and CronJobs,
- API Gateways,
- Stacks,
- Managed service instances, if you choose. If no project is picked in the **Project** field when you create an instance, the instance stays **Shared**.

The project picker in the console's top bar narrows the lists to the project you choose; **All projects** shows everything.

A project is also a permission boundary. On the **Access Control** page you can give a member **Read** or **Edit** access to everything in a project.

> **Note:** A project does not set a region. The region is chosen separately each time you create an app, a managed service instance or an API Gateway.

### App (service)

An app is a running copy, on Komuta, of an application in your Git repo: a website, an API, a background worker. The console menu calls it **Apps**; most pages and these documents call it a **service**. They are the same thing.

One repo can produce several services. Each service has its own resource package, replicas, ports and address, environment variables, pipeline history and logs.

### Jobs and CronJobs

A Job is a task that runs once and finishes; a CronJob ties the same task to a schedule. They are for work that does not need to run all the time, such as a database migration, a nightly report or a bulk email. Create them from the **Jobs & CronJobs** page.

### Managed services

The **Managed Services** menu has two kinds of product:

- **PostgreSQL, Valkey and RabbitMQ**: a relational database, an in-memory key-value store and a message queue. Each one is created as an **instance**; its CPU, memory and disk come from the **plan** you choose.
- **API Gateway**: one address in front of your APIs. You manage its keys and rate limits from the console. It is created under a project.

### Stack

A Stack defines the services, managed services, domains and secret values that make up an application in a single file. **Validate** shows the errors in the file, and **Create plan** lists what will be created and the **Estimated monthly cost**; resources are created only after the plan is approved. A repo with `komuta.yaml` at its root is sent to Stack creation from the new app flow.

---

## From Code to Live: A Service's Journey

A service's life connects the concepts you see in the console in this order:

1. **Source.** You connect your Git account on the **Integrations** page. The service follows a repo and a branch.
2. **Pipeline.** A pipeline starts when you press **Deploy**, or on every push while auto-deploy is on. The source code is prepared, the image is built, scanned for security issues and vulnerabilities, signed and deployed. Builds wait in a queue; how many run at the same time depends on your account tier.
3. **Image.** The built image is pushed to a container registry. Komuta's own registry is ready to use.
4. **Deployment.** The new version goes live according to the service's rollout strategy: **Blue/Green**, **Auto Promote** or **Canary**. Every deployment stays on the **History** page; **Rollback** returns to an earlier version.
5. **Running.** The service runs with the CPU and memory of its resource package, as the set number of replicas. Health probes take a failing replica out of traffic or restart it. Autoscaling changes the replica count with the load.
6. **Traffic.** A request from the internet reaches the service's **Public URL** (the domain Komuta gives you, or your own custom domain), passes access protection and is sent to the service's port.
7. **Watching.** Logs, metrics and traces are collected. When a condition is met, an alert rule sends a notification.

A service reaches another of your services over the private network through the **Connections** page. It connects to a managed database by adding the instance's connection details to the service's environment variables; in a Stack file, that value can be read straight from the instance.

---

## Where Things Are in the Console

The left menu is grouped under four headings:

| Heading | What it holds |
|---|---|
| **Workloads** | **Apps**, **Jobs & CronJobs**, **Stacks**, **Managed Services** (PostgreSQL, Valkey, RabbitMQ, **API Gateways**) |
| **Monitoring** | **Observability**, **Alerts**, **Security** |
| **Configuration** | **Integrations**, **Domains**, **Environment Groups** |
| **Administration** | **Access Control**, **Account** |

When you open a service, it gets its own menu: **Overview**, **Logs**, **Deployment** (**Pipelines**, **History**, **Auto-deploy**), **Observability**, **Security**, **Configuration** (**Environment variables**, **Connections**, **Access & ports**, **Health probes**, **Autoscaling**), **Bandwidth**, **Storage** and **Invoice**.

The **Account** pages have their own menu in three groups: **Personal** (**Profile**, **Password**, **Two-Factor**, **API Keys** and more), **Organization** (**Organizations**, **General**, **Notifications & Alerts**) and **Billing** (**Current Spend**, **Wallet**, **Statements**, **Payment Methods**). Only organization admins see the **Billing** group.

---

## Glossary

The console is available in English and Turkish; the names below are the English ones.

| In the console | What it means | Read more |
|---|---|---|
| **Organization** | The workspace that owns services, members, Git connections and billing. | [Organizations and Team](organizations-team.md) |
| **Project** | A folder inside an organization that groups related repos, apps, Jobs, gateways and Stacks. Also a permission boundary. | [Creating a New Service](new-service-guide.md) |
| **Apps** (service) | An application built from a Git repo that runs all the time. These documents call it a "service". | [Creating a New Service](new-service-guide.md), [Service Dashboard](service-dashboard.md) |
| **Jobs & CronJobs** | Work that runs once and finishes (Job) or runs on a schedule (CronJob). | [Jobs and CronJobs](jobs-cronjobs.md) |
| **Managed Services** | PostgreSQL, Valkey, RabbitMQ and API Gateway, set up and run by Komuta. | [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md) |
| Instance (**New instance**) | One copy of a managed service, with its own disk, user and backups. | [PostgreSQL](managed-postgresql.md) |
| **Plan** | The CPU, memory and disk size of a managed service instance. It can be upgraded later. | [PostgreSQL](managed-postgresql.md) |
| **Shared** | A managed service instance not attached to any project. | [PostgreSQL](managed-postgresql.md) |
| **API Gateways** | One address in front of your APIs, with keys and rate limits. | [API Gateway](api-gateway.md) |
| **Stacks** | A setup that defines services, managed services and domains in one file and creates them with a plan. | [Stacks](stacks.md), [Stack Manifest Reference](stack-manifest-reference.md) |
| **Region** | Where a service or instance runs. There is currently one region: Germany-East. | [Creating a New Service](new-service-guide.md) |
| **Resource Package** | A service's amount of CPU and memory; it also sets the price. | [Billing](billing.md) |
| **Replicas** | The copies of a service that run at the same time. | [Autoscaling](services-autoscaling.md) |
| **Autoscaling** | The setting that moves the replica count between a min and a max based on CPU and memory usage. | [Autoscaling](services-autoscaling.md) |
| **Pipelines** | The runs in which code is built, scanned and deployed to the service. | [Pipelines](service-pipeline-guide.md), [Build Queue](build-queue.md) |
| **Deployment** | Taking a new version live; earlier versions are on the **History** page. | [Deployment History](service-deployment-history.md) |
| Rollout strategy (**Blue/Green**, **Auto Promote**, **Canary**) | The method that decides how a new version takes over traffic. | [Deployment Strategies](deployment-strategies.md) |
| **Auto-deploy** | The pipeline starting on its own when a push reaches the tracked branch. It can also be triggered from CI. | [Auto Deploy](service-auto-deploy.md), [Deploying from CI](ci-deploy.md) |
| **Environment variables** | Configuration values the app reads while it runs. | [Environment Variables](services-environment-variables.md) |
| **Environment Groups** | Variables that several services share. Values are copied into the service; changing the group does not change a service until you push the update. | [Environment Groups](environment-groups.md) |
| **Connections** | A service reaching another of your services over the private network, without a public address. One-way. | [Connections](services-connections.md) |
| **Access & ports** | Which port traffic enters the service through, and who can open its address. | [Access & Ports](services-ports.md) |
| **Public URL** | The address that reaches the service from the internet: the **Komuta domain** and, if you add one, a **Custom domain**. | [Access & Ports](services-ports.md) |
| **Domains** | The page where you connect your own domain to an app. The certificate is issued and renewed automatically. | [Custom Domains](custom-domains.md) |
| **Access protection** | Protection that opens a service's address only to the people, networks or machines you choose. Included in every plan. | [Access Protection](service-access-protection.md) |
| **Health probes** | Checks that regularly ask whether the service is ready and alive. | [Health Probes](services-health-probes.md) |
| **Storage** | Persistent volumes attached to the service that survive pod restarts. | [Storage](storage.md) |
| **Bandwidth** | The page that shows the service's traffic and its bandwidth plan. The plan sets the outbound speed limit for each of the service's pods; the default plan is free and the higher plans have a monthly fee. | [Bandwidth Plans](bandwidth-plans.md) |
| **Sleeping** | The state in which a free service with no traffic has its replicas scaled to zero. It wakes on the first request. | [Service Sleep](service-sleep.md) |
| **Always on** | A paid add-on that keeps a service from sleeping. While it is on, the service does not use the free hours pool. | [Service Sleep](service-sleep.md) |
| **Logs** | The page where the service's container output is searched and followed live. | [Logs](service-logs.md) |
| **Observability** | The area where logs, metrics and traces are explored together. | [Observability](observability.md) |
| **Alerts** | Rules that send a notification when a metric or log condition is met. | [Alerts: Overview](alert-guide.md) |
| **Channels** | The email, Slack and Microsoft Teams destinations alerts go to. | [Channels and Delivery](notification-guide.md) |
| **Security** (Security Center) | The area that brings security findings, service protection, access activity and audit records together. | [Security Center](security-center-guide.md) |
| **Integrations** | Connections to Git providers (GitHub, GitLab, Azure DevOps) and container registries. | [Integrations](integrations.md) |
| **Access Control** | The page where you invite members and set their role and per-project permissions. | [Organizations and Team](organizations-team.md) |
| **Admin**, **Member** | The two roles in an organization. Only admins see the billing pages. | [Organizations and Team](organizations-team.md) |
| **Wallet** | Your balance, spending tier and the limit increase request. | [Billing](billing.md) |
| **Statements** | The statements issued at the end of each billing cycle. | [Billing](billing.md) |
| **Current Spend** | Spending so far in the current cycle. | [Billing](billing.md) |
| **Monthly resource estimate limit** | The monthly estimate cap that lets you create something new or move to a bigger plan. It does not stop running usage. | [Billing](billing.md), [Limits](limits.md) |
| **API Keys** | Personal keys that give your own agent or CI programmatic access to Komuta. The secret is shown once. | [Account Security](account-security.md) |
| Komuta MCP | The server that lets your AI assistant act on Komuta on your behalf. | [Installing the Komuta MCP Server](mcp-setup.md), [Komuta MCP](mcp-tools.md) |

---

## Related Documents

- [Quick Start](quick-start.md) — The steps from sign-up to your first service.
- [Creating a New Service](new-service-guide.md) — The new app wizard in detail.
- [Organizations and Team](organizations-team.md) — Members, roles and switching organizations.
- [Billing](billing.md) — Wallet, statements and the spending limit.
- [Limits](limits.md) — Account tiers and platform limits.
- [Pricing](https://www.komuta.io/pricing) — Packages, the free plan and prices.
