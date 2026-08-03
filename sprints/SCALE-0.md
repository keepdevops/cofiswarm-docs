# SCALE-0 — Baseline inventory

**Date:** 2026-06-16T22:47Z  
**Hardware:** M3 Max 36 GB (`coordinator.json` memory note)  
**Roster:** 13 agents (`swarm-config.json`)  
**Change:** Sprint 21 — live configure + mode-sdk infer; Sprint 22 sign-off.

## Gate reference

[SCALE-GATES.md](./SCALE-GATES.md) · [ML-BOTTLENECKS.md](../ML-BOTTLENECKS.md)

## Configure snapshot

- Default mode: `router` (`max_select: 2`)
- Cascade synthesizer: `synthesis`
- RAG: enabled (`top_k: 3`)
- `MATRIX_LLAMA_SERVER` in `cofiswarm-deploy/.env`

```bash
CONFIGURE_LIVE=1 make test-configure-live
make test-scale0-signoff-gate
```

## Idle pressure

Source: `slot-manager:8013` → `scale0-pressure.json`

| endpoint / port | names | idle usage | ok |
|-----------------|-------|------------|-----|
| 8086 | architect, debugger, optimizer, programmer | 0.000 | true |
| 8085 | database, foreman, frontend, synthesis, tester | 0.000 | true |
| 8083 | mlx-scout | 0.000 | true |
| 8084 | reviewer, security | 0.000 | true |
| 8087 | scout | 0.000 | true |

## Nominal workload results

Source: `scale0-workload.json`

| Mode | Prompt | Pass | Wall s | kv_pressure | Notes |
|------|--------|------|--------|-------------|-------|
| flat | P1 | yes | 1.03 | 0.000 | infer |
| pipeline | P1 | yes | 0.88 | 0.000 | infer |
| cascade | P1 | yes | 0.92 | 0.000 | infer |
| router | P1 | yes | 0.09 | 0.000 | infer |
| flat | P2 | yes | 0.83 | 0.000 | infer |
| pipeline | P2 | yes | 0.89 | 0.000 | infer |
| cascade | P2 | yes | 0.91 | 0.000 | infer |
| router | P4 | yes | 0.14 | 0.000 | infer |

## Gate verdict

- [x] Baseline pressure logged (`scale0-pressure.json`)
- [x] Nominal workload complete with live infer (`notes: infer`)
- [x] **Advance to SCALE-1:** YES (full 13-agent roster → load sprint)

## Notes

- mode-flat may bind `:8121` or `:8221` when `:8021` is taken (see `mode-ports.env`).
- Slot math: per-slot KV ≈ `ctx_cap ÷ parallel` per model server.
