# komuta.yaml Manifest Reference

This page describes version 1 of the `komuta.yaml` format used by a [Stack](stacks.md): what each field means, whether it is required, its default, the values it accepts, and the error messages you may see during **Create plan**. How to load, plan and apply the file is covered in [Stacks](stacks.md).

---

## File Rules

- The file is a single YAML document whose root is an object. It can be at most 1 MiB and nested at most 32 levels deep.
- A file can contain at most 200 resources: the total of `services`, `addons`, `uses.addons`, `domains` and `secrets` entries.
- Field names are camelCase and case-sensitive. An unknown field is refused: `Unknown field 'repo'. Expected fields: repository, branch, root.`
- A key can't appear twice in the same object. YAML anchors/aliases and custom tags (`!tag`) can't be used.
- Komuta's internal identifiers (values in GUID format) can't be written into the file; refer to everything by name.
- The `gatewayClass` field is managed by Komuta and can't be written anywhere.
- Every name (`name`, `hostname`) must be non-empty and at most 160 characters. The same name can't be used twice in the same section; this check ignores letter case.
- Values inside angle brackets, such as `<project-name>`, count as placeholders and produce a warning.
- Values such as passwords, keys and tokens can't be written into the file; use [`secrets`](#secrets) for them.

---

## Top-Level Fields

| Field | Required | Meaning |
|---|---|---|
| `version` | Yes | The format version. Only `1`. |
| `name` | Yes | The Stack's name and identity. A file planned with the same name updates the same Stack. |
| `target` | Yes | The project, region and connections the resources are set up with. |
| `apply` | No | Apply options. |
| `secrets` | No | Names of secret values that aren't written into the file. |
| `addons` | No | Managed services the Stack creates and manages. |
| `uses` | No | Existing managed services the Stack uses without managing them. |
| `services` | No | Services, Jobs and CronJobs. |
| `domains` | No | Domains attached to services. |

---

## target

| Field | Default | Meaning |
|---|---|---|
| `project` | — | The target project's name. Required; 3–100 characters. If the project doesn't exist, it is created while `apply.createMissingProject` is on. |
| `region` | — | The region's name or code, for example `de-east`. Needed for services and managed services; if it's missing, the plan passes but those steps stop with `REGION_NOT_FOUND` during the run. |
| `envGroup` | — | Accepted; today it doesn't affect the resources that are created. |
| `deploymentTarget` | `paas` | Komuta hosting. You don't need to write this field. |
| `connections.git` | — | The name of your Git connection on the **Integrations** page. Needed for every service. |
| `connections.registry` | — | The name of a container registry connection (Komuta's own registry included). Needed for every service; if it can't be found, the service step stops with `REGISTRY_CONNECTION_NOT_FOUND`. |

---

## apply

| Field | Default | Accepted | Meaning |
|---|---|---|---|
| `prune` | `false` | only `false` | Resources removed from the file aren't deleted; they stay as **Orphan**. |
| `createMissingProject` | `true` | `true` / `false` | If `false` and the project doesn't exist, the plan is blocked. |
| `adoptExisting` | `explicit` | only `explicit` | Existing resources are adopted only after you confirm each one on the plan screen. |
| `failurePolicy` | `stop-dependents` | — | Steps that depend on a failed step are always blocked and the others carry on. The value isn't checked today and doesn't change this behavior. |
| `maxParallel` | `3` | 1–10 | The maximum number of steps that run at once. |

---

## secrets

The secret value itself is never written into the file; only its name and where it comes from are declared here. An environment variable refers to the value with `valueFrom.secret`.

| Field | Default | Meaning |
|---|---|---|
| `name` | — | Required. References are case-sensitive. |
| `source` | `prompt` | `prompt`: entered through the one-time secure link. `generate`: Komuta generates it during the run. `existing`: a value already stored in this Stack is used. |
| `required` | `true` | A `prompt` value with `false` isn't asked for in the secure form; if a variable refers to it, the step stops with `SECRET_NOT_RESOLVED`. |
| `existingName` | — | Required for `existing`: the name the value is stored under in this Stack. If there is no value under that name, the secret's own name is tried. |
| `length` | `48` | The length of the value to generate for `generate`; 24–256. |

```yaml
secrets:
  - name: payment-api-key           # entered through the secure link
  - name: session-secret
    source: generate
    length: 64
```

Values are kept in the Stack's encrypted store and aren't asked for again in later plans of the same Stack. See [Stacks](stacks.md#secrets) for details.

---

## services

Each entry is a service, a Job or a CronJob.

| Field | Default | Meaning |
|---|---|---|
| `name` | — | Required. Becomes the service name in Komuta. |
| `workload` | `service` | `service`, `job` or `cronjob`. |
| `source` | — | The repository and branch the code comes from. |
| `build` | — | Build settings. |
| `compute` | — | Package and replica count. |
| `rollout` | — | Rollout strategy. |
| `autoDeploy` | — | Automatic deployment on push. |
| `ports` | — | Ports (`service` only). |
| `network` | — | Public address and routes (`service` only). |
| `env` | — | Environment variables. |
| `storage` | — | Persistent storage (`service` only). |
| `health` | — | Health checks (`service` only). |
| `security` | — | Runtime security settings. |
| `bandwidth` | — | Bandwidth plan. |
| `schedule`, `execution` | — | Job and CronJob settings. |
| `access` | — | Access protection; see [Access Protection in a Stack Manifest](stack-manifest-access.md). |

### source

| Field | Default | Meaning |
|---|---|---|
| `repository` | — | Required. The repository address, for example `https://github.com/acme/shop.git`. |
| `branch` | — | Required. The plan pins the latest commit on this branch. |
| `root` | — | The service's folder in the repository (monorepo). The default Dockerfile path and build context are based on it. |

### build

| Field | Default | Meaning |
|---|---|---|
| `strategy` | `dockerfile` | `dockerfile`: built with the Dockerfile in the repository. `image`: a prebuilt image is deployed without a build. |
| `dockerfile` | `<root>/Dockerfile` | The Dockerfile's path in the repository. With the `dockerfile` strategy the file must be in the pushed branch, or the plan is blocked. |
| `context` | `root`, else `.` | The build context. |
| `image.name` | the service name | With the `dockerfile` strategy, the name of the built image. With the `image` strategy, the full reference of the image to deploy, for example `nginx:1.27`. |
| `image.tag` | `git-sha` | The tag of the built image. |
| `dockerfileAutoFix` | `false` | If `true`, when a build fails Komuta opens an AI-fixed Dockerfile as a pull request on a separate branch and retries the build on that branch. |
| `detectedFramework` | — | The application's framework. Komuta uses it to open the folders that framework needs to write to at runtime. |
| `runtimeLibc` | — | The C library of the runtime image: `musl` or `glibc`. Used for .NET applications so automatic tracing picks the right component. |

> **Note:** With the `image` strategy too, `source.repository`, `source.branch` and `target.connections` are required.

### compute

| Field | Default | Meaning |
|---|---|---|
| `plan` | — | Required. The package name: `dz-nano` (the free package), `dz-dev`, `dz-standard`, `dz-premium`, `dz-enterprise`. Write it in lowercase, as in the console. |
| `replicas.mode` | `fixed` | `fixed`, or `autoscaling` for autoscaling (`hpa` is accepted too). |
| `replicas.count` | `1` | The replica count in `fixed` mode. |
| `replicas.min`, `replicas.max` | — | The lower and upper bounds for autoscaling. |
| `replicas.cpuTargetPercent`, `replicas.memoryTargetPercent` | `80` | Autoscaling targets. |

> **Warning:** On Komuta hosting a new service is created with a fixed replica count: `count` in `fixed` mode, 1 in `autoscaling` mode. `min` and `max` aren't applied at creation; for real autoscaling, raise the upper bound on the [Autoscaling](services-autoscaling.md) page after the service is created. The cost estimate uses `min` in `autoscaling` mode.

The free package (`dz-nano`) runs a single replica. In `fixed` mode, a `count` above 1 blocks the plan.

### rollout and autoDeploy

| Field | Default | Meaning |
|---|---|---|
| `rollout.strategy` | `auto-promote` | `auto-promote`, `blue-green` or `canary`. Details are in [Deployment Strategies](deployment-strategies.md). Updating environment variables through the Stack needs `auto-promote`. |
| `autoDeploy.enabled` | `false` | Automatic deployment on push to the branch. If the `autoDeploy` block is left out entirely, automatic deployment starts on as long as your Git connection can send push notifications. |
| `autoDeploy.includePaths`, `autoDeploy.excludePaths` | — | Which paths' changes trigger a deployment. See [Auto Deploy](service-auto-deploy.md). |

`autoDeploy.cooldownSeconds` and `autoDeploy.skipCommitPatterns` are accepted, but the plan refuses them.

### ports

`service` only. If `ports` is left out, the service is created with a single primary port 8080.

| Field | Default | Meaning |
|---|---|---|
| `name` | `http` | The port's name. |
| `protocol` | `TCP` | The protocol. |
| `containerPort` | — | The port the application listens on. |
| `servicePort` | `containerPort` | The port the service exposes. |
| `targetPort` | `containerPort` | The port traffic is sent to. |
| `primary` | `false` | The primary port. If none is marked, the first port becomes primary. |
| `active` | `true` | Accepted; at creation the port is always set up as active. |

See [Access & Ports](services-ports.md) for port concepts.

### network

| Field | Default | Meaning |
|---|---|---|
| `public` | `false` | If `true`, the service gets a public address. Needed for access protection. |
| `routes[].host` | — | The route's domain. It must be a domain attached to **the same service** under `domains`; otherwise the step stops with `ROUTE_HOST_NOT_OWNED`. |
| `routes[].matches[].path` | `/` | A path starting with `/`. |
| `routes[].matches[].pathType` | `Prefix` | `Prefix` or `Exact`. |
| `meshExpose`, `dependencies` | — | Accepted, but the plan refuses them: `Private mesh dependencies are not enabled on this target.` |

### env

Each entry is an environment variable. The value is either written directly with `value` or taken from a source with `valueFrom`; the two can't be used together.

| Field | Default | Meaning |
|---|---|---|
| `name` | — | Required; unique within the service. |
| `value` | — | A plain value. |
| `sensitive` | `false` | The value is stored as secret. |
| `buildTime` | `false` | The value is also provided during the build. |
| `valueFrom.secret` | — | The value of a name under `secrets`. Always stored as secret. |
| `valueFrom.addon` | — | Connection details of a managed service: `name`, `property`, `endpoint`. |
| `valueFrom.service` | — | The address of a service in the Stack: `name`, `property`. |

Rules:

- A variable with `sensitive: true`, or whose name reads as a credential, can't have a `value`. Names ending like `API_KEY`, `SECRET`, `PASSWORD`, `PRIVATE_KEY`, `ACCESS_TOKEN`, `CREDENTIALS`, `CONNECTION_STRING`, `DATABASE_URL` or `REDIS_URL` are always treated as secret (underscores and letter case don't matter).
- A secret value can't be `buildTime: true`.
- A variable starting with `NEXT_PUBLIC_`, `NUXT_PUBLIC_`, `EXPO_PUBLIC_`, `REACT_APP_`, `GATSBY_`, `VUE_APP_`, `VITE_` or `PUBLIC_` that isn't `buildTime: true` gets a warning; the built frontend can't see that value.
- See [Environment Variables](services-environment-variables.md) for variable details.

**`valueFrom.addon`** — the value is written during the run, after the managed service is ready.

| Field | Default | Accepted |
|---|---|---|
| `name` | — | A name under `addons` or `uses.addons`. |
| `property` | — | `host`, `port`, `database`, `user`, `password`, `connectionString`. For `password` and `connectionString` the variable must have `sensitive: true`. |
| `endpoint` | `private` | `private` or `public`. `public` needs the managed service's public access to be on. RabbitMQ can only use `public`. |

`connectionString` format: `postgresql://user:password@host:port/database?sslmode=require` for PostgreSQL, `rediss://user:password@host:port` for Valkey, `amqps://user:password@host:port/%2F` for RabbitMQ. For Valkey the `database` value is `0`; for RabbitMQ it is the virtual host.

**`valueFrom.service`** — `property: host` gives the service's address, `property: url` gives `https://<address>/`. The address is the first domain attached to that service under `domains`; if there is none, it is the address Komuta assigned to the service. The referenced service is created first; services can't refer to each other in a cycle.

### storage

`service` only. Concepts are in [Persistent Storage](storage.md).

| Field | Default | Meaning |
|---|---|---|
| `name` | — | Lowercase letters, digits and hyphens; starts and ends with a letter or digit; at most 63 characters. |
| `plan` | — | The storage plan's code or name, for example `storage-small`. |
| `mountPath` | — | An absolute path; can't be `/` or contain `..`. |
| `subPath` | — | A relative path; can't contain `..`. |
| `readOnly` | `false` | Read-only mount. |
| `active` | `true` | `false` is accepted, but the plan refuses it. |

### health

`service` only. The `readiness`, `liveness` and `startup` checks each take the same fields. Concepts are in [Health Probes](services-health-probes.md).

| Field | Default | Meaning |
|---|---|---|
| `enabled` | `true` | A disabled check isn't validated. |
| `type` | `httpGet` | `httpGet`, `tcpSocket` or `exec`. |
| `path` | — | Required for `httpGet`. |
| `port` | — | Required for `httpGet` and `tcpSocket`; 1–65535. |
| `scheme` | `HTTP` | `HTTP` or `HTTPS`. |
| `command` | — | At least one item for `exec`. |
| `initialDelaySeconds` | `0` | Can't be negative. |
| `periodSeconds` | `10` | At least 1. |
| `timeoutSeconds` | `3` | At least 1. |
| `successThreshold` | `1` | At least 1; only 1 for `liveness` and `startup`. |
| `failureThreshold` | `3` | At least 1. |

### security

These fields are applied when the service is created; most need a separate permission, and the plan is blocked if you don't have it. Concepts are in [Service Security](service-security-guide.md).

| Field | Default | Meaning | Permission needed |
|---|---|---|---|
| `allowRunAsRoot` | `false` | Lets the container run as root. | Allow a service's container to run as root |
| `addCapabilities` | — | Extra Linux capabilities; only `CHOWN`, `FOWNER`, `FSETID`, `NET_BIND_SERVICE`, `KILL` (the `CAP_` prefix is accepted). | Opt a service into the Linux capability allowlist |
| `extraWritableMounts` | — | Extra writable paths; protected paths are refused. | Add extra writable mounts to a service |
| `persistFrameworkKeys` | `false` | Keeps the framework's data protection keys across restarts. `service` only. | Add extra writable mounts to a service |
| `autoInstrumentation.disabled` | `false` | If `true`, Komuta doesn't add automatic tracing to the service; it runs without producing traces. Use it if the tracing component breaks your application. | Add extra writable mounts to a service |
| `autoInstrumentation.pythonOptIn` | `false` | Accepted and stored; it has no effect today. Python doesn't need a separate opt-in; `disabled` is what turns automatic tracing off. | Add extra writable mounts to a service |
| `networkAlerts` | `false` | Turns on network alert rules for the service. | — |
| `policyGuardMode` | `audit` | Only `audit` can be applied; `off`, `shadow` and `enforce` are accepted, but the plan refuses them. | — |

The plan shows the settings that relax restrictions on the **Security relaxations** card.

### bandwidth

| Field | Meaning |
|---|---|
| `plan` | The bandwidth plan's name: `Starter`, `Growth`, `Business`, `Enterprise`, `Scale`. If left out, the service starts on the default plan. See [Bandwidth Plans](bandwidth-plans.md). |

### Job and CronJob fields

For `workload: job` or `workload: cronjob`. These workloads don't use `ports`, `network`, `storage`, `health` or `compute.replicas`.

| Field | Default | Meaning |
|---|---|---|
| `schedule.cron` | — | Required for a CronJob; a five-field cron expression, for example `0 3 * * *`. |
| `schedule.timeZone` | — | An IANA time zone, for example `Europe/Istanbul`. |
| `schedule.suspended` | `false` | Pauses the schedule. |
| `schedule.concurrencyPolicy` | — | Accepted, but the plan refuses it. |
| `execution.restartPolicy` | `OnFailure` | Only `OnFailure` can be applied. |
| `execution.backoffLimit` | — | The retry count; 0 or more. |
| `execution.activeDeadlineSeconds` | — | The longest run time; more than 0. |
| `execution.ttlSecondsAfterFinished` | — | How long a finished run is kept; 0 or more. |
| `execution.command`, `execution.args` | — | Override the image's entrypoint and arguments. |

The cron, time zone and duration values aren't checked during planning but when the Job is created; a value that doesn't fit stops that step.

> **Warning:** At the moment, the step for a Job or CronJob created through a Stack fails during the run. Create your Jobs and CronJobs from the [Jobs & CronJobs](jobs-cronjobs.md) page.

### Fields not available yet

These fields are accepted in the file, but the plan is blocked with `CAPABILITY_NOT_AVAILABLE`:

| Field | Plan message |
|---|---|
| `observability` (`slos`, `alerts`, `dashboards`) | Declarative SLO, alert and dashboard adapters are not enabled on this target. |
| `network.meshExpose`, `network.dependencies` | Private mesh dependencies are not enabled on this target. |
| `autoDeploy.cooldownSeconds`, `autoDeploy.skipCommitPatterns` | Auto-deploy cooldown and skip patterns are not enabled on this target. |
| `storage[].active: false` | Creating inactive storage attachments is not enabled on this target. |
| `security.policyGuardMode` (other than `audit`) | Non-default Policy Guard modes require the correlated transition adapter on this target. |
| `schedule.concurrencyPolicy`, `execution.restartPolicy` (other than `OnFailure`) | The requested job execution policy is not enabled on this target. |
| `addons[].backups` | Declarative addon backup configuration is not enabled on this target. |
| `addons[].type: opensearch` | OpenSearch is not provisionable on Komuta yet. |

---

## addons

Managed services the Stack creates and manages.

| Field | Default | Meaning |
|---|---|---|
| `name` | — | Required. Starts with a lowercase letter; lowercase letters, digits and hyphens; at most 40 characters. This rule is checked when the service is created, not during planning. |
| `type` | — | `postgres`, `valkey` or `rabbitmq`. |
| `plan` | — | Required. The plan code or name, for example `postgres-2g`, `valkey-1g`, `rabbitmq-1broker`. |
| `region` | `target.region` | The region's name or code. |
| `topology` | the plan's default | The topology code or name, for example `standalone`. |
| `version` | the plan's default | The version. |
| `database` | `app` | PostgreSQL only: the name of the first database. Starts with a lowercase letter; lowercase letters, digits and underscores; at most 63 characters. |
| `network.access` | `private-only` | `private-only`: reachable only from inside Komuta. `public`: a public address is opened. RabbitMQ needs `public`. `public-allowlist` isn't accepted; add an IP allow-list in the console after creation. |
| `network.allowedCidrs` | — | Can only be used with `public-allowlist`. |
| `config.evictionPolicy` | `noeviction` | Valkey only: `noeviction`, `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `volatile-lfu`, `allkeys-random`, `volatile-random`, `volatile-ttl`. |
| `config.persistenceMode` | `rdb` | Valkey only: `rdb`, `aof` or `none`. `none` needs `topology: standalone` written explicitly. |
| `scope` | `project` | Accepted; it has no effect today. |
| `backups` | — | Accepted, but the plan refuses it. |

Managed services are attached to the target project. For plans and concepts, see [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md) and [RabbitMQ](managed-rabbitmq.md).

> **Note:** The credentials of a PostgreSQL that a Stack manages, or whose connection details a Stack has given to services, can't be rotated from the console; for example, "This addon is managed by a Stack. Credential rotation is unavailable until dependent services can be refreshed safely; no credential was changed." appears.

---

## uses

Existing managed services the Stack uses without managing them. The plan lists them with the **Use** action; they aren't included in the cost estimate.

| Field | Meaning |
|---|---|
| `addons[].name` | The name of a managed service in the target project, or of a shared one that isn't attached to any project. Letter case must match exactly. |
| `addons[].type` | `postgres`, `valkey` or `rabbitmq`; must match the existing service's type. |

The used service must be **Active** and must not be scheduled for deletion. The same name can't be under both `addons` and `uses.addons`.

---

## domains

| Field | Default | Meaning |
|---|---|---|
| `hostname` | — | Required. A valid domain name. |
| `service` | — | Required. The name of a service under `services`. The service must belong to this Stack. |
| `tls` | `managed` | Only `managed`: Komuta manages the certificate. |

The domain is added once its service is ready. Complete the DNS verification as described in [Custom Domains](custom-domains.md).

---

## Validation Messages

**Create plan** checks the file first. Each issue is listed with its code, its path in the file and its message. Messages are shown in English, as Komuta returns them.

### File structure

| Code | Message |
|---|---|
| `MANIFEST_REQUIRED` | Manifest is required. |
| `MANIFEST_TOO_LARGE` | Manifest exceeds 1048576 bytes. |
| `INVALID_YAML` | Invalid YAML or field value at line …, column …. Check the field type and manifest schema. |
| `MULTI_DOCUMENT_NOT_ALLOWED` | Exactly one YAML document is allowed. |
| `ROOT_OBJECT_REQUIRED` | The manifest root must be an object. |
| `MAX_DEPTH_EXCEEDED` | Manifest nesting cannot exceed 32. |
| `YAML_ANCHOR_NOT_ALLOWED` | YAML anchors and aliases are not allowed. |
| `YAML_CUSTOM_TAG_NOT_ALLOWED` | Custom YAML tags are not allowed. |
| `DUPLICATE_KEY` | Duplicate key '…' is not allowed. |
| `UNKNOWN_OR_INVALID_FIELD` | Unknown field '…'. Expected fields: …. |
| `OBJECT_REQUIRED` | A manifest object is required here. |
| `FIELD_PLATFORM_MANAGED` | gatewayClass is managed by Komuta and must not be declared. |
| `PLATFORM_ID_NOT_ALLOWED` | Platform identifiers must not be stored in the manifest; use resource names. |
| `SCHEMA_VERSION_UNSUPPORTED` | Only manifest version 1 is supported. |
| `RESOURCE_LIMIT_EXCEEDED` | A manifest can contain at most 200 resources. |
| `NAME_REQUIRED` | A non-empty name is required. |
| `NAME_TOO_LONG` | A name cannot exceed 160 characters. |
| `DUPLICATE_RESOURCE_NAME` | Resource name '…' is duplicated. |

### Fields

| Code | Message |
|---|---|
| `PRUNE_NOT_SUPPORTED` | prune must be false in version 1. |
| `ADOPTION_MUST_BE_EXPLICIT` | adoptExisting must be explicit. |
| `MAX_PARALLEL_INVALID` | maxParallel must be between 1 and 10. |
| `SECRET_SOURCE_INVALID` | Secret source must be prompt, existing or generate. |
| `EXISTING_SECRET_NAME_REQUIRED` | existingName is required for existing secrets. |
| `GENERATED_SECRET_LENGTH_INVALID` | Generated secret length must be between 24 and 256 when specified. |
| `ADDON_TYPE_INVALID` | Addon type must be postgres, valkey, rabbitmq or opensearch. |
| `ADDON_PLAN_REQUIRED` | Addon plan is required. |
| `ADDON_NETWORK_ACCESS_INVALID` | Addon network access must be private-only, public or public-allowlist. |
| `ADDON_NETWORK_PRIVATE_UNSUPPORTED` | RabbitMQ does not support private-only access yet; use public. |
| `ADDON_NETWORK_ALLOWLIST_UNSUPPORTED` | Initial public-allowlist enforcement is not supported by Stack creates; create with public access and restrict it through the addon API. |
| `ADDON_NETWORK_ALLOWLIST_UNEXPECTED` | allowedCidrs can only be used with public-allowlist access. |
| `ADDON_EVICTION_POLICY_INVALID` | Valkey evictionPolicy must be one of: noeviction, allkeys-lru, … |
| `ADDON_PERSISTENCE_MODE_INVALID` | Valkey persistenceMode must be one of: rdb, aof, none. |
| `ADDON_PERSISTENCE_REQUIRES_STANDALONE` | Valkey persistenceMode 'none' requires an explicit `topology: standalone`; replication requires persistence. |
| `USE_TYPE_INVALID` | A used addon's type must be postgres, valkey or rabbitmq. |
| `USE_ADOPT_CONFLICT` | Addon '…' is declared under both addons and uses.addons; a Stack either manages an addon or uses it. |
| `WORKLOAD_INVALID` | Workload must be service, job or cronjob. |
| `REPOSITORY_REQUIRED` | Repository is required. |
| `BRANCH_REQUIRED` | Branch is required. |
| `COMPUTE_PLAN_REQUIRED` | Compute plan is required. |
| `ROLLOUT_STRATEGY_INVALID` | Rollout strategy must be canary, blue-green or auto-promote. |
| `BANDWIDTH_PLAN_REQUIRED` | Bandwidth plan is required. |
| `STORAGE_WORKLOAD_UNSUPPORTED` | Persistent storage is supported only for service workloads. |
| `HEALTH_WORKLOAD_UNSUPPORTED` | Health probes are supported only for service workloads. |
| `ROUTING_WORKLOAD_UNSUPPORTED` | Routing is supported only for service workloads. |
| `FRAMEWORK_KEYS_WORKLOAD_UNSUPPORTED` | Framework key persistence is supported only for service workloads. |
| `STORAGE_NAME_INVALID` | Storage name must be a DNS-1123 label of at most 63 characters. |
| `STORAGE_PLAN_REQUIRED` | Storage plan is required. |
| `STORAGE_MOUNT_PATH_INVALID` | Storage mountPath must be an absolute, non-root path without parent traversal. |
| `STORAGE_SUB_PATH_INVALID` | Storage subPath must be relative and cannot contain parent traversal. |
| `PROBE_TYPE_INVALID` | Probe type must be httpGet, tcpSocket or exec. |
| `PROBE_TIMING_INVALID` | Probe initialDelaySeconds cannot be negative and all thresholds/periods must be at least 1. |
| `PROBE_SUCCESS_THRESHOLD_INVALID` | Liveness and startup probes require successThreshold: 1. |
| `HTTP_PROBE_TARGET_REQUIRED` | An enabled httpGet probe requires a path and a port between 1 and 65535. |
| `HTTP_PROBE_SCHEME_INVALID` | HTTP probe scheme must be HTTP or HTTPS. |
| `TCP_PROBE_PORT_REQUIRED` | An enabled tcpSocket probe requires a port between 1 and 65535. |
| `EXEC_PROBE_COMMAND_REQUIRED` | An enabled exec probe requires at least one command item. |
| `POLICY_GUARD_MODE_INVALID` | policyGuardMode must be off, shadow, audit or enforce. |
| `SECURITY_CAPABILITY_NOT_ALLOWED` | Capability '…' is not in the platform allowlist. |
| `SECURITY_WRITABLE_MOUNT_NOT_ALLOWED` | Writable mount '…' is invalid or targets a protected path. |
| `ROUTE_HOST_REQUIRED` | Route host is required. |
| `ROUTE_PATH_INVALID` | Route path must start with '/'. |
| `ROUTE_PATH_TYPE_INVALID` | Route pathType must be Prefix or Exact. |
| `ENV_NAME_REQUIRED` | Environment variable name is required. |
| `SECRET_VALUE_NOT_ALLOWED` | Sensitive values cannot be stored in the manifest. |
| `ENV_VALUE_CONFLICT` | Use either value or valueFrom, not both. |
| `BUILD_TIME_SECRET_NOT_ALLOWED` | A sensitive value cannot be made available at build time; keep it runtime-only. … |
| `VALUE_FROM_INVALID` | valueFrom must reference exactly one secret, addon or service. |
| `SECRET_REFERENCE_NOT_FOUND` | Secret '…' does not exist. |
| `SECRET_REFERENCE_CASE_MISMATCH` | Secret '…' does not exist; secret names are case-sensitive and the declared name is '…'. |
| `ADDON_REFERENCE_NOT_FOUND` | Addon '…' is not declared under addons or uses.addons. |
| `ADDON_REFERENCE_PROPERTY_INVALID` | Addon property must be host, port, database, user, password or connectionString. |
| `ADDON_REFERENCE_ENDPOINT_INVALID` | Addon endpoint must be private or public. |
| `ADDON_PUBLIC_ENDPOINT_NOT_DECLARED` | Addon '…' does not declare public network access. |
| `ADDON_PASSWORD_MUST_BE_SENSITIVE` | An addon password or connectionString binding must declare sensitive: true. |
| `SERVICE_REFERENCE_NOT_FOUND` | Service '…' does not exist. |
| `SERVICE_REFERENCE_PROPERTY_INVALID` | Service property must be host or url. |
| `SERVICE_DEPENDENCY_CYCLE` | Service dependencies must not contain a cycle. |
| `DOMAIN_SERVICE_NOT_FOUND` | Service '…' does not exist. |
| `DOMAIN_TLS_INVALID` | Only managed TLS is supported. |

The messages of the `access` block are in [Access Protection in a Stack Manifest](stack-manifest-access.md#validation).

### During planning

If the file is valid, the plan compares it with the resources in your account and may also show these issues:

| Code | Message |
|---|---|
| `PROJECT_NOT_FOUND` | Project '…' does not exist and apply.createMissingProject is false. |
| `GIT_CONNECTION_REQUIRED` | A Git connection is required to verify source-built services. |
| `GIT_CONNECTION_NOT_FOUND` | Git connection '…' was not found. |
| `SOURCE_CHANGE_REQUIRED` | Dockerfile '…' does not exist in pushed branch '…'. Create, test, commit and push it before planning. |
| `SOURCE_REVISION_VERIFICATION_FAILED` | The remote source revision could not be verified. |
| `STORAGE_PLAN_NOT_FOUND` | Storage plan '…' is unavailable. |
| `BANDWIDTH_PLAN_NOT_FOUND` | Bandwidth plan '…' is unavailable. |
| `PRICE_UNAVAILABLE` | A required resource price is unavailable. Refresh pricing and create a new plan; unknown cost is not free. |
| `PERMISSION_REQUIRED` | Permission '…' is required for this action. |
| `Komuta:Sleep:FreePlanSingleReplica` | The free compute plan runs a single replica without autoscaling; choose a paid plan to scale out. |
| `ADOPTION_CONFIRMATION_REQUIRED` | Existing resource '…' must be explicitly adopted. |
| `ADOPTION_NAME_CASE_MISMATCH` | The existing … is named '…', which differs from '…' only by letter case. … |
| `ADOPTION_TYPE_MISMATCH` / `ADOPTION_PLAN_MISMATCH` / `ADOPTION_NETWORK_ACCESS_MISMATCH` | The type, plan or network access of the managed service to adopt doesn't match the file; the message states the current value. |
| `ADOPTION_SHARED_ADDON_IN_USE` | Shared addon '…' is used by … service(s) outside the target project; … Use it with uses.addons instead. |
| `OWNERSHIP_CONFLICT` | The … '…' is already managed by Stack '…'. A resource has exactly one owning Stack, so this Stack cannot adopt it. |
| `USE_ADDON_NOT_FOUND` | No addon named '…' exists in the target project or among the shared addons. |
| `USE_OUTSIDE_TARGET` | Addon '…' belongs to project '…', not the target project; … |
| `USE_ADDON_NOT_ACTIVE` | Addon '…' is …; it must be Active before a service can use it. |
| `DOMAIN_HOSTNAME_INVALID` | Hostname '…' is not a valid domain name. |
| `CAPABILITY_NOT_AVAILABLE` | See [Fields not available yet](#fields-not-available-yet). When an unsupported field of an existing resource changes: In-place updates for Stack-managed … resources are not enabled on this target. |

If the target project is missing, "A target project is required." appears; if the given region can't be found, "The target region was not found." appears. In both cases no plan is created.

### Warnings

Warnings don't block the plan.

| Code | Message |
|---|---|
| `PLACEHOLDER_VALUE` | This value still looks like a template placeholder and must be reviewed before production use. |
| `BUILD_TIME_PREFIX_NOT_STAGED` | This name is usually read by the frontend build; a new service needs buildTime: true or the built app will not see it. … |
| `VALUE_MATERIALIZED_AT_APPLY` | Addon property '…' will be resolved securely during apply after '…' is Active. (For service references: Service property '…' will be resolved during apply from the custom domain or assigned platform address.) |

---

## Examples

### A complete example

An API, a web frontend, PostgreSQL and Valkey, with a domain. Replace the connection names with your own connections from the **Integrations** page.

```yaml copy
version: 1
name: shop

target:
  project: shop
  region: de-east
  connections:
    git: my-github
    registry: my-registry

secrets:
  - name: payment-api-key
  - name: session-secret
    source: generate
    length: 64

addons:
  - name: shop-db
    type: postgres
    plan: postgres-2g
    database: shop
  - name: shop-cache
    type: valkey
    plan: valkey-1g
    config:
      evictionPolicy: allkeys-lru

services:
  - name: api
    source:
      repository: https://github.com/acme/shop.git
      branch: main
      root: api
    compute:
      plan: dz-dev
      replicas:
        mode: fixed
        count: 2
    ports:
      - name: http
        containerPort: 8080
        primary: true
    network:
      public: true
    health:
      readiness:
        type: httpGet
        path: /healthz
        port: 8080
      liveness:
        type: httpGet
        path: /healthz
        port: 8080
        initialDelaySeconds: 10
    env:
      - name: LOG_LEVEL
        value: info
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: shop-db, property: connectionString }
      - name: REDIS_HOST
        valueFrom:
          addon: { name: shop-cache, property: host }
      - name: REDIS_PASSWORD
        sensitive: true
        valueFrom:
          addon: { name: shop-cache, property: password }
      - name: PAYMENT_API_KEY
        valueFrom: { secret: payment-api-key }
      - name: SESSION_SECRET
        valueFrom: { secret: session-secret }

  - name: web
    source:
      repository: https://github.com/acme/shop.git
      branch: main
      root: web
    compute:
      plan: dz-dev
    ports:
      - containerPort: 3000
    network:
      public: true
    env:
      - name: NEXT_PUBLIC_API_URL
        buildTime: true
        valueFrom:
          service: { name: api, property: url }

domains:
  - hostname: shop.example.com
    service: web
```

The plan for this file creates two managed services, two services and a domain; if the `shop` project doesn't exist, it creates that first. For `api` and `web`, `api/Dockerfile` and `web/Dockerfile` must be in the pushed `main` branch. The `payment-api-key` value is asked for through the secure link; `session-secret` is generated during the run. `web` receives the address of `api` at build time, so it is created after `api`.

### The smallest Stack

A single service on the free package:

```yaml copy
version: 1
name: hello

target:
  project: hello
  region: de-east
  connections:
    git: my-github
    registry: my-registry

services:
  - name: hello
    source:
      repository: https://github.com/acme/hello.git
      branch: main
    compute:
      plan: dz-nano
    ports:
      - containerPort: 8080
    network:
      public: true
```

### Using an existing database

A service that uses `main-db`, a PostgreSQL already running in the project, without managing it. The plan shows the **Use** action for `main-db`. The service connects to the managed service's private address (`endpoint: private`); if the managed service has no private address, the step stops with `ADDON_ENDPOINT_NOT_AVAILABLE`, in which case write `endpoint: public`.

```yaml copy
version: 1
name: reports

target:
  project: shop
  region: de-east
  connections:
    git: my-github
    registry: my-registry

uses:
  addons:
    - name: main-db
      type: postgres

services:
  - name: reports
    source:
      repository: https://github.com/acme/reports.git
      branch: main
    compute:
      plan: dz-dev
    env:
      - name: DATABASE_URL
        sensitive: true
        valueFrom:
          addon: { name: main-db, property: connectionString }
```

---

## Related Documents

- [Stacks](stacks.md) — loading, planning, applying and following a file.
- [Access Protection in a Stack Manifest](stack-manifest-access.md) — the `access` block.
- [Environment Variables](services-environment-variables.md) — secret and build-time variables.
- [PostgreSQL](managed-postgresql.md), [Valkey](managed-valkey.md), [RabbitMQ](managed-rabbitmq.md) — managed service plans.
- [Pricing](https://www.komuta.io/pricing) — package and plan prices.
