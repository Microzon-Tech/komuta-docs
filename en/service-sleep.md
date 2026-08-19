# Service Sleep Mode (Service Sleep / Hibernate)

Service Sleep is a cost optimization mechanism that lets your services running on the DevOpsZon platform automatically enter sleep mode during periods without traffic. A service in sleep mode runs **0 pods** — meaning it consumes no CPU or RAM — but automatically wakes up when the first HTTP request arrives and serves it. This flow is nearly invisible to the user; no manual intervention is needed to put a service to sleep or wake it up.

---

## Who is it for?

- **Development and test environments** — staging services sitting idle on weekends or after business hours.
- **Low-traffic projects** — dashboards, internal tools, demo applications used only a few hours a day.
- **Free plan users** — automatic sleep is on by default; it's the main mechanism that keeps the free plan sustainable.
- **Paid plan users** — you can enable it via the admin managing your plan if you wish. It stays off for production services that receive continuous traffic.

---

## How does it work? (High level)

1. **Measurement:** The platform measures the HTTP traffic each service receives in the background at regular intervals (15 minutes by default). We use Cilium Hubble's HTTP metrics as the measurement source — this data comes from your own traffic and never leaves your environment.
2. **Threshold check:** If a service is found to have received no HTTP requests in the last 3 hours (default), it is flagged as a "sleep candidate."
3. **Sleep operation:** The service's pod count is scaled to 0, its HPA (autoscaling) settings are saved, and the service's incoming traffic is temporarily routed to a small intermediary layer called the **Activator**.
4. **Automatic wake-up:** When a user sends a request to a sleeping service:
   - The Activator catches the request,
   - It wakes the service up in the background (the pod count is restored),
   - It holds the user's request until the pod is ready,
   - Once the pod is up, it forwards the request to the new pod and returns the response to the user.
5. **Ready for new requests:** The service now runs normally. If it sits idle for another 3 hours after the traffic ends, the sleep cycle starts again.

The **most important** part of this flow is that the wake-up operation is part of the request delivered to the user — meaning the address of the sleeping service does not change, and you don't need to make any changes on the client side.

---

## How long does waking up take? (Cold-start time)

Depending on the pod image size and the service's startup time, there is typically a **2–15 second** delay. This can be:
- 2–5 seconds for small Node.js / Go services,
- 5–15 seconds for .NET / Java services,
- longer for services with heavy startup steps (such as DB migration, cache warm-up)

The platform monitors cold-starts exceeding 20 seconds as an "SLO violation" and shows them as a warning on the dashboard.

---

## How do I see my service's sleep status?

### Service Dashboard (Overview)

A compact **sleep banner** appears at the top of the service detail's **Dashboard / Overview** tab. The banner is a single line tall and does not push the topology visual down. It contains:

- A circular **moon (🌙) icon** on the left and a "Service is sleeping" title,
- A single-line subtitle explaining the reason for sleep,
- **Time it went to sleep** information (e.g., "15.04.2026 22:30"),
- A primary **"Wake Up"** button on the right,
- Below the button, a "Keep my service always up →" link (a shortcut to upgrade your plan)

The banner only appears while your service is going to sleep / sleeping / waking up; it automatically disappears for active services.

The banner changes color during transition states:
- **In sleep mode** → amber (golden yellow) tone
- **Waking up** → blue tone (clock icon spins)
- **Sleep transition failed** → red tone ("Wake Up" can be retried)

### Service lists — cards

Sleeping services are distinguishable at a glance on the service cards on the Projects page:

- An amber-colored **"SLEEPING"** label (with a moon icon) in the top-right corner of the card,
- The card content is slightly dimmed (opacity decreases),
- The card's border turns amber and shows a subtle glow effect,
- The Deploy (rocket) button is hidden — triggering a deploy for a sleeping service isn't directly necessary; a deploy already wakes the service up.

You'll see "GOING TO SLEEP" while the service is transitioning to sleep, and "WAKING UP" while it's waking.

### Live updates

Komuta (the new React interface) refreshes the service sleep status in the background at short intervals; during transition moments (going to sleep / waking up) it updates every second. The moment you press the "Wake Up" button, the banner updates its status immediately — you get fast feedback without waiting for the API response. In the old Angular interface, the same information also arrives live via SignalR.

---

## What if I need to wake my service manually?

This is possible in three ways:

1. **The "Wake Up" button on the Dashboard** — the fastest way. Click it once, and the status card automatically updates once the pod is ready.
2. **The first HTTP request** — send a request to your service via a browser or `curl`; the Activator wakes the service up in the background, and your request receives a response after a delay of a few seconds.
3. **CI/CD pipeline deploy** — when you trigger a deploy, the service is automatically woken up (a running pipeline already prevents a service from going to sleep).

---

## Can I put my service to sleep manually?

Yes. You can put your service to sleep on demand by selecting the "Hibernate" (Put to Sleep) option from the Dashboard status card. This is particularly useful for scenarios such as:

- Shutting down short-lived test environments,
- Shutting down demo services that won't be used at night/on weekends,
- Keeping a service active only during a planned usage window (e.g., "Monday–Friday 09:00–18:00 only")

---

## I have a service I don't want to enter sleep mode

You can add two types of exemptions for your service by asking your admin via the admin panel:

- **Never Sleep (NeverSleep):** The service is never put to sleep, no matter how long it sits idle. For example, a scheduler service that runs frequent cron jobs, a webhook receiver, or a critical health check service.
- **Force Sleep (ForceSleep):** More aggressive sleep — the service is put to sleep whenever the policy runs, regardless of the traffic threshold. This is generally used for demo environments.

You cannot add these exemptions on your own account; the DevOpsZon admin team (or a user with the admin role in your organization) adds them for you via the admin panel. You can request this through support or from a teammate who has access to the admin panel.

---

## Is the sleep feature enabled on my plan?

| Plan type | Default behavior |
|---|---|
| Free plan | **On** — your services automatically sleep after sitting idle for 3 hours |
| Paid plans (Starter / Pro / Business) | **Off** — runs continuously; an admin can enable it for you |
| Enterprise plan | **Off** — enabled only upon special request |

The plan administrator can override this behavior for any plan. You can see your plan's current status in the "Sleep Mode" card at the top of the service dashboard — if the card doesn't appear, the sleep feature is off on your plan.

---

## Frequently asked questions

**Question: Does my sleeping service lose data?**
No. Sleep only scales the pod count to 0. All persistent data, such as the Persistent Volume (disk), environment variables, config, and secrets, remain exactly as they are. When the service wakes up, it continues from where it left off with the same data.

**Question: What happens to my WebSocket connections when going to sleep?**
On sleeping services, existing TCP connections (including WebSocket) are closed via graceful shutdown as the pod shuts down. If you have reconnection logic on the client side, the Activator catches the new connection and wakes the service up. For services that need a persistently open connection, request a **NeverSleep** exemption from your admin.

**Question: What if an error occurs during wake-up?**
If the service can't become ready within 30 seconds on the Activator side, the user receives an HTTP 503 "Service is starting up, please retry" response. A Retry-After header is sent. When a request is sent again after a short wait, the service will most likely be ready. Repeated errors appear on the dashboard as a red sleep status.

**Question: Should I worry about the sleep feature slowing down the platform?**
No. The sleep policy measures traffic from the metrics Hubble already collects — there's no extra network traffic. And while the service is sleeping, no CPU/RAM is consumed at all, meaning it reduces resource usage on the platform.

**Question: Could a malicious client keep sending requests to wake my services and consume resources?**
The Activator only triggers a single wake: 100 requests arriving at the same service at the same time result in 1 wake RPC (coalescing). In a scenario where a service is constantly bombarded with requests, the service never goes to sleep in the first place — it can't cross the threshold. So malicious traffic will behave just like normal traffic. If you'd like, you can also protect it at a higher level with a bandwidth limit (Bandwidth Management).

**Question: Does a service I use only once a month wake up?**
Yes. Sleep status doesn't remove the service; it only shuts down the pod. When a request comes in after a month, you'll get a normal response after waiting through the initial cold-start time (3–15 seconds).

**Question: How do I see when sleep events happened in the past?**
The **Sleep History** subtab of the service dashboard lists every sleep/wake event along with its date, trigger (automatic sweep, manual, or ingress activator), and duration information.

---

## More information

- [Platform overview](platform-overview.md)
- [Service management](service-guide.md)
- [Plans and billing](billing-plans.md)
- [HPA / autoscaling](service-guide.md#hpa)
