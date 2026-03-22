# kelm — Kubernetes Deployment Platform

GitOps platform powered by **ArgoCD**, **Helm**, and **Kustomize**.

- **Infra apps** (prometheus, etc.) are deployed directly via Helm + Kustomize.
- **Developer apps** plug in via a reusable GitHub Actions workflow. CI builds the Docker image, pushes to DockerHub, updates this repo, and ArgoCD auto-deploys.

## How it works

```
Developer's Repo                         kelm (this repo)
┌────────────────────┐                   ┌───────────────────────────────────┐
│ App code           │  git push main    │                                   │
│ Dockerfile         │ ─────────────►    │ .github/workflows/reusable-deploy │
│ .github/workflows/ │                   │   ├─ semantic-release  (version)  │
│   deploy.yaml ─────┤ calls reusable ──►│   ├─ docker build+push (DockerHub)│
│ .releaserc.json    │   workflow        │   └─ update apps/<name>/values    │
└────────────────────┘                   │                                   │
                                         │ charts/generic-app/   ← reusable │
                                         │ apps/<name>/values.yaml  ← config│
                                         │ argocd/ ← App-of-Apps            │
                                         └────────────┬────────────────────┘
                                                       │ git change detected
                                                       ▼
                                                  ┌──────────┐
                                                  │  ArgoCD  │
                                                  │  (syncs) │
                                                  └──────────┘
```

## Repository Structure

```
kelm/
├── charts/
│   └── generic-app/              # Reusable Helm chart (Deployment, Service, etc.)
├── apps/
│   ├── kube-prometheus-stack/    # Infra: monitoring (single deployment)
│   └── <app-name>/              # Created by CI for each developer app
├── argocd/
│   ├── apps/                    # Root App-of-Apps (Helm chart)
│   ├── applicationsets/         # Auto-discovery ApplicationSets
│   └── projects/                # RBAC boundaries
├── clusters/                    # Cluster-specific linkages
├── examples/
│   └── developer-app/           # Copy this into your app repo
└── .github/
    └── workflows/
        └── reusable-deploy.yaml # Called by developer repos
```

## For Developers — Deploying Your App

### 1. Add 3 files to your repo

```
your-repo/
├── Dockerfile
├── .releaserc.json              ← copy from examples/developer-app/
└── .github/workflows/
    └── deploy.yaml              ← copy from examples/developer-app/
```

### 2. Set GitHub Secrets

| Secret | Value |
|--------|-------|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |
| `KELM_DEPLOY_KEY` | SSH key with write access to kelm |

### 3. Push to main

```bash
git add . && git commit -m "feat: initial release" && git push
```

CI handles everything: versioning → Docker build → ArgoCD deploy.

## For Platform Admins — Bootstrap

```bash
# 1. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 2. Register kelm repo in ArgoCD
argocd repo add git@github.com:pawan-97/kelm.git --ssh-private-key-path ~/.ssh/id_rsa

# 3. Apply projects and bootstrap
kubectl apply -f argocd/projects/
kubectl apply -f argocd/apps/root-app.yaml
```

## Applications

| App | Type | Namespace |
|-----|------|-----------|
| kube-prometheus-stack | Infra (Helm) | monitoring |
| _your-app_ | Developer | _auto-created_ |
