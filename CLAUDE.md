# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working on this repository.

## What this repo is

`feature-torture` is a **Claude Code skill package**, not an application. There is no build step, no runtime code, no tests of code. The "product" is one prompt + supporting docs that get installed as a skill into Claude Code (or any compatible host) and loaded as a prompt at session time:

- `skills/feature-torture/SKILL.md` — the canonical prompt with frontmatter (name, description). Defines the diamond-loop process, the 6-label verdict palette, the closed technique list, the post-converge cross-label challenge, and the required output sections.
- `skills/feature-torture/tests.md` — corpus criteria the human uses to judge whether a prompt edit improved or degraded the skill. Per-report rules + aggregate distribution + verdict-correctness gates.
- `skills/feature-torture/improvements-queue.md` — open backlog of prompt-edit candidates with triage table.
- `examples/` — five real config fixtures covering five different roadmap shapes (markdown tables, task lists, FR-prose multi-axis, suggestion-list-no-status).

The skill reads project config from `.feature-torture.md` at the project root, falling back to `.personal/feature-torture/config.md`, falling back to inferring values from `AGENTS.md` / `README.md`.

## Distribution model

Three install channels, mirroring `bfw`:

1. **Claude Code plugin** — version-driven. `/plugin marketplace update feature-torture` reads `.claude-plugin/plugin.json` and refreshes. **Skipping a version bump is a silent freeze**; `just release` enforces the bump.
2. **vercel-labs/skills** — clone-driven. Always pulls latest `main`.
3. **Claude Desktop / claude.ai web** — `.skill` archive. Built by `just package`, attached to a GitHub Release.

## Editing rules

- **The prompt is the product.** Treat `skills/feature-torture/SKILL.md` like a shipped binary — every edit is a release-eligible change.
- **Run dogfood before claiming an edit improved the prompt.** `tests.md` defines what a passing run looks like. The corpus is currently spread across 4 repos (lucid-lint, daily-ops, secondary-corpus, jira_analytics, rhetorix) — keep their `.personal/feature-torture/reports/` dirs alive as fixtures.
- **No backward-compat hacks.** If a placeholder gets renamed (e.g. `UNSTARTED_PATTERN` → `UNSTARTED_RULE`), update every example config in the same commit. The release machinery is not the place to bridge config schemas.
- **CHANGELOG entries describe behaviour, not files.** Readers care about what the skill *does* differently, not which markdown lines changed.

## Release flow

`just release X.Y.Z` (after `## vX.Y.Z` header in CHANGELOG.md) → bumps both manifests → commits → tags. `just package` builds the `.skill` archive into `dist/`. `docs/RELEASING.md` has the full process notes.
