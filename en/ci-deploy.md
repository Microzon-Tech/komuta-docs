# Deploying from CI

Your own CI pipeline (GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, Bitbucket…) can call Komuta as its last step, after its tests and approvals pass. Komuta builds the branch you send and releases it; your CI can wait for the result and pass or fail its own job accordingly.

This works with a **deploy token**. A deploy token is bound to a single service, can only do what you allow, and cannot be mixed up with your account's general API keys.

---

## Creating a Deploy Token

1. Open the service's **Auto-deploy** page and switch to the **CI/CD integration** tab.
2. Click **Create token** and set it up:
   - **Name:** where the token is used (for example `github-actions-main`).
   - **Allowed actions:** build and deploy, deploy a ready-made image, or both.
   - **Allowed branches and tags:** which branches and tags the token may deploy (for example `main`, `release/*`). Left empty, only the branch the service tracks.
   - **Allowed image repositories:** repositories accepted for image deploys. Left empty, only the service's own image repository.
   - **IP allowlist:** addresses (CIDR) requests may come from. Empty means any address.
   - **Lifetime:** 365 days by default, 730 days at most.
3. The token is shown **only once**. Copy it into your CI's secrets (for example a `KOMUTA_DEPLOY_TOKEN` secret on GitHub).

Tokens start with `kmtd_`. A service can have at most 10 active tokens at a time. Users who can edit the service can create and revoke tokens.

> **If push auto-deploy is on:** every push to the service branch already starts a build. If your CI also calls Komuta, each push is built twice. Once deploying from CI works, turn auto-deploy off on the **Push auto-deploy** tab.

---

## Deploying Without a Secret (OIDC)

If you use GitHub Actions or GitLab CI, you can deploy without storing any secret in your CI. The job presents a short-lived identity token (OIDC) signed by the CI provider; Komuta verifies the signature with the provider's public keys and accepts the request only when it comes from the repository you trust.

1. In **Create token**, pick **GitHub Actions OIDC** or **GitLab CI OIDC** under **How your CI signs in**.
2. Enter the repository: `owner/repo` for GitHub, `group/project` for GitLab. Optionally add an **environment** (for example `production`); then only jobs that use that environment can deploy.
3. Allowed actions, branches, image repositories and the IP allowlist apply as they do for secret tokens. If you set branch patterns, the branch the job runs on must match them as well.
4. Switch the snippets to **OIDC (no secret)**.

The GitHub Actions job needs `permissions: id-token: write`; the GitLab CI job declares a `KOMUTA_ID_TOKEN` under `id_tokens` with `aud: https://api.komuta.io`. In this mode there is no secret to store, leak or rotate.

---

## Ready-made Snippets

The **CI/CD integration** tab provides copy-ready examples filled in for your service, for GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, Bitbucket Pipelines and `curl`. Each one starts the deploy, waits for the result, and fails the CI job if the deploy fails. The **Build and deploy / Deploy a ready-made image** switch above the snippets turns them into image mode; there the script reads the image you pushed, pinned by digest, from the `IMAGE` variable.

The shortest GitHub Actions example. It only **starts** the deploy and does not wait for the result, so the CI job passes as soon as Komuta accepts the request:

```yaml copy
name: Deploy to Komuta

on:
  push:
    branches: ["main"]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy with Komuta
        env:
          KOMUTA_DEPLOY_TOKEN: ${{ secrets.KOMUTA_DEPLOY_TOKEN }}
        run: |
          curl -sSf -X POST \
            "https://api.komuta.io/api/v1/deploy-hooks/services/<SERVICE_ID>/deployments" \
            -H "Authorization: Bearer $KOMUTA_DEPLOY_TOKEN" \
            -H "Content-Type: application/json" \
            -d '{"mode":"build","ref":"${{ github.ref_name }}","clientRequestId":"${{ github.run_id }}-${{ github.run_attempt }}"}'
```

Copy the full version that waits for the result from the **CI/CD integration** tab.

---

## Deploy Request

```http
POST https://api.komuta.io/api/v1/deploy-hooks/services/{serviceId}/deployments
Authorization: Bearer kmtd_...
Content-Type: application/json
```

```json
{
  "mode": "build",
  "ref": "main",
  "commitSha": "9f1c2ab...",
  "clientRequestId": "gh-123456-1",
  "metadata": {
    "ciProvider": "github-actions",
    "runUrl": "https://github.com/org/repo/actions/runs/123456",
    "actor": "alice",
    "commitMessage": "Fix login redirect"
  }
}
```

| Field | Description |
|-------|-------------|
| `mode` | `build`: Komuta builds the branch and releases it. `image`: releases an image you built and pushed yourself. |
| `ref` | Branch or tag to build. Must match the token's branch patterns. |
| `commitSha` | Optional. When given, exactly that commit is built. If commit pinning is not enabled on the platform the request returns `400`; leave the field out and the branch's latest commit is built. |
| `image` | Required in `image` mode. Must be pinned by digest (`registry.example.com/app@sha256:...`). |
| `clientRequestId` | Idempotency key. Sent again with the same token and body, no new deploy is opened and the same deploy is returned. The same key with a different body returns `409`. |
| `metadata` | Optional. Shows in the console which CI run the deploy came from. |

When Komuta accepts the request it returns `202 Accepted`:

```json
{
  "deploymentId": "3a2f...",
  "status": "queued",
  "queue": { "position": 1, "reason": "PreviousBuildRunning", "estimatedStartSeconds": 240 },
  "statusUrl": "https://api.komuta.io/api/v1/deploy-hooks/deployments/3a2f...",
  "consoleUrl": "https://console.komuta.io/services/.../pipelines"
}
```

`queue` carries your position, the wait reason and the estimated start while the build is queued. See [Build Queue](build-queue.md).

---

## Waiting for the Result

Poll `statusUrl` with the same token:

```http
GET https://api.komuta.io/api/v1/deploy-hooks/deployments/{deploymentId}
Authorization: Bearer kmtd_...
```

| `status` | Meaning | What CI should do |
|----------|---------|-------------------|
| `queued` | The build is queued. | Keep waiting. |
| `building` | The build is running. | Keep waiting. |
| `deploying` | The new version is being released. | Keep waiting. |
| `succeeded` | The new version is live. | Pass the job. |
| `failed` | The build or the release failed. `failureReason` says why. | Fail the job. |
| `cancelled` | The deploy was cancelled. | Fail the job. |
| `superseded` | A newer build of the same service replaced this one (merged in the queue, or the running build was cancelled under "Latest wins"). `supersededBy` points to the replacing deploy when it came from CI. | Usually treated as passed. |

Deploy records are kept for 180 days; the `statusUrl` of an older deploy returns `404`.

To cancel a queued or running deploy:

```http
POST https://api.komuta.io/api/v1/deploy-hooks/deployments/{deploymentId}/cancel
Authorization: Bearer kmtd_...
```

---

## Requiring Signed Images

If you deploy ready-made images (`image` mode), you can require that only images **signed with your key** are deployed to the service. It is off by default.

1. Create a key pair: `cosign generate-key-pair`. Keep the private key (`cosign.key`) in your CI's secrets; **never** give it to Komuta.
2. In CI, after pushing the image, sign its digest: `cosign sign --key cosign.key registry.example.com/app@sha256:...`
3. On the service's **Auto-deploy → CI/CD integration** tab, paste the public key (`cosign.pub`, starting with `-----BEGIN PUBLIC KEY-----`) into the **Signed images** card and turn it on.

While it is on, Komuta verifies the signature of every `image` mode request. An unsigned image, or one signed with another key, is rejected with `403` and recorded in the audit log. If the signature cannot be verified right now (for example the registry is unreachable) the request returns `503`; try again shortly. ECDSA keys (the cosign default) and RSA keys of at least 2048 bits are accepted.

---

## Error Codes

| HTTP | When |
|------|------|
| `400` | The request is invalid (missing field, image by tag, `commitSha` while pinning is off). |
| `401` | The token is invalid, revoked or expired. |
| `402` | The account's billing is suspended. |
| `403` | The action, branch, image repository or IP address is outside the token's permissions, or signed images are required and the image is not signed with your key. |
| `404` | The service does not belong to this token, or the deploy was not found. |
| `409` | The same `clientRequestId` was sent with a different body, or the deploy can no longer be cancelled. |
| `429` | A rate limit was hit. Wait for the `Retry-After` header and try again. |
| `503` | The deploy could not be started right now; try again shortly. |

Rate limits: per token, 30 deploys and 300 status reads per minute; per service, 60 deploys per hour.

---

## Security

- A token only works on its own service and on the branches you allow. A leaked token cannot touch other services or the rest of your account.
- Komuta does not store the token secret; it is shown only when created. If you lose it, revoke the token and create a new one.
- The token list shows when, from which IP address and from which CI provider each token was last used. Creating and revoking tokens, every deploy and rejected requests are recorded in the audit log.
- A **Deploy token expiring soon** notification is sent 7 days and 1 day before a token expires.

---

## Related Documents

- [Auto-deploy](service-auto-deploy.md) — Rules for push auto-deploy.
- [Build Queue](build-queue.md) — When the deploy will start.
- [Pipelines](service-pipeline-guide.md) — Build stages and logs.
