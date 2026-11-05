# AI LAN configuration

Kubernetes configuration for [ai-lan](https://github.com/abellizzi1/ai-lan): web/API gateway, PostgreSQL, and optional Ollama server.

For cluster setup, start with [cluster-config](https://github.com/abellizzi1/cluster-config).

## Layout

| Path | Purpose |
| --- | --- |
| `base/` | Shared workloads, ingress, storage, and provider defaults |
| `overlays/lan/` | Hosted inference configuration and gateway image tag |
| `overlays/ollama/` | Local inference with Ollama |

## Deploy an update

1. Build and push new image from [ai-lan](https://github.com/abellizzi1/ai-lan).
2. Set `newTag` value to the new image's tag in `overlays/lan/kustomization.yaml`.
3. Commit and push to the branch tracked by ArgoCD.

## Change configuration

- **Hosted provider/model:** edit `provider-api-base` and `default-model` in `base/kustomization.yaml`.
- **Ollama:** select `overlays/ollama` in the Application defined in `cluster-config`. Model changes also require updating the pull command and readiness probe in `overlays/ollama/ollama.yaml`.
