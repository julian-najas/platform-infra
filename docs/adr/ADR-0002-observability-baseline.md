# ADR-0002: Observability Baseline — Métricas y Alertas Mínimas

## Status

Accepted

## Date

2026-02-16

## Context

Sin observabilidad desde Day-1, los problemas se descubren cuando un
paciente no recibe su recordatorio. Necesitamos un baseline que nos
diga "algo está mal" sin overengineering.

Opciones evaluadas:
- A) Logs y ya → no proactivo, no medible.
- B) Full o11y stack (tracing, metrics, logs, profiling) → overengineering Day-1.
- C) Métricas Prometheus + 4 alertas críticas + logs estructurados.

## Decision

Adoptamos **C**: baseline pragmático.

### Métricas (repo 2: control-plane)
- `cacp_requests_total` — counter por endpoint y status.
- `cacp_request_duration_seconds` — histograma de latencia.
- `cacp_opa_decisions_total` — counter allow/deny.
- `cacp_opa_errors_total` — counter de fallos OPA.
- `cacp_queue_depth` — gauge de items en cola Redis.
- `cacp_up` — gauge liveness.

### Alertas (repo 3: platform-infra)
1. `CACPApiDown` — API unreachable > 2 min → critical.
2. `CACPHighErrorRate` — 5xx > 5% durante 5 min → warning.
3. `CACPQueueBacklog` — queue > 100 items durante 10 min → warning.
4. `CACPOPAUnreachable` — OPA errors > 0 durante 1 min → critical.

### Logs
- structlog con `trace_id`, `correlation_id`, `actor`, `appointment_id`.
- JSON output en producción.
- Indexados por `event_type` para búsqueda rápida.

### Health checks
- `/health` — liveness (app is up).
- `/ready` — readiness (PG + Redis + OPA reachable).

## Consequences

### Positive
- Problemas detectados en minutos, no en horas/días.
- Alertas accionables, no ruido.
- Correlación request → decision → action via trace_id.

### Negative
- Prometheus + Grafana son componentes extra a mantener.
- 4 alertas pueden no cubrir todos los failure modes.

### Mitigations
- Revisión trimestral de alertas: ¿faltan? ¿sobran? ¿ruidosas?
- Dashboard Grafana como segundo paso (no Day-1 blocker).
