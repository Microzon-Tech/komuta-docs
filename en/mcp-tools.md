# Komuta MCP

With Komuta MCP, you can handle your work on Komuta by simply asking your AI assistant, instead of opening the dashboard. Creating a project, deploying an app, setting up a database, checking service logs, or reviewing invoices can all be done with a single message. Below is everything the assistant can do for you, grouped by category.

To set it up, visit: [https://www.komuta.io/docs/mcp/mcp-setup](https://www.komuta.io/docs/mcp/mcp-setup)

## Project Tools (4)

| Tool | Description |
|---|---|
| `komuta_list_regions` | Lists available PaaS regions. |
| `komuta_list_projects` | Lists the user's projects. |
| `komuta_create_project` | Creates a new project. |
| `komuta_check_repo_connected` | Checks whether a repo is already connected to a project. |

## Service Tools (8)

| Tool | Description |
|---|---|
| `komuta_discover_repo_services` | Discovers deployable services within a repo. |
| `komuta_provision_service` | Deploys a new service into a project. |
| `komuta_list_services` | Lists services. |
| `komuta_get_service_deploy_options` | Gets a service's default deploy settings. |
| `komuta_get_service_overview` | Gets a service's status overview. |
| `komuta_get_service_pipeline_status` | Gets a service's last/specific pipeline run status. |
| `komuta_trigger_pipeline` | Redeploys a service. |
| `komuta_change_branch` | Changes the branch a service is connected to. |

## Dockerfile Tools (3)

| Tool | Description |
|---|---|
| `komuta_generate_dockerfile` | Fetches the Dockerfile-generation prompt and platform build rules (mandatory before writing/editing any Dockerfile). |
| `komuta_validate_dockerfile` | Validates a Dockerfile against platform rules (mandatory after writing/editing any Dockerfile). |
| `komuta_validate_dockerfile_location` | Validates that the Dockerfile sits in the selected service's own directory (mandatory for monorepos). |

## Addon Tools (7)

| Tool | Description |
|---|---|
| `komuta_create_database` | Creates a managed Postgres instance. |
| `komuta_create_rabbitmq` | Creates a managed RabbitMQ instance. |
| `komuta_create_valkey` | Creates a managed Valkey instance. |
| `komuta_list_addons` | Lists managed addons. |
| `komuta_get_addon` | Gets an addon's status (status poll). |
| `komuta_get_addon_connection_info` | Gets an addon's connection info. |
| `komuta_list_addon_plans` | Lists managed-addon plans. |

## Connection Tools (6)

| Tool | Description |
|---|---|
| `komuta_list_git_connections` | Lists Git service connections. |
| `komuta_list_registry_connections` | Lists container registry connections. |
| `komuta_get_github_install_url` | Gets the GitHub App install URL. |
| `komuta_list_connection_repos` | Lists repos visible through a Git connection. |
| `komuta_list_connection_branches` | Lists branches for a repo via a connection. |
| `komuta_create_registry_connection` | Creates a new container registry connection. |

## Autodeploy Tools (2)

| Tool | Description |
|---|---|
| `komuta_get_service_auto_deploy` | Gets a service's auto-deploy configuration. |
| `komuta_set_service_auto_deploy` | Applies a service's auto-deploy configuration. |

## Scaling Tools (2)

| Tool | Description |
|---|---|
| `komuta_get_service_hpa` | Gets a service's autoscaling (HPA) settings. |
| `komuta_set_service_hpa` | Applies a service's autoscaling (HPA) settings. |

## Logs Tools (1)

| Tool | Description |
|---|---|
| `komuta_query_service_logs` | Queries a service's pod logs. |

## Billing Tools (1)

| Tool | Description |
|---|---|
| `komuta_get_service_invoices` | Gets a service's current-cycle charges and invoice history. |

## Pricing Tools (1)

| Tool | Description |
|---|---|
| `komuta_list_resource_packages` | Lists PaaS resource packages (the package is always chosen by the user, never the agent). |
