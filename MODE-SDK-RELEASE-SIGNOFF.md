# mode-sdk release sign-off

**Date:** 2026-06-17T01:03Z  
**Module:** `github.com/keepdevops/cofiswarm-mode-sdk` · **Tag:** v0.1.0

## Verdict

**Versioned mode-sdk + mode plugin requires:** PASS

## Gates

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
./scripts/tag-mode-sdk.sh
make mode-sdk-release
MODE_SDK_REQUIRE_REMOTE=1 make test-mode-sdk-release-gate   # after git push --tags
```

Push tag: `git -C ~/cofiswarm/repos/cofiswarm-mode-sdk push origin v0.1.0`
