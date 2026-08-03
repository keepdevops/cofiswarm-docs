# Migration operator handoff

**Date:** 2026-06-17T02:28Z  
**Release:** v1.1.0 · 43 repos · device M3 Max · profile 16gb  
**Sign-off:** v1.1.0

## Verdict

**43-repo device migration:** READY FOR OPERATOR HANDOFF

All capstone gates passed. Origin matches pins (`REMOTE_REQUIRE=1`). Local checkouts match `repos.json`.

## Prerequisites

| Stage | Doc |
|-------|-----|
| Migration complete | [MIGRATION-COMPLETE-SIGNOFF.md](./MIGRATION-COMPLETE-SIGNOFF.md) |
| Remote complete | [REMOTE-COMPLETE-SIGNOFF.md](./REMOTE-COMPLETE-SIGNOFF.md) |
| Post-migration track | [POST-MIGRATION-SIGNOFF.md](./POST-MIGRATION-SIGNOFF.md) |

## Daily verification

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
./scripts/verify-migration.sh              # status (non-fatal)
./scripts/pin-repos.sh                     # refresh pins after local commits
make post-migration
make ops-check                             # stack health + UI smoke
```

## Capstone gates (one-time / after bulk changes)

```bash
./scripts/pin-repos.sh
make migration-handoff                     # remote-complete + verify + render
REMOTE_REQUIRE=1 make migration-handoff      # after origin push
```

## Push runbook (origin drift)

```bash
./scripts/verify-remote-push.sh
PUSH_DRY_RUN=1 ./scripts/push-all-repos.sh
./scripts/push-all-repos.sh
REMOTE_REQUIRE=1 make remote-complete
```

## Live stack (optional)

```bash
make post-migration-live                   # sidecars + device-ops gates
COFISWARM_CI_LIVE=1 make test-ci-signoff-gate
```

Operator runbook: `cofiswarm-deploy/docs/runbook.md` § Migration handoff
