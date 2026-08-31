# Connections

Connections let one application reach another one of your applications over a private network. Traffic never goes out to the internet — the two applications reach each other directly, so there's no need to define a public URL for this.

![alt text](img/services-connections.png)

---

## Connecting an Application

Use **Connect app** to choose the application to reach. Once the connection is established, Komuta generates a private address for that application and shows it in the list; setting up the connection can take a few minutes.

For the generated address to be usable, your application needs to read it. The **Add as environment variable** action in the list writes the address into the service's environment variables.

![alt text](img/services-connections-list.png)

---

## Apps You Connect To and Apps That Connect to You

The list shows both directions separately: the applications this service **connects to**, and the applications that **connect to** this service. A connection is one-directional — connecting to an application from here doesn't mean that application can reach you back.

> **Note:** Which services connect to a given service can't be listed yet. To see that direction, check that service's own connections page.

---

## Related Documents

- [Environment Variables](environment-variables.md) — Defining a connection address as a variable.
- [Ports](ports.md) — The ports a service listens on and the private network setting.
