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

You can create a professional access address by connecting your own domain to a service on DevOpsZon.

### Steps

**1. Adding a Domain**

- Go to the **Domains** page from the left menu
- Click the **Yeni Domain Ekle** button
- Enter your domain name (e.g.: `api.mycompany.com`)
- Select the service to connect it to

**2. DNS Verification**

DevOpsZon will ask you to create a DNS record to verify your domain ownership:

| Record Type | Host | Value |
|------------|------|-------|
| **CNAME** | `api.mycompany.com` | Target address provided by DevOpsZon |

or

| Record Type | Host | Value |
|------------|------|-------|
| **TXT** | `_devopszon.api.mycompany.com` | Verification code |

> DNS changes may take anywhere from a few minutes to 48 hours to propagate.

**3. Verification Check**

After creating the DNS record, click the **Verify** button. DevOpsZon will check your DNS record.

- **Success:** The domain is activated and connected to your service
- **Failure:** Check your DNS record and try again

**4. TLS Certificate**

Once the domain is verified, a TLS certificate is automatically created. Secure access to your service is provided over `https://api.mycompany.com`.

---

## Cloudflare Integration

DevOpsZon works integrated with Cloudflare in SaaS mode:

| Feature | Description |
|---------|----------|
| **DNS management** | Automatic DNS record creation and updates |
| **CDN** | Fast access with static content caching |
| **DDoS protection** | Automatic attack blocking |
| **SSL/TLS** | Full (strict) TLS mode |

You can see the Cloudflare status of each hostname in the Ingress Management tab.

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

- **Wildcard domain:** By creating a wildcard CNAME record in the form of `*.mycompany.com`, you can route all subdomains
- **DNS propagation time:** If verification fails, wait a few minutes and try again
- **HTTPS enforcement:** All services are served over HTTPS by default; HTTP requests are automatically redirected
