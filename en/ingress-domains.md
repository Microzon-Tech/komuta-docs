# Ingress and Domain Management

This guide explains how your services are accessed from the outside world, hostname configuration, and the process of connecting a custom domain.

---

## Automatic Hostname

When each service is created in DevOpsZon, a unique hostname is automatically assigned:

```
{service-name}-{unique-id}.devopszon.com
```

**Example:**
```
my-api-a1b2c3d4.devopszon.com
```

This hostname provides direct access to your service over HTTPS. The TLS certificate is automatically created and renewed with Let's Encrypt.

### Blue/Green Preview Hostname

For services using the Blue/Green deployment strategy, an additional preview hostname is created:

```
{service-name}-{unique-id}-preview.devopszon.com
```

This address is used to test the new version before making it live.

---

## Ingress Configuration

You can configure your services' traffic routing rules from the **Service Management** → **Ingress Management** tab.

### Basic Settings

| Setting | Description |
|------|----------|
| **Host** | The hostname to be routed to your service |
| **Path** | URL path-based routing (e.g.: `/api`, `/web`) |
| **Backend Port** | The port number your service listens on |
| **TLS** | SSL/TLS certificate status |

### Host Rules

You can bind more than one hostname to a service:

- The automatically generated `*.devopszon.com` hostname
- A custom domain (e.g.: `api.mycompany.com`)
- Preview hostname (in Blue/Green strategy)

### Path Rules

You can route different paths under the same hostname to different services:

```
myapp.devopszon.com
├── /api    → backend-service
├── /admin  → admin-service
└── /       → frontend-service
```

---

## Connecting a Custom Domain

In addition to the address Komuta gives your service, you can serve it on your own domain (for example `api.mycompany.com`).

### Before You Start

- The domain must point to a service that serves HTTP traffic. Workers, jobs and cron jobs can't have a custom domain.
- Start with a subdomain (`www.mycompany.com`, `api.mycompany.com`). A root domain (`mycompany.com`) needs a DNS provider that supports CNAME flattening, ALIAS or ANAME records at the zone apex.
- Wildcard domains (`*.mycompany.com`) aren't supported. Add each subdomain separately.

### 1. Add the Domain

1. Open **Domains** from the left menu.
2. Click **Add domain**.
3. Enter the hostname and choose the **Target App**.

### 2. Add the DNS Records

The domain's detail panel lists the **DNS Records to Add**. Create all of them at your DNS provider exactly as shown:

| Purpose | Type | Name | Value |
|---------|------|------|-------|
| Traffic routing | CNAME | `api.mycompany.com` | `origin.komuta.app` |
| Ownership verification | TXT | `_cf-custom-hostname.api.mycompany.com` | The code shown in the console |
| Certificate validation | CNAME | `_acme-challenge.api.mycompany.com` | `api.mycompany.com.<id>.dcv.cloudflare.com` (shown in the console) |

- Use the copy button next to each value. The ownership code and the certificate validation target are unique to your domain.
- You add the certificate validation record once. Certificates renew automatically after that, and you don't need to change DNS again.
- If your DNS is hosted on Cloudflare, set these records to **DNS only** (grey cloud). A record proxied through your own Cloudflare account (orange cloud) can't be verified, and access protection can't cover it.
- Domains added earlier may point to an address like `origin.edge-1.komuta.app`. They keep working, and you don't need to change them.

> DNS changes may take anywhere from a few minutes to 48 hours to propagate.

### 3. Wait for Activation

Each record shows whether Komuta can see it yet. When ownership verification and the certificate are complete, the domain's status changes to **Active** and your service is reachable at `https://api.mycompany.com`.

- Click **Refresh Validation** to check the status right away.
- A domain that isn't activated within 14 days is removed automatically. You can add it again at any time.
- If an active domain is marked for attention (for example because its DNS record no longer points to Komuta), check the records above.

### Removing a Domain

Select the domain and click **Remove**. Deleting a service also removes its custom domains.

---

## Cloudflare Integration

Custom domains are served through Cloudflare:

| Feature | Description |
|---------|-------------|
| **CDN** | Fast access with static content caching |
| **DDoS protection** | Automatic attack blocking |
| **SSL/TLS** | Certificates are issued and renewed automatically |

You can see each domain's status on the **Domains** page.

---

## Traffic Management

### Gateway API

Kubernetes Gateway API support is available for managing your services' traffic. This supports more advanced routing scenarios:

- Weighted traffic distribution
- Header-based routing
- gRPC support

### Rate Limiting

A fixed rate limit is applied to each service in SaaS mode. This ensures the platform's stability and fair use.

---

## Access Addresses of Managed Services

The access addresses of addon services (PostgreSQL, RabbitMQ, Valkey) use a different subdomain:

| Service | Format | Port |
|--------|--------|:----:|
| **PostgreSQL** | `pg-{id}.devopszon.app` | 30930 |
| **RabbitMQ (AMQPS)** | `rmq-{id}.devopszon.app` | 5671 |
| **RabbitMQ (Management)** | `rmq-{id}.devopszon.app` | 15672 |
| **Valkey** | `valkey-{id}.devopszon.app` | Configurable |

These addresses work with SNI (Server Name Indication) routing, and each instance is assigned its own dedicated TLS certificate.

---

## Tips

- **DNS propagation time:** If a record still shows as not detected, wait a few minutes and click **Refresh Validation**
- **HTTPS enforcement:** All services are served over HTTPS by default; HTTP requests are automatically redirected
