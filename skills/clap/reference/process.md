# process()

`plugin->process` is `[audio-thread & active & processing]`. All pointers in
`clap_process_t` and nested structs are valid only until `process` returns.

Fields (see `clap/process.h`, do not re-declare): `steady_time` (`-1` if
unknown; else `>= 0` and grows by at least `frames_count`), `frames_count`,
`transport` (null = free-running), `audio_inputs` / `audio_outputs` + counts,
`in_events` / `out_events`.

Audio buffer count must match `clap_plugin_audio_ports->count()`. Index maps
to `audio_ports->get()`. Inputs are read-only. 32-bit audio is required;
64-bit is optional (see [ports.md](ports.md)).

`clap_audio_buffer_t`: either `data32` or `data64` is set (not both). Check
which. `constant_mask` bit *n* means channel *n* is a constant (use index 0);
it is a hint — still fill outputs completely or the host may play garbage.

## Sample-accurate loop

Do **not** apply all input events at frame 0, then render the whole block.
Walk `in_events` in time order. At sample `i`, dispatch every event with
`header.time == i`, then render `[i, next_event.time)`. Ignore events whose
`space_id` is not `CLAP_CORE_EVENT_SPACE_ID` unless you registered a custom
space.

Canonical loop: `my_plug_process` in
https://github.com/free-audio/clap/blob/main/src/plugin-template.c

## Return status

| Status | Meaning |
|--------|---------|
| `CLAP_PROCESS_ERROR` | Failed; host discards output |
| `CLAP_PROCESS_CONTINUE` | Keep processing |
| `CLAP_PROCESS_CONTINUE_IF_NOT_QUIET` | Keep going if output is not quiet |
| `CLAP_PROCESS_TAIL` | Use `clap.tail` to decide |
| `CLAP_PROCESS_SLEEP` | Idle until the next event or input change |

## Events

Input list is read-only, **sorted by sample offset** (`header.time`).
Output list: insert in **sample-sorted order** via `try_push` (returns false
if the host queue is full — do not retry in a spin loop).

Every event starts with `clap_event_header_t`: `size` (bytes of the whole
event, memcpy-able), `time`, `space_id` (`CLAP_CORE_EVENT_SPACE_ID` = 0),
`type`, `flags`.

Core types:

| Type | Struct | Notes |
|------|--------|-------|
| `NOTE_ON/OFF/CHOKE/END` | `clap_event_note` | Preferred over raw MIDI. Velocity 0 on NOTE_ON is still NOTE_ON. Plugin sends `NOTE_END` so the host can free voices. |
| `NOTE_EXPRESSION` | `clap_event_note_expression` | Absolute value, not cumulative |
| `PARAM_VALUE` | `clap_event_param_value` | Sets the param; heard = value + mod |
| `PARAM_MOD` | `clap_event_param_mod` | Modulation amount |
| `PARAM_GESTURE_BEGIN/END` | `clap_event_param_gesture` | Wrap user gestures |
| `TRANSPORT` | `clap_event_transport` | Intra-block transport/tempo |
| `MIDI` / `MIDI2` / `MIDI_SYSEX` | matching structs | Do not duplicate a note as both `NOTE_ON` and `MIDI` |

Note addressing is the tuple `(port, channel, key, note_id)`. `-1` is a
wildcard. Match voices against the whole tuple.

Flags: `CLAP_EVENT_IS_LIVE`, `CLAP_EVENT_DONT_RECORD` (use DONT_RECORD when a
MIDI CC drove a param, so the host records the CC not the param).

Sysex `buffer` lifetime is this `process()` call (or this `try_push`). Copy
if you keep it.

## Audio-thread rules (process)

No malloc, no mutex, no blocking I/O, no GUI, no logging that allocates.
Read params from the event list / a lock-free queue filled on the main thread.
See [threading.md](threading.md) and [params.md](params.md).
