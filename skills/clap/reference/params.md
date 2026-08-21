# Parameters (`clap.params`)

The host treats the plugin as an atomic object and drives parameters.
The plugin keeps its audio processor and GUI in sync.

Id: `CLAP_EXT_PARAMS` = `"clap.params"`.

## Info

`clap_param_info_t`: stable `id` (never change it), `flags`, optional `cookie`,
`name`, `module` (path with `/` as tree separator — do not repeat module in
`name`), finite `min_value` / `max_value`, `default_value` in range.

Cookie: plugin may stash a pointer in `get_info` for fast process-time lookup.
Host may pass it or `NULL` — always handle `NULL`. Cookie dies on
`CLAP_PARAM_RESCAN_ALL` or destroy.

## How values move

Two paths, **not concurrent**:

1. Automation during `process()`
2. `clap_plugin_params.flush()` when not processing audio

`flush` is `[active ? audio-thread : main-thread]` and must not run
concurrently with `process()`. Prefer `process()` while active — `flush`
drops sample offsets.

Heard value = **param value + modulation amount**.

When the plugin changes a param (GUI knob), it must emit `CLAP_EVENT_PARAM_VALUE`
from `process()` or `flush()`. Wrap a user drag with
`CLAP_EVENT_PARAM_GESTURE_BEGIN` / `END`. From the GUI thread, call
`clap_host_params->request_flush()` or `host->request_process()` — never from
the audio thread (`request_flush` is `[thread-safe, !audio-thread]`).

MIDI-CC-driven param changes: set `CLAP_EVENT_DONT_RECORD` on the param event.

`get_value` / `get_info` / `count` / `value_to_text` / `text_to_value` are
`[main-thread]`. Hosts format display **only** through `value_to_text`.

## Flags (common)

`STEPPED`, `PERIODIC`, `HIDDEN`, `READONLY`, `BYPASS` (at most one; stepped
0/1), `AUTOMATABLE` (+ per-note/key/channel/port), `MODULATABLE` (+ same),
`REQUIRES_PROCESS`, `ENUM` (implies STEPPED; every integer in range has a
non-blank `value_to_text`).

Bypass must not decide whether the host calls `process()`; sleep inside
`process` if you want CPU back (`CLAP_PROCESS_SLEEP`).

Structural changes (max delay, FFT size): `host->request_restart()`, apply
on next activate.

## Rescan

`clap_host_params.rescan(flags)` `[main-thread]`:

- `VALUES` — values changed (e.g. preset); host does not record automation
- `TEXT` — value-to-text changed
- `INFO` — name/module/hidden/periodic
- `ALL` — add/remove params or critical fields (range, stepped, cookie, …).
  **Only while deactivated.** If active: `host->request_restart()` and wait.

Do not shrink ranges if you promised the sound will not change.

## Persist

Implement [state.md](state.md). Hosts must **not** invent a param-dump fallback
for plugins that skip `clap.state`.
