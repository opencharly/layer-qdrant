# qdrant

The [Qdrant](https://qdrant.tech) vector-search server for OpenCharly images.

The `qdrant` candy installs the pinned upstream static-musl release binary at
`/usr/local/bin/qdrant` and supervises it on REST 6333 + gRPC 6334 with
persistent storage under `~/.qdrant`. Auth is the admin API key, resolved from
the credential store and injected as `QDRANT__SERVICE__API_KEY` — never a
plaintext key in the image or `charly.yml`.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `qdrant` |
| Binary | `/usr/local/bin/qdrant` |
| Version | pinned `v1.19.1` (`QDRANT_VERSION` var) |
| Ports | REST `6333`, gRPC `6334` |
| Storage | `~/.qdrant` (volume `qdrant`) |
| Service | `qdrant` (supervisord, priority 20, restart always) |
| Auth | admin API key from the credential store (`charly/api-key/qdrant`) |
| Requires | [`layer-supervisord`](https://github.com/opencharly/layer-supervisord) |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-qdrant-pod:
  candy:
    base: "quay.io/fedora/fedora:43"
    distro: [fedora:43, fedora]
    candy:
      - '@github.com/opencharly/layer-qdrant:v2026.265.2042'
      - '@github.com/opencharly/plugin-qdrant/candy/plugin-qdrant:v2026.265.2042'
```

The admin API key is auto-generated 32-byte hex on first deploy. Retrieve it
with `charly secrets get charly/api-key qdrant`, or override it before first
deploy with `charly secrets set charly/api-key qdrant <v>`. `/healthz`,
`/livez`, and `/readyz` are always unauthenticated; `/collections`, `/telemetry`,
and `/metrics` require the key.

## Layout

- `charly.yml` — the `qdrant:` candy entity (the `require:`, `var:`, `env:`,
  `secret_require:`, `env_provide:`, `port:`, `volume:`, `service:`, and `plan:`
  blocks) and the embedded `qdrant-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-qdrant:qdrant` (the candy's embedded skill entity)
- CLI + `qdrant:` check verb: `/charly-qdrant:qdrant-cli` (in `plugin-qdrant`)
- Box + R10 bed: `opencharly/pod-qdrant`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
