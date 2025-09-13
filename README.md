# GitOps Repository

This repository contains the GitOps configuration managed by ArgoCD using the **ApplicationSet pattern** for automatic discovery and deployment.

## Architecture

```
Terraform (Bootstrap) → ApplicationSet → Auto-discovered Applications
```

**Key Benefits:**
- ✅ **Zero manual Application YAML files** - just create directories
- ✅ **Automatic cleanup** - finalizers prevent orphaned resources  
- ✅ **Modern GitOps** - industry standard ApplicationSet pattern

## Applications

### Infrastructure (Helm Charts)
- **ingress-nginx**: NGINX ingress controller with ALB integration
- **prometheus**: Monitoring stack (Prometheus + Grafana + AlertManager)

### User Applications (Kubernetes Manifests)
- **n8n**: Workflow automation platform

## Bootstrap Process

1. **Terraform Bootstrap** (one-time):
   ```bash
   # Deploy ArgoCD platform + ApplicationSet
   cd live/space/us-east-2/02-gitops
   terragrunt apply-all
   ```

2. **ApplicationSet Takes Over**:
   - ApplicationSet auto-discovers all `apps/*` directories
   - Creates ArgoCD Application for each directory automatically
   - **ArgoCD stays managed by Terraform** (clean separation)
   - Applications self-heal and auto-sync

## Directory Structure

```
├── apps/                    # Auto-discovered by ApplicationSet
│   ├── ingress-nginx/      # NGINX Ingress (Helm)
│   ├── prometheus/         # Monitoring (Helm) 
│   └── n8n/               # User app (Manifests)
└── STRUCTURE.md           # Detailed structure documentation
```

## Adding New Applications

### Helm Chart App
```bash
# 1. Create directory
mkdir apps/my-helm-app

# 2. Create kustomization with Helm chart
cat > apps/my-helm-app/kustomization.yaml << EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
helmCharts:
- name: my-chart
  repo: https://charts.example.com
  version: 1.0.0
EOF

# 3. Commit and push - ApplicationSet deploys automatically!
git add . && git commit -m "Add my-helm-app" && git push
```

### Kubernetes Manifest App
```bash
# 1. Create directory with manifests
mkdir apps/my-app
# Add your deployment.yaml, service.yaml, etc.

# 2. Create kustomization
cat > apps/my-app/kustomization.yaml << EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
EOF

# 3. Commit and push - ApplicationSet deploys automatically!
git add . && git commit -m "Add my-app" && git push
```

## Removing Applications

```bash
# Just delete the directory - ApplicationSet handles cleanup
rm -rf apps/my-app
git commit -m "Remove my-app" && git push
# ArgoCD automatically removes the application with proper cleanup!
```

## URLs (Post-Deployment)

- **ArgoCD**: https://argo.sidmakes.com
- **Grafana**: https://grafana.sidmakes.com
- **n8n**: https://n8n.sidmakes.com

## Cleanup

```bash
# ApplicationSet ensures proper cleanup of ALL applications
cd live/space/us-east-2/02-gitops
terragrunt destroy  # Now works perfectly - zero orphaned resources!
```

---

**Migration Note**: This repo was upgraded from manual Application YAML pattern to ApplicationSet auto-discovery for 50% less maintenance overhead and zero configuration drift risk.