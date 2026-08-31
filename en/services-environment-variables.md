# Environment Variables

Environment variables are configuration values your application reads at runtime — things like a database address, an API key, or a log level. Since they aren't hardcoded, they can differ from one environment to another.

After a variable is added, edited, or deleted, it's pushed to the runtime with **Apply Changes**.

![alt text](img/services-enviroment-variables-page.png)
---

## Variable Types

Every variable has a type, and the type determines where the value is visible.

| Type | What it means |
|---|---|
| **Plain** | The value is stored as plain text and visible in the UI. |
| **Secret** | The value is stored encrypted, hidden in the UI, and never leaves the server. |
| **Build** | The value is passed not only to the running container but also to the build stage. |

The **Build** type is needed for variables that are read during the build. When it's off, the value only exists at runtime; if it's read during the build, it will appear empty. This option is checked automatically when you enter a key with a known prefix (such as `VITE_`, `NEXT_PUBLIC_`, `REACT_APP_`).

> **Warning:** A Build-type value gets baked into the image, and if it has a public prefix, into the browser bundle too. Don't define information that needs to stay secret with this type.

![alt text](img/services-enviroment-variables-types.png)

---

## Bulk Import

To add many variables at once, paste the contents of your existing configuration file. Three formats are supported: `.env`, plain JSON, and nested appsettings JSON. The format is detected automatically from the pasted content.

---

## Export

Variables can be downloaded or copied to the clipboard in `.env`, JSON, YAML, CSV, shell, and Kubernetes manifest formats.

> **Note:** Secret values come out empty on export, since they're stored encrypted and never leave the server.

---

## Related Documents

- [Service Dashboard](service-dashboard.md) — Overall service status and deploy actions.
- [Pipelines](pipeline-guide.md) — How the build stage works.
