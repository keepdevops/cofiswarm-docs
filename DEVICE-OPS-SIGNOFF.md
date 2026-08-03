# Device ops sign-off

**Date:** 2026-06-17T00:24Z  
**Stack:** `make up` · UI :3000 · launchd optional login start

## Verdict

**Stack health + UI ops + launchd template:** PASS

## LaunchAgent (optional)

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
make install-launchd
./scripts/launchd-status.sh
LAUNCHD_REQUIRE=1 make test-launchd-live-gate
make uninstall-launchd   # remove
```

## Gates

```bash
make up
make test-device-ops-signoff-gate
```

See `cofiswarm-deploy/docs/runbook.md`.
