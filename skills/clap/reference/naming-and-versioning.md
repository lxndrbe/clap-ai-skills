# Naming & versioning

## Plugin id

Arbitrary string, unique to your plugin. Reverse URI:

```
com.u-he.diva
```

Descriptor `id` and `name` are mandatory and must not be blank.
`vendor`, `url`, `manual_url`, `support_url`, `version`, `description` may be
null or blank; blank is safer than null.

## Plugin version string

Arbitrary, but hosts compare versions. Form most hosts understand:

```
MAJOR(.MINOR(.REVISION)?)?( (Alpha|Beta) XREV)?
```

Examples: `1.4.4`, `2.0`, `1.0 Beta 3`.

## Extension ids

Stable macros expand to **dot-separated** strings, e.g. `CLAP_EXT_PARAMS` →
`"clap.params"`. Custom ids: unique, and include a version if the ABI breaks
(`"com.vendor.ext/2"`).

## ABI versioning

- `0.x.y` — development, API/ABI unstable.
- `1.x.y` — stable. Plugin built against `1.x` loads in any `1.y` host.
- `clap_version_is_compatible(v)` → `v.major >= 1`.

Source of truth: `CLAP_VERSION_MAJOR/MINOR/REVISION` and `CLAP_VERSION` in
`clap/version.h`.

## Feature strings

Descriptor `features` is a **null-terminated** array. Hosts classify with it.
No spaces; use `-`. Custom features: `$namespace:$feature`.

Categories (from `clap/plugin-features.h`):

- `instrument`, `audio-effect`, `note-effect`, `note-detector`, `analyzer`
- Sub: `synthesizer`, `sampler`, `drum`, `drum-machine`, `filter`,
  `equalizer`, `compressor`, `delay`, `reverb`, `utility`, `stereo`, `mono`, …

Standard strings drive host UI and search. Pick one primary category.
