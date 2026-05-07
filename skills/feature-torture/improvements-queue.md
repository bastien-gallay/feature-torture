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

## 9. F-id synthesis acknowledgment in prompt body — **SUPERSEDED by #11 (2026-05-07)**

**Problem.** v3's body says *"git grep for the feature ID"*
assuming one exists. Suggestion-list projects (rhetorix) have no
IDs — only numbered titles. Config tells the agent to synthesise,
but the prompt body doesn't acknowledge the case.

**Sketch.** One sentence in the *Repo orientation* block:
*"If the project's roadmap shape lacks F-IDs, synthesise an
`F-<slug>` from the row title; use it consistently in scratchpad,
report, and adjacency references."*

**Status.** Superseded by **#11 — Title-as-stable-key** (the structural
fix). Synthesising F-IDs is a piecemeal patch; making title the
required key and demoting F-ID to optional is the right level. See
`.personal/brainstorm/20260507-make-it-generic.md`.

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

## 11. Title-as-stable-key + demote F-ID to optional — **MUST (next release)**

**Problem.** `SKILL.md:88-101, 132` hard-codes F-ID as the cross-reference
key for adjacency probes, scratchpad, and report references. Web research
(2026-05-07) confirmed F-ID is the **outlier**, not the norm: GitHub
Issues, Linear, Jira all auto-generate IDs; most ROADMAP.md files use
title-as-key with no stable ID. This is the load-bearing
bastien-driven-development assumption in the shipped prompt.

**Sketch.** Make **title** the required cross-reference key. Demote
**F-ID** to *optional and recommended* with a one-liner: "if you have
stable IDs, adjacency probes are sharper." Update `SKILL.md:88-101`
(input-shape rules), `:132` (adjacency probe), and propagate the
optional-ID story through all 5 fixtures. Keep your own workflow
(F-NN) intact — it just stops being the only valid shape.

**Why now, not queue.** Highest-impact / lowest-prompt-cost lever for
the v1 generic-mode push (see brainstorm 2026-05-07). Pairs with #12
in a single release.

**Test.** Re-run the rhetorix fixture (suggestion-list, no IDs) and a
fresh GH-issue-like roadmap. Both should produce reports with title
as the cross-reference key without the agent synthesising an `F-<slug>`.
Existing F-ID corpora (lucid-lint, daily-ops, jira_analytics's BL-NNN)
should still produce identical reports.

## 12. `AUDIENCE` placeholder in `.feature-torture.md` — **MUST (next release)**

**Problem.** The technique table's Audience column (Eng / PM / All) is
informational today. Reports get tone-tuned implicitly via the project
frame, but there's no explicit hook telling the agent which lens to
favour. Solo-OSS-dev reports drift toward "All", missing the chance to
bias toward the actual reader.

**Sketch.** Add an `AUDIENCE` placeholder to `.feature-torture.md` (one
of: `Eng`, `PM`, `Eng + PM`, `PM + design`, `All`, `solo dev`). Wire
it into Phase 2 — Choose techniques: prefer techniques whose Audience
column matches. Document in README + propagate to all 5 fixtures.

**Why now, not queue.** Trivial schema add, big report-quality lift.
Pairs with #11 in the v1 generic-mode release.

**Test.** Set `AUDIENCE: solo dev` on lucid-lint fixture, re-run a
torture; chosen techniques should skew toward Value/Effort 2×2,
T-shirt sizing (audience = "PM + solo dev"). Set `AUDIENCE: PM` on
jira_analytics; should skew toward Cost-of-delay, Kano.

## 13. GitHub Issues read-only adapter — **COULD (gated on external dogfooder)**

**Problem.** Web research confirmed GitHub Issues is the dominant input
surface for solo-OSS (~95% of the target audience). Today feature-torture
only reads markdown roadmap files. An external user with no `ROADMAP.md`
has no entry point.

**Sketch.** Read-only adapter: `feature-torture <github-issue-url>` or
`<repo>#<n>`. Fetch title + body via `gh issue view --json`. Treat the
issue as a single feature row — no F-ID synthesis (uses #11's
title-as-key). No write-back; report still lands in the local repo.

**Why queue, not now.** Highest absolute impact, but heaviest prompt
budget. **Gated:** do not build until ≥1 non-Bastien user has
successfully tortured a markdown-roadmap feature. Otherwise this is
scope built for a hypothetical user.

**Test.** Pick 3 real OSS issues from medium-sized repos (e.g. a Bun
issue, a Vite issue, a Linear-like indie tool); torture each; report
quality vs `tests.md` aggregate criteria should match markdown-fixture
quality.

## 14. Batch mode (torture N features in one run) — **WON'T (this round)**

**Problem.** Power-user wish: "I have 5 stuck roadmap items, torture
them all and emit 5 reports."

**Sketch.** Loop over a list of feature IDs/titles; emit one report
per. Session state would need a between-feature reset.

**Why parked.** Solves no onboarding problem. The dominant use is "I
have *one* feature today." Adds session-state complexity to the prompt
for a minority case. Re-evaluate after ≥3 external users specifically
ask for it.

---

> **Items #15–#24 added 2026-05-07 from
> `.personal/brainstorm/20260507-extend-features.md` (this session is the
> roadmap of record; we do not maintain a separate `ROADMAP.md`).**
> User-stated philosophy that drove these: **CUPID / SOLID,
> explicit > implicit, options for the architecture > all-mighty
> architecture.** Two new hard constraints govern the set: **light
> adapter payload** (pluggable, intermediate contract) and **auditable
> configuration** (config + auto-detect fallbacks printed in output).

## 15. Adapter contract — typed seam — **MUST (next release, lands before #11)**

**Problem.** Today the skill assumes one input shape (markdown roadmap
row). Future adapters (paste-mode, GH issues, …) would each grow their
own prompt branch — a fat, unauditable surface. The new "light adapter
payload" + "auditable configuration" constraints make this a debt that
compounds with every adapter we add.

**Sketch.** A small typed contract every adapter speaks through:

```text
{ title, body, optional_id, source_ref }
```

The central skill consumes only this shape. Each adapter is a thin
mapping: roadmap.md row → contract; pasted URL body → contract;
GH-issues JSON → contract. Print the contract instance (and which
adapter produced it) in the report footer to satisfy the
auditable-configuration constraint.

**Why now, not queue.** Lands *before* #11 (title-as-key) — the
contract is what lets `optional_id` exist cleanly. Doing #11 first
would mean re-doing it once #15 ships. Also unblocks #18, #20.

**Test.** After landing, the existing 5 fixtures must produce
byte-identical reports (the contract is a pure refactor today). Adding
a 6th fixture (e.g. a fake "GH-issue body" markdown blob) to the
corpus should require **only** a new ~20-line adapter, no `SKILL.md`
prompt changes.

## 16. Tombstone-aware mode — **MUST (next release)**

**Problem.** When a feature has been killed (👎) and recorded in
`TOMBSTONE.md`, nothing prevents the skill from re-proposing it on a
later session. Wastes user time, erodes trust in the verdict palette.

**Sketch.** If a `TOMBSTONE.md` (or configured equivalent) exists at
project root or in `.personal/feature-torture/`, read it during repo
orientation. If the current feature's title or ID matches a tombstone
entry, refuse to re-torture and cite the tombstone in the *Adjacency
map* section instead.

**Why now.** Cheap (effort = 1, mostly path-lookup + one prompt
clause) and high-trust. No reason to defer.

**Test.** Place a known-killed feature in `TOMBSTONE.md`, request
torture on it. The skill should refuse and reference the tombstone
entry. Adjacency map should still surface the kill rationale to inform
the *current* feature being designed.

## 17. First-feature wizard — **SHOULD (release+1, chains from `--init`)**

**Problem.** `--init` ends in "now what?" The user gets a configured
project but no momentum. Activation drops at the next session boundary.

**Sketch.** After `--init` writes `.feature-torture.md`, prompt: *"Want
to torture one feature now?"* If yes, list the unstarted rows the
shape-detector (#18) found, let the user pick one, and chain into the
torture flow with the just-written config.

**Why queue, not now.** Depends on `--init` (already locked-in
release+1) and benefits from #18.

**Test.** Run `--init` against a fresh repo with a populated
roadmap. Wizard should surface ≥3 unstarted features and produce a
report on the picked one without re-asking any config question.

## 18. Shape-detector — **SHOULD (release+1)**

**Problem.** New users don't know which of the 5 fixture shapes their
roadmap resembles. The format-flex gallery (gated, from prior session)
shows shapes but not *fit*; users still guess.

**Sketch.** A read-only detector that reads the roadmap path and
heuristically matches against the 5 fixtures (markdown table /
checkbox list / FR-prose multi-axis / suggestion-list / numbered
titles). Outputs a confidence-ranked match with one-line rationale.
Logs the match in `--init` output (auditable-configuration).

**Why queue, not now.** Cheap (effort = 2) but only matters once
`--init` lands — the detector's output feeds `--init`'s scaffolded
config and #17's wizard.

**Test.** Run against the 5 fixtures themselves; each must self-match
at >80% confidence. Run against a 6th, deliberately ambiguous roadmap;
confidence should drop to ≤60% and the output should suggest the top
two candidates.

## 19. AUDIENCE CLI override — **SHOULD (release+1, rides #12)**

**Problem.** `AUDIENCE` (locked-in #12) is config-time only. Teams
shift; a single feature might want a different lens than the project
default. No per-session escape hatch.

**Sketch.** A `--audience <Eng|PM|Eng+PM|...>` flag that wins over the
`.feature-torture.md` value for the current run. Print the
effective AUDIENCE (and its source: `flag` vs `config`) in the report
footer.

**Why queue, not now.** Trivial; piggybacks on #12 landing.

**Test.** With `AUDIENCE: solo dev` in config, run torture with
`--audience PM` — chosen techniques should skew toward
Cost-of-delay / Kano (PM-leaning) and the report footer should read
`AUDIENCE: PM (source: flag, overrides config: solo dev)`.

## 20. Paste-a-URL / paste-body adapter — **COULD (gated on ≥1 external dogfooder)**

**Problem.** Lower-bar variant of #13 (full GH-issues adapter). Users
without a `ROADMAP.md` and without `gh` CLI need an entry point.

**Sketch.** Accept either a URL (skill fetches the body via WebFetch)
or pasted issue body. Output goes through the contract (#15) — title +
body + url-as-source_ref + no optional_id. No write-back.

**Why queue, not now.** Same gate as #13 — wait for ≥1 external
dogfooder. Otherwise this is scope built for a hypothetical user. If
gate clears, **prefer #20 over #13** (lighter dep tree — no `gh`
binary required, just WebFetch which the host already provides).

**Test.** Paste 3 real OSS issues from medium-sized repos; report
quality vs `tests.md` aggregate criteria should match
markdown-fixture quality.

## 21. Report-quality self-check vs `tests.md` — **COULD (gated on ≥30 reports)**

**Problem.** Reports drift from `tests.md` aggregate criteria over
time; nobody notices until a manual re-read. The skill could grade its
own output before emitting.

**Sketch.** Add a final phase: skill re-reads its own draft report
against `tests.md` criteria (verdict-distribution band, required
sections present, length budget, ≤20-word sentences). Surface failures
as a self-check footer; don't block emission.

**Why queue, not now.** Compelling but the corpus needs to grow first.
Per existing #6 (tooling), aggregate criteria are only meaningful
past ~30 reports.

**Test.** Run on 5 known-good reports — self-check should pass.
Run on a deliberately-bloated report (1500 words for a 👍) — it
should flag the length-budget violation in the footer.

## 22. Session-resume — **WON'T (this round)**

**Problem.** Sessions are stateless; an interruption mid-torture loses
the scratchpad.

**Sketch.** Persist the scratchpad to a session file; resume on
next invocation if file exists.

**Why parked.** I/E = 0.5. Sessions are short (~30 min); interruption
is rare. Plumbing cost outweighs the benefit. Re-evaluate if multiple
users report mid-session loss.

## 23. Verdict diff across re-runs — **WON'T (this round)**

**Problem.** Re-tortured features can produce a different verdict
weeks later (the world changed, or the prompt changed). No way to
detect that the prior report is stale.

**Sketch.** Store run history per feature; compare verdicts on
re-torture; emit a diff section.

**Why parked.** I/E = 0.75. Needs run-history infrastructure. User
can manually re-torture and read both reports if they suspect
staleness.

## 24. Rename-resilience (alias log / content-hash key) — **WON'T (this round)**

**Problem.** Title-as-key (#11) breaks if a feature gets renamed —
adjacency probes fail across the rename, prior reports become
un-greppable.

**Sketch.** Maintain an alias log (renames recorded with date) or
move to a content-hash key for the body, with the title as a display
field.

**Why parked.** User confirmed they can live without it (2026-05-07).
Re-evaluate if titles drift in practice.

## Triage

| Item | Cost | Value | Run order |
|---|---|---|---|
| **15 — Adapter contract** | S–M | **H** (enables #11, #18, #20) | **Must — next release** (lands BEFORE #11) |
| **11 — Title-as-key + demote F-ID** | S–M | **H** (Bastien-DD removal) | **Must — next release** (rides #15; pair with #12) |
| **12 — `AUDIENCE` placeholder** | S | **M–H** (report-fit) | **Must — next release** (pair with #11) |
| **16 — Tombstone-aware mode** | S | **M–H** (trust) | **Must — next release** |
| **1 — Init mode** | L–M | **H** (activation-cost killer) | **Should — release+1** (refined 2026-05-07: 5-question scaffold + #8 auto-detect) |
| **17 — First-feature wizard** | S | **H** (activation lift on #1) | **Should — release+1** (chains from #1) |
| **18 — Shape-detector** | S–M | **M–H** (feeds #1, #17) | **Should — release+1** |
| **19 — AUDIENCE CLI override** | S | M | **Should — release+1** (rides #12) |
| 10 — North-star scope-around | S | M | Bundle with #11+#12 release |
| 7 — EN ↔ FR section names | M | M | Pick up on second FR dogfood |
| 8 — FR length budget | S | L–M | Pick up on second FR dogfood |
| 3 — Kata check | S | L (premise weakened by post-converge already covering this) | Standalone, when bored |
| 5 — Palette review | S | L–M | Standalone, when bored |
| 2 — Bullet pass | S | L | Bundle with any other edit |
| 6 — Tooling | M | L | Defer past 30 reports |
| **20 — Paste-a-URL adapter** | M | **H** | **Could — gated on ≥1 external dogfooder** (preferred over #13 — lighter dep tree) |
| **13 — GH Issues adapter** | **L** | **H** | **Could — gated on ≥1 external dogfooder** (consider deferring in favour of #20) |
| **21 — Report self-check vs tests.md** | M | M | **Could — gated on ≥30 reports** |
| 14 — Batch mode | M | L | **Won't this round** — re-evaluate after 3 external asks |
| 22 — Session-resume | M–L | L | **Won't this round** (I/E 0.5) |
| 23 — Verdict diff across re-runs | L | M | **Won't this round** (I/E 0.75; needs run-history infra) |
| 24 — Rename-resilience | M | M | **Won't this round** — re-evaluate if titles drift |
| 9 — ~~F-id synthesis~~ | — | **SUPERSEDED** | Replaced by #11 (2026-05-07) |
| 4 — ~~Tune ✂️~~ | — | **CLOSED** | Implemented + premise falsified 2026-05-05 |
