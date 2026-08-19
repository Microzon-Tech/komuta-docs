# Platform Overview

DevOpsZon is a comprehensive DevOps platform that lets you manage your modern applications end-to-end on Kubernetes. This page summarizes all the capabilities and working models the platform offers.

---

## Working Models

DevOpsZon offers three different working models:

### SaaS (Platform as a Service)

Publish your applications on infrastructure managed by DevOpsZon. Kubernetes cluster setup, maintenance, and monitoring are handled entirely by DevOpsZon.

- Choose a plan based on your resource needs
- Connect your Git repo, discover services, and publish them
- An automatic hostname is generated for each service (e.g., `my-api-a1b2c3d4.devopszon.com`)
- Pay as you go with monthly billing

### Hybrid

Connect your own Kubernetes cluster to DevOpsZon and take advantage of all the management panel's capabilities.

- **Existing cluster:** Connect with Kubeconfig
- **New cluster:** Automatically provision one on your cloud provider from the DevOpsZon interface
- You can connect multiple clusters
- All resources belong to you; a fixed pricing policy applies

### On-Premise

A DevOpsZon installation on a Kubernetes platform running entirely within your isolated environment. Your data and infrastructure remain 100% under your control.

---

## Core Capabilities

### Project and Service Management

| Capability | Description |
|---------|----------|
| **Project organization** | Group your applications under projects |
| **Automatic service discovery** | Dockerfiles in your Git repos are automatically detected |
| **Multi-repo support** | You can connect multiple repositories to a project |
| **Bulk operations** | You can deploy multiple services at the same time |

### CI/CD Pipeline

| Capability | Description |
|---------|----------|
| **Komuta Pipeline** | Kubernetes-native CI/CD infrastructure |
| **Automatic build & push** | Building an image from the Dockerfile and pushing it to the registry |
| **Live pipeline monitoring** | Track the build process step by step in real time |
| **Pipeline history** | View and compare your entire build history |
| **Queue management** | Automatic queuing for concurrent builds |

### Deployment Strategies

| Strategy | Description |
|----------|----------|
| **Rolling Update** | The default strategy; pods are updated gradually |
| **Canary** | Test the new version by routing a small percentage of traffic to it |
| **Blue/Green** | Prepare the new version in a parallel environment and switch over all at once |
| **Auto-Promote** | Automatic full rollout once the specified conditions are met |

### Managed Services (Addons)

| Service | Features |
|--------|-----------|
| **PostgreSQL** | Fully managed database; Single and HA (Primary + Replica) topologies; automatic backups; connection pooling (PgBouncer); secure connection via TLS |
| **RabbitMQ** | Fully managed message queue; Single Broker and HA Quorum Cluster; Management UI; automatic backups; TLS |
| **Valkey** | Fully managed in-memory data store; a high-performance caching solution |

### Monitoring and Observability

| Capability | Description |
|---------|----------|
| **Real-time dashboard** | Pod status, traffic flow, and resource consumption |
| **Live traffic visualization** | Monitor Ingress → Pod traffic flow live |
| **[Komuta Observability](observability-guide.md)** | Trace-derived RED, endpoint, trace, network dependency, SLO, anomaly, and evidence coverage |
| **Komuta Logs** | Centralized log collection, filtering, histograms, and trace correlation |
| **Komuta Alerts** | Metric- and log-based alerting rules |
| **Multiple notification channels** | Email, Slack, Telegram, Teams, PagerDuty, SMS, WhatsApp |

### Network and Access

| Capability | Description |
|---------|----------|
| **Ingress management** | Host-based routing rules |
| **Custom domain** | Connect your own domain name to services |
| **TLS/SSL** | Automatic certificate management (Let's Encrypt) |
| **Cloudflare integration** | DNS and CDN integration |
| **Automatic hostname** | A unique `*.devopszon.com` address for each service |

### Security and Authorization

| Capability | Description |
|---------|----------|
| **Access control** | Read/Edit permissions at the project, service, cluster, and integration level |
| **Multi-user** | Invite your team members and manage their permissions |
| **Tenant isolation** | Each organization runs in a fully isolated environment |
| **Network policies** | Pod-level network isolation and rule-based traffic management |

---

## Architecture Overview

```
┌──────────────────────────────────────────────────┐
│                DevOpsZon Console                  │
│                 (Web Application)                 │
└────────────────────┬─────────────────────────────┘
                     │ REST API + Live Updates
┌────────────────────▼─────────────────────────────┐
│                  DevOpsZon API                    │
│    Project / Service / Cluster / Pipeline / Alert │
└───┬──────────┬──────────┬──────────┬─────────────┘
    │          │          │          │
┌───▼────┐ ┌──▼────┐ ┌───▼────┐ ┌───▼────┐
│ Komuta │ │Komuta │ │ Komuta │ │ Komuta │
│Pipeline│ │Rollout│ │Observe │ │Gateway │
│(CI/CD) │ │(Deploy│ │ + Logs │ │  + DNS │
└────────┘ └───────┘ └────────┘ └────────┘
```

- **Komuta Pipeline:** Kubernetes-native CI/CD pipelines
- **Komuta Rollout:** Advanced deployment strategies (Canary, Blue/Green, Auto-Promote)
- **Komuta Observability + Logs:** Trace, endpoint RED, network flows, data health, and centralized log management
- **Komuta Gateway:** Traffic routing, DNS management, and automatic hostname registration
- **Real-time updates:** Notifications and status changes are reflected on the panel instantly

---

## Supported Integrations

### Git Providers
- GitHub
- GitLab
- Bitbucket
- Azure DevOps

### Container Registries
- Docker Hub
- GitHub Container Registry (GHCR)
- GitLab Container Registry
- Azure Container Registry (ACR)
- Amazon Elastic Container Registry (ECR)
- Google Container Registry (GCR)
- Custom Registry

### Notification Channels
- Email (SMTP)
- Slack
- Telegram
- Microsoft Teams
- PagerDuty
- SMS
- WhatsApp
- Webhook

### Cloud Providers
- Hetzner Cloud (primary)
- Extensible cloud provider infrastructure
</content>
</invoke>
