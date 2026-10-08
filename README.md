# sherpa-onnx

Offline text-to-speech for OpenCharly images — the [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx)
ONNX engine plus a baked `en_US` VITS voice, no runtime download.

The `sherpa-onnx` candy downloads the sherpa-onnx shared runtime release
(executables under `runtime/bin`, the ONNX Runtime + C-API shared libraries under
`runtime/lib`) and the `vits-piper-en_US-lessac-high` voice into
`~/.local/share/sherpa-onnx`. Synthesis runs fully offline with no network fetch.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `sherpa-onnx` |
| Version | pinned `v1.12.23` (`SHERPA_VERSION` var) |
| Engine | `~/.local/share/sherpa-onnx/runtime/bin/sherpa-onnx-offline-tts` |
| Libraries | `libsherpa-onnx-c-api.so`, `libonnxruntime.so` |
| Voice | `vits-piper-en_US-lessac-high` (baked into the image) |
| Env | `SHERPA_ONNX_RUNTIME_DIR`, `SHERPA_ONNX_MODEL_DIR`, `SHERPA_ONNX_VITS_DATA_DIR` |
| Service / port | none |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-tts-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-sherpa-onnx:v2026.240.0119'
```

Then, inside the built image (or on a dev host):

```bash
~/.local/share/sherpa-onnx/runtime/bin/sherpa-onnx-offline-tts \
  --vits-model=$SHERPA_ONNX_MODEL_DIR/vits-piper-en_US-lessac-high/en_US-lessac-high.onnx \
  --vits-tokens=$SHERPA_ONNX_MODEL_DIR/vits-piper-en_US-lessac-high/tokens.txt \
  --vits-data-dir=$SHERPA_ONNX_VITS_DATA_DIR \
  --output-filename=/tmp/out.wav "hello"
```

Three things about that invocation, all measured against the pinned runtime
(`SHERPA_VERSION: v1.12.23`): **every option takes the `=` form**, the output flag is
**`--output-filename`** — there is no `--output-wav` (it exits 255 with
`Invalid option --output-wav=<your path>`) — and **`--vits-data-dir` is required**: the
piper voice speaks espeak-ng phonemes, so leaving it empty exits 255 with `Not a model
using characters as modeling unit. Please provide --vits-lexicon if you leave
--vits-data-dir empty`. That data ships inside the voice tarball this candy bakes and is
exported as `$SHERPA_ONNX_VITS_DATA_DIR`, so the invocation above produces a WAV with no
network fetch.

The candy's `plan:` asserts the engine binary, its executability, the two shared
libraries, the voice model, its tokens file, and the espeak-ng phoneme data all ship in
the image — and runs that same synthesis invocation as a `check:`, so a voice that cannot
synthesise fails the plan instead of passing every check.

## Layout

- `charly.yml` — the `sherpa-onnx:` candy entity (the `env:`, the
  `SHERPA_VERSION` var, the runtime + voice download steps, and the `check:`
  probes) and the embedded `sherpa-onnx-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-tools:sherpa-onnx`
- Speech-to-text: `/charly-tools:whisper`
- Alternative TTS: `/charly-tools:sag`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
