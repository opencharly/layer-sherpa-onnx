# AGENTS.md — layer-sherpa-onnx

Standalone candy repo for the `sherpa-onnx` layer — the offline text-to-speech
ONNX engine plus a baked `en_US` VITS voice. The candy lives in `charly.yml` at
the repo root: the `env:` block, the `SHERPA_VERSION` var, the runtime + voice
download steps, the `check:` probes, and the embedded `skill:` entity projected
into the marketplace corpus as `/charly-tools:sherpa-onnx`.

Canonical files:

- `charly.yml` — the `sherpa-onnx:` candy entity and the `sherpa-onnx-skill:`
  skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-tools:sherpa-onnx` — the owning skill. The pinned runtime release, the
  baked voice, the runtime/model dir env vars, and the offline-synthesis
  contract. Load before editing or troubleshooting the candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `command:`/`check:`, `env:`). Load before editing any
  entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- `charly box validate` at the repo root checks the manifest parses and
  validates.
- The candy's `plan:` `check:` steps are the functional evidence: the
  `sherpa-onnx-offline-tts` binary and its executable bit, the C-API + ONNX
  Runtime shared libraries, the VITS voice model, and its tokens file all ship in
  the image — so synthesis runs with no network fetch.
- The download step is architecture-switched (`x86_64` → `linux-x64`,
  `aarch64` → `linux-aarch64`); keep both arms valid.

## Modify this repo

- Edit the `sherpa-onnx:` candy entity AND the `sherpa-onnx-skill:` skill entity
  in `charly.yml` together. The skill is the projected usage source, so a version
  or path change not mirrored in the skill leaves the corpus stale.
- Bump `SHERPA_VERSION` deliberately; the runtime and the voice model are
  separate release assets.
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
