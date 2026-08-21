# clap-ai-skills

[AI skill](https://github.com/anthropics/skills) for [CLAP](https://cleveraudio.org/) (CLever Audio Plugin).

Not official CLAP. Not affiliated with Bitwig or [free-audio](https://github.com/free-audio/clap). Not a vendor framework. Headers are the spec.

## What's inside

- `skills/clap/` — CLAP format and C ABI: entry point, factory, process, events, params, ports, state, threading, extensions, clap-validator. Layout: `SKILL.md` + `setup.md` + `tools-install.md` + `reference/`.

## Install

```sh
git clone https://github.com/lxndrbe/clap-ai-skills.git
```

Point the agent at `skills/clap/` (symlink or copy into its skills dir). Or add this to the project's `AGENTS.md`:

```markdown
When working on CLAP plugins, read `skills/clap/SKILL.md` first and follow its reference files.
```

If it does not auto-trigger: `Read skills/clap/SKILL.md and follow it.`

## Scope

- CLAP **1.x** (ABI-stable). Version is in `clap/version.h`. Upstream as of 2026-08: **1.2.10**.
- The **format**, not JUCE / iPlug2 / nice-plug / any house stack. Frameworks still sit on this ABI; thread specs and extension ids still apply.
- Draft extensions (`include/clap/ext/draft/`) may change.

## Authority

1. Headers: https://github.com/free-audio/clap (`include/clap/`)
2. This skill (workflow + common failures)
3. Blog posts last

## License

MIT — [LICENSE](LICENSE).

CLAP is a separate project (MIT, [free-audio/clap](https://github.com/free-audio/clap)). This repo does not vendor the headers.
