# CI/CD Pipeline Management

DevOpsZon provides Kubernetes-native CI/CD pipeline infrastructure on **Komuta Pipeline**. Your source code is automatically built, converted into a Docker image, and deployed to your cluster.

---

## How the Pipeline Works

When you trigger a deploy, the following steps happen in the background, in order:

```
Git Repo ──→ Source Fetch ──→ Docker Build ──→ Image Push ──→ Deploy
   (1)            (2)              (3)            (4)         (5)
```

| Step | Description |
|------|----------|
| **1. Trigger** | Clicking the Deploy button or an automatic trigger |
| **2. Source Fetch** | The code for the specified branch is pulled from the Git repository |
| **3. Docker Build** | A container image is created using the Dockerfile |
| **4. Image Push** | The created image is pushed to the connected container registry |
| **5. Deploy** | The new image is deployed to the cluster with Komuta Rollout |

---

## Triggering the Pipeline

### Manual Trigger

1. Click the **Deploy** button on the service row from the **Projects** page
2. Or trigger it from the **Service Management** → **Pipeline Summary** tab

### Bulk Trigger

To deploy multiple services at the same time:
1. Select the services on the **Projects** page
2. Use the **Bulk Deploy** action

### Queue Management

When multiple pipelines are triggered at the same time, DevOpsZon uses an intelligent queue system:
- The number of pipelines that can run concurrently is determined by the cluster's resources
- Pipelines waiting in the queue are processed in order
- Each pipeline's queue status is shown in the project list

---

## Monitoring the Pipeline

### In the Project List

The pipeline status is shown with a badge on each service row in the project list:

| Badge | Meaning |
|-------|--------|
| **Queued** | The pipeline has been queued and is waiting its turn |
| **Pending** | The pipeline's resources are being prepared |
| **Build** | The Docker image is being created |
| **Successful** | The pipeline completed successfully |
| **Error** | The pipeline ended with an error |

### Pipeline Summary Tab

More detailed information is available in the **Service Management** → **Pipeline Summary** tab:

- **Live pipeline:** Every step of an ongoing build is shown in real time
- **Task list:** The name, duration, and status of each pipeline step
- **History:** All previous pipeline runs

### Pipeline Detail

Clicking a pipeline run opens the detail view:

| Information | Description |
|-------|-------------|
| **Task table** | Each step's name, start/end time, and duration |
| **Status** | The success/error status of each task |
| **Logs** | Detailed logs for each task step |

---

## Pipeline Tasks

A typical DevOpsZon pipeline consists of the following tasks:

| Task | Description |
|-------|-------------|
| **fetch-credentials** | Prepares Git and registry access credentials |
| **git-clone** | Pulls the source code from the repository |
| **build-and-push** | Builds the image using the Dockerfile and pushes it to the registry |
| **update-manifest** | Updates the deployment manifest file with the new image tag |
| **deploy** | Starts the deploy via Komuta Rollout |

---

## Pipeline Logs

To access the logs for each pipeline task:

1. Go to the **Pipeline Summary** tab
2. Click the relevant pipeline run
3. In the task list, click the step whose log you want to see
4. The detailed log output is displayed

### Searching Within Logs

When debugging pipeline logs:
- Search for build errors using the keywords `error` or `failed`
- Detect issues in the Dockerfile from the output of `COPY` and `RUN` commands
- Check registry push errors from the `push` logs

---

## Error Cases and Solutions

| Error | Possible Cause | Solution |
|------|------------|-------|
| **Build error** | Syntax error in the Dockerfile or a missing dependency | Check the Dockerfile; review the error details in the logs |
| **Push error** | Registry access credentials are invalid or expired | Update the registry connection from the Integrations page |
| **Git clone error** | No repository access permission or the branch was not found | Check your Git connection and the branch name |
| **Queue timeout** | Insufficient cluster resources | Run fewer concurrent pipelines or increase resources |

---

## Automatic Dockerfile Fix

When a build error occurs, DevOpsZon can automatically analyze your Dockerfile and offer a fix suggestion.

### How It Works

1. An error occurs during the pipeline's build stage
2. The error logs are automatically analyzed
3. A fix suggestion is prepared
4. A **fix branch** and a **Pull Request** are created in your Git repository
5. After reviewing and merging the PR, you can redeploy

> **Note:** The automatic fix feature is optional and is enabled from the service settings.

---

## Tips

- **To shorten pipeline duration:** Use Docker multi-stage builds and reduce unnecessary layers
- **Cache:** Take advantage of the Docker cache by keeping dependency installation layers separate
- **Health check:** Wait for pods to pass their readiness probe after deploy
- **Parallel build:** You can run multiple pipelines in parallel for independent services
