# .feature-torture.md (project config) — jira_analytics

> **Note**: this project's roadmap does not use checkbox markers. Status
> is encoded as an emoji on the row title (`✅` = done) and as inline
> bold text in the body (`**Statut : Corrigé**`, `**Statut : Bloqué**`).
> The "unstarted" set is rows whose title does NOT end with `✅` and
> whose body does not contain `**Statut : Corrigé**`. The agent must
> reason about status, not match a checkbox glyph.

## Paths

| Placeholder | Value |
|---|---|
| `{{PROJECT_NAME}}` | `jira_analytics` |
| `{{ROADMAP_PATH}}` | `ROADMAP.md` (FR-language) |
| `{{PICK_SCRIPT}}` | none |
| `{{ARCH_DOCS_PATH}}` | none — flat tool, no docs/architecture/ |
| `{{REPORTS_DIR}}` | `.personal/feature-torture/reports/` |
| `{{UNSTARTED_RULE}}` | H3 heading `### BL-NNN : ...` whose title does NOT end with `✅` AND whose body does NOT contain `**Statut : Corrigé**` (multi-axis: title glyph + body negative) |

## Project frame

**One-line stance:** internal Jira-analytics tool for a Berger-Levrault
team — Python script that pulls Jira data and renders an HTML report
with Lean / cycle-time / SLA / quality metrics.

**Audience:** the team that owns the tool + their managers reading
the rendered report.

**Non-negotiables**:
- Deterministic — same Jira export → same report.
- Speed (the tool runs ad-hoc; report generation should stay quick).
- Single-file deployable (one Python script + a templates/ folder).

## Roadmap conventions

- **Status markers:** `✅` on H3 title = done. Otherwise unstarted or
  in-progress (check body for `**Statut : Bloqué**` etc.).
- **Feature ID shape:** `BL-NNN` — already on each H3.
- **Picker rule:** select H3 rows starting with `### BL-` whose title
  does NOT end with `✅`.
- **Style:** prose is **French**. The torture report should follow
  the prompt's "match user language" rule — write the report in FR
  using the FALC discipline if the project's documents are FR.

## Repo orientation hints (for the skill)

1. `README.md` (FR).
2. `ROADMAP.md` for the feature row + section context.
3. The path mentioned in the row body (e.g. `**Fichier :** templates/report.html`).
4. `metrics/` and `templates/` directory listing for code adjacency.
