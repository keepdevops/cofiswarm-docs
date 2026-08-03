# SCALE-6 — TurboQuant MLX pilot

**Date:** _fill on first run_  
**Hardware:** M3 Max 36 GB  
**Roster:** 13 agents (load sprint)  
**Change:** SCALE-5 + mlx-scout 4bit lane (`:8083`) — audit, direct infer, dual concurrent.

**Prerequisite:** mlx-scout server running on `:8083` (configure or manual `mlx_lm.server`).

## Commands

```bash
export COFISWARM_MODE_EXECUTE_TIMEOUT=600
make test-scale6-gate
make test-scale6-signoff-gate
make render-scale6-signoff
```

## Gate verdict

- [ ] MLX audit (engine mlx, max_concurrency 1, 4bit model)
- [ ] MLX-P1 direct infer
- [ ] MLX-dual concurrent (2 cases)
- [ ] Peak KV &lt; 0.75
- [ ] **Advance to SCALE-7:** YES / NO
