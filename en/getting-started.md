# Quick Start Guide

Welcome to DevOpsZon! This guide walks you through the step-by-step onboarding process for those using the platform for the first time.

---

## What is DevOpsZon?

DevOpsZon is a DevOps platform that lets you easily publish, manage, and monitor your applications on Kubernetes. Connect your Git repo, discover services, and go live with a single click — without dealing with infrastructure complexity.

---

## First Steps

### 1. Create an Account

To access DevOpsZon Console, create your account at [console.devopszon.com](https://console.devopszon.com). After signing up, complete your email verification to log in to the dashboard.

### 2. Connect Your Git Provider

To be able to publish your projects, you first need to connect the Git provider where your source code is located.

- Go to the **Integrations** page from the left menu
- Select your provider from the **Git Connection** section: **GitHub**, **GitLab**, **Bitbucket**, or **Azure DevOps**
- Complete the OAuth authorization
- Your connection status will appear as green

### 3. Connect a Container Registry

Define a container registry where your built Docker images will be stored.

- On the same **Integrations** page, go to the **Registry Connection** section
- Supported registries: **Docker Hub**, **GitHub Container Registry (GHCR)**, **GitLab Container Registry**, **Azure Container Registry (ACR)**, **Amazon ECR**, **Google GCR**, or a **custom registry**
- Enter your access credentials to save the connection

### 4. Determine Your Cluster

Determine the Kubernetes cluster on which your applications will run. You have two options:

- **SaaS Mode:** Use DevOpsZon's managed infrastructure — no additional cluster setup required
- **Hybrid Mode:** Connect your own Kubernetes cluster with a **kubeconfig** file, or create a new cluster from the DevOpsZon interface

### 5. Create Your First Project

- Click the **"+"** button in the top header or the **New Project** option
- The project creation wizard will open:

| Step | Description |
|------|----------|
| **Basic Information** | Project name and description |
| **Service Connection** | Select your Git provider and registry |
| **Repo Selection** | Mark the repositories you want to publish |
| **Service Discovery** | Dockerfiles in the repos are automatically detected and services are suggested |
| **Resource & Cluster** | Determine the cluster and resource package the services will run on |
| **Environment Variables** | Define the required environment variables |
| **Summary & Confirmation** | Review all settings and create the project |

### 6. Make Your First Deploy

Once the project is created, you will see your services in the project list. To deploy a service:

- Click the **Deploy** button on the service row
- Select an image tag or trigger a new build
- The pipeline will start automatically and you can track its progress
- Once the build and deploy are complete, you can access your service via the assigned hostname

---

## Platform Structure

DevOpsZon Console is organized into main sections in the left menu:

| Menu | Purpose |
|------|------|
| **Projects** | Manage your projects and services |
| **Services** | Detailed management panel for the selected service |
| **Cluster** | Manage your Kubernetes clusters |
| **Addons** | Managed services (PostgreSQL, RabbitMQ, Valkey) |
| **Integrations** | Git and container registry connections |
| **Alerts** | Alert rules and notifications |
| **Notifications** | Notification channels and settings |
| **Domains** | Custom domain management |
| **Access Control** | User permissions and access control |

---

## Next Steps

- Check out the **Project Management** guide to manage your projects in detail
- See the **Deployment Strategies** section to learn about deployment strategies (Canary, Blue/Green)
- Go to the **Managed Services** guide for a managed database or message queue
- Review the **Alert Management** section to set up alerts and notifications
- Go to the [Observability Guide](observability-guide.md) section to examine service performance along with evidence coverage
