# ============================================================
# Example: Developer App README
#
# This directory shows what a developer adds to THEIR OWN repo
# to integrate with the kelm platform.
# ============================================================

# Example App — Deploying on kelm

This shows the **minimum** a developer needs in their own repo
to get their app built, released, and deployed via ArgoCD.

## What you need in your repo

```
your-app-repo/
├── Dockerfile                           # Build your app
├── .releaserc.json                      # Semantic release config
├── .github/
│   └── workflows/
│       └── deploy.yaml                  # Calls kelm's reusable workflow
└── src/                                 # Your app code
```

## Setup Steps (one-time)

### 1. Add GitHub Secrets to your repo

Go to **your repo → Settings → Secrets and variables → Actions** and add:

| Secret | Value |
|--------|-------|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |
| `KELM_DEPLOY_KEY` | SSH private key with **write** access to `pawan-97/kelm` |

### 2. Copy these files

- `.github/workflows/deploy.yaml` → your `.github/workflows/deploy.yaml`
- `.releaserc.json` → your repo root

### 3. Commit and push

```bash
git add .
git commit -m "feat: add kelm CI/CD integration"
git push
```

### 4. That's it!

On every push to `main`:
1. **semantic-release** determines the next version from commit messages
2. **Docker image** is built and pushed to DockerHub
3. **kelm repo** is updated with the new image tag
4. **ArgoCD** detects the change and deploys automatically

## Commit Message Convention

semantic-release uses [Conventional Commits](https://www.conventionalcommits.org):

| Prefix | Version Bump | Example |
|--------|-------------|---------|
| `fix:` | Patch (1.0.1) | `fix: handle null response` |
| `feat:` | Minor (1.1.0) | `feat: add user search` |
| `feat!:` or `BREAKING CHANGE:` | Major (2.0.0) | `feat!: change API response format` |

## Customisation

Override any value from the generic chart by editing your `deploy.yaml`:

```yaml
with:
  container_port: 3000        # Your app's port
  namespace: my-team          # Custom namespace
```

Or create a `helm/values.yaml` in your repo with full overrides:

```yaml
# helm/values.yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: my-app.example.com
      paths:
        - path: /
          pathType: Prefix
```

And reference it:
```yaml
with:
  values_file: helm/values.yaml
```
