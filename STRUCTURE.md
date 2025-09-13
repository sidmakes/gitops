# GitOps Repository Structure

## Overview
This repository contains all Kubernetes application manifests managed by ArgoCD using the **ApplicationSet pattern** for automatic discovery and deployment.

## Directory Structure
```
├── apps/                           # Auto-discovered by ApplicationSet
│   ├── ingress-nginx/             # NGINX Ingress Controller (Helm)
│   │   ├── kustomization.yaml     # Helm chart configuration
│   │   └── namespace.yaml         # Namespace resource
│   ├── prometheus/                # Monitoring Stack (Helm)
│   │   ├── kustomization.yaml     # Prometheus + Grafana config
│   │   └── namespace.yaml         # Namespace resource
│   └── n8n/                       # User Application (Manifests)
│       ├── kustomization.yaml     # Kustomization config
│       ├── namespace.yaml         # Namespace resource
│       ├── deployment.yaml        # Application deployment
│       ├── service.yaml          # Service definition
│       └── ingress.yaml          # Ingress configuration
├── README.md                     # Documentation
└── STRUCTURE.md                  # This file
```

## ApplicationSet Pattern

### How It Works
1. **Terraform deploys** an ApplicationSet that auto-discovers directories in `apps/`
2. **Each directory** in `apps/` becomes an ArgoCD Application automatically
3. **Zero manual** Application YAML files needed
4. **Proper cleanup** via finalizers when applications are removed

### ApplicationSet Configuration
```yaml
# Deployed by Terraform in argocd-bootstrap module
generators:
- git:
    directories:
    - path: apps/*        # Auto-discovers all app directories
```

## Application Types

### Infrastructure Applications (Helm Charts)
- **ingress-nginx**: NGINX Ingress Controller with ALB integration
  - Deployed via Helm chart in kustomization.yaml
  - Fixed NodePorts: 30080 (HTTP), 30443 (HTTPS)
  - ALB-optimized configuration

- **prometheus**: Monitoring stack (Prometheus + Grafana + AlertManager)
  - Deployed via kube-prometheus-stack Helm chart
  - Grafana ingress: https://grafana.sidmakes.com
  - Persistent storage for TSDB

### User Applications (Kubernetes Manifests)
- **n8n**: Workflow automation platform
  - Native Kubernetes manifests
  - Ingress: https://n8n.sidmakes.com  
  - Production-ready with resource limits

## Design Principles

### 1. ApplicationSet Auto-Discovery
- **Zero manual Application YAML**: Just create directory → auto-deployed
- **GitOps-native**: Industry standard for multi-app management
- **Proper cleanup**: Finalizers ensure cascade deletion

### 2. Clean Structure
- **One directory per app**: `apps/app-name/`
- **All resources included**: namespaces, manifests, configs
- **Production ready**: Resource limits, health checks, proper labels

### 3. Deployment Patterns
- **Helm charts**: Use kustomization.yaml with `helmCharts` field  
- **Kubernetes manifests**: Direct YAML files with kustomization
- **Consistent labeling**: `app.kubernetes.io/managed-by: gitops`

## Deployment Flow

### 1. Bootstrap (Terraform - one time)
```bash
cd live/space/us-east-2/02-gitops
terragrunt apply-all
```

### 2. GitOps (Automatic)
- ApplicationSet discovers all `apps/*` directories
- Creates ArgoCD Application for each directory
- Deploys applications with automated sync + self-heal
- Handles cleanup when directories are removed

## URLs (Post-Deployment)
- **ArgoCD**: https://argo.sidmakes.com
- **Grafana**: https://grafana.sidmakes.com  
- **n8n**: https://n8n.sidmakes.com

## Adding New Applications

### Helm Chart Application
```bash
# 1. Create app directory
mkdir apps/my-helm-app

# 2. Create kustomization with Helm chart
cat > apps/my-helm-app/kustomization.yaml << EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
helmCharts:
- name: my-chart
  repo: https://charts.example.com
  version: 1.0.0
  namespace: my-namespace
  valuesInline:
    key: value
EOF

# 3. Create namespace
cat > apps/my-helm-app/namespace.yaml << EOF  
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
EOF

# 4. Commit and push - ApplicationSet deploys automatically!
```

### Kubernetes Manifest Application
```bash
# 1. Create app directory with manifests
mkdir apps/my-app
# Add deployment.yaml, service.yaml, etc.

# 2. Create kustomization
cat > apps/my-app/kustomization.yaml << EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- namespace.yaml
- deployment.yaml
- service.yaml
EOF

# 3. Commit and push - ApplicationSet deploys automatically!
```

## Cleanup 

### Removing Applications
```bash
# Just delete the directory - ApplicationSet handles cleanup
rm -rf apps/my-app
git commit -m "Remove my-app"
git push  # ArgoCD automatically removes the application!
```

### Complete Teardown
```bash
# ApplicationSet ensures proper cleanup of all applications
cd live/space/us-east-2/02-gitops  
terragrunt destroy  # Now works perfectly with zero orphaned resources!
```

## Migration Notes

**Migrated from**: Manual Application YAML pattern → ApplicationSet auto-discovery
**Benefits**:
- ✅ **50% less maintenance**: No manual Application YAML files
- ✅ **Zero drift risk**: Auto-discovery prevents configuration mismatch  
- ✅ **Proper cleanup**: Finalizers ensure no orphaned resources
- ✅ **Modern GitOps**: Industry standard ApplicationSet pattern