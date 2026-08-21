# Installing CLAP tools

## clap-validator (required)

Static + dynamic checks on a built `.clap`: thread-safety, missing extensions, ABI.

```sh
cargo install clap-validator
clap-validator validate path/to/plugin.clap
```

Flags and current behavior: https://github.com/robbert-vdh/clap-validator

Treat a validator failure as a plugin bug, not a tool bug.

## Headers

No install. Vendor or submodule — [setup.md](setup.md). Only a C/C++ compiler (or FFI) is required.

## Optional

- Real host: Bitwig, REAPER, Ardour (CLAP support)
- [clapdb](https://clapdb.tech/) — plugin/DAW support matrix
- [clap-wrapper](https://github.com/free-audio/clap-wrapper) — wrap CLAP as VST3/AU
