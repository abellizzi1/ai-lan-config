# AI LAN configuration

Kubernetes configuration for the web/API gateway, PostgreSQL, and optional Ollama server.

For cluster setup, Argo CD access, secrets, and backups, start with [cluster-config](https://github.com/abellizzi1/cluster-config).

## Layout

| Path | Purpose |
| --- | --- |
| `base/` | Shared workloads, ingress, storage, and provider defaults |
| `overlays/lan/` | Hosted inference configuration and gateway image tag |
| `overlays/ollama/` | Local inference with Ollama |

## Deploy an update

1. Build and push the gateway image from the service source repository. It includes the HTML and JavaScript.
2. Set `images[0].newTag` in `overlays/lan/kustomization.yaml` to the published version.
3. Commit and push to the branch tracked by the Argo CD Application.

Both overlays inherit the version from `overlays/lan/kustomization.yaml`; this is the only image tag to update. The base Deployment contains only the image name. Argo CD deploys committed changes automatically.

For the frontend migration, publish `angelobellizzi/ai-gateway:0.0.2` before pushing these manifests. Version `0.0.1` does not include the frontend. Argo removes the old nginx Deployment, Service, and frontend ConfigMap during sync.

## Change configuration

- **Hosted provider/model:** edit `provider-api-base` and `default-model` in `base/kustomization.yaml`.
- **Frontend:** edit `services/gateway/app/static/` in the [service source repository](https://github.com/abellizzi1/model-api), then rebuild and publish the gateway image.
- **Ollama:** select `overlays/ollama` in the Application defined in `cluster-config`. Model changes also require updating the pull command and readiness probe in `overlays/ollama/ollama.yaml`.

Provider/model ConfigMap changes automatically roll the gateway pods. Manage runtime secrets through `cluster-config`; keep credentials and backups out of Git.

## Check manifests

```bash
kubectl kustomize overlays/lan
kubectl kustomize overlays/ollama
```
