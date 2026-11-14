# 🚢 web-game-shipping-loop

> **Turn a runnable browser-game prototype into something you can ship.**
> An agent skill: deterministic probes to guard against regressions, triage of feel / performance / device issues from the user's own words, single-file size budgets, bilingual UI, and GitHub/Gitee publishing with Pages.

## Install

```bash
git clone https://github.com/EternalNight996/web-game-shipping-loop.git \
  ~/.dsh/skills/web-game-shipping-loop
# or any host that reads a skills directory
npx skills add https://github.com/EternalNight996/web-game-shipping-loop
```

## What it encodes

- Gate 1: the probe JSON must be green before you touch anything
- Gate 2: the build must read the freshly minified bundle (a stale bundle silently swallows every change)
- Gate 3: one problem class per round, so you can attribute the result
- Nine hard rules, each from a real incident: stale bundle, `stopPropagation` eating Space, per-shot point lights forcing full material recompiles, one giant InstancedMesh losing frustum culling, probes contaminated by level geometry, a translator named `t`, non-ASCII PowerShell scripts, "looks fine" as acceptance, and claiming a release without checking the live URL

## Verify

```bash
node <skills>/dsh-skill-forge/validate-skill.mjs SKILL.md   # expect PASS (errors=0, warns=0)
```

## License

MIT © mengbot
