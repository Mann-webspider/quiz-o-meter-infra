# Environment Configuration

## Development (quiz-dev)
- **Namespace:** `quiz-dev`
- **Domain:** `quiz-dev.your-domain.com`
- **Replicas:** 1 per service
- **Resources:** Minimal (100m CPU, 256Mi RAM)
- **Auto-scaling:** Disabled
- **SSL:** Let's Encrypt Staging
- **Log Level:** Debug
- **Auto-deploy:** Yes (via ArgoCD)

## Production (quiz-prod)
- **Namespace:** `quiz-prod`
- **Domain:** `quiz.your-domain.com`
- **Replicas:** 3 per service (min), 10 (max)
- **Resources:** Higher (200m-1000m CPU, 512Mi-1Gi RAM)
- **Auto-scaling:** Enabled (HPA)
- **SSL:** Let's Encrypt Production
- **Log Level:** Info
- **Auto-deploy:** Manual approval required

## Key Differences

| Feature | Dev | Prod |
|---------|-----|------|
| Replicas | 1 | 3-10 |
| CPU Request | 100m | 200m |
| Memory Request | 256Mi | 512Mi |
| Auto-scaling | No | Yes |
| Zero-downtime | No | Yes |
| SSL Cert | Staging | Production |
| Deployment | Auto | Manual |
