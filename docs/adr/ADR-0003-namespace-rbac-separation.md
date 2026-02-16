# ADR-0003: Separación de Namespaces y RBAC por App

## Status

Accepted

## Date

2026-02-16

## Context

Múltiples componentes (API, workers, OPA, databases) coexisten en el
cluster. Sin separación, un compromise en el worker podría escalar
privilegios y acceder a secrets de la API o de otros namespaces.

Opciones evaluadas:
- A) Todo en `default` namespace → blast radius máximo.
- B) Un namespace por entorno → mejor, pero no aísla componentes.
- C) Namespaces por entorno + ServiceAccounts + RBAC por componente.

## Decision

Adoptamos **C**: namespaces separados + RBAC least-privilege.

### Namespaces
- `cacp-dev` — entorno de desarrollo.
- `cacp-prod` — entorno de producción.
- `observability` — Prometheus, Grafana, alerting.

### ServiceAccounts
- `cacp-api` — para el control-plane API.
- `cacp-worker` — para los workers de ejecución.
- Cada uno con su propio ServiceAccount, sin `automountServiceAccountToken`
  donde no sea necesario.

### RBAC
- Roles con permisos mínimos: `get`, `list` sobre `configmaps` y `secrets`.
- No `create`, `update` ni `delete` sobre recursos del cluster.
- RoleBindings scoped al namespace correspondiente.
- Prod y dev tienen Roles idénticos pero en namespaces separados.

## Consequences

### Positive
- Aislamiento: compromise en worker no accede a API secrets.
- Blast radius contenido por namespace.
- Auditabilidad: RBAC explícito en git.

### Negative
- Más manifiestos K8s a mantener.
- Duplicación de Role definitions entre namespaces.

### Mitigations
- Kustomize overlays para reducir duplicación.
- RBAC revisado trimestralmente.
