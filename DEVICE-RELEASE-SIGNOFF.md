# Device release sign-off

**Date:** 2026-06-17T00:17Z  
**Release:** v1.1.0  
**Device:** M3 Max · profile 16gb

## Verdict

**Migration + SCALE-7 + UI ops + observability:** PASS

## Sign-offs

| Track | Doc |
|-------|-----|
| Migration structure | [MIGRATION-SIGNOFF.md](./MIGRATION-SIGNOFF.md) |
| SCALE 0–7 | [MIGRATION-SCALE-SIGNOFF.md](./MIGRATION-SCALE-SIGNOFF.md) |
| Observability | [OBSERVABILITY-SIGNOFF.md](./OBSERVABILITY-SIGNOFF.md) |

## Gates

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
./scripts/pin-repos.sh
make test-release-signoff-gate
```

Pins: `43` repos · `migration_signoff=v1.1.0` · `observability_signoff=v1.1.0`
