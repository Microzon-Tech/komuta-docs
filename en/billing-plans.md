# Billing and Resource Plans

DevOpsZon's billing system offers transparent pricing based on the resources you use. This guide explains resource plans, invoice management, and payment processes.

---

## Billing Model

### SaaS Mode

In SaaS mode, billing is done on a **monthly basis** according to the resource plans you select:

- A resource plan is selected for each service
- Managed services (PostgreSQL, RabbitMQ, Valkey) are billed separately
- An invoice is automatically generated at the end of the billing period

### Hybrid Mode

In Hybrid mode, a **fixed monthly fee** applies for each connected Kubernetes cluster:

- You are billed separately for each cluster you connect
- How much resource you use on the cluster is your own choice
- Managed services (addons) are billed additionally

---

## Resource Plans

When choosing a resource plan for your services, you determine the CPU, memory, and disk capacity according to your application's needs.

### Service Resource Packages

A resource package is selected for each service during or after project creation. This package determines the resource request and limit values in Kubernetes.

### Addon Plans

Dedicated plans are available for managed services:

**PostgreSQL Plans:**

| Plan | vCPU | RAM | Disk | Connection Limit |
|------|:----:|:---:|:----:|:---------------:|
| postgres-2g | 0.5 | 2 GB | 40 GB | 250 |
| postgres-4g | 1 | 4 GB | 80 GB | 500 |
| postgres-8g | 2 | 8 GB | 160 GB | 1,000 |
| postgres-16g | 4 | 16 GB | 320 GB | 2,000 |
| postgres-32g | 8 | 32 GB | 640 GB | 4,000 |

HA plans offer a Primary + Replica topology at an additional cost.

---

## Wallet and Balance

Your DevOpsZon account has a **wallet**:

- You can top up your account balance
- Invoices are automatically deducted from the wallet
- You receive a notification when your balance is low

### Viewing Balance

You can view your wallet balance and transaction history from the **Account** page:

| Information | Description |
|-------|----------|
| **Current Balance** | The current amount in your wallet |
| **Estimated Monthly Cost** | Estimated monthly invoice based on your current resource usage |
| **Recent Transactions** | History of balance top-ups and invoice deductions |

---

## Invoice Management

### Viewing Invoices

You can access your invoices from **Account** → **Invoices**:

| Information | Description |
|-------|----------|
| **Billing Period** | The date range covered by the invoice |
| **Total Amount** | Invoice total |
| **Status** | Paid, Pending, Overdue |
| **Line Items** | Details of the services that make up the invoice |

### Invoice Line Items

Each invoice can include the following types of line items:

| Item | Description |
|-------|----------|
| **Service resources** | Cost of the resource package selected for each service |
| **PostgreSQL** | Managed database plan |
| **RabbitMQ** | Managed message queue plan |
| **Valkey** | Managed in-memory store plan |
| **Cluster (Hybrid)** | Fixed fee per connected cluster |

---

## Changing Plans

You can change the plan of an existing service or addon at any time:

### Changing a Service Plan

1. Go to the **Resource Plan** tab in the service management panel (SaaS mode)
2. Your current plan and the available plans are displayed
3. Select the new plan and confirm
4. The change takes effect starting from the next billing period

### Changing an Addon Plan

1. Click the **Change Plan** button in the addon detail panel
2. Choose an upgrade (higher plan) or downgrade (lower plan)
3. Review the effects and confirm

> **Note:** For downgrade operations, if your current data volume exceeds the disk capacity of the new plan, the operation cannot be performed.

---

## Payment Policy

### On-Time Payment

Invoices are automatically generated at the end of the period and deducted from the wallet balance.

### Late Payment

If an invoice is not paid on time:

| Duration | Action |
|------|-------|
| **Invoice date** | A notification is sent |
| **After 7 days** | The service's replica count is reduced to zero (the service is stopped) |
| **After 14 days** | Resources are permanently deleted |

> **Warning:** Resources cannot be recovered once deleted. It is important to keep track of your payment plan.

---

## Tips

- **Balance tracking:** Keep low-balance notifications enabled
- **Choosing the right plan:** Start with a small plan and scale up as needed
- **HA plans:** Prefer HA plans for production environments; weigh the additional cost against the risk of downtime
- **Invoice history:** Review past invoices regularly and monitor unexpected cost increases
