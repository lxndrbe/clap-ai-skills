# Entry point

The plugin DSO exports exactly one symbol:

```c
CLAP_EXPORT extern const clap_plugin_entry_t clap_entry;
```

## `clap_plugin_entry`

```c
typedef struct clap_plugin_entry {
    clap_version_t clap_version;   // = CLAP_VERSION
    bool (*init)(const char *plugin_path);
    void (*deinit)(void);
    const void *(*get_factory)(const char *factory_id);
} clap_plugin_entry_t;
```

- `clap_version` must be `CLAP_VERSION` (`clap/version.h`).
- `init(plugin_path)` — first call, before any other CLAP symbol. Fast (hosts
  scan this way). No GUI, no user interaction. Return `true` on success; on
  `false` the host must not call `deinit` or anything else. `plugin_path` is
  the DSO path (Linux/Windows) or the bundle (macOS).
- `deinit()` — frees what `init` allocated. After it, the DSO is reusable via
  another `init`.
- `get_factory(factory_id)` — requested factory or `NULL`. For plugins, answer
  `CLAP_PLUGIN_FACTORY_ID` (`"clap.plugin-factory"`) with a
  `clap_plugin_factory_t*`. **[thread-safe]**; may run concurrently.
- `init` / `deinit` may run on any thread, **not simultaneously** with each
  other or with any other CLAP symbol from this DSO.

### init/deinit multi-call (CLAP ≥ 1.2.0)

Hosts should pair `init`/`deinit`, but a host and a wrapper can both load the
same DSO. If `init` does non-trivial work, keep a static counter + mutex and
run the real work only while the counter is 0. Increment on `init`, decrement
on `deinit`.

## Factory

```c
typedef struct clap_plugin_factory {
    uint32_t (*get_plugin_count)(const struct clap_plugin_factory *factory);
    const clap_plugin_descriptor_t *(*get_plugin_descriptor)(
        const struct clap_plugin_factory *factory, uint32_t index);
    const clap_plugin_t *(*create_plugin)(
        const struct clap_plugin_factory *factory,
        const clap_host_t *host, const char *plugin_id);
} clap_plugin_factory_t;
```

- All factory methods are `[thread-safe]`. Scan must be fast.
- `create_plugin` returns an instance but must **not** use host callbacks —
  that is `clap_plugin->init` (full host access there).
- `plugin_id` is the descriptor `id` string ([naming-and-versioning.md](naming-and-versioning.md)).
- Descriptor is owned by the plugin; valid until `clap_entry.deinit()`.
- Host pointer is valid until after `plugin->destroy()`.

## Scan / discovery

Hosts recursively search the CLAP directories (and `CLAP_PATH`) for `.clap`
files. Paths: [setup.md](../setup.md). Fast `init` + correct descriptors is
the whole scan story.
