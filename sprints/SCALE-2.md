# SCALE-2 — Long-context load sprint

**Date:** 2026-06-16T22:52Z  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents (load sprint)  
**Change:** P3 long-context + concurrent flat; `mode_config` passthrough.

## Commands

```bash
export COFISWARM_MODE_EXECUTE_TIMEOUT=600
make test-scale2-gate
make test-scale2-signoff-gate
```

Artifact: `scale2-workload.json` · peak kv **0.000**

## Results

| Mode | Prompt | Pass | Wall s | kv_pressure | Notes |
|------|--------|------|--------|-------------|-------|
| flat | P1 | yes | 1.12 | 0.000 | infer |
| pipeline | P1 | yes | 0.84 | 0.000 | infer |
| cascade | P1 | yes | 0.88 | 0.000 | infer |
| router | P1 | yes | 0.09 | 0.000 | infer |
| flat | P2 | yes | 0.8 | 0.000 | infer |
| pipeline | P2 | yes | 0.84 | 0.000 | infer |
| cascade | P2 | yes | 0.88 | 0.000 | infer |
| router | P4 | yes | 0.13 | 0.000 | infer |
| flat | P3 | yes | 1.72 | 0.000 | infer |
| pipeline | P3 | yes | 1.84 | 0.000 | infer |
| cascade | P3 | yes | 14.79 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.23 | 0.000 | infer |
| flat | P1-concurrent | yes | 0.24 | 0.000 | infer |

## Gate verdict

- [x] P3 + baseline workload logged
- [x] Peak KV &lt; 0.75 (0.000)
- [x] **Advance to SCALE-3:** YES
