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

## Catalog (key extensions)

Fundamental — implement these for a usable plugin:

| Extension | Id string | Role | Detail |
|-----------|-----------|------|--------|
| state | `clap.state` | save/load | [state.md](state.md) |
| params | `clap.params` | parameters | [params.md](params.md) |
| note-ports | `clap.note-ports` | note I/O | [ports.md](ports.md) |
| audio-ports | `clap.audio-ports` | audio I/O | [ports.md](ports.md) |
| render | `clap.render` | realtime / offline | |
| latency | `clap.latency` | report latency | |
| tail | `clap.tail` | processing tail | |
| gui | `clap.gui` | GUI | [gui.md](gui.md) |

Support:

| Extension | Id string | Role |
|-----------|-----------|------|
| thread-check | `clap.thread-check` | assert current thread |
| thread-pool | `clap.thread-pool` | host thread pool |
| log | `clap.log` | host-aggregated logging |
| timer-support | `clap.timer-support` | timers |
| posix-fd-support | `clap.posix-fd-support` | I/O handlers |

Deeper host integration: preset-load, preset-discovery (factory),
param-indication, note-name, remote-controls, context-menu, voice-info,
track-info, surround, ambisonic, audio-ports-config, audio-ports-activation,
configurable-audio-ports, state-context.

Draft (`include/clap/ext/draft/`, may change): tuning, triggers,
transport-control, resource-directory, extensible-audio-ports, webview, …

Authoritative list: https://github.com/free-audio/clap#learn-about-clap

Third-party example: `cockos.reaper_extension` (REAPER API).

## Custom extensions

You can define and share your own ([naming-and-versioning.md](naming-and-versioning.md)):

- unique id
- version in the id if the ABI breaks
- every method documented with a thread spec
