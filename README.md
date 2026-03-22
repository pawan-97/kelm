# ArgoCD GitOps Repository — kelm

Manages Kubernetes applications using **ArgoCD**, **Helm**, and **Kustomize**
following the **App-of-Apps** pattern.

## Repository Structure

```
kelm/
├── argocd/
│   ├── projects/                        # AppProject RBAC boundaries
│   ├── apps/                            # Root App-of-Apps Helm chart
│   │   ├── root-app.yaml                # ★ Bootstrap once (kubectl apply)
│   │   ├── Chart.yaml
│   │   ├── values.yaml                  # Register applications here
│   │   └── templates/
│   │       ├── applications.yaml        # Renders Application CRs
│   │       └── applicationsets.yaml     # Renders ApplicationSet CRs
│   └── applicationsets/                 # Standalone ApplicationSets
│       ├── cluster-addons-appset.yaml   # Auto-discovers apps in clusters/prod/
│       └── multi-cluster-monitoring-appset.yaml
│
├── apps/                                # Application manifests
│   └── kube-prometheus-stack/           # Prometheus + Grafana + Alertmanager
│       ├── kustomization.yaml           # Single deployment, all namespaces
│       └── values.yaml                  # Helm values
│
└── clusters/
    └── prod/
        └── monitoring/                  # Picked up by cluster-addons-appset
```

## Bootstrap

```bash
# 1. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 2. Add this repo to ArgoCD (SSH key required — see repo setup guide)
argocd repo add git@github.com:pawan-97/kelm.git --ssh-private-key-path ~/.ssh/id_rsa

# 3. Apply the root App-of-Apps (one time only)
kubectl apply -f argocd/apps/root-app.yaml
# ArgoCD auto-creates all registered Applications from values.yaml
```

## Adding a New Application

1. Create `apps/<app-name>/kustomization.yaml` (+ any supporting files)
2. Add an entry to `argocd/apps/values.yaml` under `applications:`
3. Commit and push — ArgoCD syncs automatically

## Applications

| App | Namespace | Chart Version |
|-----|-----------|---------------|
| kube-prometheus-stack | monitoring | 68.3.2 |

## Grafana Access (port-forward)

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
# http://localhost:3000
```
