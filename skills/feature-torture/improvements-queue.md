# Feature-torture — improvements queue

> Captured 2026-05-05 during Block B (standalone-skill seed). Each
> item is a candidate prompt edit that *might* improve the skill but
> hasn't been verified. Order = rough priority. Run one at a time,
> dogfood once per corpus, judge against `tests.md` aggregate
> criteria before committing to the edit.

## 1. Init option / sub-skill

**Problem.** Bootstrapping the skill in a new repo today means
copying `config.md`, editing the placeholders, and remembering the
file shape from `.impeccable.md`. The secondary-corpus dogfood worked because I
knew the shape; an OSS user wouldn't.

**Sketch.** A small `init` mode the agent enters when no config file
is present at any of the documented paths. It runs a 6-question
interview (project name, roadmap path, picker, ID shape, unstarted
pattern, reports dir) and writes `.feature-torture.md` at root.
Mirrors what `/init` does for `CLAUDE.md`.

**Why queue, not now.** This is a sub-skill, not a tweak. It changes
the surface from "one prompt" to "two modes (init + torture)". Worth
designing only after the home decision is made (`~/.claude/skills/`
vs plugin) — the answer changes whether `init` writes a markdown
file, a TOML config, or runs against a JSON schema.

**Test.** Run `init` against one of: daily-ops, the secondary-corpus repo
(after deleting its config), one fresh repo with no AGENTS.md.
Inferred values should match what a human would write within ±1
field.

## 2. More bullet / numbered lists in the prompt body

**Problem.** v3 uses prose for several rules where a list would scan
faster. The "Web search vs repo grep" section, the "Style rules"
section, and the "Hard rules" section all paragraph-then-dash;
bullets-only would shave reader time.

**Sketch.** Pass through the prompt converting "rule-ish" prose to
bullet lists. Keep prose where it's actually argumentative (e.g.
the diamond-loop motivation).

**Why queue, not now.** Cosmetic. Worth doing alongside the next
substantive prompt edit, not on its own.

**Test.** Word count delta should be ≤ ±10 %. Density of the prompt
itself improves; reports it produces should not change.

## 3. Kata-table check on the chosen solution

**Problem.** The Kniberg Kata appears as an *optional* closing move.
When skipped, the report's Choice is sometimes a verdict without a
matching "next concrete step somebody can take Monday" line. The
gap shows up most in secondary-corpus reports where the verdict is ✂️ but the
"Next Concrete Step" is implied rather than written.

**Sketch.** Promote one Kata cell — "Next Concrete Step" — from
optional-section to a required ADR Consequence row. Keep full Kata
optional. Add to self-check: "Choice has a named next concrete step
that fits in one sentence and is doable Monday."

**Why queue, not now.** Edits a required-section spec. Needs a
dogfood pair to confirm it doesn't bloat the report.

**Test.** Re-read the 5 lucid-lint reports — count how many already
have an implicit next-concrete-step in Choice. If ≥ 4, this edit is
mostly cosmetic. If ≤ 2, it's load-bearing.

## 4. Tune verdicts — premise falsified, kept as refutation discipline — **CLOSED 2026-05-05**

> **Status.** Implemented 2026-05-05 as option 3 (post-converge cross-
> label challenge). Acceptance test ran the same day (8 reports
> across 4 repos: lucid-lint, daily-ops, jira_analytics, rhetorix —
> secondary-corpus ✂️ pool was unavailable for re-test because 7/8 had shipped).
>
> **The original premise was wrong.** "✂️ at 53 % means the bucket is
> a catch-all" was falsified by the secondary-corpus ship-rate: 7 of 8 ✂️
> recommendations shipped within a week, with at least one commit
> message echoing the report's reshape word-for-word. The bucket was
> correctly named.
>
> **The check stays anyway** — generalised on 2026-05-05 from
> ✂️-specific to all-labels (any non-default convergence triggers a
> two-neighbour challenge). Value is **refutation discipline**, not
> bucket-correction. Reports that converge on ✂️ now explain why
> 👍-with-amendments and 🧬 don't fit, which makes the verdict
> defensible without changing the distribution.
>
> **Threshold widened**: `tests.md` ✂️ band 25–45 % → 25–60 %; new
> verdict-correctness gates added (✂️→ship within 14 days ≥ 60 %).
> Original analysis preserved below for traceability.

**Problem.** ✂️ reshape is 53 % of verdicts across both corpora
(see `tests.md` aggregate snapshot). The label hides three different
recommendations:

- **(a) Reshape the slice** — keep the goal, ship a smaller scope
  this cycle. Most secondary-corpus ✂️ reports.
- **(b) Reshape and re-time** — keep the goal, ship later. Some
  lucid-lint ✂️ reports drift here.
- **(c) Reshape because the abstraction is wrong** — keep the
  user-visible behaviour, change the implementation. F-treemap-
  heatmap fits this.

**Sketch options** — pick one when this comes up the queue:

1. **Split ✂️ into ✂️-narrow and ✂️-rebuild.** Two labels with
   different decisions: ✂️-narrow ships a smaller scope; ✂️-rebuild
   keeps the goal but rewrites the approach.
2. **Sub-tag inside ✂️.** Keep the single label, require a
   one-word qualifier from a closed list: `narrow` / `re-time` /
   `re-shape`. Lower friction than splitting; same disambiguation.
3. **Stress-test ✂️ at the end of every diamond.** When the verdict
   converges to ✂️, run a forced check: "would 👍-with-amendments
   fit?" If yes, demote to 👍. "Would 🧬 split fit?" If yes,
   promote to 🧬. ✂️ becomes the residual category, not the
   default.

**Recommendation.** Try option 2 first — lowest blast radius, easy
to roll back. Run SCAMPER on the qualifier vocabulary:
- Substitute "re-shape" with a noun the user can act on.
- Combine with the Kata's "Next Concrete Step" so the qualifier
  predicts the step shape.
- Eliminate any qualifier that doesn't change the next action.

**Why queue, not now.** This is the most consequential edit on the
list; doing it before tests.md exists means we'd have no way to tell
if it worked.

**Test.** After the edit, run 6 fresh dogfoods (3 lucid-lint, 3
secondary-corpus), measure verdict-share against `tests.md` thresholds. Pass
iff ✂️ ≤ 45 % AND no other label drops to 0 %.

## 5. Stress-test the verdict label itself

**Problem.** Related to #4 but broader. The 6-label palette is a
hypothesis. We've never asked: are these the right six? Should one
exist that's currently absent (e.g. 🪞 "mirror — duplicate of
shipped F-X, drop")?

**Sketch.** Run a one-pass review of all 19 reports asking: "what
verdict would I assign?" If any report's optimal verdict is not in
the current six, the palette has a gap.

**Why queue, not now.** Discovery work, not an edit.

**Test.** Run the review. If 0 reports want a missing label, palette
is right. If ≥ 2 reports want the same missing label, propose
adding it.

## 6. Cross-corpus adjacency check (tooling)

Not a prompt edit; a small script that surfaces aggregate signal
from the corpus (verdict counts, reciprocity, output-type fit). Lets
us run `tests.md` aggregate criteria in 5 seconds instead of 5
minutes. Worth building only when the prompt stabilises and the
corpus grows past ~30 reports.

## 7. Canonical section-name translations (EN ↔ FR)

**Problem.** When the project is FR (jira_analytics), the agent must
translate every required-section heading inline (Roman thumb verdict
→ Verdict pouce romain, Make me dream → Fais-moi rêver, Adjacency
map → Carte d'adjacence). Different runs will translate the same
heading differently — reports become un-greppable across languages.

**Sketch.** Ship a small EN ↔ FR mapping table inside the prompt
(or a referenced file) with the canonical translations. Same idea
for any future locale.

**Why queue, not now.** Discovered 2026-05-05 in the BL-104 dogfood;
not blocking. Worth picking up when a second FR project gets tortured.

## 8. FR length-budget calibration

**Problem.** French is ~15 % wordier than English at equal density.
The current 500–900 word budget + ≤ 20-word sentence rule pull
in opposite directions for FR — the BL-104 report came in at 1116
words for a 🧬 verdict that was not substantively long.

**Sketch.** Either (a) bump the soft target to 1000 words for FR
(or any non-EN), or (b) raise the FR sentence cap to 25 words. (a)
is simpler; (b) better preserves FALC discipline.

**Why queue, not now.** Same reason as #7 — single-data-point
finding from BL-104.

## 9. F-id synthesis acknowledgment in prompt body

**Problem.** v3's body says *"git grep for the feature ID"*
assuming one exists. Suggestion-list projects (rhetorix) have no
IDs — only numbered titles. Config tells the agent to synthesise,
but the prompt body doesn't acknowledge the case.

**Sketch.** One sentence in the *Repo orientation* block:
*"If the project's roadmap shape lacks F-IDs, synthesise an
`F-<slug>` from the row title; use it consistently in scratchpad,
report, and adjacency references."*

**Why queue, not now.** Trivial fix; bundle with the next
substantive prompt edit.

## 10. "North star scope-around" clarification

**Problem.** v3 says "if the project lacks a one-page product north
star and the gap matters, name it in Open Questions and stop." The
rhetorix dogfood found a middle path — *scoping around* the missing
north star (e.g. parking the political-leaning sub-feature pending
content policy) instead of stopping. The agent reasonably interpreted
this as compliance; the prompt's binary "stop or proceed" framing
left it ambiguous.

**Sketch.** Add a third option: *"or scope explicitly around the
gap (named-out, parked-children) and say so in Open Questions."*

**Why queue, not now.** Edge case; surfaces twice before fixing.

## Triage

| Item | Cost | Value | Run order |
|---|---|---|---|
| 1 — Init mode | L–M | **H** (now-load-bearing for promotion) | After v3 stabilises in `~/.claude/skills/` |
| 9 — F-id synthesis | S | M | Bundle with next prompt edit |
| 10 — North-star scope-around | S | M | Bundle with next prompt edit |
| 7 — EN ↔ FR section names | M | M | Pick up on second FR dogfood |
| 8 — FR length budget | S | L–M | Pick up on second FR dogfood |
| 3 — Kata check | S | L (premise weakened by post-converge already covering this) | Standalone, when bored |
| 5 — Palette review | S | L–M | Standalone, when bored |
| 2 — Bullet pass | S | L | Bundle with any other edit |
| 6 — Tooling | M | L | Defer past 30 reports |
| 4 — ~~Tune ✂️~~ | — | **CLOSED** | Implemented + premise falsified 2026-05-05 |
