# Extensions

Most CLAP features are **extensions** — C interfaces queried by id string at runtime.

## The pattern

```c
const clap_host_log *log = host->get_extension(host, CLAP_EXT_LOG);
const clap_plugin_params *params = plugin->get_extension(plugin, CLAP_EXT_PARAMS);
```

The struct field is `get_extension` (the official README sometimes writes
`extension` as shorthand — that is not the ABI name).

- Host interfaces: `struct clap_host_xxx`
- Plugin interfaces: `struct clap_plugin_xxx`
- Extension id macro: `CLAP_EXT_XXX` → string like `"clap.params"` (dots, not slashes)
- Query returns `NULL` if unsupported — always null-check
- `get_extension` is `[thread-safe]` on both sides
- Plugin's returned pointer is valid from `plugin->init()` until `plugin->destroy()`
- Host's returned pointer is valid until after `plugin->destroy()`
- Forbidden to call `get_extension` before `plugin->init()`; allowed *during* `init`

## Catalog (implement what you need)

Ids below are the real strings (`CLAP_EXT_*` macros). Upstream map:
https://github.com/free-audio/clap#learn-about-clap
Which ones to ship: table in `SKILL.md`. Do not invent new `clap.*` ids.

| Extension | Id string | Role | Detail |
|-----------|-----------|------|--------|
| state | `clap.state` | save/load | [state.md](state.md) |
| params | `clap.params` | parameters | [params.md](params.md) |
| note-ports | `clap.note-ports` | note I/O | [ports.md](ports.md) |
| audio-ports | `clap.audio-ports` | audio I/O | [ports.md](ports.md) |
| latency | `clap.latency` | delay in samples; `get` is `[main-thread & (being-activated \| active)]`; change only during `activate` (else `request_restart`) | |
| tail | `clap.tail` | decay length; `>= INT32_MAX` = infinite; `host_tail->changed` is `[audio-thread]` | |
| render | `clap.render` | host says realtime vs offline; skip if DSP is identical | |
| gui | `clap.gui` | host window embed / floating | [gui.md](gui.md) |
| voice-info | `clap.voice-info` | voice count for polyphonic modulation | |

Support:

| Extension | Id string | Role |
|-----------|-----------|------|
| thread-check | `clap.thread-check` | assert current thread |
| thread-pool | `clap.thread-pool` | host thread pool |
| log | `clap.log` | host-aggregated logging |
| timer-support | `clap.timer-support` | timers |
| posix-fd-support | `clap.posix-fd-support` | I/O handlers |

Deeper host integration: preset-load, preset-discovery (factory),
param-indication, note-name, remote-controls, context-menu,
track-info, surround, ambisonic, audio-ports-config, audio-ports-activation,
configurable-audio-ports, state-context.

Draft (`include/clap/ext/draft/`, may change): tuning, triggers,
transport-control, resource-directory, extensible-audio-ports, webview, …

Third-party example: `cockos.reaper_extension` (REAPER API).

## Custom extensions

Do **not** mint ids under `clap.*` — that namespace is the spec.
A private extension (`com.vendor.…`) is allowed only if a host you target
documents it. Version the id if that ABI breaks. Every method needs a
thread spec ([naming-and-versioning.md](naming-and-versioning.md)).
