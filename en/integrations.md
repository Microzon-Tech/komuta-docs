# Integrations: Git Accounts and Container Registries

Komuta builds your services from the code in your Git repository and pushes the resulting image to a container registry. That takes two connections: a **Git account** the code is read from and a **container registry** the image is pushed to. Both are managed on the [Integrations](https://console.komuta.io/integrations) page.

You need to connect a GitHub, GitLab or Azure DevOps account before your first service. You don't have to do anything for the registry: Komuta's own registry is ready to use. Add your own registry only if you want your images kept there.

---

## The Integrations Page

| Section | What it shows |
|---|---|
| **Git Providers** | The GitHub, GitLab and Azure DevOps cards. Each card has a **Connect** button and lists the accounts connected for that provider. |
| **Container Registries** | The registries you can use, and an **Add** button for a new one. |
| **GitHub App Installations** | The GitHub accounts and organizations the Komuta App is installed on. |
| **GitLab auto-deploy** / **Azure DevOps auto-deploy** | The projects and repositories that redeploy on push. Shown only when you have a connected account for that provider. |

Every connected account appears as a chip on its provider card. The **Org** or **User** badge on the chip tells you whether it is an organization or a personal account. The badges in the page header sum up the number of connections, Git accounts and registries.

For GitHub and Azure DevOps connections, clicking the chip opens the **Repositories** panel with the repositories the connection can reach. A **Private** / **Public** badge shows each repository's visibility; Azure DevOps does not report visibility, so its repositories have no badge.

> **Note:** Organization admins see every connection. Other members see the connections they added and the ones they were given access to. Komuta's registry is visible to everyone.

---

## Providers at a Glance

| | GitHub | GitLab | Azure DevOps |
|---|---|---|---|
| How you connect | Install the Komuta App on your GitHub account or organization | gitlab.com: sign in with GitLab. Self-managed: access token | Personal access token (PAT) |
| Access required | The repositories you select during installation | The `api` scope | **Code (Read)** and **Service Connections (Read, query, manage)** |
| Repositories listed | Repositories the installation can reach | Projects the token owner is a member of | Repositories in the organization the token can see |
| Auto-deploy on push | Ready for every repository the installation covers | One webhook per project, set up when a service is deployed | One subscription per repository, set up when a service is deployed |
| Token renewal | Not needed | **Update token** | **Update token** |

---

## GitHub

GitHub is connected through the **Komuta App**. You install the app on your personal GitHub account or on a GitHub organization, and Komuta can reach only the repositories you allow during installation. Pushes to those repositories reach Komuta with no further setup.

### Connecting a GitHub Account

1. Click **Connect** on the GitHub card, or **Connect GitHub Account** in the **GitHub App Installations** section. You are sent to GitHub.
2. Choose the account or organization to install the app on.
3. Choose the access: all repositories, or only the ones you select.
4. Confirm the installation. GitHub may ask you to authorize Komuta; this step confirms that the person installing owns the account.
5. You return to the Integrations page. The installation appears under **GitHub App Installations**, and the connection appears on the GitHub card as `github-app-<account>`.

On a personal account, the person authorizing must be that account. On an organization, the person authorizing must be an owner of the GitHub organization or the person who installed the app. If the check fails, "Could not complete the connection. Please try again." appears and no connection is created.

> **Note:** An installation on a GitHub account is tied to a single Komuta organization. If the same GitHub account is already connected in another Komuta organization, no new connection is created in the second one.

### The Installation Card

Each installation card shows the account name, an **Organization** or **Personal** badge, a status and the access scope (**All repositories** / **Selected repositories**).

| Status | Meaning | What to do |
|---|---|---|
| **Active** | The installation works. | — |
| **Suspended** | The installation is suspended on GitHub. | Re-enable the installation on GitHub. The connection stays the same. |
| **Disconnected on GitHub** | The app was removed on GitHub; Komuta cannot reach the repositories. | Read "Removing and Reinstalling" below. |

**Configure on GitHub** opens the installation's settings page on GitHub. Adding or removing repositories, reviewing permissions and uninstalling the app are done there.

### Adding Repositories Later

If you installed with **Selected repositories**, add a new repository like this:

1. Click **Configure on GitHub** on the installation card.
2. On GitHub, add the repository to the repository access list and save.
3. Open the New App wizard again. The repository list is read from GitHub, so the new repository shows up.

Nothing else is needed in Komuta; the installation and the connection stay the same.

### Removing and Reinstalling

Removing the installation and installing it again (removing it in Komuta, or deleting the app on GitHub) is not the same as adding a repository. A reinstall makes Komuta create a **new connection**.

> **Warning:** Services you created with the previous installation stay on the old connection and do not move to the new one by themselves. Their next builds cannot pull the code and their auto-deploy stops working. The console has no screen that moves a service to another Git connection. To add or remove repositories, don't remove the installation; use **Configure on GitHub**. If you are already in this situation, open a [support ticket](support-tickets.md).

If a card shows **Disconnected on GitHub**, **Reconnect** starts the installation flow again. This also creates a new connection, so the warning above applies here too.

---

## GitLab

**Connect** on the GitLab card opens the **Connect GitLab** dialog. First choose where your GitLab runs:

- **gitlab.com** — Sign in with GitLab. This path is for gitlab.com only.
- **Self-managed** — Connect with an access token. This is the only path for a GitLab on your own server, and it also works for gitlab.com.

### Signing In with gitlab.com

1. In the **Connect GitLab** dialog, keep **gitlab.com** selected and click **Continue to GitLab**.
2. Sign in to GitLab and authorize Komuta. Komuta asks for the `api` scope; it uses it to list your projects and to register the push webhook that drives auto-deploy.
3. Back on the Integrations page, "GitLab connected" appears.

GitLab always shows its sign-in screen; make sure you sign in with the right GitLab account. If you connect the same GitLab account again, no new connection is created; the existing connection's access is refreshed.

### Self-Managed GitLab

1. Create a token with the `api` scope in GitLab. A Personal, Group or Project Access Token all work.
2. In the **Connect GitLab** dialog, choose **Self-managed**.
3. Fill in the fields and click **Connect**.

| Field | Description |
|---|---|
| **Server URL** | Your GitLab server address, for example `https://gitlab.example.com`. If you paste a project URL, only the server part is used. Leave it empty to use gitlab.com. |
| **Access Token** | A token with the `api` scope. The **Create one** link under the field opens GitLab's token page. |
| **Connection Name** | Optional. If left empty, the name is `gitlab-<server>-<username>`. |

Komuta verifies the token against GitLab and reads your username from there before saving, so your server must be reachable from Komuta. If the token is rejected, "GitLab rejected that token. Check the server URL and make sure the token has the "api" scope." appears and nothing is saved.

If you used a Personal Access Token, Komuta reads its expiry from GitLab. When 14 days or fewer remain, the **GitLab auto-deploy** card shows a badge with the days left (for example **Expires in 5d**); once the date passes it shows **Token expired**. The expiry cannot be read for Group and Project Access Tokens.

### Updating the Token

On the **GitLab auto-deploy** card, click **Update token** next to the connection, enter the new token and save with **Update token**. The new token is verified first; if it is rejected, "GitLab rejected that token. The existing connection was left unchanged." appears. The connection stays the same, so its projects, webhooks and deployed services stay connected.

### Auto-Deploy on GitLab

GitLab delivers push events through one webhook per project. When you deploy a service from a GitLab repository, Komuta registers the webhook on that project itself; the webhook listens to push events only. If the webhook cannot be registered, the service is still created, but its auto-deploy starts off.

For each connection, the **GitLab auto-deploy** card shows:

- The projects with a webhook, and their count (for example **2 repo(s)**).
- A refresh icon (Re-check this webhook in GitLab): registers the webhook again if it was deleted on the GitLab side. On success, "Webhook verified" appears.
- A trash icon: if you click **Disable auto-deploy** in the confirmation, the webhook is removed from GitLab as well and pushes stop triggering a deployment.
- The **Add a project…** list and the **Add** button: register a webhook on a project that doesn't have one yet.

To register a webhook, the token needs the `api` scope and Maintainer access to the project. Otherwise "Could not enable auto-deploy. The token needs the api scope and Maintainer access to the project." appears. If you added the webhook later, check that auto-deploy is on in the service's **Auto-deploy** tab.

---

## Azure DevOps

Azure DevOps is connected with a personal access token (PAT).

### Connecting Azure DevOps

1. In Azure DevOps, create a PAT with **Code (Read)** and **Service Connections (Read, query, manage)**.
2. Click **Connect** on the Azure DevOps card.
3. Fill in the **Connect Azure DevOps** dialog and click **Connect**.

| Field | Description |
|---|---|
| **Organization** | The organization name (for example `contoso`) or a full Azure DevOps URL. If you paste `https://dev.azure.com/contoso/...` or `https://contoso.visualstudio.com`, the name is taken from it; a `https://dev.azure.com/<organization>` preview appears under the field. |
| **Personal Access Token** | A PAT with the access listed above. Once you enter the organization, a **Create one** link opens the Azure DevOps token page. |
| **Connection Name** | Optional. If left empty, the name is `azuredevops-<organization>`. |
| **Username** | Optional. Azure DevOps looks only at the token. |
| **Token expiry (optional)** | The expiry you gave the PAT in Azure DevOps. Azure doesn't report it, so Komuta can warn you only if you enter it. A past date cannot be picked. |

The button first shows **Connecting...**, then **Verifying...** while Komuta tries to read the organization's projects with the token. On success, a message like "Connected to contoso — 3 project(s) found" appears. If the token is rejected, the connection is rolled back and "Azure DevOps rejected that token. Check the organization name and make sure the PAT grants Code (Read) and Service Connections (Read, query, manage)." appears.

### Token Expiry and Updating

If you entered an expiry, a badge with the days left (for example **Expires in 5d**) appears next to the connection chip when 14 days or fewer remain, and **Token expired** once the date passes. With an expired token, builds fail when they pull the code.

To renew the token, click **Update token** next to the connection on the **Azure DevOps auto-deploy** card. Enter the new PAT and, if you like, the new expiry, then save with **Update token**. The new token is verified first; if it is rejected, "Azure DevOps rejected that token. The existing connection was left unchanged." appears.

> **Warning:** Don't renew the token by connecting Azure DevOps again with **Connect**. Every connect creates a new connection, and your existing services and auto-deploy subscriptions stay on the old one. **Update token** keeps the connection.

### Auto-Deploy on Azure DevOps

Azure DevOps delivers push events through one subscription (service hook) per repository. When you deploy a service from an Azure DevOps repository with the New App wizard, Komuta creates the subscription itself. If it can't, the service is still deployed and this warning appears: "Service deployed, but automatic redeploy on push could not be enabled. Check that the PAT grants Service Connections (Read, query, manage), then retry from Integrations."

Manage subscriptions on the **Azure DevOps auto-deploy** card:

- To turn auto-deploy on, pick the repository in the **Add a repository…** list and click **Enable**.
- To turn it off, click the trash icon on the repository row and confirm with **Remove**. The subscription is removed from Azure DevOps as well; existing services keep running.

Azure DevOps push events don't include the list of changed files. Path filters are therefore not available for Azure DevOps services, and every push to the tracked branch deploys; cooldown and skip directives still apply. See [Auto Deploy](service-auto-deploy.md) for details.

---

## Using Connections in the Wizard

The **Connect** step of the wizard you open with **New App** on the [**Apps**](https://console.komuta.io/services) page uses these connections: **Git account** sets where the code is read from, **Container registry** sets where the built image is pushed, and then you pick the repository and branch.

- If you have exactly one Git connection and one registry, both are selected automatically, the cards are hidden and you go straight to the repository list.
- If you have more than one, you pick the one to use. When a registry is already selected, its section starts collapsed and shows the selected name; expand it to change it.
- **Open integrations** on the "Repositories are pulled from your connected git accounts." line, and the **Connect another** card at the end of the list, open the Integrations page in a new tab. With no Git connection at all, the step says "Komuta needs a git connection to fetch your source." After adding a connection, go back to the wizard; if it isn't in the list, reload the page.

You can search the repository list by name and description. Every repository the connection can reach is listed, private ones included; private repositories need no extra setting. A public repository can also be picked only through a connected account. The wizard doesn't continue without a Git account.

Selecting a repository opens a branch picker under it. No branch is selected for you; pick one with **Select a branch…**. The repository's default branch sits at the top with a **default** label, and protected branches carry a shield icon. If branches can't be loaded, **Failed** with the status code and **Retry** appear.

| Message | What to do |
|---|---|
| Pick a git account or connect a new one. | Pick a Git account, or connect one on Integrations. |
| Pick a container registry for the built image. | Expand the registry section and pick one. |
| Pick a repository to deploy from. | Pick a repository from the list. |
| Pick a branch. | Pick a branch in the branch picker. |
| No repositories returned for this account. | Make sure the connection can reach the repositories; on GitHub, add the repository to the installation. |

---

## Free Plan and a Personal Git Account

The free plan needs the organization to be tied to a personal GitHub, GitLab or Azure DevOps account. Komuta first tries to set this up by itself, using the Git account that owns the free service's repository. Each Git account can use the free plan in one organization only.

If it could not be set up automatically, use the **Free plan Git account** card on the [External Logins](https://console.komuta.io/account/external-logins) page of your account settings:

- For GitHub, click **Link GitHub**. You confirm on GitHub, and the account is also added as a sign-in method.
- For GitLab or Azure DevOps, first connect your personal account on the Integrations page, then click **Bond with this account** in the card's **Your Git accounts** list. You must be the person who added that connection.

Only personal Git accounts count. Organization accounts, GitHub App installations and self-hosted Git servers cannot be used for the free plan. For the rules of free services, see [Service Sleep](service-sleep.md) and [Billing](billing.md).

---

## Removing a Git Connection

**GitLab and Azure DevOps:** On the provider card, click the trash icon (**Remove connection**) next to the connection chip and confirm with **Remove**. Komuta also deletes the connection's webhooks and subscriptions.

**GitHub:** A GitHub connection is managed together with its installation. In the **GitHub App Installations** section, click the trash icon on the installation card and confirm with **Remove**. This removes the installation and the connection from Komuta; the app stays installed on GitHub. To remove it from GitHub too, use **Configure on GitHub**.

Removal cannot be undone. Services built from the connection's repositories keep running, but they can no longer pull source, and the auto-deploy subscriptions on the connection are removed.

> **Warning:** Komuta does not stop you from removing a connection that services still use, and there is no screen that moves services to another connection. Before removing, make sure no service uses it. To renew a token, use **Update token** instead of removing and reconnecting.

---

## Container Registries

### Komuta's Registry

Komuta's registry is ready in every organization and needs no setup. It is marked with the **Managed by platform** badge in the list. You cannot edit or remove it. If you haven't added another registry, the wizard selects it automatically.

### Adding Your Own Registry

1. Click **Add** in the **Container Registries** section.
2. Choose the type in the **Container Registry** field.
3. Enter a **Connection Name** (at least 2 characters). This name is shown in the wizard.
4. Fill in the fields for the type and click **Add Registry**.

| Type | Fields |
|---|---|
| Docker Hub | **Registry URL** (prefilled with `https://docker.io`), **Username**, **Password / Token** |
| GitHub Container Registry | **Registry URL** (`https://ghcr.io`), **Username**, **Password / Token** |
| GitLab Container Registry | **Registry URL** (`https://registry.gitlab.com`), **Username**, **Password / Token** |
| Azure Container Registry | **Registry URL** (prefilled with `https://<your-registry>.azurecr.io`; replace it with your registry's name), **Username**, **Password / Token** |
| AWS ECR | **ECR Registry URL** (for example `123456789.dkr.ecr.us-east-1.amazonaws.com`) |
| Google Container Registry | **Configuration Mode**: **Keyfile JSON** (the content of keyfile.json) or **Docker Config** (the **Docker Config JSON** content) |
| Custom Registry | **Registry URL** (prefilled with `https://your.custom.registry`; replace it with your address), **Username**, **Password / Token** |

The **Password / Token** value is stored encrypted. If a connection with the same name already exists in your organization, the save is refused with `A service connection with the name '<name>' already exists.` The name of Komuta's registry cannot be used either.

### Checking the Connection

The console has no connection test for registries. Komuta doesn't try the registry when you save; the details are used when a build pushes an image. Check the username, token and address in your registry's own dashboard. If they are wrong, the problem shows up in the first build; see [Pipelines](service-pipeline-guide.md) for the build steps.

### Editing and Removing

Registries you added have a pencil icon and a trash icon on their card.

**Editing:** The **Edit Container Registry** dialog opens. If you leave **Password / Token** empty, the stored password is kept. The **Registry URL** field, however, opens empty, and saving it empty deletes the stored address; enter the address again before saving. The same applies to **ECR Registry URL** and the Google Container Registry fields. On success, "Registry updated" appears.

**Removing:** Confirm with **Remove** in the **Remove Registry** dialog. This cannot be undone. Komuta does not stop you from removing a registry that services use; make sure no service uses it before you remove it.

### Picking a Registry in the Wizard

The **Container registry** section in the **Connect** step of the New App wizard lists every registry you can use; Komuta's registry shows the **Managed by platform** badge there as well. The registry you pick is where the service's built image is pushed.

---

## Running an Existing Image

**Services:** The New App wizard always asks for a repository and a branch; a service is built from the repository. The wizard has no option for an existing image. To deploy an image built in your own CI to an existing service, use a deploy token with the **Deploy a ready-made image** action from the **CI/CD integration** tab of the service's **Auto-deploy** page; see [Deploying from CI](ci-deploy.md).

**Jobs and CronJobs:** You can run an existing image without a build.

1. Start a new job on the [Jobs & CronJobs](https://console.komuta.io/jobs) page.
2. Choose **An image you already have** as the source.
3. In the **Registry** step, pick the registry connection used to pull the image.
4. Enter the full image name in **Image reference**, for example `ghcr.io/acme/cleanup:1.4`. Use an immutable tag or digest from the selected registry.

See [Jobs and CronJobs](jobs-cronjobs.md) for details.

---

## Common Messages

| Message | Where | What to do |
|---|---|---|
| Could not complete the connection. Please try again. | On return from GitHub or GitLab | Try again. On GitHub, make sure you authorize as the account owner. |
| Couldn't load repositories | **Repositories** panel | The connection may have been removed or suspended, or its token is no longer valid. Check the installation status or update the token. |
| No repositories found | **Repositories** panel | The connection can't reach any repository. On GitHub, add repositories with **Configure on GitHub**. |
| GitHub App disconnected | The service's **Auto-deploy** tab | The installation was removed or suspended on GitHub. Check its status under **GitHub App Installations**. |
| Could not enable auto-deploy. The PAT needs Service Connections (Read, query, manage). | **Azure DevOps auto-deploy** | Grant the PAT that access, renew it with **Update token**, then try again. |
| Token expired | Next to a GitLab or Azure DevOps connection | Enter a new token with **Update token**. |

---

## Related Documents

- [Quick Start](quick-start.md) — What to prepare before your first service.
- [Creating a New Service](new-service-guide.md) — Every step of the wizard.
- [Auto Deploy](service-auto-deploy.md) — The rules of deploy on push.
- [Pipelines](service-pipeline-guide.md) — The build and release process.
- [Jobs and CronJobs](jobs-cronjobs.md) — Running a job from an existing image.
- [Support](support-tickets.md) — Reaching the support team.
