# Application Analysis — Cofiswarm

> Analysis based on the **running instance** at `http://localhost:3000` plus the
> live coordinator on `:3002`, cross-checked against the source.
> Repo: `keepdevops/cofiswarmdev` · npm package: `@keepdevops/matrix` v2.1.0 · License: AGPL-3.0-only
>
> Naming: the project has been rebranded to **Cofiswarm**. The running UI title
> is "Cofiswarm — Local Multi-Agent LLM Workbench" and the header logo reads
> "Cofiswarm". The npm package id (`@keepdevops/matrix`) and the internal engine
> name (`swarm-matrix`, returned by `/api/health`) still carry the old "Matrix"
> name, but the product is Cofiswarm.

---

## 0. What's actually running right now (ground truth)

Observed live from `:3000` (UI) and `:3002` (`/api/*`):

- **Branding**: "Cofiswarm", light/cream theme active (theme toggle in header).
- **Status**: `ONLINE — LLAMA + MLX`. **vLLM is not running** despite being
  supported in code. `/api/health` → `{"status":"ok","engine":"swarm-matrix"}`.
- **Active mode**: `router` (header MODE dropdown). All four modes exist:
  `flat`, `pipeline`, `cascade`, `router`.
- **Deployed agents: 13** (not the "16+" the README markets) —
  `architect, database, debugger, foreman, frontend, mlx-scout, optimizer,
  programmer, reviewer, scout, security, synthesis, tester`.
- **Engine split**: 12 × `llama`, 1 × `mlx`. The swarm is **mostly one shared
  model**, not a richly heterogeneous mix:
  - 9 agents → `Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf` on port **8081**
  - 3 agents (foreman, reviewer, scout) → `Codestral-22B-v0.1-Q4_K_M.gguf` on **8082**
  - 1 agent (mlx-scout) → `Llama-3.2-1B-Instruct-4bit` (MLX) on **8083**
- **Ports are shared by model**, not one-per-agent (same-model agents share a
  `llama-server` process). These are 8081–8083, **not** the README's vLLM 8080–8083.
- **RAG**: enabled, `embedder: mlx`, `top_k: 3`, `min_score: 0.0`.
- **Host**: ~36 GB unified memory (20.8 GB free at observation), `source: host`.
- **UI shell**: Brewlate layout with five tabs — **Session · Agents · Modes ·
  Live · RAG** — header controls: ONLINE status, MODE, KV gauge, HISTORY(0),
  CLEAR KV, CONFIGURE, theme toggle, MONITOR. Session tab has a prompt box with
  Temperature slider (0.20), "Use RAG context" toggle, QP target dropdown,
  LLAMA/MLX engine toggle, SUBMIT PROMPT and REFINE buttons.

> The sections below describe full product capabilities. Where the README claims
> more than the live deploy shows (16 agents, three engines), the running
> instance is the authority: **13 agents, two live engines (LLAMA+MLX)**.

---

## 1. What it is

cofiswarm is a **local-first, multi-agent LLM coding workbench**. It turns an
Apple Silicon or NVIDIA machine into a team of up to 16 specialised coding
agents (architect, programmer, security, reviewer, tester, …) that answer a
single prompt together under one of several orchestration strategies.

Everything runs on local inference servers — no cloud, no API keys. It is
installable via `npm i -g @keepdevops/matrix` and launched with the `matrix`
CLI, which boots the whole stack.

Its niche versus tools like Cursor/Aider or frameworks like CrewAI/LangGraph:
**privacy-first, multi-backend mixing (MLX + llama.cpp + vLLM in one swarm),
pre-built specialised agents, and multi-turn conversation threads** with a
real-time React UI.

---

## 2. Architecture (3-tier)

```
┌──────────┐    ┌──────────┐    ┌──────────────────────────┐
│ React UI │───▶│  proxy   │───▶│  coordinator (C++)       │
│  :3000   │    │  :3002   │    │  :8000 / :3002 native    │
└──────────┘    └──────────┘    │  modes · rosters ·       │
                                │  presets · breaker · SSE │
                                │  └─▶ N agent backends    │
                                │      (llama / mlx / vllm) │
                                └──────────────────────────┘
```

| Tier | Tech | Role |
|---|---|---|
| **UI** | React 18 (CRA / react-scripts), CodeMirror | Browser workbench on `:3000`; dispatches prompts, renders streamed agent output, config panels, metrics. |
| **proxy** | compiled native binary (`proxy`, Mach-O arm64) | Front door on `:3002`; routes UI requests to the coordinator and MLX native API. |
| **coordinator** | compiled C++ binary (`coordinator`, Mach-O arm64) | Orchestration engine: runs the modes, manages agent backends, KV-cache, sessions, SSE streaming, and a native MLX path (`/api/mlx/*`). |
| **orchestration/** | Python | Reference/sidecar orchestration: `SwarmFactory` config loader, RAG service, MLX coordinator sidecar, lifecycle (launch/check/shutdown), telemetry, extra modes. |
| **backends/** | Python | Backend base abstractions for inference engines. |

The `proxy` and `coordinator` ship as **pre-built native binaries** (built from
C++ sources via `scripts/build_cpp_binaries.sh`). The Python `orchestration/`
tree provides the launch lifecycle, RAG, MLX sidecar, and additional
experimental modes.

---

## 3. Inference engines (mixable per agent)

> Live now: **LLAMA + MLX only**. vLLM is supported in code/configs but is
> **not running** in the current instance.

- **LLAMA** — `llama-server` (llama.cpp), loads `.gguf` files; `--parallel N`
  lets same-model agents share one process; supports KV-cache clear.
- **MLX** — `mlx_lm.server` on Apple Silicon/Metal; loads model directories;
  handled natively by the C++ coordinator on `:3002` (`/api/mlx/*`) with
  per-port serialisation and session management.
- **vLLM** — a **single shared** Docker Model Runner / OpenAI-compatible
  endpoint at `http://127.0.0.1:12434/v1` (port **12434**), configured in
  `docker/docker-vllm-config.json`. The proxy binary hardcodes 12434
  (`PROXY_CONFIGURE_DOCKER_PORT`). **Not** four servers on 8080–8083 — the
  README's "8080–8083" is wrong; those are the llama.cpp/MLX server ports.

**Port configurability**: llama/MLX agent ports are set per agent in
`config/agents/*.json` and the generated `swarm-config*.json` (regenerate with
`scripts/build_swarm_config.py`) — fully configurable. The vLLM endpoint port
lives in `docker/docker-vllm-config.json` but is also baked into the proxy
binary as a constant, so changing it cleanly needs a rebuild.

Any agent can be pointed at any model via the CONFIGURE panel / per-agent JSON.

---

## 4. Orchestration modes

The active mode decides how one prompt becomes one or more agent calls:

| Mode | Behaviour | Reducer |
|---|---|---|
| `flat` | Broadcast in parallel to **every** deployed agent. | None |
| `pipeline` | Sequential chain; each stage gets the previous stage's output. | Optional synthesizer |
| `cascade` | Parallel broadcast + a synthesizer (mixture-of-agents). | Required synthesizer |
| `router` | Classifier agent (default `foreman`) picks ≤ `max_select` agents from the roster, enriched with live KV-pressure load hints. | None |

Additional experimental Python modes live in `orchestration/modes/`:
`critic_debate`, `map_reduce`, `speculative`, `tree_of_thought`.

Each mode has a **per-mode roster** (which agents participate; order matters in
pipeline) and **presets** (save mode + roster + synthesizer + max_select under
a name). Empty roster falls back to the full deployed swarm.

---

## 5. The agents

**13 agents are actually deployed** (live `/api/agents`), each with a tuned
system prompt and UI colour: `architect, database, debugger, foreman, frontend,
mlx-scout, optimizer, programmer, reviewer, scout, security, synthesis, tester`.
This matches the 13 files in `config/agents/`. The README/marketing "16+ roles"
(adding api, devops, documenter, specialist) is aspirational, not what runs.

In the current deploy the swarm is **homogeneous by model**, not one model per
role: 9 agents share Llama-3.1-8B, 3 share Codestral-22B (foreman, reviewer,
scout), and mlx-scout runs the MLX Llama-3.2-1B. Same-model agents share a
single backend process/port.

Per-agent config is the **source of truth** in `config/agents/<name>.json`
(plus `config/coordinator.json`). Model paths use `${MATRIX_MODEL_DIR}/...`
placeholders, expanded at load time, **failing loudly** on any unresolved var.
`python3 scripts/build_swarm_config.py` regenerates the gitignored
`swarm-config.json` build artifact (and its `public/` copy). Authored variants:
`swarm-config-16gb.json`, `swarm-config-32gb.json`,
`swarm-config-8agents-text-image.json`.

---

## 6. Key features

- **Multi-turn conversation threads** — sessions auto-continue after the first
  broadcast; `ConversationThread` shows turn history; persisted in
  `sessions.json`, reset by CLEAR KV.
- **RAG context injection** — sqlite-vec-backed (serverless, local `.db` file),
  per-agent targeting; ingest / manage / retrieve via the RAG panels and
  `orchestration/rag/` (chunker, embed, retrieve, store, jobs).
- **Per-agent circuit breakers** and live agent health badges/dashboard.
- **Live metrics & Monitor popout** — KV-cache pressure per port, Unified
  Memory gauge, MLX per-port Q/W/D pressure, opt-in RSS event feeds
  (model load/evict, token-regulation).
- **Five+ UI layouts** — **Brewlate** (default), plus classic, dashboard,
  terminal, minimal, sidebar; selectable via `?layout=`. Visual layout editor
  (flow / freeform / grid) with localStorage persistence.
- **CodeMirror response viewer** — multi-language auto-detect, edit, copy,
  full-screen expand per card.
- **Auto code extraction** — the `programmer` agent's first code block is pulled
  into a dedicated CODE OUTPUT pane; SAVE CODE exports all agents' code blocks
  to one timestamped file.
- **Broadcast history** — last 10 prompts + full responses, click to reload.
- **Model converter** — GGUF → MLX conversion tooling (`scripts/gguf_to_mlx.py`,
  `ModelConverter` UI).
- **Quality passes, variant comparison, response rating, token budgets,
  trajectory/impact dashboards, distillation** — additional analysis panels.
- **SSE streaming** end-to-end for both llama and MLX paths.

---

## 7. Codebase shape

- **Frontend**: ~259 source `.js` files in `src/` with ~47 test files.
  Organised into `api/` (23 typed API clients), `components/`, `layouts/`
  (Brewlate + alternates), `hooks/` (~45 React hooks for state/streaming/
  polling), `utils/`, `config/` (Zod-style schemas), `themes/`, `styles/`.
- **Backend orchestration**: ~32 Python files in `orchestration/` + `backends/`.
- **Native binaries**: `proxy`, `coordinator` (committed pre-built); C++ test
  suites under `xxxxx/`.
- **CLI**: `bin/matrix.mjs` (launcher; platform-gated to macOS arm64 / Linux
  x64+arm64) and `bin/matrix-uninstall.mjs`.
- **Deploy**: `docker/`, `production/` (Dockerfile, nginx, compose), `scripts/`
  (build, staging, RAG docker, config setup).

Engineering conventions (per CLAUDE.md) are visibly applied: small modular
files, Pydantic/Zod validation at boundaries, and "fail loudly" error handling
in the config loader.

---

## 8. How to run

```bash
npm i -g @keepdevops/matrix   # install
matrix                        # launch proxy + coordinator + UI

# from source (dev):
npm start                     # CRA dev server on :3000
```

The `matrix` CLI verifies the platform, locates the bundled `proxy`/`coordinator`
binaries and web build, then starts the stack. Set `MATRIX_MODEL_DIR` to your
local model directory (defaults to `/Users/Shared/llama/models` on macOS).

---

## 9. Reference docs in this directory

- `CAPABILITIES.md` — exhaustive endpoint/event/knob reference.
- `BACKEND_ROUTING.md` — how requests route across engines.
- `configuration.md`, `model-management.md` — config and model handling.
- `mlx-coordinator-benchmark.md`, `mlx-native-api-parity.md`,
  `architecture/mlx-inference-roadmap.md` — MLX native path details.
- `distillation-schema.md`, `HELP.md`, `block_sequence.puml`.
