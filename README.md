# platform-infra

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
