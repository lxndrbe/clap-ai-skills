# Language bindings

CLAP is a C ABI. Any language with FFI can use it. Bindings expose the raw C
structs. Thread specs, lifetimes, and ABI rules in the headers still apply —
a binding cannot relax "no allocation on the audio thread".

## Rust

[`clap-sys`](https://github.com/glowcoil/clap-sys) — raw FFI. Use the current
crates.io version (do not copy a stale pin from a tutorial).

```toml
[dependencies]
clap-sys = "0.5" # check crates.io
```

Higher-level frameworks sit on `clap-sys`. Follow the framework; this
skill's rules still apply underneath.

## C / C++

Headers directly. They wrap themselves in `extern "C"` for C++.

## Others

Listed upstream: https://github.com/free-audio/clap#programming-language-bindings

- Delphi: CLAP-for-Delphi
- Zig: clap-zig-bindings
- Ada: CLAP for Ada
