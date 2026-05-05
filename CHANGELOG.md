# Changelog

## v0.1.0 — 2026-05-05

Initial promotion from `~/.claude/skills/feature-torture/` to a standalone repo with full distribution scaffolding (mirrors `bfw`).

### Added

- v3 prompt with 6-label verdict palette (👍 ship · ✂️ reshape · ⏸ park · 🧬 split · 👎 kill · 🤷 defer-decision).
- **Post-converge cross-label challenge** (mandatory; tests the two nearest neighbours and either flips the verdict or records refutations in *Choice*).
- **Start-state check as primary gate** — `git log` / branch grep / `gh pr list` / codebase grep run before committing to a torture, even when the marker says unstarted.
- `{{UNSTARTED_RULE}}` placeholder (prose rule, supersedes the single-axis `{{UNSTARTED_PATTERN}}`).
- Closed technique list (18 techniques across 4 phases) with audience-fit column.
- 5 example configs covering 5 roadmap shapes: lucid-lint (markdown table with `☐`/`🚧`/`✅`), secondary-corpus (`- [ ]` task list), daily-ops (`- [ ]` + `- [/]` in-progress), jira_analytics (FR-prose, multi-axis status), rhetorix (suggestion-list, no status).
- `tests.md` corpus criteria (per-report, aggregate, verdict-correctness gates).
- `improvements-queue.md` (10 items, 1 closed, triage table).

### Pre-promotion validation (8 reports across 4 repos)

Verdict distribution: 👍 25 % · ✂️ 12.5 % · ⏸ 25 % · 🧬 25 % · 👎 12.5 % · 🤷 0 %. All six labels populated except 🤷 (rare-by-design). Start-state check caught 2 stale roadmap rows. secondary-corpus ✂️ ship-rate 7/8 (88 %) within one week confirmed bucket-naming was correct.
