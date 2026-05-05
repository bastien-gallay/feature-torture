# .feature-torture.md (project config) — rhetorix

> **Note**: this project has no formal roadmap with status markers.
> `docs/evol.md` is a "Feature Suggestions" list — every numbered
> H3 (`### N. <Title>`) is a suggestion. There is no started/unstarted
> distinction *in the document*; the agent must perform a start-state
> check (commits, branches, code grep) to determine whether a
> suggestion has begun in code.

## Paths

| Placeholder | Value |
|---|---|
| `{{PROJECT_NAME}}` | `rhetorix` |
| `{{ROADMAP_PATH}}` | `docs/evol.md` (suggestion list, EN) |
| `{{PICK_SCRIPT}}` | none |
| `{{ARCH_DOCS_PATH}}` | `docs/` |
| `{{REPORTS_DIR}}` | `.personal/feature-torture/reports/` |
| `{{UNSTARTED_RULE}}` | every numbered H3 (`### N. <Title>`) under `## 🚀 Feature Suggestions` is a candidate (the rule matches every row — start-state check is the only gate; "unstarted" determined by codebase grep + git log) |

## Project frame

**One-line stance:** Rhétorix is an AI-powered web app that analyses
articles for rhetorical quality (Toulmin model, CRAAP test, fallacy
detection). React + Bun + Firecrawl + LLM scoring.

**Audience:** end users (curious readers, students, journalists) +
the maintainer.

**Non-negotiables**:
- Shareable analysis links must keep working (URL stability).
- LLM cost stays bounded — no full-corpus re-analysis on every visit.
- Reliability scores remain on a 1–5 scale (don't drift to 1–10 or
  letters).

## Roadmap conventions

- **Status markers:** none — `evol.md` is a suggestion document. The
  start-state check is the *only* gate.
- **Feature ID shape:** none. Synthesise an `F-<slug>` from the H3
  title.
- **Picker rule:** select any numbered H3 under `## 🚀 Feature
  Suggestions` whose title does not appear in the codebase as a
  shipped feature (verified via grep + git log).
- **Style:** prose is **English**. The torture report should follow
  plain English with short sentences, active voice, concrete verbs.

## Repo orientation hints

1. `README.md` (EN).
2. `docs/evol.md` for the feature row.
3. `src/` for the React app — components, pages, hooks.
4. `package.json` for the dependency surface (Firecrawl, LLM SDK).
5. `GEMINI.md` for any AI-prompt conventions.
