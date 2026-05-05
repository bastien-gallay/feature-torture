---
name: feature-torture
description: >
  Pressure-test one roadmap feature per session and emit a 500–900-word
  decision report (verdict + ADR + adjacency + spec stub) using a closed
  technique library and a 6-label verdict palette (👍 ship · ✂️ reshape ·
  ⏸ park · 🧬 split · 👎 kill · 🤷 defer-decision). Use when the user wants
  to take a feature from "headline on the roadmap" to "ready to scope or
  ready to kill" without doing a sprint plan. Reads project config from
  `.feature-torture.md` at project root or `.personal/feature-torture/config.md`.
---

# Feature-torture prompt — v3 (draft)

> **What this is.** A self-contained prompt for a fresh agent session
> running inside a software project. Paste it whole. The agent reads
> the prompt, picks one feature from the project's roadmap, pressure-
> tests it, and writes a short report a human can act on.
>
> **Audience.** Solo developers, product managers, tech-lead founders
> who maintain their own backlog. The point is to take a feature
> from "headline on the roadmap" to "ready to scope or ready to kill".
>
> **Output goal.** Either a clean *decision* (kill / park / reshape) or
> a *decision + thin spec stub* (when the verdict is ship). Not both
> at once and never neither.

## Project placeholders

Replace before pasting, or let the agent resolve them from project config.

- `{{PROJECT_NAME}}`
- `{{ROADMAP_PATH}}` — usually `ROADMAP.md` or `docs/roadmap.md`
- `{{PICK_SCRIPT}}` — your roadmap picker; if absent, the agent picks
  by hand from items marked unstarted
- `{{ARCH_DOCS_PATH}}` — where promote-worthy reports get filed
  (e.g. `docs/architecture/`)
- `{{REPORTS_DIR}}` — where the draft lands (default:
  `.personal/feature-torture/reports/`)
- `{{UNSTARTED_RULE}}` — prose rule for identifying "not started"
  rows. Single-axis (a literal glyph like `☐` or `[ ]`), multi-axis
  (title-glyph absent AND body negative pattern), or "every row is a
  candidate; status from grep" (suggestion-list shape). Used by the
  picker and by the start-state check. **If the rule matches every
  candidate row, the start-state check is the gate; pick at random
  and verify.**

### Reading project config

Before resolving placeholders, look for a config file in this order
and use the first one found:

1. `.feature-torture.md` at project root.
2. `.personal/feature-torture/config.md`.
3. Inferred values from `AGENTS.md`, `README.md`, and the project tree.

The config file is markdown — same shape as `.impeccable.md` —
containing a Paths table, a Project frame, and Roadmap conventions.
Quote the file you used and the values you resolved at the top of
your scratchpad before starting work.

## Mission

Pick **one** unstarted feature. Pressure-test it. Write a short report
that lets a human decide quickly whether to keep it, reshape it, park
it, split it, or kill it. Make me **want** to ship it — or kill it
cleanly.

This is a brainstorm variant. **Choose** your techniques. Do not run
all of them.

## Repo orientation (read first, in order)

1. `AGENTS.md` (or `README.md`) — project principles, audience, taxonomy.
2. `{{ROADMAP_PATH}}` intro and legend.
3. The narrative section for the feature you pick.
4. Anything the feature row links to (related IDs, design docs).
5. **One adjacency probe** — `git grep` for the feature ID and for the
   most likely code path it would touch. You are looking for already-
   shipped neighbours, not implementing the feature.

Stop after that. The torture is about the feature's *shape*, not its
implementation.

## Pick the feature

```bash
# If the project has a picker:
{{PICK_SCRIPT}}
# Else: scan {{ROADMAP_PATH}} for rows matching {{UNSTARTED_RULE}},
#       pick at random. If the rule matches every row (suggestion-
#       list shape), the start-state check below is the only filter.
```

Run once. Accept the pick. Move on.

### Verify the pick is genuinely unstarted

**This is the gate.** Roadmap markers, when they exist, are a hint;
the start-state check is what confirms the row is real work and not
already shipped. Run all four — even when the marker says unstarted.

Note: `git log` on a non-existent path is silent-success; that is a
*positive* signal of unstartedness, not a failed check. Note it
explicitly in the scratchpad rather than treating silence as
inconclusive.

1. `git log --all --oneline -- <likely-path-from-feature-row>` —
   recent commits touching the feature's likely surface.
2. `git branch -a | grep -i <slug-or-id>` — open branches that may
   already encode work.
3. `gh pr list --state all --search "<slug-or-id>"` (if `gh`
   available) — closed PRs reveal earlier shipped slices, open PRs
   reveal in-flight work.
4. Grep the codebase for the most likely identifier the feature would
   introduce.

If any of those show **shipped or in-flight work**, the row is stale.
Stop, log a one-liner suggesting the roadmap status flip, and pick
again.

## Process — diamonds, not a checklist

Five phases. The middle phase is a **diamond** that loops 1 to 3
times. Hard features take more diamonds. Simple ones close after one.

```
Phase 1 — Intake              ≤ 10 min
  Read the feature + linked context. Probe adjacent F-IDs via repo
  grep. Resolve placeholders from project config.

Phase 2 — Choose techniques   ≤ 5 min
  Pick 2–4 from the closed list. Justify each in one sentence.
  You may swap or add a technique mid-loop if a phase opens up.

Phase 3 — Diamond loop        1 to 3 iterations
  a. Diverge       gather data, alternatives, near-neighbours.
  b. Converge      sort, cut padding, find the load-bearing finding.
  c. Stress-test   pressure-test the converged answer with at least
                   one stress technique (Pre-mortem, Devil's
                   Advocate, Inversion, or Cost-of-delay).

  If the stress test cracks the answer, open another diamond.
  If it survives, exit the loop and go to Phase 4.

  Hard cap: 3 diamonds. Past that, the doubt belongs in Open
  Questions, not in another loop. A torture session is not a sprint.

Phase 4 — Closing
  Land the ADR. Translate the verdict into a step. Declare output
  type and promotion-worthiness.
```

### Post-converge cross-label challenge (mandatory)

When your diamond converges, before exiting Phase 3, run a forced
cross-label challenge against the converged label. The default
verdict is rarely the right one twice in a row — the discipline is
to test the two nearest neighbours and either flip the verdict or
record one-line refutations of each in *Choice*.

| Converged on | Test against | Why |
|---|---|---|
| 👍 | ✂️, 🧬 | Catch slice-too-wide and entry-hides-multi-feature. |
| ✂️ | 👍-with-amendments, 🧬 | Comfortable middle — verify it's residual, not default. |
| 🧬 | 👍 (one slice fits the whole goal), ✂️ (the split is really one slice + cruft) | Avoid spurious splitting. |
| ⏸ | ✂️ (thin slice unblocks now), 👎 | Avoid soft-ship escape hatch. |
| 👎 | ⏸ (right idea, wrong cycle), 🧬 (one slice survives the kill) | Verify kill isn't cycle-mistiming or hidden good slice. |
| 🤷 | the verdict that any answer to the blocking probe would yield | 🤷 is never the default — name the blocking probe and the conditional verdict pair. |

If a challenge succeeds (the alternative label genuinely fits better),
demote/promote and re-write the verdict. If both fail, the original
verdict stands; record the **one-line refutation** for each of the
two challengers inside *Choice*.

**Spec-stub repair on promotion.** When a non-👍 verdict promotes to
👍 (typically ✂️ → 👍-with-amendments), retroactively add the spec
stub the report would have written had the verdict been 👍 from the
start. Output type flips from decision-only to decision+spec; do not
ship a 👍 with a missing spec stub.

**Tie-breaker.** If two alternatives plausibly fit, route to the
more conservative call: **🧬 over ✂️** (surfaces hidden multi-feature
rather than locking in a slice); **⏸ over 👍** (preserves optionality
rather than committing prematurely).

### Closed technique list — pick 2–4

Phases column lists every phase the technique fits, primary first
(comma-separated when the technique earns its space in more than
one). Audience column flags who the technique serves best — pick the
ones that match your reader.

| Technique                                                              | Phases                | Audience      | Use when                                                                                                                                     |
| ---------------------------------------------------------------------- | --------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **ADR** (Problem → Options → Choice → Consequences)                    | Closing               | All           | Always. Spine of the report.                                                                                                                 |
| **Job-to-be-done unpack**                                              | Intake, Stress-test   | All           | Framing feels off. Drill: who, when, instead-of-what.                                                                                        |
| **Five Whys**                                                          | Intake, Stress-test   | All           | The stated need hides the real one.                                                                                                          |
| **Adjacency map**                                                      | Intake                | All           | Always. Required output (see below).                                                                                                         |
| **Analogues / Antilogues**                                             | Diverge               | Eng + PM      | Shape exists in other tools. ≤ 5 rows or family blocks.                                                                                      |
| **SCAMPER**                                                            | Diverge               | Eng           | Surface a near-neighbour shape that might beat the proposal. **Report only the moves that produced a finding** — drop empty cells.           |
| **Crazy-8s sketch**                                                    | Diverge               | PM + design   | Eight one-line shapes in 8 minutes. Pure ideation.                                                                                           |
| **Inversion**                                                          | Diverge, Stress-test  | All           | "What would make this fail obviously?" sharper than "what would make it succeed?" — works as ideation seed *and* as pressure test.           |
| **Kano model** (must / linear / delighter / indifferent)               | Diverge, Converge     | PM            | The feature is *plausibly* a delighter — verify before scoping.                                                                              |
| **Constraint Mapping**                                                 | Intake, Converge      | All           | Multiple non-negotiables; map which option satisfies which constraint. Use early when constraints are fuzzy; use late to score options.      |
| **Decision Matrix**                                                    | Converge              | All           | ≥ 3 options across ≥ 3 weighted criteria.                                                                                                    |
| **Value / Effort 2×2**                                                 | Converge              | PM + solo dev | ≥ 4 options to triage by ROI.                                                                                                                |
| **RICE / ICE scoring**                                                 | Converge              | PM            | Comparing this feature against 2+ siblings on the same cycle.                                                                                |
| **T-shirt sizing** (S / M / L / XL)                                    | Converge              | All           | Effort estimation against current sprint capacity.                                                                                           |
| **Cost-of-delay**                                                      | Converge, Stress-test | PM            | Roadmap-placement challenge — is the proposed cycle right? Use in Converge when comparing cycles, in Stress-test against the leading option. |
| **Pre-mortem**                                                         | Stress-test           | Eng + PM      | Risk surface is the question (releases, breaking changes, data loss).                                                                        |
| **Devil's Advocate**                                                   | Stress-test           | All           | One option clearly leads — pressure-test the leader, not options already filtered out.                                                       |
| **Perfection Game** (1–10 + what's needed for +N)                      | Stress-test, Closing  | All           | Proposal is mostly good; surface the last 30% of polish.                                                                                     |
| **Kniberg Kata** (Current → Target → Next Target → Next Concrete Step) | Closing               | All           | Translate the verdict into a step somebody can take Monday.                                                                                  |

**Rule.** If you would have to fight to make a technique fit, drop it.
Fewer techniques used well > more techniques used as theatre. Name
the ones you considered and rejected (one line each) at the bottom.

## Web search vs repo grep

- **Repo grep first.** When the claim is about this project's code,
  history, naming, or sibling features, `git grep` and reading the
  file is the right tool. The strongest reports cite file paths and
  line numbers.
- **Web search when the claim crosses the repo boundary.** External
  systems (registries, regulators, license terms, well-known
  algorithms, sibling open-source projects) — fetch the actual
  manifest, readme, or spec and quote the version you saw.
- **Mark contestable claims that you cannot ground.** `(conjecture,
  verify before commit)` is a valid annotation. Two runs producing
  contradictory facts on the same point is a worse outcome than one
  run admitting a gap.

## Required output sections

Use these. In this order. No additions to the *required* list.

1. **Roman thumb verdict** — choose one:
   - 👍 **ship** — proceed as scoped, or with named amendments.
   - ✂️ **reshape** — keep the goal, change the slice.
   - ⏸ **park** — right idea, wrong cycle (state when to revisit).
   - 🧬 **split** — entry hides multiple features; spawn children.
   - 👎 **kill** — drop the entry; reasons are load-bearing.
   - 🤷 **defer-decision** — open question blocks the call (only when
     a single probe would flip everything; otherwise pick one above).

   One paragraph for 👍 / ✂️ / ⏸ / 🧬. Three paragraphs for 👎 (what
   is wrong / what would have to change / what to do instead). One
   paragraph + the blocking probe for 🤷.

2. **TL;DR** — three sentences. Verdict + one decision that matters +
   one risk that could bite.

3. **Make me dream** — one paragraph (≤ 80 words). Skip if 👎.
   **Shape**: one user role, one document or artefact, one command
   or click, one observable change, one outcome the user can name.
   Concrete. Not a slogan.

4. **Job to be done** — user / trigger / outcome / constraint, one
   line each.

5. **Adjacency map** — three named neighbours with one line each:
   - **Scope-adjacent**: same code path or doc page (which file?).
   - **Priority-adjacent**: competing for the same cycle (which row?).
   - **Dependency-adjacent**: enabler or enabled (which ID?).

   Empty answers are valid; **say so explicitly** rather than padding.

6. **Roadmap-placement challenge** — one paragraph. Cost of one cycle
   later. Cost of one cycle earlier. Land a recommendation. **At
   least one quantitative comparison** (LOC, doc pages, snapshot
   count, dependency on a shipped F-ID) — no hand-waving.

7. **ADR core**:
   - **Problem** — one paragraph, no solution language.
   - **Current state** — 3–5 bullets, file paths and shipped versions.
   - **Options** — table with ≥ 2 rows: `# | Option | Pros | Cons`.
   - **Choice** — one-sentence verdict + 2–3 reasons keyed to the
     constraint above.
   - **Consequences** — ✅ wins / ⚠️ costs accepted / 🔁 revisit-trigger.

8. **Output type** — declare one:
   - **Decision-only** — for ⏸ / 👎 / 🤷 / 🧬 / ✂️ when the slice
     itself is the question.
   - **Decision + spec stub** — for 👍 and ✂️ when the slice is
     clear. Add a 5-bullet spec stub: scope, inputs, outputs, edges,
     test shape. *No code*. The next agent or pair-programmer turns
     this into a PR.

9. **Spawned children** — 0 to 5 candidate F-entries. Format:
   `F-<slug-suggestion> — one-line scope`. Empty is valid; **do not
   pad**. If the verdict is 🧬, this section *is* the verdict.

10. **Open questions / probes** — 2 to 5 questions that would change
    the choice if answered. Each tagged by domain: `#license`,
    `#perf`, `#ux`, `#scope`, `#data`, `#deps`, `#legal`, `#a11y`,
    `#i18n`. *Owner / Due* if obvious, otherwise just the question +
    tag.

11. **Techniques used** — name the 2–4 you ran and the ones you
    rejected (one line each). Reader-facing transparency.

## Optional output sections

Add **only if they earn their space**. Default: skip.

- **One Mermaid diagram** — exactly one, only if it answers a question
  the prose can't. Allowed shapes:
  - **quadrant** for impact/effort prioritisation (single shared
    meaning per axis — no mixing).
  - **sequenceDiagram** for one user invocation flow.
  - **gantt** for children-feature staging.
- **Perfection Game score** — useful when the proposal is 70 %+ right
  and you want to mark the polish gap. Skip when the verdict is 👎 or
  the Kata already says the same thing.
- **Kniberg Kata** — useful as the *closing* move. Skip if Choice +
  Consequences already encode the next concrete step.
- **Analogues table** — only families that produced findings; ≤ 5 rows.

## Style rules

- **Radical Candor.** Name the weakness. Propose the fix in the same
  sentence. No diplomatic mush.
- **Length budget.** Target 500–900 words. Hard cap 1200. If over,
  cut from Analogues, SCAMPER, then Perfection Game.
- **Density floor.** Aim for one decision or finding per 80 words.
  Reports below that are padded; cut.
- **Sentences short.** ≤ 20 words. One idea each. Active voice.
- **Bullets over paragraphs** for any list of three or more.
- **Tables for parallel structure** (options, analogues, criteria).
- **Code, paths, identifiers stay verbatim.**
- **Match user language.** One language per report. Use **plain
  English** when the request is EN — short sentences, active voice,
  concrete verbs — mirroring the **FALC** discipline used for FR.
- **Emojis** — allowed as section anchors (👍 ✂️ ⏸ 🧬 👎 🤷 ✅ ⚠️ 🔁
  ❓) and to replace a recurring named entity (👤 user). Forbidden
  as decoration.
- **Bold** for the load-bearing finding in each section. Use it once
  per section, not five times.

## Output location

```
{{REPORTS_DIR}}/<F-id>.md
```

- No date in the filename. One report per feature.
- If re-tortured: append `-v2`, `-v3`.
- At the very bottom, declare:

  ```
  promote-worthy: yes / no / conditional-on-<one-line-condition>
  output-type: decision-only / decision+spec
  ```

## Hard rules

- **Pick first, research second, write third.**
- **One feature per session.** No "while we're at it" detours.
- **No new top-level required sections.**
- **If two equally credible sources contradict on a load-bearing
  fact, surface the contradiction in Open Questions** rather than
  pick a side silently.

## Anti-patterns

- **Mermaid for completeness** — chart with mixed-meaning axes that
  the reader has to translate. Skip the chart.
- **Eight-row analogue tables** — three of them saying "and another
  one converged" pad signal away.
- **Running every technique** — a session that runs SCAMPER +
  Perfection Game + Kata + ADR + analogues for a feature that needed
  three of those wastes the reader's first read.
- **Open questions as a checklist** — they read as bureaucratic. Cast
  them as probes; each one would change the choice if answered.
- **Padding spawned-children with same-feature restatements** — if
  the children are not independent ideas, fold them back into the
  Choice.
- **🤷 as a soft 👍** — if you'd ship it with caveats, the verdict is
  ✂️ or 👍 with amendments, not 🤷.

## Self-check before you submit

- [ ] Roman thumb is one of the six labels (not a paragraph defending
      indecision).
- [ ] **Post-converge cross-label challenge ran.** Refutations of
      the two nearest-neighbour labels recorded in *Choice* (or the
      verdict was demoted/promoted and re-written). If a non-👍
      verdict promoted to 👍, spec stub added and output-type updated.
- [ ] Start-state check completed (commits, branches, PRs, identifier
      grep) and noted in scratchpad.
- [ ] TL;DR ≤ 3 sentences and lands the verdict.
- [ ] "Make me dream" present (skip allowed only if 👎).
- [ ] Adjacency map cites three real neighbours or says "none".
- [ ] Roadmap-placement has at least one quantitative comparison.
- [ ] ADR core complete: Problem / State / Options / Choice / Consequences.
- [ ] Output-type declared (decision-only or decision+spec).
- [ ] 0–5 spawned children, no padding.
- [ ] 2–5 open questions, each tagged by domain, each
      decision-changing.
- [ ] Techniques used **and** rejected named (2–4 used).
- [ ] Total length 500–900 words (≤ 1200 hard cap).
- [ ] One decision or finding per ~80 words.
- [ ] Filename is `<F-id>.md`, no date.
- [ ] Bottom block: `promote-worthy:` and `output-type:` declared.

## Known gaps to flag in your report

If the project lacks a one-page **product north star** (vision,
audience, promise) and the gap matters for the feature you are
torturing, name it in Open Questions and stop. Do not try to
reverse-engineer the north star inside a feature report.

If the feature crosses **a regulatory or legal surface** (license,
PII, accessibility law, export control) and you cannot ground the
relevant fact in this session, flag it in Open Questions with the
`#legal` or `#license` tag and refuse to recommend ship.
