# Bank GitOps

GitOps deployment companion for the `bank-website` application.

The app source, Dockerfile, CI, and Docker image publishing live in:

https://github.com/AstralUniverse1/bank-website

This repo owns the deployment state:

- Helm chart for the bank app and MySQL.
- ArgoCD Application manifest.
- Manual promotion workflow that updates the deployed image tag.

## Promotion Flow

1. `bank-website` CI builds and pushes `astraluniverse/bank-app`.
2. Choose the image tag to deploy, usually the short or full commit SHA.
3. Run the `Promote` workflow in this repo with `image_tag`.
4. The workflow updates `helm/bank-app/values.yaml`, bumps the chart patch version, and commits to `main`.
5. ArgoCD detects the Git change and reconciles the cluster.

## ArgoCD

Apply the ArgoCD Application manifest:

```bash
kubectl apply -f argocd/bank-app.yaml
```

The Application watches this repo at `helm/bank-app` and deploys to the `bank` namespace with automated sync enabled.

## Helm

Render locally:

```bash
helm template bank-app helm/bank-app
```

Validate locally:

```bash
helm lint helm/bank-app
```

The chart includes demo MySQL credentials in `values.yaml` so it can render and run as a portfolio example. Override them with environment-specific values before using this pattern outside a demo.
