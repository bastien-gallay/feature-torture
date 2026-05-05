# .feature-torture.md (project config) — daily-ops

Project configuration for the feature-torture skill on `daily-ops`.

> **Location now:** `.personal/feature-torture/config.md` (gitignored).
> **Promotion path:** project root as `.feature-torture.md` once stable.

## Paths

| Placeholder | Value |
|---|---|
| `{{PROJECT_NAME}}` | `daily-ops` |
| `{{ROADMAP_PATH}}` | `roadmap.md` (project root, post-2026-05-02 promotion) |
| `{{PICK_SCRIPT}}` | none — pick by hand |
| `{{ARCH_DOCS_PATH}}` | `docs/` |
| `{{REPORTS_DIR}}` | `.personal/feature-torture/reports/` |
| `{{UNSTARTED_RULE}}` | line begins with `- [ ]`; `- [/]` is in-progress (treat as unstarted for picking purposes); `- [x]` is done |

## Project frame

**One-line stance:** Python CLI + Claude-Code skill adapters for
start-of-day / end-of-day rituals. The CLI owns state writes; skills
are thin LLM-driven adapters that route through CLI subcommands.

**Audience:** the maintainer (single-user tool, dogfooded daily).

**Non-negotiables**:

- Skill ↔ CLI boundary is contract-tested.
- Daily files (`.personal/YYYY-MM-DD-today.md`, `TODO.md`, `rollup.md`)
  are the source of truth; CLI mutations preserve hand-edits.
- markdown-lint clean output, since rituals run `markdownlint --fix`
  on every write.

## Roadmap conventions

- **Status markers:** `- [ ]` not started · `- [x]` done · `- [/]`
  in-progress (used by the `whatsnext` action menu).
- **Feature ID shape:** absent — backlog items are titled with a
  bold lead phrase. Synthesise an `F-<slug>` from the title.
- **Picker rule:** select rows matching `- [ ]` under any `## Phase`
  heading.
