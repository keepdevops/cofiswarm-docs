# cofiswarm-docs

Cofiswarm component: `docs`.

- Layout: [REPO-STANDARD-LAYOUT](https://github.com/keepdevops/cofiswarmdev/blob/main/docs/REPO-STANDARD-LAYOUT.md)
- Migration: [MIGRATION-SPRINTS](https://github.com/keepdevops/cofiswarmdev/blob/main/docs/MIGRATION-SPRINTS.md)
- Bus topology: [observer-zmq-topology](./observer-zmq-topology.md) — ZMQ carrier flow (ingress :5556 → egress :5557 → observer)

## FHS paths

| Path | Purpose |
|------|---------|
| `/etc/cofiswarm/docs/` | config |
| `/var/lib/cofiswarm/docs/` | state |
| `/var/log/cofiswarm/docs/` | logs |

## Test

```bash
./test/scripts/assert-layout.sh docs
```
