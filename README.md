# platform-infra

> **Public by design** — This repository is public for demo / portfolio purposes.
> It contains **no secrets, credentials, or PHI**. Production configurations live in private infrastructure.

Esqueleto de infraestructura como código para desplegar la plataforma clínica
de reducción de no-shows.

## Estructura

```
environments/          ← Valores por entorno (dev, prod)
k8s/
  base/               ← Recursos base (namespaces, RBAC)
  apps/               ← Manifiestos por aplicación
  observability/      ← Prometheus, Grafana, alertas
argocd/
  projects/           ← AppProjects de ArgoCD
  applications/       ← Applications de ArgoCD
docs/runbooks/        ← Procedimientos operativos
```

## Principios

1. **GitOps-first** — toda configuración de infra vive en Git; ArgoCD sincroniza.
2. **Separación dev/prod** — valores distintos por entorno, secrets nunca en Git.
3. **Least-privilege** — RBAC restrictivo; cada componente tiene su ServiceAccount.
4. **Observabilidad** — métricas, alertas y dashboards definidos como código.

## Componentes desplegados

| Componente | Namespace | Descripción |
|---|---|---|
| `clinical-agentic-control-plane` | `cacp` | API FastAPI + workers |
| PostgreSQL | `cacp` | Event store (append-only) |
| Redis | `cacp` | Cola de acciones |
| OPA | `cacp` | Motor de políticas |
| Prometheus + Grafana | `observability` | Métricas y dashboards |

## Quick Start (local)

```bash
# Requiere: kubectl, kustomize, argocd CLI
kubectl apply -k k8s/base/
kubectl apply -k k8s/apps/clinical-agentic-control-plane/
```

## Propietario

[@julian-najas](https://github.com/julian-najas)

## Inter-repo contracts

This repo is the **deployer** — it takes what was approved in
`clinic-gitops-config` and delivers it to the cluster. It **never**
creates, modifies, or approves clinical artifacts.

### Inbound — from `clinic-gitops-config`

| What | Mechanism |
|------|-----------|
| Merged policies, plans, templates | ArgoCD Application watches `clinic-gitops-config` `main` branch. Sync is **automated** in `dev`, **manual** in `prod`. |

> ArgoCD **never reads unmerged branches**. Only post-merge `main` is
> considered the deployment source of truth.

### Inbound — from `clinical-agentic-control-plane`

| What | Mechanism |
|------|-----------|
| Container image | CI in `clinical-agentic-control-plane` pushes to registry. ArgoCD Application references the image tag in `k8s/apps/`. |

> This repo pins image tags via Kustomize overlays per environment.
> `dev` may use `:latest`; `prod` uses immutable SHAs.

### Expected health endpoints (contract with deployed services)

| Endpoint | Used by |
|----------|---------|
| `GET /health` | Kubernetes `livenessProbe` |
| `GET /ready` | Kubernetes `readinessProbe` |
| `GET /metrics` | Prometheus scrape config (`k8s/observability/prometheus-config.yaml`) |

### Dependency — `casf-core`

`casf-core` (scope-frozen v0.13.0) may be deployed as an edge gateway
in front of the control-plane. Its manifests are **not** managed here —
`casf-core` maintains its own deployment. This repo only provisions the
namespace and RBAC if needed.

### Contract invariants

1. **Deploy only merged state** — ArgoCD source is always `main` post-merge.
2. **Environment parity** — `dev` and `prod` share base manifests; only values differ.
3. **Secrets never in Git** — `secrets.example.yaml` documents shape; real secrets live in external stores.
4. **Least-privilege RBAC** — each component gets its own ServiceAccount with a scoped Role.
5. **Observability baseline** — Prometheus alerts fire if `/health` or `/metrics` are unreachable.

## Related repos

| Repo | Role |
|------|------|
| `clinic-gitops-config` | Approval boundary — source of truth for what gets deployed |
| `clinical-agentic-control-plane` | Service that produces signed PRs and executes on merge |
| `casf-core` | Zero-trust policy gateway (scope-frozen, own deployment) |

## License

[Apache-2.0](LICENSE)
