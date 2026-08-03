# Sidecars sign-off

**Date:** 2026-06-17T00:41Z  
**Services:** convert :8015 · rag-worker :8018

## Verdict

**MLX convert queue + RAG index worker in stack:** PASS

## Gates

```bash
CGO_ENABLED=0 make build-convert
make up
make test-sidecars-signoff-gate
```
