# Project setup

## Headers

CLAP is header-only + a C ABI. Vendor or submodule:

```sh
git submodule add https://github.com/free-audio/clap third-party/clap
```

Compile against `include/clap/clap.h`. Include `clap/all.h` only if you use draft extensions.

## C

Start from `src/plugin-template.c` in the clap repo:
https://github.com/free-audio/clap/blob/main/src/plugin-template.c

Build a shared library with the `.clap` extension. Include `clap/clap.h`;
do not re-declare CLAP structs.

- Linux: `.clap` (shared object)
- Windows: `.clap` (DLL renamed / linked as such). `CLAP_EXPORT` is required
  so `clap_entry` is visible.
- macOS: `.clap` **bundle** (not a flat dylib named `.clap`)

## C++

Headers wrap themselves in `extern "C"`. Include them directly. No codegen.

## Other languages

See [bindings.md](reference/bindings.md).

## Install path

Hosts scan these recursively for `.clap` files/bundles, plus `CLAP_PATH`
(same separator as the OS `PATH`: `:` on Unix, `;` on Windows):

| OS | Paths |
|----|-------|
| Linux | `~/.clap`, `/usr/lib/clap` |
| Windows | `%COMMONPROGRAMFILES%\CLAP`, `%LOCALAPPDATA%\Programs\Common\CLAP` |
| macOS | `/Library/Audio/Plug-Ins/CLAP`, `~/Library/Audio/Plug-Ins/CLAP` |

On 64-bit Windows, `%COMMONPROGRAMFILES%` is typically `C:\Program Files\Common Files`.

## Reference examples

- Plugins: https://github.com/free-audio/clap-plugins
- Host: https://github.com/free-audio/clap-host
- Helpers: https://github.com/free-audio/clap-helpers
