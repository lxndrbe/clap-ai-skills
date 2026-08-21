# State (`clap.state`)

Id: `CLAP_EXT_STATE` = `"clap.state"`. Persist params **and** non-param state
(project reload, duplicate, host presets).

```c
bool save(const clap_plugin_t *plugin, const clap_ostream_t *stream); // [main-thread]
bool load(const clap_plugin_t *plugin, const clap_istream_t *stream); // [main-thread]
```

`read`/`write` may be partial — loop. Return: `>0` bytes, `0` = EOF (istream
only), `-1` = error. Do not assume `FILE*`.

After a successful `load`:

- `clap_host_params.rescan` if params changed ([params.md](params.md))
- `clap_host_latency.changed` if latency changed
- If the new state needs port/latency/param-tree changes and the plugin is
  **active**, wait for deactivate (`host->request_restart()`). Apply breaking
  changes only while inactive.

`clap_host_state.mark_dirty()` `[main-thread]` — tell the host to save again.
A param-value change already implies dirty; use this for non-param state
(loaded sample, sequencer data).

## Context (optional)

`clap.state-context` distinguishes preset vs duplicate vs project. Implement
on top of `clap.state` if those paths need different blobs.

Draft: `resource-directory` — host folder for bulky extras (multisamples).

## Host rule

If the plugin has no state extension, the host must **not** save parameters
on its behalf. Round-trip test: save → load → save, compare.
