# Release cut sign-off

**Date:** 2026-06-17T01:36Z  
**Tag:** v1.1.0 on 43 pinned repos · monorepo `v1.1.0-migration`

## Verdict

**Annotated release tags at pin SHAs:** PASS

## Gates

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
./scripts/pin-repos.sh
./scripts/tag-all-repos.sh
make release-cut
RELEASE_REQUIRE_REMOTE=1 make test-all-release-tags-gate   # after git push --tags
```

Push all tags: loop `git -C ~/cofiswarm/repos/<name> push origin v1.1.0`
