# Quiz-O-Meter Infrastructure

GitOps repository for Kubernetes manifests and DevOps configuration.

## Structure
- `k8s/` - Kubernetes manifests (Kustomize)
- `jenkins/` - Jenkins CI/CD pipelines
- `monitoring/` - Prometheus, Grafana, Loki configs
- `scripts/` - Setup and utility scripts

## Environments
- **dev** - Development environment (auto-deploy)
- **prod** - Production environment (manual approval)
