# Limits and Quotas

This page collects the limits you can run into on Komuta in one place: account and organization, spending limit, free plan, services, builds, Jobs, Stacks, domains, managed services, API Gateway, logs and alerts. Each row gives the value and, where there is one, the way past it; how each feature works is explained on the linked page.

> **Note:** Limits can change over time. The console shows the value that currently applies to your organization: your spending limit and tier on **Account → Wallet**, packages and plans in the creation wizards, and a free service's remaining hours in the **Free hours** item on the service dashboard.

---

## Account and Organization

| Limit | Value | Notes |
|---|---|---|
| Organizations per account | At most 5 | Organizations you created and organizations you joined by invitation count together. At the limit you can neither create an organization nor accept an invitation. |
| Organization name | At most 64 characters | At least 2 characters when renaming. |
| Invitation validity | 72 hours | Resending an invitation starts a new 72 hours. A new invitation to the same address replaces the previous one. |
| API key name | At most 128 characters | **Account → API Keys** |
| API key lifetime | 30 days, 90 days or 1 year; at most 365 days | A key created without an expiry date expires after 90 days; this also applies to the **No expiry** option in the console. |

For inviting teammates and roles, see [Organizations and Team](organizations-team.md).

---

## Spending Limit and Billing

Every organization has a monthly resource-estimate limit. It is checked only when you create something or upgrade its plan: if the monthly estimate would go over the limit, the action is refused, while usage that is already running is never stopped. You cannot set the limit yourself; it rises automatically as your tier goes up.

| Tier | Monthly limit | How it unlocks |
|---|---|---|
| Starter | USD 20 | Starting tier of a new organization |
| Verified | USD 50 | A verified card |
| Basic | USD 100 | USD 10 in total top-ups or USD 10 in paid statements |
| Pro | USD 300 | USD 50 in total top-ups or USD 50 in paid statements |
| Scale | USD 1,000 | USD 300 in total top-ups or USD 300 in paid statements |

Your tier is the highest one whose condition you meet. If you need a limit above Scale, or more than your tier gives, use **Request a limit increase** on **Account → Wallet** or open a [support ticket](support-tickets.md).

| Limit | Value | Notes |
|---|---|---|
| Single top-up | USD 5–1,000 | Split larger amounts into several top-ups. |
| Welcome credit | USD 10, valid for 30 days | Given to the first organization only; the unused part lapses when it expires. |
| Billing cycle | From the 6th to the 6th (UTC) | Statements are issued on the 6th of each month. |
| Smallest statement | USD 0.50 | No statement is issued for a cycle below this amount; the usage carries over to the next cycle. |
| Unpaid invoice | 14-day grace period, then suspension | During the grace period scaling up, plan upgrades and managed service plan changes are paused; the banner in the console shows the suspension date. A suspended account cannot make changes; viewing and paying stay available. |

For prices and billing details, see [Billing](billing.md) and [Pricing](https://www.komuta.io/pricing).

---

## Free Plan

| Limit | Value | Notes |
|---|---|---|
| Free services per organization | 1 | Some coupons grant extra free services. |
| Free package | `dz-nano`: 0.1 vCPU, 256 MB RAM | More resources come with paid packages. |
| Awake-hour pool | 450 hours in total per calendar month (UTC) | An organization's free services share one pool. When it runs out, free services sleep until the 1st of the next month and cannot be woken or deployed in the meantime. |
| Idle time | 30 minutes | A free service that receives no traffic for this long goes to sleep and wakes on the first request. Waking can take about a minute. |
| Replicas | One replica, no autoscaling | — |
| Git account | A personal GitHub, GitLab or Azure DevOps account linked to the organization | An account released from the free plan cannot be used for another organization's free plan for at least 30 days and not before the end of that month. |

To keep a free service out of the pool, turn on the **Always on** add-on: the service never sleeps and does not use your free hours. It costs USD 3 per billing cycle plus tax, charged for the part of the cycle it is on, and needs a verified card, a single replica and a Git account linked to the free plan. Details: [Service Sleep](service-sleep.md).

---

## Services

| Limit | Value | Notes |
|---|---|---|
| Service name | 1–53 characters | Lowercase letters, digits and hyphens; starts with a letter and ends with a letter or digit. Must be unique within the repository. |
| Project name | 3–100 characters | A description is required too: 3–500 characters. |
| Autoscaling: replicas | Minimum and maximum replicas 1–100 | Set both to the same number to run a fixed replica count. |
| Autoscaling: targets | CPU and memory target 1–100% | At least one target must be on. |
| Autoscaling: windows | 0–3600 seconds | Pods added or removed per minute: 1–1000. |
| Ports | 1–65535 | The same range applies to the port, service port and target port. |
| Environment variable key | Letters, digits, `-`, `_` and `.` | Spaces and other special characters are not allowed. |
| Bulk import | At most 500 variables at a time | Import larger sets in several passes. |
| Environment group | At most 500 variables per group | Group name at most 128 characters, key at most 256 characters. |
| Persistent storage plans | Free 1 GB, Small 10 GB, Medium 50 GB, Large 200 GB, XLarge 1024 GB | A service can hold one storage attachment on the Free plan. Free and paid storage cannot be mixed on the same service. |
| Storage name and path | Volume name at most 63 characters; mount path 2–256 characters | The same name or the same path cannot be used twice on one service. |

### Resource packages

| Package | vCPU | RAM | Temporary disk | Monthly |
|---|---|---|---|---|
| `dz-nano` | 0.1 | 256 MB | 512 MB | Free plan |
| `dz-dev` | 0.5 | 1 GB | 1 GB | USD 9 |
| `dz-standard` | 1 | 2 GB | 2 GB | USD 18 |
| `dz-premium` | 2 | 4 GB | 3 GB | USD 36 |
| `dz-enterprise` | 4 | 8 GB | 4 GB | USD 72 |

Prices exclude tax; usage is metered hourly and the monthly amount is the hourly price times 720. Current prices are on the [Pricing](https://www.komuta.io/pricing) page.

Related pages: [Autoscaling](services-autoscaling.md), [Ports](services-ports.md), [Environment Variables](services-environment-variables.md), [Environment Groups](environment-groups.md), [Storage](storage.md).

---

## Network Traffic

Every service has a bandwidth plan. The plan limits the outbound (egress) speed of each of the service's pods; inbound traffic is unlimited and not billed.

| Plan | Monthly | Outbound speed per pod |
|---|---|---|
| Starter (default) | Free | 5 Mbit/s |
| Growth | USD 4.99 | 25 Mbit/s |
| Business | USD 14.99 | 100 Mbit/s |
| Enterprise | USD 49.99 | 500 Mbit/s |
| Scale | USD 149.99 | 2 Gbit/s |

Details: [Bandwidth Plans](bandwidth-plans.md).

---

## Builds and Deploys

How many builds run at once and how many can wait in the queue depends on your spending tier:

| Tier | Concurrent builds | Builds that can wait in the queue |
|---|---|---|
| Starter | 1 | 5 |
| Verified | 1 | 10 |
| Basic | 2 | 20 |
| Pro | 4 | 30 |
| Scale | 8 | 50 |

| Limit | Value | Notes |
|---|---|---|
| Build duration | At most 1 hour | A build that runs longer ends with a timeout. |
| Queue wait | At most 6 hours | A build that cannot start within this time times out in the queue. |
| Included build minutes | 2,000 minutes per billing cycle | Beyond that, USD 0.008 per minute. Time spent waiting in the queue is not counted. |
| Auto-deploy cooldown | 0–3600 seconds, 30 seconds by default | The shortest time between two automatic triggers. |
| CI deploy tokens | At most 10 active tokens per service | Lifetime 365 days by default, 730 days at most. |
| CI deploy requests | Per token, 30 deploys and 300 status reads per minute; per service, 60 deploys per hour | Over the limit you get `429`; wait for `Retry-After`. |
| CI deploy records | 180 days | The status URL of an older deploy returns `404`. |

Related pages: [Build Queue](build-queue.md), [Pipelines](service-pipeline-guide.md), [Auto-deploy](service-auto-deploy.md), [Deploying from CI](ci-deploy.md).

---

## Jobs and CronJobs

| Limit | Value | Notes |
|---|---|---|
| Script size | At most 64 KB (65,536 bytes) | Run larger work from a source built from your repository. |
| Schedule | A five-field cron expression, or `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly` | At most once a minute. A seconds field, `@every` and a `CRON_TZ=` prefix are not accepted; a date that never occurs (for example `0 0 30 2 *`) is refused. |
| Time zone | An IANA name (for example `Europe/Istanbul`) or `UTC` | — |

Details: [Jobs and CronJobs](jobs-cronjobs.md).

---

## Stacks

| Limit | Value | Notes |
|---|---|---|
| Manifest version | `version: 1` only | — |
| Manifest size | At most 1 MiB, a single YAML document | — |
| Nesting depth | At most 32 levels | — |
| Resources | At most 200 per manifest | Services, created and reused managed services, domains and secrets count together. |
| Name length | At most 160 characters | — |
| `apply.maxParallel` | 1–10 | — |
| Plan validity | 30 minutes | An expired plan cannot be applied; create a new one. |
| Secure secret link | 15 minutes, single use | Only the person who created the link can open it. |

Details: [Stacks](stacks.md), [Stack Manifest Reference](stack-manifest-reference.md).

---

## Domains

| Limit | Value | Notes |
|---|---|---|
| Adding custom domains | At most 10 per organization per day | The counter resets every day at 00:00 UTC. |

Details: [Custom Domains](custom-domains.md).

---

## Access Protection

The limits of access protection (IP lists, path rules, shares, service tokens, session lengths, code sending rates, access log retention) are collected in their own table: [Access Protection: Reference → Limits](access-protection-reference.md#limits).

---

## Managed Services

For PostgreSQL, Valkey and RabbitMQ, the plan sets the CPU, memory, disk and server count; the plans and their current sizes are listed in the creation wizard. Plan-dependent connection, memory and query limits are described on each service's own page.

| Limit | Value | Notes |
|---|---|---|
| PostgreSQL databases on the 512 MB and 1 GB plans | 5 per organization | Counted in every state, suspended and pending deletion included. Larger plans have no such limit. |
| Backup retention | 2–365 days, 7 days by default | The same for all three services. Shortening it cannot be undone: backups outside the new window are deleted. |
| Scheduled deletion | 7 days | Can be cancelled before the period ends. Valkey and RabbitMQ keep running and being billed meanwhile; PostgreSQL is taken offline and only its disk is billed. |

Details: [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md).

---

## API Gateway

| Limit | Value | Notes |
|---|---|---|
| APIs | At most 40 per gateway | — |
| Endpoints | At most 300 per gateway | All APIs share this limit. |
| API keys | At most 1,000 per gateway | Keys that are not revoked count, expired ones included. |
| **Largest request body (MiB)** | 1–100, 10 by default | — |
| Timeout | At most 300 seconds, 10 seconds by default | — |
| Rate limit | Requests per second per key; 0–10,000 in the endpoint form (0 = unlimited) | Each replica counts on its own; when the counter is kept in your own Valkey instance, all replicas share one budget. A caller cannot have both a quota and a rate limit. |
| Cache lifetime | At least 1 second, 60 seconds by default | At most 3600 seconds in the endpoint form. |
| **Largest cacheable response (KiB)** | 64–32,768, 512 by default | Cannot exceed a quarter of the gateway's cache zone. |
| Replicas | 1–10 | Up to 20 when the rate limit counter is kept in your own Valkey instance. |

Details: [API Gateway](api-gateway.md).

---

## Logs

| Limit | Value | Notes |
|---|---|---|
| Time range of one query | At most 12 hours | Go further back with **Load older logs**. |
| Cross-app search | At most 25 apps in one search | Narrow the scope with the app picker when you have more. |

Details: [Service Logs](service-logs.md), [Observability](observability.md).

---

## Alerts

| Limit | Value | Notes |
|---|---|---|
| Rule name | At most 128 characters | In the console's rule form. |
| Summary | At most 256 characters | — |
| Query | At most 2,000 characters | — |
| Notification repeat interval | At least 5 minutes | Leaving it blank does not turn notifications off. |

Details: [Alert Rules](alerts-rules.md).

---

## Support Tickets

| Limit | Value | Notes |
|---|---|---|
| Subject | 3–256 characters | — |
| Message and reply | 3–4000 characters | — |

---

## When You Hit a Limit

- **Spending limit:** Your tier rises on its own as you verify a card, top up and pay statements. For more, use **Request a limit increase** on **Account → Wallet**.
- **Free services:** If you have a coupon that grants extra free services, redeem it; run other services on paid packages.
- **Free hours:** The pool applies only to free services. Services on paid packages, and a service with the **Always on** add-on, do not use it.
- **Other limits:** Tell us what you need in a [support ticket](support-tickets.md).

---

## Related Documents

- [Quick Start](quick-start.md) — Account, organization and first steps.
- [Billing](billing.md) — Wallet, statements and the spending limit.
- [Service Sleep](service-sleep.md) — How free services sleep and wake.
- [Build Queue](build-queue.md) — Concurrent builds and queue order.
- [Access Protection: Reference](access-protection-reference.md) — Every access protection limit.
- [Support Tickets](support-tickets.md) — Limit increases and other requests.
