# SCALE-7 — Full roster migration sign-off

**Date:** 2026-06-16T23:45Z  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents  
**Change:** SCALE-6 + full llama roster flat + pressure snapshot.

Peak kv **0.000** · artifact `scale7-workload.json`

## Roster coverage

| Lane | Count |
|------|-------|
| Llama agents (flat ROSTER-full) | 12/12 |
| MLX scout (SCALE-6 pilot) | yes |
| Total roster | 13 |

## Mode matrix (P1 / P2)

| Prompt | flat | pipeline | cascade | router |
|--------|------|----------|---------|--------|
| P1 | pass | pass | pass | pass |
| P2 | pass | pass | pass | — |

## Pressure snapshot

| Port | Agents | usage | ok |
|------|--------|-------|-----|
| 8086 | architect, debugger, optimizer, … | 0.000 | yes |
| 8085 | database, foreman, frontend, … | 0.000 | yes |
| 8083 | mlx-scout | 0.000 | yes |
| 8084 | reviewer, security | 0.000 | yes |
| 8087 | scout | 0.000 | yes |

## Recent workload rows

| Mode | Prompt | Pass | Wall s | kv |
|------|--------|------|--------|-----|
| pipeline | P4-pipeline-dual | yes | 1.41 | 0.000 |
| cascade | P4-cascade-triple | yes | 0.2 | 0.000 |
| cascade | P4-cascade-triple | yes | 0.22 | 0.000 |
| cascade | P4-cascade-triple | yes | 0.24 | 0.000 |
| router | P2-mixed-burst | yes | 0.32 | 0.000 |
| flat | P2-mixed-burst | yes | 0.96 | 0.000 |
| cascade | P2-mixed-burst | yes | 1.09 | 0.000 |
| pipeline | P2-mixed-burst | yes | 2.33 | 0.000 |
| mlx | MLX-P1 | yes | 1.03 | 0.000 |
| mlx | MLX-dual | yes | 0.33 | 0.000 |
| mlx | MLX-dual | yes | 0.33 | 0.000 |
| flat | ROSTER-full | yes | 57.16 | 0.000 |

## Gate verdict

- [x] All 12 llama agents exercised via ROSTER-full
- [x] mlx-scout pilot (SCALE-6)
- [x] P1 all four modes · P2 flat/pipeline/cascade
- [x] Peak KV &lt; 0.75 (0.000)
- [x] **43-repo migration SCALE sign-off:** YES

## Stream coverage

```bash
make test-scale7-stream-signoff
```
