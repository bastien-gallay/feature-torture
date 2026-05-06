# Feature-torture — test harness

> What counts as a passing dogfood. There are no automated tests; the
> "tests" are the reports themselves, scored against the criteria
> below. The harness exists so we can tell whether a prompt change
> made the skill better or just different.

## Test corpora

Two report directories form the live test set. Read across both when
judging whether a prompt edit helped.

| Corpus | Path | Project shape |
|---|---|---|
| **lucid-lint** | `.personal/feature-torture/reports/` | Rust core + bilingual docs; mature roadmap, `F<n>` / `F-<slug>` IDs. |
| **secondary-corpus** | `(private)` | Python CLI + dashboard; flat `F-<slug>` IDs only; one batch dogfood on 2026-05-04. |

The secondary-corpus corpus is the cross-repo dogfood. Drift between the two
corpora is the signal: a prompt edit that improves lucid-lint
reports while degrading secondary-corpus ones is over-fitted to lucid-lint's
roadmap conventions.

## Per-report pass criteria

A report passes when it satisfies all of:

1. **Verdict is one of the six labels** (👍 / ✂️ / ⏸ / 🧬 / 👎 / 🤷)
   and is the *first* heading-level emoji in the file.
2. **Verdict is not 🤷** unless the report names the single probe
   that would flip it. 🤷 without a probe = fail.
3. **TL;DR ≤ 3 sentences**, lands the verdict, names one decision +
   one risk.
4. **Make me dream** present (or explicitly skipped because verdict
   is 👎).
5. **Adjacency map names ≥ 2 real neighbours**, each with a file path
   or a roadmap row reference. "None" is acceptable for one slot, not
   for two.
6. **Roadmap-placement challenge has ≥ 1 quantitative comparison**
   (LOC, doc page count, dependency on a shipped F-ID, etc.).
7. **ADR core complete**: Problem / Current state / Options ≥ 2 /
   Choice / Consequences with 🔁 revisit-trigger.
8. **Output type declared** at the bottom.
9. **Spawned children: 0 to 5, no padding.** Same-feature
   restatements fail.
10. **Open questions: 2 to 5, each domain-tagged**, each
    decision-changing.
11. **Length 500–900 words** (1200 hard cap). Reports below 400
    words usually skipped a section; reports over 1200 padded.
12. **Density ≥ 1 finding per 80 words.** Below that = padded.
13. **Config file quoted in scratchpad** (or the absence
    explicitly noted with the inferred values).
14. **Start-state verified.** The report names the check that
    confirmed the feature wasn't already in flight (recent commits,
    open branch, closed PR). A report on a feature that turns out to
    be shipped fails regardless of other quality.

## Aggregate corpus criteria

Run these across all reports in a corpus, not per-report. They catch
prompt-design problems that don't show up in a single run.

### Verdict distribution sanity

Healthy distribution across a corpus of ≥ 10 reports:

| Verdict | Healthy share | Failure mode |
|---|---|---|
| 👍 ship | 15–35 % | < 15 % means the prompt is biased toward critique. |
| ✂️ reshape | **25–60 %** | **> 60 % means the post-converge cross-label challenge isn't biting** — verdicts are sliding into ✂️ instead of being demoted to 👍-with-amendments or promoted to 🧬. |
| ⏸ park | 5–20 % | 0 % means time-axis reasoning is absent. |
| 🧬 split | 5–25 % | 0 % means the prompt isn't surfacing hidden multi-features (or the corpus is small). |
| 👎 kill | 5–15 % | 0 % means the prompt can't say no. |
| 🤷 defer | ≤ 10 % | > 10 % means probes aren't sharp enough to force a call. |

**Note on the ✂️ band.** An earlier version of this file used 25–45 %.
The secondary-corpus 2026-05-04 batch shipped 7 / 8 ✂️ recommendations within
one week, with at least one commit message echoing the report's
reshape word-for-word. That ship-rate evidence falsified the
"✂️ is a catch-all" hypothesis: the bucket was correctly named.
The band was widened accordingly (2026-05-05).

### Verdict correctness (preferred over distribution where data exists)

Distribution alone is suggestive, not load-bearing. The stronger
test is whether verdicts predict shipping behaviour. Track:

| Signal | Pass | Failure mode |
|---|---|---|
| **✂️ → ship within 14 days, with the reshape recommendation visible in the commit message** | ≥ 60 % | < 40 % means ✂️ verdicts aren't actionable — reshape recommendations are too hand-wavy. |
| **👍 → ship within 30 days as scoped** | ≥ 70 % | < 50 % means 👍 verdicts overstate readiness. |
| **🧬 → ≥ 1 child entry created (in roadmap or follow-up doc) within 30 days** | ≥ 50 % | < 25 % means 🧬 is theatre; the reshape was actually a single slice. |

Sample rows from the secondary-corpus 2026-04-30 → 2026-05-05 window (8 ✂️
verdicts, 7 shipped):

| Report | Shipped commit | Match quality |
|---|---|---|
| F-treemap-heatmap (✂️ "ship horizontal bar chart") | `7349a16` "replace sparse heatmap with repo-grouped bars" | word-for-word |
| F-coverage-reporting (✂️ "default `--cov` + CI artefact") | `adfbb9f` "coverage reporting baseline" | scoped match |
| F-strict-repo-filtering (✂️ "honour the explicit-override clause") | `074dbbb` "strict repo filtering" | inverted-bug match |
| F-edge-case-tests · F-lizard-validation · F-metric-row-schema | `f18e8da` "parser foundations — ParserError + MetricRow + edge tests" | bundled match (3 ✂️ verdicts → 1 PR) |
| F-trim-filenames | `3b25317` "shorten_path helper" | scoped match |

### Adjacency reciprocity

When report A names report B as scope-adjacent, report B (if it
exists in the corpus) should name A back. One-way adjacency suggests
the picker found genuinely-orthogonal features; mutual adjacency
suggests the corpus has families and the prompt is correctly seeing
them.

### Output-type consistency

Every 👍 report should be `decision+spec`. Every 👎 / ⏸ / 🤷 report
should be `decision-only`. ✂️ and 🧬 can be either. A 👍 with
`decision-only` is a fail (the report dodged the spec stub).

### Shape-check regression cases

When the user names a specific target that isn't a roadmap row, the
skill must emit the *Detected / Why / Pick (a)/(b)/(c)* block and
wait — not improvise a bounce, not force a feature frame.

Three live regression cases (from the lucid-lint 2026-05-06 session):

| Input | Expected detection | Expected pick path |
|---|---|---|
| `/feature-torture Block A` (a daily-file release-cut block) | `release-execution` | (c) bail to `/brainstorm` *or* (a) name the underlying F-ID; never (b) — release execution has no decision to torture. |
| "review the v0.2.5 release scope" | `policy` (release-scope question) | (b) torture as single decision → `policy-v0-2-5-scope.md`. |
| "v0.2.5 scope + when is a new rule a breaking change + patch cadence" | `decision-bundle` | Refuse the bundle; ask which single decision to torture first; route the rest to `/brainstorm` or later sessions. |

A run that produces an ad-hoc refusal paragraph (no *Detected / Why /
Pick* block) on any of the three inputs above is a regression.

### Cross-corpus drift

When a prompt edit lands, run one new dogfood in *each* corpus before
judging the edit. A prompt that helps lucid-lint reports while
hurting secondary-corpus ones is over-fitted to lucid-lint's `AGENTS.md` /
`.impeccable.md` orientation files.

## How to run a test

There is no CI. The "test runner" is you, reading the report against
the criteria above. To make this fast:

1. Pick the report (newest is usually most informative).
2. Walk the per-report list. Note any fail with one line.
3. If you're judging a prompt edit, also walk the aggregate list
   across the corpus snapshot before *and* after.
4. Log the result in the daily file's drop-in buffer or in
   `session-recap-<date>.md`.

## What this is not

- Not a CI gate. The skill is human-authored prose; an LLM scoring
  the prose against this rubric would be a pleasant Saturday project
  but not a current need.
- Not a substitute for the v3 prompt's own self-check. The self-check
  catches per-report defects; this file catches corpus-level drift.
- Not a contract with the agent. The criteria are how *we* judge the
  output, not how the agent judges its own work mid-session.
