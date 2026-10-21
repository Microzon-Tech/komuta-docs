# Komuta MCP

With Komuta MCP, you can handle your work on Komuta by simply asking your AI assistant, instead of opening the dashboard. Creating a project, publishing an app, setting up a database, checking service logs, or reviewing invoices can all be done with a single message. Below is everything the assistant can do for you, grouped by category.

To set it up, visit: [https://www.komuta.io/docs/mcp/mcp-setup](https://www.komuta.io/docs/mcp/mcp-setup)

---

## How Many Tools Are There?

Komuta MCP offers **210 tools**, all listed below by category.

Not all of these tools are used at once. The assistant first works out which task your request belongs to (`komuta_select_workflow`) and then proceeds only with that task's tools.

---

## Workflow & Onboarding Tools (6)

| Tool | Description |
|---|---|
| `komuta_select_workflow` | Determines which task a request belongs to; called before any other tool. |
| `komuta_end_workflow` | Closes the active workflow when the task is finished or abandoned. |
| `komuta_get_started` | Returns a guide for connecting Komuta MCP and the first steps. |
| `komuta_list_organizations` | Lists the organizations the account belongs to and the one it is signed in to. |
| `komuta_get_local_service_context` | Reads the default project/service saved for your working folder. |
| `komuta_set_local_service_context` | Saves a default project/service pair for your working folder. |

## Project Tools (4)

| Tool | Description |
|---|---|
| `komuta_list_regions` | Lists available PaaS regions. |
| `komuta_list_projects` | Lists the user's projects. |
| `komuta_create_project` | Creates a new project. |
| `komuta_check_repo_connected` | Checks whether a repo is already connected to a project. |

## Stack Tools (14)

New services are published with a Stack: the `komuta.yaml` at the repo root is validated, a plan is created and applied with your approval. Details: [Stacks](stacks.md).

| Tool | Description |
|---|---|
| `komuta_verify_stack_manifest_file` | Checks that the local `komuta.yaml` is ready for validation. |
| `komuta_get_stack_publish_context` | Returns the account, target, connection and pricing choices needed to publish. |
| `komuta_resolve_repo_source` | Finds which Git connection sees a repo URL and which branch to use. |
| `komuta_set_stack_profile` | Saves the preferred Stack authoring style. |
| `komuta_list_stacks` | Lists existing Stacks. |
| `komuta_validate_stack` | Validates a `komuta.yaml` file. |
| `komuta_plan_stack` | Creates an immutable Stack plan valid for 30 minutes. |
| `komuta_get_stack_plan` | Re-reads a plan that was already created. |
| `komuta_create_stack_secret_handoff` | Creates a single-use, 15-minute console link for entering missing secrets. |
| `komuta_apply_stack` | Applies a valid plan. |
| `komuta_get_stack_run` | Returns the status of a Stack run. |
| `komuta_resume_stack_run` | Retries only the failed steps of a partially failed run. |
| `komuta_get_stack_drift` | Reports whether live resources still match what the Stack last applied. |
| `komuta_export_stack` | Exports the active Stack as `komuta.yaml`. |

## Service Tools (16)

| Tool | Description |
|---|---|
| `komuta_discover_repo_services` | Discovers deployable services in a repo branch. |
| `komuta_list_services` | Lists services. |
| `komuta_get_service_overview` | Returns the service's status overview card. |
| `komuta_get_service_details` | Returns the service configuration. |
| `komuta_get_service_health_probes` | Returns the service's health probes. |
| `komuta_get_service_routing` | Returns the service's routing settings. |
| `komuta_list_service_dependencies` | Lists the service's dependencies. |
| `komuta_list_service_ports` | Lists the service's ports. |
| `komuta_get_service_deploy_options` | Returns the default deploy settings for a service. |
| `komuta_get_service_pipeline_status` | Returns the pipeline status of the latest/a specific deploy. |
| `komuta_get_pipeline_task_logs` | Returns the raw logs of a pipeline step. |
| `komuta_trigger_pipeline` | Redeploys the service. |
| `komuta_change_branch` | Changes the branch the service is connected to. |
| `komuta_set_service_replicas` | Changes the replica count while HPA is off. |
| `komuta_start_service_maintenance` | Puts the service into maintenance; visitors see a maintenance page. |
| `komuta_end_service_maintenance` | Turns maintenance off and starts the service again. |

## Rollout Tools (10)

| Tool | Description |
|---|---|
| `komuta_list_service_rollouts` | Lists the service's rollouts. |
| `komuta_get_service_rollout_status` | Returns rollout status. |
| `komuta_get_service_rollout_history` | Returns rollout history. |
| `komuta_pause_service_rollout` | Pauses a rollout. |
| `komuta_resume_service_rollout` | Resumes a paused rollout. |
| `komuta_promote_service_rollout` | Promotes a rollout to live. |
| `komuta_abort_service_rollout` | Aborts a rollout. |
| `komuta_retry_service_rollout` | Retries a rollout. |
| `komuta_rollback_service_rollout` | Rolls back to the previous version. |
| `komuta_restart_service_rollout` | Restarts a rollout. |

## Sleep & Always On Tools (9)

| Tool | Description |
|---|---|
| `komuta_get_service_sleep_state` | Returns whether the service is running or asleep, and why. |
| `komuta_get_service_sleep_history` | Lists sleep, wake and Always on events. |
| `komuta_hibernate_service` | Puts the service to sleep. |
| `komuta_wake_service` | Wakes the service. |
| `komuta_set_service_auto_sleep` | Turns automatic sleep on or off. |
| `komuta_get_free_hours_usage` | Returns the organization's free hours usage this month. |
| `komuta_get_service_keep_awake` | Returns the status of the service's Always on add-on. |
| `komuta_purchase_service_keep_awake` | Buys the Always on add-on for the service. |
| `komuta_cancel_service_keep_awake` | Turns the Always on add-on off. |

## Service Doctor Tools (8)

| Tool | Description |
|---|---|
| `komuta_assess_service` | Assesses the service with Service Doctor and produces recommendations. |
| `komuta_accept_critical_recommendation` | Accepts a recommendation. |
| `komuta_defer_critical_recommendation` | Defers a recommendation until a chosen date. |
| `komuta_dismiss_critical_recommendation` | Dismisses a recommendation. |
| `komuta_prepare_recommendation_action_plan` | Prepares an action plan for an accepted recommendation. |
| `komuta_approve_recommendation_action_plan` | Approves the prepared plan. |
| `komuta_record_recommendation_applied` | Records a manually applied plan as completed. |
| `komuta_verify_recommendation_action` | Verifies an applied recommendation by reassessing the service. |

## Dockerfile Tools (3)

| Tool | Description |
|---|---|
| `komuta_generate_dockerfile` | Returns the prompt + platform build rules for generating a Dockerfile (mandatory BEFORE writing/editing a Dockerfile). |
| `komuta_validate_dockerfile` | Validates a Dockerfile against platform rules (mandatory AFTER writing/editing). |
| `komuta_validate_dockerfile_location` | Verifies the Dockerfile is in the selected service's own directory (mandatory for monorepo scenarios). |

## Environment Variable Tools (4)

| Tool | Description |
|---|---|
| `komuta_list_service_env_vars` | Lists the service's environment variables. |
| `komuta_set_service_env_vars` | Adds/updates one or more environment variables in one call. |
| `komuta_copy_service_secrets` | Copies secrets from another service without revealing their values. |
| `komuta_apply_service_env_vars` | Applies saved environment variables to the running service. |

## Environment Group Tools (7)

| Tool | Description |
|---|---|
| `komuta_list_env_groups` | Lists environment groups. |
| `komuta_get_env_group` | Returns a group's variables and which services use them. |
| `komuta_create_env_group` | Creates a new, empty environment group. |
| `komuta_set_env_group_variables` | Adds/updates variables in a group. |
| `komuta_preview_env_group_import` | Shows the differences before importing a group into a service. |
| `komuta_import_env_group` | Imports the chosen variables of a group into a service. |
| `komuta_push_env_group` | Pushes the group's current values to the chosen services. |

## Autodeploy Tools (2)

| Tool | Description |
|---|---|
| `komuta_get_service_auto_deploy` | Returns the service's auto-deploy configuration. |
| `komuta_set_service_auto_deploy` | Applies the service's auto-deploy configuration. |

## Scaling Tools (2)

| Tool | Description |
|---|---|
| `komuta_get_service_hpa` | Returns the service's autoscaling (HPA) settings. |
| `komuta_set_service_hpa` | Applies the service's autoscaling (HPA) settings. |

## Domain Tools (5)

| Tool | Description |
|---|---|
| `komuta_list_domains` | Lists custom domains bound to services and their DNS/TLS status. |
| `komuta_list_attachable_services` | Lists services a domain can be attached to. |
| `komuta_create_domain` | Binds a custom domain to a service. |
| `komuta_refresh_domain_validation` | Re-runs DNS and certificate validation. |
| `komuta_delete_domain` | Permanently removes a domain binding. |

## Storage Tools (6)

| Tool | Description |
|---|---|
| `komuta_list_storage_attachments` | Lists persistent storage volumes attached to a service. |
| `komuta_list_storage_packages` | Lists storage plans. |
| `komuta_list_service_directories` | Lists the service's directories to pick a mount path. |
| `komuta_create_storage_attachment` | Adds a new storage volume to a service. |
| `komuta_update_storage_attachment` | Changes a volume's plan or settings. |
| `komuta_delete_storage_attachment` | Permanently deletes a volume and its data. |

## Access Protection Tools (18)

| Tool | Description |
|---|---|
| `komuta_get_service_access_policy` | Shows who can open the service. |
| `komuta_preview_service_access_change` | Shows the effect of an access change without applying it. |
| `komuta_apply_service_access_change` | Applies the approved access change. |
| `komuta_explain_service_access` | Explains whether a visitor can open a page and which rule decides. |
| `komuta_list_service_access_shares` | Lists who can sign in to a protected service. |
| `komuta_list_service_access_share_candidates` | Lists members and organizations the service can be shared with. |
| `komuta_save_service_access_share` | Adds or updates a share. |
| `komuta_delete_service_access_share` | Removes a share. |
| `komuta_list_service_access_tokens` | Lists service tokens for machines. |
| `komuta_create_service_access_token` | Creates a new service token. |
| `komuta_delete_service_access_token` | Deletes a service token. |
| `komuta_get_service_access_mesh_peers` | Shows services that can reach it directly over the private network. |
| `komuta_set_service_access_mesh_peers` | Sets services that can reach it directly over the private network. |
| `komuta_list_service_access_events` | Lists sign-in, access and refusal events. |
| `komuta_get_organization_access_sharing` | Shows the setting for sharing outside the organization. |
| `komuta_set_organization_access_sharing` | Turns sharing outside the organization on or off. |
| `komuta_list_organization_access_identity_providers` | Lists the organization's identity providers. |
| `komuta_get_access_identity_keys` | Returns the keys an app uses to verify the visitor's identity. |

## Job Tools (8)

| Tool | Description |
|---|---|
| `komuta_get_job_configuration` | Returns the job configuration. |
| `komuta_get_cronjob_state` | Returns the CronJob state. |
| `komuta_get_active_job_runs` | Returns runs in progress. |
| `komuta_list_job_runs` | Lists run history. |
| `komuta_run_job_now` | Runs the job once, right now. |
| `komuta_cancel_job_run` | Cancels a run in progress. |
| `komuta_suspend_cronjob` | Stops scheduled runs. |
| `komuta_resume_cronjob` | Resumes scheduled runs. |

## Addon Tools (7)

| Tool | Description |
|---|---|
| `komuta_create_database` | Creates a managed Postgres instance. |
| `komuta_create_rabbitmq` | Creates a managed RabbitMQ instance. |
| `komuta_create_valkey` | Creates a managed Valkey instance. |
| `komuta_list_addons` | Lists managed addons. |
| `komuta_get_addon` | Returns addon status (status poll). |
| `komuta_get_addon_connection_info` | Returns addon connection details. |
| `komuta_list_addon_plans` | Lists managed addon plans. |

## Addon Lifecycle Tools (28)

The same operations for Postgres, Valkey and RabbitMQ are done with separate tools.

| Tool | Description |
|---|---|
| `komuta_restart_postgres` / `_valkey` / `_rabbitmq` | Restarts the instance. |
| `komuta_backup_postgres` / `_valkey` / `_rabbitmq` | Starts a backup. |
| `komuta_schedule_deletion_postgres` / `_valkey` / `_rabbitmq` | Schedules the instance for deletion. |
| `komuta_cancel_deletion_postgres` / `_valkey` / `_rabbitmq` | Cancels a scheduled deletion. |
| `komuta_preview_postgres_plan_change` / `valkey` / `rabbitmq` | Shows a plan change without applying it. |
| `komuta_change_postgres_plan` / `valkey` / `rabbitmq` | Switches to the approved plan. |
| `komuta_soft_suspend_postgres` | Temporarily suspends Postgres. |
| `komuta_hard_suspend_postgres` | Fully suspends Postgres. |
| `komuta_resume_postgres` | Resumes a suspended Postgres. |
| `komuta_get_postgres_restore_points` | Lists available restore points. |
| `komuta_restore_latest_postgres` | Restores from the latest backup. |
| `komuta_cancel_postgres_restore` | Cancels a restore in progress. |
| `komuta_rotate_postgres_credentials` | Rotates Postgres credentials. |
| `komuta_get_postgres_backup_health` | Returns backup health. |
| `komuta_refresh_addon_backup_status` | Refreshes addon backup status. |
| `komuta_list_addon_alerts` | Lists addon alert rules. |

## Connection Tools (6)

| Tool | Description |
|---|---|
| `komuta_list_git_connections` | Lists Git service connections. |
| `komuta_list_registry_connections` | Lists container registry connections. |
| `komuta_get_github_install_url` | Returns the GitHub App installation URL. |
| `komuta_list_connection_repos` | Lists repos visible through a Git connection. |
| `komuta_list_connection_branches` | Lists a repo's branches through a connection. |
| `komuta_create_registry_connection` | Creates a new container registry connection. |

## API Gateway Tools (19)

| Tool | Description |
|---|---|
| `komuta_list_api_gateways` | Lists API Gateways. |
| `komuta_get_api_gateway` | Returns gateway details. |
| `komuta_get_api_gateway_pricing` | Returns gateway pricing. |
| `komuta_list_gateway_apis` | Lists APIs on the gateway. |
| `komuta_list_gateway_api_candidates` | Lists services that can be added to the gateway. |
| `komuta_list_gateway_operations` | Lists gateway operations. |
| `komuta_list_gateway_keys` | Lists gateway keys. |
| `komuta_revoke_gateway_key` | Revokes the selected key. |
| `komuta_preview_gateway_config` | Previews the saved configuration. |
| `komuta_get_gateway_pending_changes` | Returns pending changes. |
| `komuta_apply_gateway_config` | Applies the previewed configuration. |
| `komuta_get_gateway_live_config` | Returns the live configuration. |
| `komuta_get_gateway_version` | Returns the gateway version. |
| `komuta_get_gateway_metrics` | Returns request metrics. |
| `komuta_get_gateway_scale` | Returns replica settings. |
| `komuta_set_gateway_scale` | Changes replica settings. |
| `komuta_get_gateway_bandwidth` | Returns bandwidth plans and usage. |
| `komuta_set_gateway_bandwidth_plan` | Changes the bandwidth plan. |
| `komuta_delete_api_gateway` | Deletes the gateway. |

## Alert Tools (12)

| Tool | Description |
|---|---|
| `komuta_list_alert_rules` | Lists alert rules. |
| `komuta_get_alert_rule` | Returns an alert rule. |
| `komuta_list_alert_templates` | Lists alert templates. |
| `komuta_create_alert_rule` | Creates a new alert rule. |
| `komuta_update_alert_rule` | Updates an alert rule. |
| `komuta_delete_alert_rule` | Deletes an alert rule. |
| `komuta_deploy_alert_rule` | Deploys an alert rule. |
| `komuta_undeploy_alert_rule` | Undeploys an alert rule. |
| `komuta_set_alert_enabled` | Turns an alert rule on or off. |
| `komuta_get_alert_history` | Returns alert history. |
| `komuta_list_notification_channels` | Lists notification channels. |
| `komuta_set_notification_channel_active` | Turns a notification channel on or off. |

## Observability & Audit Tools (4)

| Tool | Description |
|---|---|
| `komuta_query_service_logs` | Queries the service's pod logs. |
| `komuta_query_service_traces` | Searches the service's traces. |
| `komuta_get_trace_detail` | Returns the detail of a trace. |
| `komuta_query_security_audit` | Returns security audit events. |

## Billing & Pricing Tools (12)

| Tool | Description |
|---|---|
| `komuta_get_service_invoices` | Returns the service's current-cycle charges and invoice history. |
| `komuta_get_account_cost` | Returns the account's current-cycle cost. |
| `komuta_get_account_cost_history` | Returns the account's cost history. |
| `komuta_get_build_minutes_usage` | Returns build minute allowance and usage. |
| `komuta_get_account_billing_status` | Returns the account's billing status. |
| `komuta_get_account_wallet` | Returns the wallet balance. |
| `komuta_list_wallet_transactions` | Lists wallet transactions. |
| `komuta_list_account_statements` | Lists account statements. |
| `komuta_get_statement_items` | Returns statement line items. |
| `komuta_get_statement_payment_link` | Creates a payment link for the selected statement. |
| `komuta_pay_statement_from_balance` | Pays the statement from the wallet balance. |
| `komuta_list_resource_packages` | Lists PaaS resource packages (package selection always belongs to the user, never the agent). |
