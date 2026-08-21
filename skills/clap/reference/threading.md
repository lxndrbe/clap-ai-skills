# Threading

Every CLAP method carries a thread spec. Violating it = crashes, xruns, or a
validator failure. This is the #1 source of CLAP bugs.

## Thread classes

- `[main-thread]` — the thread that called `create_plugin` and drives
  `init` / `activate` / `deactivate` / `destroy`. GUI, most param queries, and
  most host calls live here.
- `[audio-thread]` — `process()` (and `start_processing` / `stop_processing` /
  `reset`). Real-time: no allocation, no lock, no blocking, no GUI, no syscalls.
- `[thread-safe]` — callable from any thread concurrently (factory,
  `get_extension`, `request_*`).

Compound specs — `[main-thread & !active]`, `[audio-thread & active & processing]` —
declare the lifecycle state in force when the method is legal.

## Plugin lifecycle state machine

```
created → init → activate → start_processing → process (loop) → stop_processing → deactivate → destroy
                          ↑ active                                ↓
                          └───────────────────────────────────────┘
```

- `active` — between `activate` and `deactivate`
- `processing` — between `start_processing` and `stop_processing`
- `reset` — `[audio-thread & active]`; full DSP reset (filters, voices, LFOs);
  parameter values stay. `steady_time` may jump backward.

Latency and port configuration must stay constant while active. Change them
only while deactivated (or `host->request_restart()` and wait).

`activate` receives `sample_rate`, `min_frames_count`, `max_frames_count`.
Those stay valid until `deactivate`. Allocate DSP buffers here, not in
`process()`.

## Bouncing to the main thread

The audio thread must never call `[main-thread]` methods directly:

```c
host->request_callback(host);   // [thread-safe]
```

schedules `plugin->on_main_thread(plugin)` on the main thread (usually within
~33 ms, **not guaranteed** — do not assume timing). Under load the GUI thread
can starve.

Other `[thread-safe]` host calls: `request_restart`, `request_process`.

## Debugging tools

- `CLAP_EXT_THREAD_CHECK` (`clap.thread-check`) — ask the host which thread
  you are on; assert in debug builds.
- `CLAP_EXT_LOG` (`clap.log`) — log from anywhere; the host aggregates.
  Severities include `CLAP_LOG_HOST_MISBEHAVING` / `CLAP_LOG_PLUGIN_MISBEHAVING`.

See [validation.md](validation.md) for how clap-validator catches violations.
