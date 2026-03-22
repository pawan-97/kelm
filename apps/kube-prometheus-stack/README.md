# ============================================================
# kube-prometheus-stack App README
# ============================================================

# kube-prometheus-stack

Deploys the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
into the `monitoring` namespace via **Kustomize + Helm**.

## Components Deployed

| Component | Description |
|-----------|-------------|
| **Prometheus** | Metrics scraping and storage |
| **Alertmanager** | Alert routing and notifications |
| **Grafana** | Metrics visualization / dashboards |
| **Prometheus Operator** | Manages Prometheus/Alertmanager CRDs |
| **Node Exporter** | OS/node-level metrics |
| **Kube State Metrics** | Kubernetes object metrics |

## Directory Layout

```
kube-prometheus-stack/
├── base/
│   ├── kustomization.yaml   # Helm chart reference + common settings
│   └── values.yaml          # Default Helm values for all environments
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── values-dev.yaml  # Lightweight dev-specific overrides
    └── prod/
        ├── kustomization.yaml
        └── values-prod.yaml # Production sizing, HA, Ingress, Alerting
```

## ArgoCD Application

This app is registered in `argocd/apps/values.yaml` and points to
`apps/kube-prometheus-stack/overlays/prod` (or `dev` for a dev cluster).

## Accessing Grafana (dev)

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
# Open http://localhost:3000 — default creds admin/admin
```

## Adding Custom Dashboards

Create a ConfigMap with label `grafana_dashboard: "1"` in any namespace:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  my-dashboard.json: |
    { ... Grafana JSON dashboard ... }
```

## Customization Checklist

Before deploying to production, update:

- [ ] `overlays/prod/values-prod.yaml` → Storage class name (`gp3`, `ssd`, etc.)
- [ ] `overlays/prod/values-prod.yaml` → Grafana ingress hostname
- [ ] `overlays/prod/values-prod.yaml` → Alertmanager Slack/PagerDuty webhook URLs
- [ ] Create `grafana-admin-secret` Kubernetes Secret in the `monitoring` namespace
- [ ] `argocd/projects/monitoring-project.yaml` → Source repo URL
- [ ] `argocd/apps/root-app.yaml` → Source repo URL
- [ ] `argocd/apps/values.yaml` → Source repo URL
