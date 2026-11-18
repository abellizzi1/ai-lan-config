# AI LAN configuration

Kubernetes configuration for [ai-lan](https://github.com/abellizzi1/ai-lan): web/API gateway, PostgreSQL, and optional Ollama server.

For cluster setup, start with [cluster-config](https://github.com/abellizzi1/cluster-config).

The UI is exposed at `http://localhost:8080/ai/` on the Docker host, or `http://<HOST_LAN_IP>:8080/ai/` from the LAN. Argo CD shares this port at `/argocd/`. Replace `8080` if you configured another shared HTTP port.

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

The subpath release uses `angelobellizzi/ai-gateway:1.0.3`; publish that image before syncing these manifests. Both overlays inherit this version.

## Path routing

The `ai-lan` Ingress matches `/ai` and its child paths. Traefik first redirects the bare `/ai` URL to `/ai/`, preserving any query string, then strips `/ai` before forwarding requests to `gateway:8000`. The gateway is configured with `ROOT_PATH=/ai`; browser assets, chat events, API requests, and admin navigation stay under that prefix. Its internal readiness probe remains `/healthz`.

The Traefik Middleware CRD and ingress controller are supplied by K3s. The `ai-lan` AppProject in `cluster-config` must allow `traefik.io/Middleware`; update that project before syncing the new application manifests.

## Change configuration

- **Hosted provider/model:** edit `provider-api-base` and `default-model` in `base/kustomization.yaml`.
- **Ollama:** select `overlays/ollama` in the Application defined in `cluster-config`. Model changes also require updating the pull command and readiness probe in `overlays/ollama/ollama.yaml`.
