# Creating a New Service

A service is Komuta's deployment target: code from a git repository — repo, for short — is built and shipped as a running application. Every service carries its own name, port, environment variables, resource package, and deployment history. The service creation wizard resolves three questions in order — which project it belongs to, where the code comes from, and how it runs.

You can also publish new services through the Komuta MCP; see the [MCP setup](https://www.komuta.io/docs/mcp/mcp-setup) page for how to set it up.

---

## What Is a Service?

A service corresponds to a single application. If a repo has more than one app in it (say, an API and a frontend), each of those is a separate service; they can all be created together from the same repo in one pass.

Services are grouped under a **project**. A project is the structure that keeps services working together in one place; a new service is always attached to a project. Once created, a service's name becomes the identifier for every record on the platform tied to that service — which is why it needs to be meaningful and unique.

A service's source is a repo and a specific branch of that repo. Komuta scans that branch, finds buildable applications, and presents what it finds as configurable services.

---

## 1. Project and Location

The first step determines where the service belongs and where it will run.

### Choosing a Project

The service is added under the selected project. If there's no suitable project, **Create new project** is used; the project is created right before the service.

### Choosing a Region

The region the service will run in is selected here.

> **Note:** Once a service is created, its region can't be changed.

![Project step — project selector and region list side by side](https://cdn.komuta.io/docs/tr/images/create-service/create-service-first-step.png)

---

## 2. Repo and Branch

The second step determines where the code comes from.

### Choosing a Repo

The repos in the list come from your connected Git accounts. If the repo you're looking for isn't listed, the account that has it may not be connected; connections are added from the [Integrations](https://console.komuta.io/integrations) page.

### Choosing a Branch

Once a repo is selected, the branch list opens up. The branch you pick is the source for both the scan and the first deployment.

![Repo step — selected repo card expanded, branch list below it](https://cdn.komuta.io/docs/tr/images/create-service/create-service-second-step.png)

---

## 3. Configuring Services

Komuta scans the selected branch and lists the buildable applications it finds. Each one can be created as a separate service with its own settings.

### Services Found

Komuta presents every app it finds buildable in the repo during the scan as a separate service.

If the service you're expecting isn't in the list, you can check again with **Rescan**.

### Port

The port is where the application listens for incoming requests; Komuta routes traffic to that port. Komuta pulls the port value from the project; if it can't find one, it falls back to a generic default.

### Configuration File

Which Dockerfile the service is built with is set in the **Configuration file** field; Komuta finds this path during the scan. If the Dockerfile is somewhere else in the repo, the path is corrected here.

### If No Dockerfile Is Found

Komuta only builds from a Dockerfile. If the selected branch has no Dockerfile at all, generating one with AI is suggested; if that isn't possible, a Dockerfile needs to be added before the repo can be deployed.

You can also generate a Dockerfile through the Komuta MCP — see the [MCP setup](https://www.komuta.io/docs/mcp/mcp-setup) page for how to set it up.

> **Warning:** When a Dockerfile is generated with AI, a pull request is opened in the repo and it needs to be merged. Until it's merged, subsequent deployments can't find the Dockerfile and will fail.

### Resource Package

The resource package determines how much CPU and memory the service uses, and its monthly cost. It's set separately for each selected service.

The scan also suggests a package based on the application's structure.

> **Tip:** The resource package can be upgraded after the service is created.

### Environment Variables

The environment variables the application needs to run (`DATABASE_URL`, `API_KEY`, etc.) can be entered while creating the service. Values that need to stay secret are marked as **Secret**.

### Auto Deploy

With [auto deploy](https://www.komuta.io/docs/services/service-auto-deploy) turned on, every push to the selected branch starts a new deployment, taking the new version of the service live.

This setting lives under **Advanced options**, on a per-service basis — in a repo with multiple apps, one service can update on every push while another stays manually deployed.

**Path filters** make a deployment trigger only on changes in specific folders — for example, setting `api/**` means only pushes touching the `api` folder update the service, while other changes are ignored. Left empty, the service watches its own folder.

---

## Automatically Set Up Alerts

Three alerts are set up for every service with no configuration needed: **pod restart**, **pod not ready**, and **termination due to out-of-memory**.

> **Note:** You can create and edit your own alerts after the service goes live, from the [Alerts](https://console.komuta.io/alerts/overview) page.

---

## Deployment

The **Deploy** button shows the number of selected services and creates all of them in a single operation. Once deployment finishes, you're taken back to the [service list](https://console.komuta.io/services), where the build and rollout process can be tracked.

![Pre-deployment summary panel](https://cdn.komuta.io/docs/tr/images/create-service/create-service-third-step.png)
