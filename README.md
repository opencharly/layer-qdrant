# layer-qdrant

The Qdrant vector-search service candy of the
[opencharly/charly](https://github.com/opencharly/charly) candy library, as a
standalone kind-prefixed repo. The candy manifest lives at the repo root; the
charly resolver fetches this repo at the pinned tag.

Installs the pinned upstream Qdrant release binary (static-musl, the one asset
Qdrant publishes for both `linux/amd64` and `linux/arm64`) and supervises it on
REST `6333` + gRPC `6334` with persistent storage under `~/.qdrant`. Auth is the
admin API key, resolved from the credential store at `charly/api-key/qdrant` and
injected as `QDRANT__SERVICE__API_KEY` — never a plaintext key in the image or
`charly.yml`. Configuration is entirely environment-driven (`QDRANT__*`), so the
candy ships no static config file.

Compose it into a box:

```yaml
candy:
  base: "quay.io/fedora/fedora:43"
  distro: [fedora:43, fedora]        # REQUIRED for an external base
  candy:
    - '@github.com/opencharly/layer-qdrant:<ref>'
    - '@github.com/opencharly/plugin-qdrant/candy/plugin-qdrant:<ref>'
```

- [`pod-qdrant`](https://github.com/opencharly/pod-qdrant) — the box image + the
  `check-qdrant-pod` R10 bed that prove this candy.
- [`plugin-qdrant`](https://github.com/opencharly/plugin-qdrant) — the
  `charly qdrant` CLI + the `qdrant:` check verb that manage and probe it.

## Ports, endpoints, auth

| Surface | Port | Notes |
|---|---|---|
| REST + Web UI | `6333` | `/healthz`, `/livez`, `/readyz` are unauthenticated; everything else needs the key |
| gRPC | `6334` | the Go/Rust/Python clients connect here |
| Distributed p2p | `6335` | NOT published — single-node only |

```bash
charly config qdrant
charly start qdrant
charly secrets get charly/secret QDRANT__SERVICE__API_KEY   # the generated key
charly check run check-qdrant-pod                            # in the pod-qdrant repo
```
