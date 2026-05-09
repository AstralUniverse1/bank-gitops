# Bank GitOps

GitOps deployment repo for the demo banking app built in [`bank-website`](https://github.com/AstralUniverse1/bank-website).

This repo owns the Helm chart, ArgoCD Application manifest, validation workflow, and manual image promotion workflow.

## Repository Structure

| Path | Purpose |
| --- | --- |
| `.github/workflows/validate.yml` | Helm lint and render validation |
| `.github/workflows/promote.yml` | Manual workflow for promoting a Docker image tag |
| `argocd/bank-app.yaml` | ArgoCD Application manifest |
| `helm/bank-app/` | Helm chart for the app and MySQL |
| `helm/bank-app/values.yaml` | Image tag, Gunicorn runtime config, MySQL config, resource limits |
| `helm/bank-app/templates/` | Kubernetes manifests rendered by Helm |

## Deployment Model

| Area | Details |
| --- | --- |
| App image | `astraluniverse/bank-app`, published by `bank-website` CI |
| App workload | Kubernetes Deployment running the Flask/Gunicorn container on port `5000` |
| Service | NodePort service forwarding port `80` to the app container |
| Database | In-cluster MySQL by default; external MySQL can be used through Helm values |
| State | MySQL StatefulSet with PVC when in-cluster MySQL is enabled |
| ArgoCD | Watches `helm/bank-app` on `main` and deploys to the `bank` namespace |

## Promotion Workflow

The `Promote` workflow takes a manual `image_tag` input.

It updates the Helm chart image tag, bumps the chart version, updates `appVersion`, and commits the change to `main`.

ArgoCD then detects the Git change and syncs the updated desired state to the cluster.

## Requirements

- Kubernetes cluster
- ArgoCD installed in the `argocd` namespace
- Default StorageClass, or `mysql.storage.storageClassName` set in Helm values
- Access to the configured NodePort for external testing

## Helm Chart

The chart includes:

- app Deployment
- NodePort Service
- ConfigMap for non-sensitive runtime settings
- Kubernetes Secret template populated from Helm values
- readiness and liveness probes
- app resource requests and limits
- non-root app security context
- optional NetworkPolicy, disabled by default
- optional in-cluster MySQL StatefulSet with persistent storage

## Demo Credentials

`values.yaml` contains demo MySQL credentials so the chart can run as a self-contained portfolio deployment.

For any real environment, override the database credentials outside the repo and use a proper secret-management flow.
