# Ports

Ports determine which door traffic enters your service through and which port inside the application it's routed to. If a service is going to receive requests from outside, at least one port must be defined.

![alt text](img/services-ports-page.png)
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

![alt text](img/services-ports-add-port.png)

---

## Private Network (Mesh)

A service can be exposed to your other clusters over a private network. In this case, applications reach each other directly without public ingress — used for communication between services that shouldn't go out to the internet.

Once enabled, the clusters that can reach it are listed; setting up the connection can take a few minutes.

![alt text](img/services-ports-private-mesh.png)

---

## Related Documents

- [Ingress and Domains](ingress-domains.md) — Managing addresses exposed to the outside world.
- [Service Dashboard](service-dashboard.md) — Traffic flow from ingress to pods.
