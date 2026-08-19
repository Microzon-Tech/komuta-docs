# Cluster Management

Cluster management lets you add, create, and configure your Kubernetes clusters from a single panel.

---

## What Is a Cluster?

A Kubernetes cluster is the server infrastructure on which your applications run. In DevOpsZon, every project and service is associated with a cluster. Different cluster options are available depending on your working model.

---

## Cluster Addition Methods

### 1. Connecting an Existing Cluster (Hybrid Mode)

If you have your own Kubernetes cluster, you can connect it to DevOpsZon using a kubeconfig file.

**Steps:**

1. Go to the **Cluster** page from the left menu
2. In the **Cluster Add** tab, click the **Yeni Cluster Ekle** button
3. Give your cluster a name
4. Paste the **kubeconfig** content
5. DevOpsZon will test the connection to the cluster
6. If the connection is successful, it will be added to your cluster list

> **Important:** Make sure your kubeconfig has sufficient permissions for DevOpsZon to operate. Cluster-admin or equivalent permissions are required.

**Automatic configuration after connection:**

DevOpsZon automatically installs the following components on the connected cluster:
- Komuta Pipeline (CI/CD pipeline)
- Komuta Rollout (deployment strategies)
- Komuta Metrics and Komuta Alerts (monitoring and alerts)
- Komuta Logs (log collection)
- Required namespaces and access permissions

### 2. Creating a New Cluster

You can create a new Kubernetes cluster directly from the DevOpsZon interface.

**Options:**

| Method | Description |
|--------|----------|
| **Managed Cluster** | Automatic setup on a cloud provider |
| **Bare Metal** | Kubernetes installation on physical servers |
| **BYOC (Bring Your Own Cloud)** | Provisioning resources in your own cloud account |

**Managed Cluster creation wizard:**

1. **Cloud provider selection:** Hetzner Cloud is currently supported
2. **Region selection:** The data center region where the cluster will be located
3. **Node configuration:** Server type, count, and resources
4. **Kubernetes version:** The K8s version to be installed
5. **Additional settings:** Network, security, and storage configuration

### 3. SaaS Mode

In SaaS mode, cluster management is handled by DevOpsZon. As a user, you don't need to add or manage clusters — your projects automatically run on DevOpsZon's infrastructure.

---

## Cluster List

On the Cluster page, you can see a list of all your connected clusters:

| Info | Description |
|-------|----------|
| **Cluster Name** | The name you gave the cluster |
| **Status** | Connection status (Connected, Disconnected, Error) |
| **Kubernetes Version** | The K8s version on the cluster |
| **Node Count** | Number of worker nodes |
| **Topology** | Cluster architecture (single-master, multi-master, etc.) |
| **Region** | The data center region where the cluster is located |

---

## Cluster Details

When you click a row in the cluster list, the details page opens:

### Summary Cards

- **Cluster type:** Managed, Imported, Bare Metal
- **Kubernetes version**
- **Topology:** Distribution of master and worker nodes
- **Region:** Data center location

### Node Management

In the cluster details, you can view and manage the nodes (servers):

| Action | Description |
|-------|----------|
| **Node ekleme** | Add a new worker node to the cluster (for Managed clusters) |
| **Node removal** | Safely remove an existing worker node |
| **Node bilgileri** | CPU, memory, disk usage, and pod capacity |

### Kubeconfig

You can view the cluster's kubeconfig information and copy it to the clipboard. This is useful when you need direct access via `kubectl`.

---

## Addons

You can see the addons that are installed and available for installation for each cluster.

### Installed Addons

The addons required for DevOpsZon to operate are automatically installed when the cluster is connected:

| Addon | Role |
|-------|-----|
| **Komuta Pipeline** | Running CI/CD pipelines |
| **Komuta Rollout** | Advanced deployment strategies |
| **Komuta Metrics** | Metric collection and monitoring |
| **Komuta Alerts** | Alert management and routing |
| **Komuta Logs** | Log collection and querying |
| **Komuta TLS** | Automatic TLS certificate management |
| **Komuta Gateway** | HTTP/HTTPS traffic routing |
| **Komuta Runtime Security** | Container and node runtime protection with KubeArmor — [details](runtime-security-guide.md) |

### Addon Health Status

You can monitor the operating status of addons:

- **Healthy:** The addon is working properly
- **Degraded:** The addon is running but has some issues
- **Failed:** The addon is not working, intervention required

> **Tip:** If the addon health status appears problematic, you can retry the installation steps using the step rerun feature.

---

## Cluster Scaling

For Managed clusters:

- **Scale Up:** Increase cluster capacity by adding new worker nodes
- **Scale Down:** Reduce costs by removing unused worker nodes

> **Warning:** During node removal, the pods on it are moved to other nodes. You may experience a brief performance drop during this process.

---

## Tips

- **Multiple clusters:** Use separate clusters for different environments (development, staging, production)
- **Capacity planning:** Ensure sufficient node capacity by considering the number of services and resource needs
- **Region selection:** Minimize latency by choosing the data center region closest to your users
- **Backup kubeconfig:** Back up your cluster's kubeconfig in a secure location
