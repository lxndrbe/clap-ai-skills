# GUI (`clap.gui`)

Id: `CLAP_EXT_GUI` = `"clap.gui"`. All plugin GUI methods are `[main-thread]`
unless noted. Do not touch the GUI from `process()`.

Two modes:

1. **Embedded** (host parent window) — support this. Every host does.
2. **Floating** (plugin-owned window) — fallback when embed is impossible.

## Window APIs

| API | Constant | Size | Embed |
|-----|----------|------|-------|
| Win32 | `"win32"` | physical px | `SetParent` |
| Cocoa | `"cocoa"` | logical px | NSView; do **not** call `set_scale` |
| UIKit | `"uikit"` | logical px | do **not** call `set_scale` |
| X11 | `"x11"` | physical px | XEmbed |
| Wayland | `"wayland"` | physical px | embed **not** supported — use floating |

## Show sequence (from the header)

1. `is_api_supported(api, is_floating)`
2. `create(api, is_floating)` — allocates; does not show
3. Floating: `set_transient(parent)`, `suggest_title`
4. Embedded: `set_scale` (physical APIs only), `can_resize`,
   `set_size` (restore) or `get_size`, then `set_parent`
5. `show` / `hide` (hide does not destroy; stop paint timers)
6. `destroy` when done

`get_preferred_api` is a hint. Hosts may ignore it. Assign `*api` to a
`CLAP_WINDOW_API_*` constant pointer — do not copy the string into a buffer.

## Resize (embedded)

Plugin-initiated: `clap_host_gui->request_resize(w, h)`. `true` = accepted;
host need not call `set_size`. If not on the main thread, `true` only means
acknowledged — host may later `set_size` to revert.

User drag: only if `can_resize`. Host calls `adjust_size` (closest usable
size, does not apply) then `set_size`.

`get_resize_hints` — aspect ratio / axis constraints. Host
`resize_hints_changed()` if they change (`[thread-safe & !floating]`).

## Host GUI

`request_show` / `request_hide` / `closed(was_destroyed)` are `[thread-safe]`.
If `closed(..., true)`, the host must call `plugin_gui->destroy()`.

## Threading reminder

Paint, widget state, and toolkit calls: main thread. Param changes from the
GUI go through `request_flush` / `request_process` and events — [params.md](params.md).
