# GitOps Repository - n8n Deployment

This repository contains Kubernetes manifests for deploying n8n workflow automation tool using GitOps practices.

## Structure

```
├── apps/
│   └── n8n/
│       ├── base/              # Base n8n configuration
│       └── overlays/
│           └── production/    # Production-specific configuration
├── clusters/
│   └── default/
│       └── kustomization.yaml # Root kustomization for the cluster
└── infrastructure/
    └── namespaces/            # Namespace definitions
```

## Deployment

Point your GitOps tool (ArgoCD, Flux, etc.) to:
- **Path**: `clusters/default`
- **Branch**: `main`

## Components

- **n8n**: Workflow automation platform deployed without persistent storage (ephemeral setup)
- **Namespace**: Dedicated `n8n` namespace for isolation

## Next Steps

- Add Crossplane for infrastructure provisioning
- Configure persistent storage (database)
- Add monitoring and observability
