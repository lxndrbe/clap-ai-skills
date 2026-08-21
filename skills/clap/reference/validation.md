# Validation

Install and flags: [tools-install.md](../tools-install.md).

```sh
clap-validator validate path/to/plugin.clap
```

Treat a failure as a plugin bug.

## Common failures

| Failure | Cause | Fix |
|---------|-------|-----|
| host won't load the DSO | missing/incorrect `clap_entry`, wrong `clap_version` | exported symbol + `CLAP_VERSION` init |
| thread-safety violation | `[main-thread]` on audio thread (or vice versa) | bounce via `request_callback` / `on_main_thread`; [threading.md](threading.md) |
| missing ports | instrument with no `clap.note-ports`, or effect with no `clap.audio-ports` | add the extension the DSP uses; feature strings do **not** imply an extension |
| blank mandatory fields | descriptor `id` / `name` empty | fill them ([naming-and-versioning.md](naming-and-versioning.md)) |
| state not round-tripping | save/load mismatch | save → load → save; [state.md](state.md) |
| GUI attach fail | skipped embed path / wrong window API | [gui.md](gui.md) |
| param not automatable | events without `PARAM_VALUE` / missing gestures | [params.md](params.md) |

## Also do a real host scan

Validator covers ABI and threads. A real host (Bitwig, REAPER, Ardour) catches
GUI attach, param mapping, preset discovery, port naming.
