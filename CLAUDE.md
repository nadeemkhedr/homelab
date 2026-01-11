# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kubernetes homelab running on Raspberry Pi using Flux CD for GitOps-based cluster management. All infrastructure is defined as code and automatically deployed via git pushes.

## Common Commands

```bash
# Force Flux to reconcile immediately (instead of waiting for auto-sync)
flux reconcile kustomization apps
flux reconcile kustomization monitoring

# Encrypt a Kubernetes secret with SOPS before committing
export AGE_PUBLIC={public_age_key}
sops --age=$AGE_PUBLIC --encrypt --encrypted-regex '^(data|stringData)$' --in-place secret.yaml
```

## Architecture

### Directory Structure

- `apps/` - Application deployments using Kustomize base/overlay pattern
  - `base/` - Shared base configurations
  - `staging/` - Environment-specific overlays with secrets and ingress
- `clusters/staging/` - Flux CD cluster configuration
  - `flux-system/` - Flux bootstrap components
  - `apps.yaml` - Kustomization CR pointing to apps/staging
  - `monitoring.yaml` - Kustomization CR for monitoring stack
- `monitoring/` - Prometheus and Grafana stack via Helm

### GitOps Flow

1. Push changes to git (main branch)
2. Flux automatically reconciles (10-30 min intervals)
3. SOPS-encrypted secrets are decrypted in-cluster using the `sops-age` secret in flux-system namespace

### Adding a New Application

Follow the existing pattern in `apps/`:
1. Create `apps/base/{app-name}/` with: namespace.yaml, deployment.yaml, service.yaml, storage.yaml (if needed), kustomization.yaml
2. Create `apps/staging/{app-name}/` with: kustomization.yaml (references base), ingress.yaml, any SOPS-encrypted secrets
3. Add the app path to `apps/staging/kustomization.yaml` if not auto-discovered

### Key Technologies

- **Flux CD v2**: GitOps controller
- **Kustomize**: Configuration management (base/overlay pattern)
- **Helm**: Used for monitoring stack (kube-prometheus-stack)
- **SOPS + AGE**: Secret encryption
- **Traefik**: Ingress controller (routes to `*.nadeemkhedr.com`)

### Security Conventions

- All secrets must be SOPS-encrypted before committing
- Containers run as non-root (see linkding: uid 33)
- Privilege escalation disabled in security contexts
