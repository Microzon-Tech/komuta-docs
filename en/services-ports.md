# Access & Ports

The **Service Detail → Configuration → Access & ports** page shows who can open the service's public URL (access protection) and which port traffic enters through. The **Public exposure** summary at the top shows step by step how traffic reaches the service (domains → gateway → access control → ports → pods).

Ports determine which door traffic enters your service through and which port inside the application it's routed to. If a service is going to receive requests from outside, at least one port must be defined.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-page.png)
---

## Port Definition

Values entered for each port record:

- **Name** — a label used to tell ports apart (e.g. `http`, `grpc`).
- **Port** — the number an incoming request from outside reaches the service on.
- **Target port** — the number the application listens on inside the container. The two can differ; traffic is routed from the port to the target port.
- **Protocol** — TCP, UDP, or SCTP.
- **Primary** — the service's default entry point. When a domain is attached, traffic goes to this port.
- **Active** — when off, the port stays defined but isn't connected to the application.

Both port and target port take a value between 1 and 65535.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-add-port.png)

---

## Private Network (Mesh)

A service can be exposed to your other clusters over a private network. In this case, applications reach each other directly without public ingress — used for communication between services that shouldn't go out to the internet.

Once enabled, the clusters that can reach it are listed; setting up the connection can take a few minutes.

![alt text](https://cdn.komuta.io/docs/tr/images/ports/services-ports-private-mesh.png)

---

## Access Protection

The page's **Rules**, **People**, **Machines**, **Activity** and **Settings** tabs manage access protection: visitors can be asked to sign in with Komuta, access can be limited to specific IP addresses, specific paths can be protected further or closed, and webhooks and CI tools can be let in separately. Ports, the public URL and the private mesh are on the **Network** tab. See [Access Protection](service-access-protection.md) for details.

---

## Related Documents

- [Ingress and Domains](ingress-domains.md) — Managing addresses exposed to the outside world.
- [Service Dashboard](service-dashboard.md) — Traffic flow from ingress to pods.
- [Service Access Protection](service-access-protection.md) — Protecting the service's public URL with sign-in, an IP allow-list and path rules.
