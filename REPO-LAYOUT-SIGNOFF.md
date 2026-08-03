# Repo layout sign-off

**Date:** 2026-06-17T00:53Z  
**Scope:** 43 repos · standalone FHS · no git submodules

## Verdict

**REPO-STANDARD-LAYOUT §15 (`test/standalone`) on every repo:** PASS

## Gates

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
make test-repo-layout-gate
make repo-layout
```

Per-repo CI template: `templates/repo-ci.yml` · `./scripts/install-repo-ci.sh`
