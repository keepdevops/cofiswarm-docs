# SCALE-4 — KV quant audit + dual pipeline load

**Date:** 2026-06-16T23:17Z  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents (load sprint)  
**Change:** SCALE-3 + cache-type audit + 2× concurrent pipeline P4.

## Commands

```bash
export COFISWARM_MODE_EXECUTE_TIMEOUT=600
make test-scale4-gate
make test-scale4-signoff-gate
make test-architect-stream-cascade-gate
```

Peak kv **0.000** · artifact `scale4-workload.json`

## KV quant audit

| Agent | cache-type-k | cache-type-v | Pass |
|-------|--------------|--------------|------|
| architect | q4_0 | q8_0 | yes |
| database | q4_0 | q8_0 | yes |
| debugger | q4_0 | q8_0 | yes |
| foreman | q4_0 | q8_0 | yes |
| frontend | q4_0 | q8_0 | yes |
| optimizer | q4_0 | q8_0 | yes |
| programmer | q4_0 | q8_0 | yes |
| reviewer | q4_0 | q8_0 | yes |
| … | | | |

## Results (summary)

| Mode | Prompt | Pass | Wall s | kv_pressure | Notes |
|------|--------|------|--------|-------------|-------|
| flat | P1 | yes | 1.33 | 0.000 | infer |
| pipeline | P1 | yes | 0.96 | 0.000 | infer |
| cascade | P1 | yes | 0.76 | 0.000 | infer |
| router | P1 | yes | 0.09 | 0.000 | infer |
| flat | P2 | yes | 0.8 | 0.000 | infer |
| pipeline | P2 | yes | 0.84 | 0.000 | infer |
| cascade | P2 | yes | 0.72 | 0.000 | infer |
| router | P4 | yes | 0.14 | 0.000 | infer |
| flat | P3 | yes | 0.05 | 0.000 | infer |
| pipeline | P3 | yes | 0.09 | 0.000 | infer |
| cascade | P3 | yes | 0.4 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.13 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.15 | 0.000 | infer |
| router | P3-heavy | yes | 0.02 | 0.000 | relay |
| cascade | P3-heavy | yes | 0.25 | 0.000 | infer |
| flat | P3-heavy | yes | 0.05 | 0.000 | infer |
| router | P4-concurrent | yes | 0.14 | 0.000 | infer |
| router | P4-concurrent | yes | 0.16 | 0.000 | infer |
| … | | | | | |

## Gate verdict

- [x] KV quant audit (pass)
- [x] Dual pipeline concurrent (2 cases)
- [x] Peak KV &lt; 0.75 (0.000)
- [x] **Advance to SCALE-5:** YES
