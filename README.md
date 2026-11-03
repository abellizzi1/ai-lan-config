# AI LAN application configuration

This directory is ready to become the root of its own Git repository. It contains everything Argo CD needs to render the AI workloads. It does not contain Python source, Terraform, or runtime credentials.

| Path | Purpose |
| --- | --- |
| `base/` | Gateway, PostgreSQL, frontend, ingress, namespace, and provider/model defaults. |
| `base/frontend/` | Plain HTML/CSS/JavaScript served by nginx through a generated ConfigMap. |
| `overlays/lan/` | Hosted inference deployment; **edit the gateway image tag here**. Initially `angelobellizzi/ai-gateway:0.0.1`. |
| `overlays/ollama/` | Same image tag plus Ollama, model storage, and local provider defaults. |

After manually building and pushing a new image, edit `images[0].newTag` in `overlays/lan/kustomization.yaml`, commit, and push. Argo CD's `ai-lan` Application watches this repo's `main` branch at `overlays/lan`, or `overlays/ollama` when selected in the cluster configuration repo. Both variants inherit the same tag. Build/push is an operator action, never part of manifest sync.

Render locally with:

```bash
kubectl kustomize overlays/lan
kubectl kustomize overlays/ollama
```

Edit the hosted `provider-api-base` and `default-model` literals in `base/kustomization.yaml`. The Ollama overlay overrides both; changing its model also requires updating `ollama.yaml`'s pull command and readiness probe. Generated ConfigMap name hashes cause the affected pods to roll on config or browser changes.

Before the first sync, create namespace `ai-lan` and the externally managed Secret `gateway-secrets`, and restore any PostgreSQL backup before the gateway starts. The infrastructure repository's `scripts/prepare-ai-lan.sh` performs that initial preparation. Required Secret keys are:

| Key | Purpose |
| --- | --- |
| `gateway-api-key` | Initial administrator password (the historical environment variable name). |
| `gateway-admin-username` | Initial administrator username. |
| `provider-api-key` | Hosted provider key; use `ollama` for local inference. |
| `postgres-password` | PostgreSQL role password; preserve it for existing databases. |

Secret updates need a gateway restart. Changing the PostgreSQL Secret does not rotate an already initialized database role's password. Never commit populated Secret manifests or Docker/Git credentials.

The gateway image is assumed public. For a private Docker Hub image, create an image pull Secret named `dockerhub` directly in `ai-lan`, then add a Kustomize patch to the chosen overlay setting the gateway Deployment's `spec.template.spec.imagePullSecrets` to `[{name: dockerhub}]`. Provision that Secret again after cluster recreation.

PostgreSQL syncs before the gateway, and its Deployment uses Recreate with one data volume. The gateway remains one replica because user locks are in memory. Namespace and PVC annotations protect them from Argo pruning/deletion, including Ollama storage when switching back to hosted inference. Removed protected resources need explicit cleanup and may leave the Application OutOfSync. Cluster deletion still destroys local storage; backups belong outside Git and outside cluster volumes.

AI ingress currently catches all hostnames. Give other applications explicit hostnames when sharing the same ingress controller and port. The chat site serves HTTP on a trusted LAN; TLS is not configured here.
