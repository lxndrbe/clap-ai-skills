# Ports

Change port configuration only while **deactivated**. While active, latency
and ports are frozen ([threading.md](threading.md)).

If you omit an extension, you have **no** ports of that kind.

## Audio (`clap.audio-ports`)

32-bit audio is required; 64-bit is optional (`CLAP_AUDIO_PORT_SUPPORTS_64BITS`
/ `PREFERS_64BITS` / `REQUIRES_COMMON_SAMPLE_SIZE`).

Main input and main output: at most one each, **index 0**, flag
`CLAP_AUDIO_PORT_IS_MAIN`.

`port_type`: `"mono"`, `"stereo"`, or an extension type (`surround`,
`ambisonic`). Null/empty = unspecified.

`in_place_pair`: other port id if the host may reuse the same buffer;
otherwise `CLAP_INVALID_ID`.

`count` / `get` are `[main-thread]`. Host `rescan` flags that change list,
channel count, type, flags, or in-place pair require `!active`.

## Notes (`clap.note-ports`)

Dialects (bitfield `supported_dialects`, one `preferred_dialect`):

| Flag | Events |
|------|--------|
| `CLAP_NOTE_DIALECT_CLAP` | `clap_event_note` / note expression |
| `CLAP_NOTE_DIALECT_MIDI` | `clap_event_midi` |
| `CLAP_NOTE_DIALECT_MIDI_MPE` | MIDI + MPE |
| `CLAP_NOTE_DIALECT_MIDI2` | `clap_event_midi2` |

Prefer CLAP note events. Do not send the same note twice (NOTE_ON **and** MIDI).

Host `supported_dialects()` says what the host speaks. `RESCAN_ALL` only when
deactivated; `RESCAN_NAMES` can run live.

Port `id` is stable. Input and output id spaces may overlap.
