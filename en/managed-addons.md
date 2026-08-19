# Managed Services (Managed Addons)

DevOpsZon offers fully managed database, message queue, and cache services for the needs of your applications. These services are automatically provisioned, configured, backed up, and monitored.

---

## Supported Services

| Service | Type | Use Case |
|--------|-----|----------------|
| **PostgreSQL** | Relational Database | Structured data storage, ACID-compliant transactions |
| **RabbitMQ** | Message Queue | Asynchronous messaging, event-driven architecture, task queuing |
| **Valkey** | In-Memory Data Store | Caching, session management, real-time data |

---

## PostgreSQL — Managed Database

### Features

| Feature | Detail |
|---------|-------|
| **Version** | PostgreSQL 16 and 17 |
| **Topology** | Single Instance and HA (Primary + Replica) |
| **Connection Pooling** | Built-in PgBouncer (transaction mode) |
| **Automatic Backup** | Daily backup, S3/B2 storage |
| **TLS** | Encrypted connection with automatic certificate |
| **Access** | Unique hostname via SNI routing (e.g. `pg-a1b2c3d4.devopszon.app`) |
| **Performance Tuning** | Automatic PostgreSQL tuning based on plan |

### Plans

#### Single Instance

| Plan | vCPU | RAM | Disk | Max Connection |
|------|:----:|:---:|:----:|:--------------:|
| **postgres-2g** | 0.5 | 2 GB | 40 GB | 250 |
| **postgres-4g** | 1 | 4 GB | 80 GB | 500 |
| **postgres-8g** | 2 | 8 GB | 160 GB | 1,000 |
| **postgres-16g** | 4 | 16 GB | 320 GB | 2,000 |
| **postgres-32g** | 8 | 32 GB | 640 GB | 4,000 |

#### HA (Primary + Replica)

| Plan | vCPU (per instance) | RAM (per instance) | Disk (per instance) |
|------|:-------------------:|:------------------:|:-------------------:|
| **postgres-8g-ha** | 2 | 8 GB | 160 GB |
| **postgres-16g-ha** | 4 | 16 GB | 320 GB |
| **postgres-32g-ha** | 8 | 32 GB | 640 GB |

> HA plans include automatic failover. When the Primary server goes down, the Replica automatically takes over the Primary role.

### Creating PostgreSQL

1. Go to the **Addons** page from the left menu
2. Select the **PostgreSQL** tab
3. Click the **Create New Instance** button
4. Choose a plan and topology
5. The creation process will start, and progress will be shown step by step

**Setup steps (automatic):**

| Step | Description |
|------|----------|
| Namespace creation | Isolated workspace |
| Database cluster setup | Via CloudNativePG operator |
| PgBouncer deployment | Connection pooling |
| TLS certificate generation | Via Let's Encrypt |
| DNS record creation | SNI hostname |
| Health check | Connection verification |
| Backup configuration | Automatic daily backup |

### Connection Information

Once the instance is created, you can access your connection information from the addon detail panel:

- **Hostname:** `pg-{id}.devopszon.app`
- **Port:** `30930`
- **Database name:** Automatically generated
- **Username and password:** Stored securely and can be copied from the panel

**Connection string examples:**

```
postgresql://user:password@pg-a1b2c3d4.devopszon.app:30930/mydb?sslmode=require
```

### Plan Change

You can upgrade or downgrade the plan of your existing PostgreSQL instance:

1. Click the **Change Plan** button in the addon detail panel
2. Select the new plan
3. Confirm

> **Note:** A brief interruption may occur during a plan change. This duration is minimized on HA plans.

---

## RabbitMQ — Managed Message Queue

### Features

| Feature | Detail |
|---------|-------|
| **Topology** | Single Broker and HA Quorum Cluster (3 nodes) |
| **Protocol** | AMQPS (5671) — encrypted with TLS |
| **Management UI** | Web-based management interface (port 15672) |
| **Automatic Backup** | Daily definition and message snapshot |
| **SNI Routing** | Unique hostname (e.g. `rmq-{id}.devopszon.app`) |
| **Guardrails** | MaxMemory, MaxConnections, MaxQueues, MaxChannels limits |

### Creating RabbitMQ

1. On the **Addons** page, select the **RabbitMQ** tab
2. Click the **Create New Instance** button
3. Choose the topology and plan
4. The 8-step setup process can be monitored in real time via SignalR

### Connection Information

- **AMQPS Hostname:** `rmq-{id}.devopszon.app:5671`
- **Management UI:** `https://rmq-{id}.devopszon.app:15672`
- **Username and password:** Can be copied from the panel

### Plan Change

You can change CPU, RAM, and disk resources live. Zero-downtime plan changes are supported on HA topologies.

---

## Valkey — Managed In-Memory Data Store

### Features

| Feature | Detail |
|---------|-------|
| **Usage** | Caching, session management, real-time data |
| **High performance** | In-memory data access |
| **TLS** | Encrypted connection support |
| **SNI Routing** | Access via unique hostname |

### Creating Valkey

Follows the same flow as RabbitMQ and PostgreSQL:
1. On the **Addons** page, select the **Valkey** tab
2. Choose a plan and create it
3. Get your connection information from the panel

---

## Addon Management Panel

The detail panel of each addon instance includes the following sections:

| Section | Description |
|-------|----------|
| **Genel Bilgiler** | Status, plan, topology, creation date |
| **Connection Info** | Hostname, port, username, password, connection string |
| **Actions** | Restart, plan change, deletion |
| **Yedekleme** | Backup history and manual backup trigger |
| **Plan Change** | Increase/decrease resources |

---

## Security

For all managed services:

- **TLS/SSL:** All connections are encrypted
- **Network isolation:** Pod-level isolation via NetworkPolicy
- **Secure storage:** Access credentials are stored as Kubernetes Secrets
- **SNI Routing:** Access via a dedicated subdomain for each instance

---

## Tips

- **Connection pooling:** Route your PostgreSQL connections through PgBouncer; avoid using direct connections
- **HA plans:** Prefer HA topology for production environments
- **Backup check:** Monitor from the addon panel that automatic backups are being created regularly
- **Plan selection:** You can start with a small plan and scale up as your needs grow
