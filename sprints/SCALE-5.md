# SCALE-5 — Extended concurrent load

**Date:** 2026-06-16T23:20Z  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents (load sprint)  
**Change:** SCALE-4 + 3× concurrent cascade P4 + 4-way mixed-mode P2 burst.

## Commands

```bash
export COFISWARM_MODE_EXECUTE_TIMEOUT=600
make test-scale5-gate
make test-scale5-signoff-gate
```

Peak kv **0.000** · artifact `scale5-workload.json`

## Load additions

- **Cascade triple:** 3× concurrent `cascade` on P4
- **Mixed burst:** 4 modes concurrent on P2 (flat, pipeline, cascade, router)

## Results (summary)

| Mode | Prompt | Pass | Wall s | kv_pressure | Notes |
|------|--------|------|--------|-------------|-------|
| flat | P1 | yes | 0.82 | 0.000 | infer |
| pipeline | P1 | yes | 0.88 | 0.000 | infer |
| cascade | P1 | yes | 0.72 | 0.000 | infer |
| router | P1 | yes | 0.09 | 0.000 | infer |
| flat | P2 | yes | 0.75 | 0.000 | infer |
| pipeline | P2 | yes | 0.84 | 0.000 | infer |
| cascade | P2 | yes | 0.79 | 0.000 | infer |
| router | P4 | yes | 0.14 | 0.000 | infer |
| flat | P3 | yes | 0.05 | 0.000 | infer |
| pipeline | P3 | yes | 0.08 | 0.000 | infer |
| cascade | P3 | yes | 0.41 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.13 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.15 | 0.000 | infer |
| router | P3-heavy | yes | 0.02 | 0.000 | relay |
| cascade | P3-heavy | yes | 0.24 | 0.000 | infer |
| flat | P3-heavy | yes | 0.05 | 0.000 | infer |
| router | P4-concurrent | yes | 0.14 | 0.000 | infer |
| router | P4-concurrent | yes | 0.15 | 0.000 | infer |
| router | P4-concurrent | yes | 0.17 | 0.000 | infer |
| pipeline | P4-pipeline-dual | yes | 1.38 | 0.000 | infer |
| … | | | | | |

## Gate verdict

- [x] Cascade triple concurrent (3 cases)
- [x] Mixed-mode burst (4 cases)
- [x] Peak KV &lt; 0.75 (0.000)
- [x] **Advance to SCALE-6:** YES
