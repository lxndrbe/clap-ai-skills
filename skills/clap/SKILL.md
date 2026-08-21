---
name: clap
description: >
  Use when writing, editing, debugging, or reviewing CLAP plugins (C, C++, Rust,
  or any binding) — clap_entry, factory, process(), params, GUI, ports, state,
  thread-safety, naming, and clap-validator. Triggers: CLAP, .clap, clap-sys,
  clap-validator, audio plugin ABI, clap.gui, clap.params.
---

# CLAP Development Skill

For building, debugging, or reviewing audio plugins that target
[CLAP](https://cleveraudio.org/) (CLever Audio Plugin).

**Authority:** C headers in [free-audio/clap](https://github.com/free-audio/clap)
(`include/clap/`). This skill is a workflow. Header comments win on thread
specs, lifetimes, and ABI. All strings are UTF-8.

## Workflow

1. Read the CLAP version the project compiles against:
   `CLAP_VERSION_MAJOR/MINOR/REVISION` in `clap/version.h`. This skill targets
   CLAP **1.x**. A plugin built against `1.x` loads in any `1.y` host
   (`clap_version_is_compatible` → `v.major >= 1`). Draft extensions
   (`include/clap/ext/draft/`) can change.
2. After edits: compile, then run [clap-validator](https://github.com/robbert-vdh/clap-validator)
   on the built `.clap`. Do not claim the plugin works without that report.
3. Validator catches ABI and thread bugs. GUI attach, param mapping, presets,
   and port names also need a real host scan (Bitwig, REAPER, Ardour) when possible.

Most "host won't load it" / "validator fails" / "crackles on the audio thread"
questions: [threading.md](reference/threading.md),
[entry-point.md](reference/entry-point.md),
[process.md](reference/process.md),
[validation.md](reference/validation.md).

## Reference files

Skim the matching file *before* building in that area, not only when stuck.

| File | Read when… |
|---|---|
| [setup.md](setup.md) | Starting a project / wiring headers or the build. |
| [entry-point.md](reference/entry-point.md) | `clap_entry`, init/deinit, factory, scanning. |
| [extensions.md](reference/extensions.md) | Querying or implementing host/plugin extensions. |
| [threading.md](reference/threading.md) | Thread spec or lifecycle state machine. |
| [process.md](reference/process.md) | `process()`, events, audio buffers, process status. |
| [params.md](reference/params.md) | Parameters, gestures, flush, cookies. |
| [ports.md](reference/ports.md) | Audio I/O or note ports / dialects. |
| [gui.md](reference/gui.md) | Embedded or floating GUI (`clap.gui`). |
| [state.md](reference/state.md) | Save/load, presets, project recall. |
| [naming-and-versioning.md](reference/naming-and-versioning.md) | Plugin ids, versions, feature strings, ABI. |
| [validation.md](reference/validation.md) | clap-validator failures. |
| [bindings.md](reference/bindings.md) | Rust (`clap-sys`) or other languages. |
| [tools-install.md](tools-install.md) | Installing clap-validator. |

## CLAP in 30 seconds

A plugin is a shared library (`.clap`) exporting one symbol:

```c
CLAP_EXPORT extern const clap_plugin_entry_t clap_entry;
```

Three objects do the work:

- **`clap_plugin_entry`** — DSO entry: `init` / `deinit` / `get_factory`.
- **`clap_plugin_factory`** — creates instances; `get_factory(CLAP_PLUGIN_FACTORY_ID)` (`"clap.plugin-factory"`) returns it.
- **`clap_plugin`** — one instance. Lifecycle:

```
init → activate → start_processing → process … → stop_processing → deactivate → destroy
```

`reset` may run while active (audio thread): full DSP reset, params unchanged.

Host and plugin talk through **extensions** (C interfaces). Query with
`get_extension`; null-check. Field name is `get_extension`, not `extension`.

```c
const clap_host_log *log = host->get_extension(host, CLAP_EXT_LOG);
if (log) log->log(host, CLAP_LOG_INFO, "hello");
```

Every method has a thread spec (`[main-thread]`, `[audio-thread]`,
`[thread-safe]`). Wrong thread → crashes, xruns, or validator failure.

## Documentation

Headers under `include/clap/` — `entry.h`, `plugin.h`, `host.h`, `process.h`,
`events.h`, `factory/*.h`, `ext/*.h`. Raw:

```
https://raw.githubusercontent.com/free-audio/clap/main/include/clap/<path>
```

Extension map: https://github.com/free-audio/clap#learn-about-clap
