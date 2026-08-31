# Autoscaling

Autoscaling adjusts the service's replica count based on load: when CPU or memory usage rises above a set target, a replica is added, and when load drops, one is removed.

Settings you can configure:

- **Min and max replicas** — the lower and upper bounds the service can scale to, between 1 and 100. If both are set to the same number, the service always runs with that many replicas: the count doesn't change whether load rises or falls, and the target percentages are never taken into account. There's no need to turn off autoscaling for services that want a fixed replica count.
- **Target CPU and memory percentage** — the threshold used to create a new replica or remove an existing one. A replica is added when usage rises above this value and removed when it drops below it.
- **Scaling windows** — set in seconds, this determines how long to wait after a scaling decision, preventing the replica count from constantly bouncing up and down due to momentary fluctuations. When left empty, the default of 0 seconds applies, meaning no wait is enforced.
  - **Scale-up window** — the time to wait after adding a replica before the next increase. Keeping it short responds faster to sudden load spikes.
  - **Scale-down window** — the time to wait after removing a replica before the next decrease. Keeping it long prevents a service from prematurely losing capacity when traffic dips temporarily.

![alt text](img/sercices-autoscaling.png)
---

## Related Documents

- [Service Dashboard](service-dashboard.md) — The service's resource package and live usage.
- [Deployment History](deployment-history.md) — Running replica count and version information.
