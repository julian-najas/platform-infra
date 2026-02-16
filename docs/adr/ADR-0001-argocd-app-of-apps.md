# ADR-0001: ArgoCD App-of-Apps / Projects

## Status

Accepted

## Date

2026-02-16

## Context

La plataforma clínica se compone de múltiples componentes (API, workers,
OPA, PostgreSQL, Redis, observabilidad). Necesitamos una estrategia de
despliegue GitOps que escale y mantenga separación de ambientes.

Opciones evaluadas:
- A) kubectl apply manual → no reproducible, no auditable.
- B) Helm releases en un solo Application → blast radius alto.
- C) ArgoCD con AppProject + Applications por componente/entorno.

## Decision

Adoptamos **C**: ArgoCD con estructura app-of-apps.

### AppProject
- `clinical-platform` — agrupa todas las applications.
- Source repos restringidos a `platform-infra` y `clinic-gitops-config`.
- Destinations restringidos a namespaces específicos.

### Applications
- `cacp-dev` — despliega control-plane en `cacp-dev`, sync automático.
- `cacp-prod` — despliega control-plane en `cacp-prod`, sync manual.
- Separación explícita: no es "el mismo deploy con distinto tag".

### Sync policies
- Dev: automated prune + self-heal.
- Prod: manual sync, PruneLast, no CreateNamespace.

## Consequences

### Positive
- Despliegue declarativo y auditable (git history).
- Rollback sencillo (ArgoCD rollback o git revert).
- Blast radius contenido por namespace.

### Negative
- ArgoCD como dependencia de infraestructura.
- Learning curve para el equipo.

### Mitigations
- Runbooks detallados para deploy y rollback.
- kubectl fallback documentado para emergencias.
