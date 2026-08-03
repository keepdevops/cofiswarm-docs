# Observability sign-off

**Date:** 2026-06-17T00:13Z  
**Stack:** observer :8016 · Prometheus :9090 · Grafana :3030 · zmq-bridge :5555

## Verdict

**Host metrics + optional Prometheus/Grafana + ZMQ:** PASS

## Gates

| Gate | Scope |
|------|-------|
| `test-observability-gate` | observer plugins, grafana layout, /metrics |
| `test-zmq-bridge-gate` | topics + publish |
| `test-prometheus-up-gate` | scrape + PromQL + Grafana |

```bash
make up
make observability-up
make test-observability-signoff-gate
```

See `cofiswarm-deploy/docs/observability.md`.
