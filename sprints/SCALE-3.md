# SCALE-3 — Concurrent + heavy-token load sprint

**Date:** 2026-06-16T23:09Z  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents (load sprint)  
**Change:** SCALE-2 + P3-heavy + 3× concurrent router P4.

## Commands

```bash
export COFISWARM_MODE_EXECUTE_TIMEOUT=600
make test-scale3-gate
make test-scale3-signoff-gate
make test-architect-stream-router-gate
make test-architect-stream-cascade-gate
```

Peak kv **0.000** · artifact `scale3-workload.json`

## Results (summary)

| Mode | Prompt | Pass | Wall s | kv_pressure | Notes |
|------|--------|------|--------|-------------|-------|
| flat | P1 | yes | 1.42 | 0.000 | infer |
| pipeline | P1 | yes | 0.97 | 0.000 | infer |
| cascade | P1 | yes | 0.78 | 0.000 | infer |
| router | P1 | yes | 0.09 | 0.000 | infer |
| flat | P2 | yes | 0.8 | 0.000 | infer |
| pipeline | P2 | yes | 0.85 | 0.000 | infer |
| cascade | P2 | yes | 0.72 | 0.000 | infer |
| router | P4 | yes | 0.13 | 0.000 | infer |
| flat | P3 | yes | 0.06 | 0.000 | infer |
| pipeline | P3 | yes | 0.09 | 0.000 | infer |
| cascade | P3 | yes | 0.44 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.12 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.14 | 0.000 | infer |
| router | P3-heavy | yes | 0.02 | 0.000 | relay |
| cascade | P3-heavy | yes | 0.23 | 0.000 | infer |
| flat | P3-heavy | yes | 0.05 | 0.000 | infer |
| router | P4-concurrent | yes | 0.14 | 0.000 | infer |
| router | P4-concurrent | yes | 0.16 | 0.000 | infer |
| router | P4-concurrent | yes | 0.17 | 0.000 | infer |


## Gate verdict

- [x] SCALE-3 workload logged (19 cases)
- [x] Peak KV &lt; 0.75 (0.000)
- [x] **Advance to SCALE-4:** YES

## Stream coverage (Sprint 27)

All four modes stream live infer: flat, pipeline, router (`selected`), cascade (`synthesis_start`).
