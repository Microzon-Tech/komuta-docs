# Integrations

DevOpsZon provides integration with various Git providers and container registries to access your source code and container images. These integrations form the foundation of the project creation and deployment processes.

---

## Git Integrations

Git integrations provide access to the repositories where your source code lives. A secure connection is established via OAuth-based authorization.

### Supported Git Providers

| Provider | Connection Method | Features |
|-----------|------------------|-----------|
| **GitHub** | OAuth App | Repo list, branch viewing, Dockerfile discovery |
| **GitLab** | OAuth App | Repo list, branch viewing, Dockerfile discovery |
| **Bitbucket** | OAuth App | Repo list, branch viewing, Dockerfile discovery |
| **Azure DevOps** | OAuth App | Repo list, branch viewing, Dockerfile discovery |

### Setting Up a Git Connection

1. Go to the **Integrations** page from the left menu
2. In the **Git Connections** section, click the **Connect** button for the provider you want to connect
3. You will be redirected to the provider's OAuth authorization page
4. Approve the required permissions
5. You will be redirected back to DevOpsZon, and the connection status will appear as **Connected**

### After Connecting

Once the connection is established:

- **Repo list:** You can see all accessible repositories in your provider
- **Branch viewing:** You can list the branches of each repo
- **File discovery:** You can inspect the repo contents (Dockerfile, project structure)
- **Service discovery:** You can discover deployable services by automatically detecting Dockerfiles

### Connection States

| State | Meaning |
|-------|--------|
| **Connected** | Connection is active, access is provided |
| **Expired** | Token has expired, re-authorization required |
| **Error** | Connection error, needs review |

> **Tip:** You can easily renew connections whose token has expired using the **Reconnect** button.

---

## Container Registry Integrations

Container registries are the resources where your built Docker images are stored and from which they are pulled during deployment.

### Supported Registries

| Registry | Type | Description |
|----------|-----|----------|
| **Docker Hub** | Public | Docker's official registry |
| **GitHub Container Registry (GHCR)** | Public | Registry integrated with GitHub |
| **GitLab Container Registry** | Public | Registry integrated with GitLab |
| **Azure Container Registry (ACR)** | Cloud | Azure's managed registry |
| **Amazon ECR** | Cloud | AWS's managed registry |
| **Google Container Registry (GCR)** | Cloud | Google Cloud's registry |
| **Custom** | Custom | Registry on your own server |

### Setting Up a Registry Connection

1. On the **Integrations** page, go to the **Registry Connections** section
2. Click the **Yeni Registry Ekle** button
3. Select the registry type
4. Enter the access information:

| Field | Description |
|------|----------|
| **Registry URL** | Registry address (e.g., `ghcr.io`, `registry.example.com`) |
| **Username** | Registry username or service account |
| **Password / Token** | Access password or personal access token |

5. Verify the connection with the **Test** button
6. Save the connection with the **Kaydet** button

### Registry Use Cases

- **During build (Push):** The pipeline pushes the Docker image it created to the selected registry
- **During deploy (Pull):** Kubernetes pulls the deployed image from the registry

Each project's service settings specify which registry will be used.

---

## Role in the Project Creation Flow

Integrations form the foundation of the project creation wizard:

```
Git Connection → Repo Selection → Service Discovery → Registry Selection → Pipeline → Deploy
```

1. **Service connection step:** One of the previously established Git connections is selected
2. **Repo selection step:** Repos are listed via the selected Git connection
3. **Service discovery step:** Dockerfiles in the selected repos are discovered
4. **Pipeline configuration:** The pipeline is configured to push the image to the selected registry

---

## Connection Management

### Updating a Connection

To update the access information of an existing connection:
1. Click the **Edit** button next to the relevant entry in the connection list
2. Enter the new information
3. Click the **Kaydet** button

### Deleting a Connection

You can delete unused connections. However, be careful:

> **Warning:** If you delete a connection that is used by active projects, the pipelines of those projects will fail. Check which projects use the connection before deleting it.

---

## Tips

- **Separate accounts:** Use different registry connections for production and development environments
- **Token renewal:** Track the expiration of your Personal Access Tokens and update them before they expire
- **Minimal privilege:** Grant access only to the necessary repos in your Git connections
- **Custom registry:** Prefer using a custom registry for projects with privacy requirements
