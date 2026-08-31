# Health Probes

Health probes let Komuta periodically check whether your service is running properly. When a probe fails, the service is either taken out of traffic or restarted — which one happens depends on the probe type.

Saved settings are applied to the service within about 30 seconds.

![asd](img/services-health-controls.png)
---

## Probe Types

The three probes are turned on and off independently, and each produces a different outcome.

| Probe | What happens on failure |
|---|---|
| **Readiness** | The service is removed from the load balancer; it stops receiving traffic but keeps running. |
| **Liveness** | The container is restarted. |
| **Startup** | The container is given time to finish starting up; the other two probes don't kick in until this one passes. |

For slow-starting services, without a **Startup** probe, Liveness can kick in and restart the container before it's even ready. If your application has a long startup time, define this probe first.

---

## Probe Method

Each probe is performed using one of three methods:

- **HTTP GET** — a request is sent to the specified path, expecting a successful response. Default path is `/health`, port 80.
- **TCP** — the probe is considered successful if a connection can be made to the specified port. Used for services without an HTTP endpoint.
- **Exec** — a command is run inside the container; the probe passes if the command exits with a zero exit code.

---

## Timing Settings

| Setting | What it determines | Default |
|---|---|---|
| **Initial delay** | Time to wait after the container starts before the first probe | 10s |
| **Period** | Interval between probes | 10s |
| **Timeout** | Time after which an unanswered probe is considered failed | 5s |
| **Success threshold** | Number of consecutive successes needed to go from failed back to healthy | 1 |
| **Failure threshold** | Number of consecutive failures needed to be considered failed | 3 |

> **Warning:** Setting the failure threshold and period too low on the Liveness probe causes the container to be restarted unnecessarily during a temporary slowdown. The service experiences a brief outage during that restart.

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — The service's live health status and pods.
- [Autoscaling](autoscaling.md) — Adjusting replica count based on load.
