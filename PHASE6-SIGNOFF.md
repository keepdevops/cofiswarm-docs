# Phase 6 optional repos sign-off

**Date:** 2026-06-17T01:24Z  
**Scope:** infer-vllm · infer-sglang · infer-ollama · backend-vllm · adapter-openai-compat · tools  
**Runtime:** not in default stack (`required: false`)

## Verdict

**Phase 6 scaffold + static tests:** PASS

## Gates

```bash
cd ~/cofiswarm/repos/cofiswarm-deploy
make phase6
```

| Repo | Role |
|------|------|
| cofiswarm-infer-vllm | infer stub + Dockerfile |
| cofiswarm-infer-sglang | infer stub |
| cofiswarm-infer-ollama | infer stub |
| cofiswarm-backend-vllm | backend stub |
| cofiswarm-adapter-openai-compat | OpenAI-compat adapter |
| cofiswarm-tools | orchestrate modes (map_reduce, …) |
