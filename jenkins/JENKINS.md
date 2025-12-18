# Jenkins CI/CD Pipeline

## Overview
Jenkins builds Docker images and updates Kubernetes manifests for GitOps deployment.

## Pipelines

### 1. Development Pipeline (Jenkinsfile.dev)
- **Trigger:** Push to `dev` branch
- **Actions:**
  1. Clone app repository
  2. Build Docker images (frontend + backend)
  3. Push to registry with `dev-${BUILD_NUMBER}` tag
  4. Update `k8s/overlays/dev/kustomization.yaml`
  5. Commit changes to Git
  6. ArgoCD auto-syncs within 3 minutes

### 2. Production Pipeline (Jenkinsfile.prod)
- **Trigger:** Manual (with parameters)
- **Parameters:**
  - VERSION: Semantic version (e.g., v1.0.0)
  - CONFIRM_DEPLOY: Checkbox confirmation
- **Actions:**
  1. Build production images
  2. Tag with version number
  3. Push to registry
  4. Update `k8s/overlays/prod/kustomization.yaml`
  5. Create Git tag
  6. **WAIT** for manual ArgoCD sync

## Initial Setup

### 1. Access Jenkins

Get Jenkins URL
echo "http://your-server-ip:32000"

Login
Username: admin
Password: admin123 # Change immediately!

### 2. Install Required Plugins
Already pre-installed via Helm:
- Kubernetes Plugin
- Docker Pipeline
- Git Plugin
- Blue Ocean
- Pipeline Stage View

### 3. Configure Credentials

#### a. Docker Registry
1. Go to: Manage Jenkins → Credentials → Global
2. Add Credentials:
   - Kind: Username with password
   - ID: `docker-username`
   - Username: your-docker-username
   - Password: your-docker-token

3. Add another:
   - Kind: Secret text
   - ID: `docker-password`
   - Secret: your-docker-token

#### b. GitHub Credentials
1. Add Credentials:
   - Kind: Username with password
   - ID: `github-credentials`
   - Username: your-github-username
   - Password: your-github-token (PAT)

### 4. Create Pipeline Jobs

#### Dev Pipeline
1. New Item → Pipeline
2. Name: `quiz-dev-pipeline`
3. Pipeline:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: `https://github.com/YOUR_USERNAME/quiz-o-meter-infra.git`
   - Branch: `dev`
   - Script Path: `jenkins/Jenkinsfile.dev`
4. Build Triggers:
   - Poll SCM: `H/5 * * * *` (every 5 minutes)

#### Prod Pipeline
1. New Item → Pipeline
2. Name: `quiz-prod-pipeline`
3. Pipeline:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository: same as above
   - Branch: `main`
   - Script Path: `jenkins/Jenkinsfile.prod`
4. This project is parameterized: ✅
   - Add String Parameter: VERSION
   - Add Boolean Parameter: CONFIRM_DEPLOY

## Workflow

### Development Workflow

Developer pushes to app repo (dev branch)
↓

Jenkins auto-detects change (polls every 5 min)
↓

Builds Docker images
↓

Pushes to registry
↓

Updates infra repo (k8s manifests)
↓

ArgoCD detects change in Git
↓

ArgoCD auto-syncs to quiz-dev namespace
↓

Application deployed! ✅

### Production Workflow

Merge dev → main in app repo
↓

Manually trigger Jenkins prod pipeline
↓

Enter version (e.g., v1.0.0)
↓

Confirm deployment checkbox
↓

Jenkins builds and pushes images
↓

Updates infra repo main branch
↓

ArgoCD shows "Out of Sync"
↓

MANUAL: Review in ArgoCD UI
↓

Click "Sync" in ArgoCD
↓

Production deployed! ✅

## Troubleshooting

### Build Fails - Docker Permission Denied

On K3s server, add jenkins to docker group
kubectl exec -it jenkins-0 -n jenkins -- chmod 666 /var/run/docker.sock

### Git Push Fails
- Check GitHub credentials are correct
- Ensure GitHub token has repo write permissions

### Images Not Pushed
- Verify Docker registry credentials
- Check network connectivity from Jenkins pod

## Monitoring Builds

### Via UI
- Dashboard → Build History
- Blue Ocean UI: http://your-server:32000/blue

### Via CLI

Get build logs
kubectl logs -n jenkins jenkins-0 -f

Check pod status
kubectl get pods -n jenkins
