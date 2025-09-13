# GitOps Repository

This repository contains the GitOps configuration managed by ArgoCD for the Kubernetes cluster.

## Architecture

```
Terraform (Bootstrap) → ArgoCD → GitOps Applications
```

## Applications

### Infrastructure
- **ingress-nginx**: NGINX ingress controller with ALB integration
- **prometheus**: Monitoring stack (Prometheus + Grafana + AlertManager)

### User Applications  
- **n8n**: Workflow automation platform

## Bootstrap Process

1. **Terraform Bootstrap** (one-time):
   ```bash
   # Deploy ArgoCD platform + create root application
   cd live/space/us-east-2/argocd-platform && terragrunt apply
   cd live/space/us-east-2/argocd-bootstrap && terragrunt apply
   ```

2. **GitOps Takes Over**:
   - ArgoCD automatically deploys: ingress-nginx, prometheus, n8n
   - **ArgoCD stays managed by Terraform** (not self-managing)
   - Clean separation: Terraform = infrastructure, GitOps = applications

## Directory Structure

```
├── apps/                    # Application definitions
│   ├── ingress-nginx/      # Ingress controller
│   ├── prometheus/         # Monitoring stack
│   └── n8n/               # User applications
├── infrastructure/         # Infrastructure resources
│   └── namespaces/        # Namespace definitions
└── clusters/              # Cluster configurations
    └── default/           # Default cluster entrypoint
```

## Adding New Applications

1. Create app structure in `apps/my-app/`
2. Add to `clusters/default/kustomization.yaml`
3. Commit and push - ArgoCD will automatically deploy

## Monitoring

- **Grafana**: https://grafana.sidmakes.com
- **ArgoCD**: https://argo.sidmakes.com