# Access Control

DevOpsZon's access control system lets you manage which resources users in your organization can access and which actions they can perform. Create a secure multi-user environment by assigning read and edit permissions on a per-entity (record) basis.

---

## Access Control Model

DevOpsZon uses **ACL (Access Control List)**-based authorization. Each user can be assigned separate permissions for each resource (entity).

### Permission Types

| Permission | Description |
|-------|----------|
| **Read** | View the resource; inspect its details; access monitoring data |
| **Edit** | Update the resource, change configuration, trigger deploys, delete |

> **Note:** Edit permission also includes Read permission. When you grant a user Edit, read permission is automatically granted as well.

### Resource Types

Access control is applied over four fundamental resource types:

| Resource Type | Scope |
|-------------|---------|
| **Project** | Project viewing, listing services, triggering deploys |
| **Service** | Service details, pipeline, environment variables, ingress, logs |
| **Cluster** | Cluster information, node management, addon status |
| **Integration** | Git and registry connections, repo listing |

---

## Access Control Page

Go to the **Access Control** page from the left menu. The page consists of two sections:

### Left Panel — User List

- All users in your organization are listed
- You can find a user using the search field
- Select the user you want to authorize by clicking on them

### Right Panel — Authorization

Permission management for the selected user, in four tabs:

| Tab | Display |
|-------|----------|
| **Project** | All projects and the user's Read/Edit status on each project |
| **Service** | All services and the user's Read/Edit status on each service |
| **Cluster** | All clusters and the user's Read/Edit status on each cluster |
| **Integration** | All integrations and the user's Read/Edit status on each integration |

---

## Assigning Permissions

### Individual Assignment

1. Select the user from the left panel
2. Go to the relevant tab in the right panel (e.g., **Project**)
3. Check/uncheck the **Read** and **Edit** checkboxes on each record row
4. Click the **Kaydet** button

### Bulk Assignment

Bulk assignment options are available at the top of each tab:

- **Grant Read to All:** Grants read permission to all resources of the selected type
- **Grant Edit to All:** Grants edit permission to all resources of the selected type
- **Remove All:** Removes all permissions of the selected type

---

## Usage Scenarios

### Scenario 1: Developer Access

A developer should only be able to view and deploy the projects and services they work on:

| Resource | Permission |
|--------|-------|
| The project they work on | **Edit** |
| Services in the project | **Edit** |
| Other projects | No permission |
| Cluster | **Read** |
| Integrations | **Read** |

### Scenario 2: Team Lead

A team lead should be able to view all projects but only edit the projects belonging to their own team:

| Resource | Permission |
|--------|-------|
| Own team's projects | **Edit** |
| Other projects | **Read** |
| All clusters | **Read** |
| Integrations | **Edit** |

### Scenario 3: Monitoring / Operations

The operations team should be able to monitor all resources but not make changes:

| Resource | Permission |
|--------|-------|
| All projects | **Read** |
| All services | **Read** |
| All clusters | **Read** |
| Integrations | **Read** |

---

## Effect of Permissions

Access control settings apply at every point of the platform:

| Area | For an Unauthorized User |
|------|-------------------------|
| **Project list** | Only authorized projects are visible |
| **Service list** | Only authorized services are visible |
| **Service management** | Deploys and setting changes cannot be made without Edit permission |
| **Cluster list** | Only authorized clusters are visible |
| **Integrations** | Unauthorized connections are hidden |

---

## Tenant Isolation

In DevOpsZon, each organization (tenant) operates in a fully isolated environment:

- One organization's data cannot be accessed by other organizations
- Each organization has its own user list and permission structure
- Kubernetes namespaces are separated on a per-tenant basis

Access control governs permission management among users within the same organization.

---

## Tips

- **Principle of least privilege:** Grant users only the minimum permissions they need
- **Regular review:** Periodically review permission assignments; remove permissions for employees who have left
- **Project-based assignment:** Don't forget to assign permissions to relevant users whenever you create a new project
- **Use Edit carefully:** Edit permission allows all changes, including deletion
