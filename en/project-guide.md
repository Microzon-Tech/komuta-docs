# Project Management

Projects are the top-level organizational unit in DevOpsZon. Under each project there are one or more Git repositories and the services discovered from those repositories.

---

## Project Concept

In DevOpsZon, a project represents the following hierarchy:

```
Project
├── Repository 1
│   ├── Service A (defined via Dockerfile)
│   └── Service B (defined via Dockerfile)
├── Repository 2
│   └── Service C
└── Repository 3
    └── Service D
```

Each project is associated with a **cluster** and a **registry**. All services connected to the project run on the designated cluster, and built images are pushed to the selected registry.

---

## Project List

When you log in to the Console, you land on the **Project Management** page. On this page:

- **Project selector in the top bar:** Switch between your existing projects
- **Favorite projects:** Star the projects you use often for quick access
- **Service table:** A list of all services belonging to the selected project

### Information Shown in the Service Table

| Column | Description |
|-------|----------|
| **Service Name** | The name of the discovered service (derived from the Dockerfile) |
| **Branch** | The active Git branch |
| **Namespace** | The Kubernetes namespace |
| **Cluster** | The cluster the service runs on |
| **Status** | The current status of the service (Running, Degraded, Progressing, etc.) |
| **Pipeline** | The status of the latest pipeline run |
| **Actions** | Actions such as deploy, details, integration, delete |

---

## Creating a New Project

To create a new project, click the **"+"** button in the top menu, or select **New Project** from the overflow menu. A 7-step wizard will guide you:

### Step 1: Basic Information

Give your project a name and an optional description. The project name must be unique across the platform.

### Step 2: Service Connection

Select the Git provider and container registry you previously defined on the **Integrations** page. These connections will be used to access your repos and to store built images.

> **Tip:** If you don't have a Git or registry connection yet, you will be redirected to the integrations page from this step.

### Step 3: Repo Selection

The repositories in your connected Git provider are listed. Check the repos you want to add to the project. You can connect more than one repo to a project.

### Step 4: Service Discovery

DevOpsZon scans your selected repos and automatically detects directories containing a **Dockerfile**. Each Dockerfile is suggested as a deployable service.

- Review the list of discovered services
- Uncheck any services you don't want
- Select a branch for each service

> **Note:** Service discovery is Dockerfile-based only. Applications without a Dockerfile cannot be discovered automatically.

### Step 5: Resources and Cluster

- Select the **Kubernetes cluster** on which the services will run
- Specify the **resource package** (CPU, memory, disk)
- In SaaS mode, resource packages directly affect pricing

### Step 6: Environment Variables

Define the **environment variables** required for the services to run. You can add variables individually for each service or in bulk.

| Field | Description |
|------|----------|
| **Key** | Variable name (e.g., `DATABASE_URL`) |
| **Value** | Variable value |
| **Secret** | When checked, the value is stored as a Kubernetes Secret |

### Step 7: Summary and Confirmation

Review all your settings. Once you confirm, DevOpsZon automatically does the following:

1. Creates the project
2. Connects the repos
3. Defines the services
4. Prepares the required Kubernetes resources
5. Triggers the first pipeline (optional)

---

## Adding a Repo to a Project

To add a new repository to an existing project:

1. Select **Add Repo** from the overflow menu on the project page
2. Select the new repo from the repo list in your Git provider
3. Confirm the service discovery
4. The new services will be added to the project list

---

## New Service Discovery

When you add a new Dockerfile to your repos, to discover this service:

1. Select **Discover New Service** from the overflow menu
2. DevOpsZon rescans your repos
3. Newly found services are shown as a preview
4. Services you confirm are added to the project

---

## Project Operations

### Deploying a Service

You can start a new deployment by clicking the **Deploy** button next to any service in the project list. In the deploy dialog:

- Select one of the existing image tags, or
- Trigger a new build

### Deleting a Project

Deleting a project permanently deletes **all services, pipeline histories, and Kubernetes resources** connected to that project. This action cannot be undone.

> **Warning:** Deleting a project also cleans up all associated Kubernetes resources (deployment, service, ingress, etc.).

### Connecting to a Cluster

Using the **Connect to Cluster** option in the overflow menu, you can move your project to a different cluster or set up an additional cluster connection.

---

## Tips

- **Favorite projects:** Star the projects you use often for quick access from the selector in the top bar
- **Bulk deploy:** You can deploy multiple services at the same time
- **Per-project monitoring:** Track the health status of each project from the status indicators in the project list
