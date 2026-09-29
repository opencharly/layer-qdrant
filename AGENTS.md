# AGENTS.md — layer-qdrant

Standalone candy repo for the `qdrant` layer — the Qdrant vector-search service
candy: the pinned upstream release binary, a supervisord service on REST 6333 +
gRPC 6334, persistent storage under `~/.qdrant`, and admin API-key auth from the
credential store. The candy lives in `charly.yml` at the repo root, including the
embedded `skill:` entity projected into the marketplace corpus as
`/charly-qdrant:qdrant` (the `qdrant:` skill, family `qdrant`, owner
`layer-qdrant`).

The repo split: **this repo** owns the candy; `opencharly/pod-qdrant` owns the box
image + the `check-qdrant-pod` R10 bed; `opencharly/plugin-qdrant` owns the
`charly qdrant` CLI + the `qdrant:` check verb.

Canonical files:

- `charly.yml` — the `qdrant:` candy entity and the `qdrant-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-qdrant:qdrant-cli` — the CLI + `qdrant:` check verb (in
  `plugin-qdrant`); the way a running server is probed and managed.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, `service:`, `port:`, `volume:`,
  `secret_require`). Load before editing any entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- `charly box validate` at the repo root checks the manifest parses and
  validates.
- The candy's `plan:` `check:` steps split into build-context (`qdrant-binary`,
  `qdrant-version`) and runtime probes (`qdrant-readyz`, `qdrant-healthz`,
  `qdrant-service-running`, `qdrant-auth-enforced`, `qdrant-auth-accepted`).
  The runtime probes are driven by the `check-qdrant-pod` bed in `pod-qdrant`.
- Auth is never plaintext: the API key resolves from the credential store and is
  injected as `QDRANT__SERVICE__API_KEY`.

## Modify this repo

- Edit the `qdrant:` candy entity AND the `qdrant-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a version,
  port, or auth change not mirrored in the skill leaves the corpus stale.
- The static-musl build is deliberate: it is the ONE asset Qdrant publishes for
  both linux/amd64 and linux/arm64, so a single `${BUILD_ARCH}` URL covers every
  platform. Do not swap it for a gnu build.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
