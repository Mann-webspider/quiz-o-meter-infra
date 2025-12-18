# ArgoCD GitOps Configuration

## Overview
ArgoCD monitors Git repository and automatically deploys changes to Kubernetes.

## Architecture

Git Push → ArgoCD Detects Change → Sync to K8s → Application Deployed


## Environments

### Development (quiz-o-meter-dev)
- **Git Branch:** `dev`
- **Namespace:** `quiz-dev`
- **Sync:** Automatic
- **Prune:** Enabled (removes deleted resources)
- **Self-Heal:** Enabled (reverts manual changes)

### Production (quiz-o-meter-prod)
- **Git Branch:** `main`
- **Namespace:** `quiz-prod`
- **Sync:** Manual (requires approval)
- **Prune:** Disabled (safer for production)
- **Self-Heal:** Enabled

## Workflow

### Dev Deployment
1. Developer pushes code to `dev` branch
2. Jenkins builds Docker images
3. Jenkins updates `k8s/overlays/dev/kustomization.yaml` with new image tags
4. Jenkins commits changes to Git
5. **ArgoCD auto-syncs** within 3 minutes
6. Application deployed to `quiz-dev` namespace

### Prod Deployment
1. Merge `dev` → `main` branch
2. Jenkins builds production images
3. Jenkins updates `k8s/overlays/prod/kustomization.yaml`
4. Jenkins commits to `main` branch
5. **ArgoCD shows "Out of Sync"** (manual approval needed)
6. DevOps team reviews in ArgoCD UI
7. Click "Sync" button to deploy
8. Application deployed to `quiz-prod` namespace

## ArgoCD UI Access
- URL: `https://argocd.your-domain.com` (via Cloudflare Tunnel)
- Username: `admin`
- Password: Check `~/argocd-setup/argocd-password.txt` on server

## Common Commands

Login to ArgoCD CLI
```bash
argocd login argocd.your-domain.com
```
List applications
```bash
argocd app list
```
Get app details
```bash
argocd app get quiz-o-meter-dev
```
Sync dev manually (if needed)
```bash
argocd app sync quiz-o-meter-dev
```

Sync prod (manual approval)
```bash
argocd app sync quiz-o-meter-prod
```

View sync status
```bash
argocd app wait quiz-o-meter-dev
```

Rollback to previous version
```bash
argocd app rollback quiz-o-meter-prod
```

## Troubleshooting

### App shows "Out of Sync"
- Normal for prod (manual sync required)
- For dev, check if auto-sync is enabled
- Click "Sync" in UI or run `argocd app sync APP_NAME`

### Sync Failed
- Check application events in UI
- Review pod logs: `kubectl logs -n quiz-dev -l app=backend`
- Check resource limits/quotas

### Self-Heal Not Working
- Verify `selfHeal: true` in Application spec
- Check ArgoCD has proper RBAC permissions
- Review ArgoCD logs: `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller`



View application logs
argocd app logs quiz-o-meter-dev