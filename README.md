# clap-ai-skills

Tool-agnostic [AI skill](https://github.com/anthropics/skills) for [CLAP](https://cleveraudio.org/) (CLever Audio Plugin) development.

Not an official CLAP project. Not affiliated with Bitwig or [free-audio](https://github.com/free-audio/clap). The C headers are the spec; this pack is a compact agent loadout.

A plain `skills/` folder. No editor plugin, no vendor lock-in. Point any assistant at `skills/clap/SKILL.md`.

## Install

```sh
git clone https://github.com/lxndrbe/clap-ai-skills.git
```

Then one of:

| Agent | How |
|-------|-----|
| Any (AGENTS.md) | Copy the one-liner from this repo's `AGENTS.md` into yours. |
| Claude Code / Cursor / Codex / similar | Clone into the agent's skills dir, or symlink `skills/clap`. |
| Grok | Copy or symlink `skills/clap` into the project or user skills folder. |

The skill auto-triggers on CLAP work if the agent reads `SKILL.md` frontmatter (`name` + `description`). If it doesn't, tell it:

```text
Read skills/clap/SKILL.md and follow it.
```

## Structure

```
skills/
  clap/
    SKILL.md                    # entry — read this first
    setup.md                    # headers, build, install paths
    tools-install.md            # clap-validator + optional tools
    reference/
      entry-point.md            # clap_entry, init/deinit, factory, scanning
      extensions.md             # extension model + catalog
      threading.md              # thread specs + lifecycle
      process.md                # process(), events, audio buffers
      params.md                 # parameters, gestures, flush
      ports.md                  # audio-ports + note-ports
      gui.md                    # clap.gui embed / floating
      state.md                  # save/load
      naming-and-versioning.md  # ids, versions, ABI, features
      validation.md             # clap-validator + common failures
      bindings.md               # Rust and other languages
```

## Scope

- CLAP **1.x** (ABI-stable). Headers carry the exact version (`CLAP_VERSION_*` in `clap/version.h`). Upstream as of 2026-08 is **1.2.10**.
- The **format and C ABI** — not a framework (JUCE, iPlug2, nice-plug, AURA, …). If you use a framework, follow it; threading, lifecycle, and extension ids still apply underneath.
- Draft extensions live in `include/clap/ext/draft/` and may change. This skill flags them; do not treat drafts as stable.

## Authority

1. Headers: https://github.com/free-audio/clap (`include/clap/`)
2. This skill (workflow + common failure modes)
3. Blog posts last

Header comments carry thread specs and lifetimes. When in doubt, read the header.

## License

MIT — see [LICENSE](LICENSE).

CLAP itself is a separate project (MIT, [free-audio/clap](https://github.com/free-audio/clap)). This repo does not vendor the headers.
