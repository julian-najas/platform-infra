# Runbook: Rollback

## Cuándo hacer rollback

- Error rate > 5% sostenido por más de 5 minutos post-deploy.
- `/health` no retorna 200.
- OPA inalcanzable (fail-closed activo, todas las peticiones denegadas).
- Errores críticos en logs que impiden operación normal.

## Procedimiento — ArgoCD

### Opción 1: Rollback via ArgoCD (preferido)

```bash
# Ver historial de deploys
argocd app history cacp-prod

# Rollback a la revisión anterior
argocd app rollback cacp-prod <REVISION_NUMBER>

# Verificar
argocd app get cacp-prod
kubectl -n cacp-prod get pods
```

### Opción 2: Revert Git + Sync

```bash
# Revertir el commit en platform-infra
git revert HEAD
git push origin main

# ArgoCD sincroniza automáticamente en dev
# En prod, sync manual:
argocd app sync cacp-prod
```

### Opción 3: kubectl directo (emergencia)

```bash
# Rollback del deployment directamente
kubectl -n cacp-prod rollout undo deployment/cacp-api
kubectl -n cacp-prod rollout undo deployment/cacp-worker

# Verificar
kubectl -n cacp-prod rollout status deployment/cacp-api
```

## Post-rollback

1. Verificar que `/health` retorna 200.
2. Revisar métricas en Grafana — confirmar que error rate baja.
3. Crear incidencia con:
   - Timestamp del deploy fallido
   - Versión desplegada
   - Error observado
   - Acción de rollback tomada
4. Root cause analysis dentro de 48h.

## Contacto de escalación

- **Owner**: @julian-najas
- **Canal**: #clinical-platform-ops
