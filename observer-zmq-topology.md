# Observer + ZMQ Bus Topology (Go)

Reconstructed from `cofiswarm-common/zmq/topics.yaml`, `cofiswarm-zmq-bridge`,
`cofiswarm-observer`, and `cofiswarm-grafana/README.md`. Shows how the Go
components join the ZMQ bus, how the observer control plane carries
presence/alerts/dispatch, and where telemetry exits to Prometheus/Grafana.

## 1. Bus topology — components ⇄ zmq-bridge ⇄ observer

The bridge is a **real ZMQ forwarder** (`COFISWARM_BUS=zmq`): an ingress SUB binds
`:5556`, collects every `swarm.*` frame component PUBs send, feeds the recent-events
ring + the `/v1/stream` SSE endpoint, and re-emits each frame on an egress PUB at
`:5557`. The observer subscribes to that egress over a real ZMQ socket; its
presence/hello translations are republished back via the bridge HTTP control plane
(`:5555 /v1/publish`).

```
   component PUB                ┌────────────────────────────────────────────────┐
   swarm.*  ───────────────────▶          cofiswarm-zmq-bridge (Go)              │
   (announce/goodbye/metrics)   │   ingress SUB :5556 ─▶ ring + /v1/stream SSE    │
                                │                       │                        │
   presence republish          │   HTTP control :5555  ▼                        │
   ◀───────POST /v1/publish─────┤            egress PUB :5557                     │
   hello (bridge -> comps)      └──────────────────────────┬─────────────────────┘
                                                           │  swarm.* (ZMQ wire)
                                                           ▼
                                     ┌──────────────────────────────────┐
                                     │      cofiswarm-observer (Go)      │
                                     │  SUB :5557  (COFISWARM_ZMQ_       │
                                     │             EGRESS_ADDR)          │
                                     │  :8016  /v1/observed  /metrics    │
                                     │  presence + roster + alerts +     │
                                     │  dispatch + log sinks + liveness  │
                                     └────────────────┬─────────────────┘
                                                      │ aggregates KV pressure → /metrics
                                                      ▼
   Prometheus :9090  ──scrape host.docker.internal:8016──►  Grafana :3030
```

The observer can alternatively tail the SSE stream (`COFISWARM_BRIDGE_URL` →
`/v1/stream`); ZMQ egress ingest takes precedence when both are set. The
`swarm.observer.model.*` (per-model dispatch, req/reply) and `swarm.observer.tokens.*`
(streamed tokens + usage) topics flow over the same carrier.

## 2. Who publishes what (from topics.yaml)

```
 DATA / TELEMETRY topics                    swarm.<domain>.<event>
 ─────────────────────────────────────────────────────────────────
 cofiswarm-kvpool        ──▶  swarm.kvpool.evict   · swarm.kvpool.pressure
 cofiswarm-slot-manager  ──▶  swarm.slot.erase     · swarm.slot.pressure   (:8013)
 cofiswarm-dispatch      ──▶  swarm.dispatch.session
 cofiswarm-infer-llama   ──▶  swarm.infer.llama.metrics
 cofiswarm-infer-mlx     ──▶  swarm.infer.mlx.metrics
 cofiswarm-adapter-*     ──▶  swarm.adapter.opencode.step

 OBSERVER CONTROL PLANE                     direction
 ─────────────────────────────────────────────────────────────────
 swarm.observer.announce    component  ──▶  bus     (join / re-announce)
 swarm.observer.goodbye     component  ──▶  bus     (graceful leave)
 swarm.observer.presence    bus        ──▶  observers (online/offline)
 swarm.observer.alert       bus        ──▶  observers (needed-comp-down / timeout)
 swarm.observer.request     observer   ──▶  bridge  (inference request)
 swarm.observer.roster      observer   ──▶  bridge  (snapshot for late joiners)
 swarm.observer.cancel      observer   ──▶  models  (cancel in-flight request)
 swarm.observer.hello       bridge     ──▶  comps   (re-announce on restart)
 swarm.observer.model.*     bridge     ──▶  model   (per-model dispatch, req/reply)
 swarm.observer.tokens.*    model      ──▶  observers (streamed tokens + final usage)
```

## 3. Component set on the bus (Go services)

```
 front door     cofiswarm-gateway            (nginx front door)
 orchestration  cofiswarm-dispatch           (architect dispatch, SSE, sessions)
 modes          cofiswarm-mode-{flat,pipeline,cascade,router}
 roster         cofiswarm-agent-registry     (agent roster + mode config API)
 concurrency    cofiswarm-slot-manager :8013 (physical slots + KV pressure)
 kv policy      cofiswarm-kvpool             (eviction, token budget)
 inference      cofiswarm-infer-{llama,mlx,vllm,ollama,sglang}
 telemetry hub  cofiswarm-observer :8016     (presence, alerts, dispatch, metrics)
 the bus        cofiswarm-zmq-bridge :5555 control · :5556 ingress · :5557 egress
```

## Carrier backends and wiring

`cofiswarm-zmq-bridge` selects its carrier via `COFISWARM_BUS`:

| Backend | Selector | Carrier |
|---------|----------|---------|
| `mem` (default) | unset | in-process pub/sub (tests / offline) |
| `nats` | `COFISWARM_BUS=nats` | real NATS broker (`COFISWARM_NATS_URL`) |
| `zmq` | `COFISWARM_BUS=zmq` | ingress SUB `:5556` + egress PUB `:5557` (this doc) |

Relevant env:

- Bridge: `COFISWARM_ZMQ_ADDR` (ingress, default `tcp://*:5556`),
  `COFISWARM_ZMQ_EGRESS_ADDR` (egress, default `tcp://*:5557`, `off` for ingress-only).
- Observer: `COFISWARM_ZMQ_EGRESS_ADDR` (subscribe target), `COFISWARM_ZMQ_FILTER`
  (subject prefix, default `swarm.`), `COFISWARM_BRIDGE_URL` (HTTP base for presence
  republish + SSE fallback).

Implemented in `cofiswarm-zmq-bridge` PR #8 (forwarder) and `cofiswarm-observer`
PR #3 (egress subscriber). The topic names (`swarm.observer.*`) map 1:1 onto NATS
subjects, so the `nats` backend carries the identical control plane.
