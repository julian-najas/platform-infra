# Runbook: Deploy

## Pre-requisitos

- Acceso al cluster Kubernetes
- ArgoCD CLI configurado (`argocd login`)
- Imagen construida y publicada en GHCR

## Procedimiento — Dev

1. Merge PR en `platform-infra` → ArgoCD sincroniza automáticamente.
2. Verificar:
   ```bash
   argocd app get cacp-dev
   kubectl -n cacp-dev get pods
   kubectl -n cacp-dev logs -l app=cacp-api --tail=50
   ```

## Procedimiento — Prod

1. Asegurar que la versión pasó validación en dev.
2. Actualizar `environments/prod/values.yaml` con el tag de la imagen.
3. Crear PR con el cambio → requiere aprobación de `@julian-najas`.
4. Tras merge, ejecutar sync manual:
   ```bash
   argocd app sync cacp-prod
   ```
5. Verificar:
   ```bash
   argocd app get cacp-prod
   kubectl -n cacp-prod get pods
   kubectl -n cacp-prod logs -l app=cacp-api --tail=50
   curl -s https://api.clinica.example.com/health
   ```

## Validación post-deploy

- [ ] `/health` retorna `{"status": "ok"}`
- [ ] `/metrics` expone métricas Prometheus
- [ ] OPA accesible (verificar logs, no hay `opa_unreachable`)
- [ ] Redis conectado (verificar logs)
- [ ] PostgreSQL conectado (verificar logs)
