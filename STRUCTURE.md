# GitOps Repository Structure

## Overview
This repository contains all Kubernetes application manifests managed by ArgoCD using the GitOps methodology.

## Directory Structure
```
├── apps/                           # Application definitions (ArgoCD Applications)
│   ├── ingress-nginx/             # NGINX Ingress Controller
│   │   ├── base/
│   │   │   ├── ingress-nginx-app.yaml    # ArgoCD Application
│   │   │   └── kustomization.yaml
│   │   └── overlays/production/
│   │       └── kustomization.yaml
│   ├── prometheus/                # Monitoring Stack (Prometheus + Grafana)
│   │   ├── base/
│   │   │   ├── prometheus-app.yaml       # ArgoCD Application  
│   │   │   └── kustomization.yaml
│   │   └── overlays/production/
│   │       └── kustomization.yaml
│   └── n8n/                       # User Application (Workflow Automation)
│       ├── base/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   ├── ingress.yaml
│       │   └── kustomization.yaml
│       └── overlays/production/
│           ├── deployment-patch.yaml
│           └── kustomization.yaml
├── infrastructure/                # Infrastructure resources
│   └── namespaces/               # Namespace definitions
│       ├── kustomization.yaml
│       ├── monitoring-namespace.yaml
│       └── n8n-namespace.yaml
├── clusters/                     # Cluster configurations
│   └── default/                  # Default cluster entrypoint
│       └── kustomization.yaml
├── README.md                     # Documentation
└── STRUCTURE.md                  # This file
```

## Application Types

### Infrastructure Applications (ArgoCD Applications)
- **ingress-nginx**: NGINX Ingress Controller with ALB integration
  - Fixed NodePorts: 30080 (HTTP), 30443 (HTTPS)
  - ALB-optimized configuration
- **prometheus**: Monitoring stack (Prometheus + Grafana + AlertManager)
  - Grafana ingress: https://grafana.sidmakes.com
  - Persistent storage for TSDB

### User Applications (Kubernetes Manifests)
- **n8n**: Workflow automation platform
  - Ingress: https://n8n.sidmakes.com
  - Production overlays for scaling

## Design Principles

### 1. Clean Separation
- **Terraform**: Infrastructure + ArgoCD bootstrap
- **GitOps**: Application deployment + management

### 2. Standard Patterns
- External Helm charts: Inline `values:` (ArgoCD best practice)
- Kubernetes manifests: Traditional Kustomize base/overlays
- Consistent labeling and metadata

### 3. Production Ready
- Automated sync with self-healing
- Environment-specific overlays
- Resource limits and health checks
- Proper ingress configuration for ALB

## Deployment Flow

1. **Bootstrap** (Terraform - one time)
   ```bash
   terragrunt apply  # argocd-platform
   terragrunt apply  # argocd-bootstrap
   ```

2. **GitOps** (Automatic)
   - ArgoCD discovers `clusters/default/kustomization.yaml`
   - Deploys all applications automatically
   - Self-heals and syncs changes

## URLs (Post-Deployment)
- **ArgoCD**: https://argo.sidmakes.com
- **Grafana**: https://grafana.sidmakes.com  
- **n8n**: https://n8n.sidmakes.com

## Adding New Applications

1. Create application structure in `apps/my-app/`
2. Add to `clusters/default/kustomization.yaml`  
3. Commit and push - ArgoCD deploys automatically
