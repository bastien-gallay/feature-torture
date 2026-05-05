# .feature-torture.md (project config)

Project configuration for the feature-torture skill on `lucid-lint`.
The skill reads this file to fill its placeholders. Mirrors the
`.impeccable.md` style — humans first, tooling second.

> **Location now:** `.personal/feature-torture/config.md` (gitignored,
> matches the rest of the draft skill).
> **Promotion path:** when the skill stabilises, move this file to
> project root as `.feature-torture.md` so it travels with the repo.

## Paths

| Placeholder | Value |
|---|---|
| `{{PROJECT_NAME}}` | `lucid-lint` |
| `{{ROADMAP_PATH}}` | `ROADMAP.md` |
| `{{PICK_SCRIPT}}` | `python3 .personal/feature-torture/scripts/pick_feature.py` |
| `{{ARCH_DOCS_PATH}}` | `docs/architecture/` |
| `{{REPORTS_DIR}}` | `.personal/feature-torture/reports/` |
| `{{UNSTARTED_RULE}}` | row's Status cell contains the literal `☐` |

## Project frame

**One-line stance:** bilingual cognitive-accessibility linter for prose.
Deterministic-core in Rust; no LLM, no network at runtime; LLM and
NLP work lives in plugin crates.

**Audience:** three groups, see `.impeccable.md` for full detail.

1. **Evaluators** — tech writers, docs maintainers, staff engineers
   scanning for 90 seconds before deciding.
2. **Contributors** — rule, dictionary, and translation authors.
3. **Cognitive-accessibility community** — ADHD, dyslexic,
   low-attention, fatigued, L2, FALC / RGAA / EAA practitioners.
   The docs UI itself is a credibility test for this group.

**Non-negotiables** (any feature recommendation must respect):

- Atomic rules — one signal per rule.
- Cognitive-load grounded — cite research or a recognised standard
  (WCAG, RGAA, FALC, BDA, IFLA, CDC Clear Communication Index,
  plainlanguage.gov).
- Deterministic in core — LLM, POS, dependency-tree work goes to
  plugin crates.
- Bilingual-viable — EN + FR concrete path at proposal time.
- Category-coherent — fits one of `Structure / Syntax / Rhythm /
  Lexicon / Readability` cleanly.

## Roadmap conventions

- **Status markers:** `☐` not started · `🚧` in progress · `✅` done.
- **Feature ID shape:** `F<number>` (legacy, F1–F146) or `F-<slug>`
  (new entries; kebab-case slug).
- **Catalog table row shape:**
  `| <a id="..."></a>[ID](#id) | Topic | Status | Target | Summary |`
- **Picker rule:** select rows where the Status column contains `☐`.

## Style profiles

- **Chat / report prose (EN):** plain English — short sentences,
  active voice, concrete verbs. Mirrors the FALC discipline used in
  FR.
- **Chat / report prose (FR):** FALC profile (`falc`) — see
  `docs/src/guide/profiles.md`.
- **Commit messages, PR bodies, code review:** `dev-doc` profile.
  Identifiers and flags fight FALC thresholds; this profile expects
  it.

## Repo orientation hints (for the skill)

When orienting on a feature, read in this order:

1. `AGENTS.md` — principles, contracts, taxonomy.
2. `.impeccable.md` — design context (only if the feature touches a
   user-facing surface).
3. `{{ROADMAP_PATH}}` intro + legend + the feature's narrative section.
4. The feature row's linked F-IDs.
5. One adjacency probe via `git grep` on the feature ID and the most
   likely code path.
