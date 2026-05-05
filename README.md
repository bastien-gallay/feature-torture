# feature-torture

A **decision-discipline skill** for Claude Code (and compatible AI coding assistants). Pressure-tests one roadmap feature per session and emits a 500–900-word report a human can act on — verdict + ADR + adjacency map + spec stub when shipping.

## What this is *not*

`feature-torture` is **not** a stress-test, load-test, fuzzer, or chaos-engineering tool. It does not run code, exercise APIs under load, or break a deployed system to see what breaks. It runs *before* implementation, on a roadmap entry. The "torture" in the name is the interrogation of an idea — does the slice make sense, what's the better cut, when should it ship — not the abuse of a built artefact.

If you landed here looking for stress / load / chaos testing, you want one of: [Gatling](https://gatling.io/), [k6](https://k6.io/), [Locust](https://locust.io/), [Chaos Mesh](https://chaos-mesh.org/), or [Toxiproxy](https://github.com/Shopify/toxiproxy).

## Why

A roadmap entry is a *headline*. A sprint plan is a *commitment*. The gap between them is where unverified assumptions hide. `feature-torture` forces the assumptions out before the sprint inherits them.

Most AI feature-review sessions drift: you ask "what should we do with X?", the model lists pros and cons, and you leave with a longer paragraph but no decision. `feature-torture` treats the AI as a **decision facilitator**, not a participant — it runs a structured diamond-loop process, picks 2–4 techniques from a closed list, and forces the verdict into one of six labels:

- 👍 **ship** — proceed as scoped, or with named amendments
- ✂️ **reshape** — keep the goal, change the slice
- ⏸ **park** — right idea, wrong cycle
- 🧬 **split** — entry hides multiple features; spawn children
- 👎 **kill** — drop the entry; reasons are load-bearing
- 🤷 **defer-decision** — only when a single probe would flip everything

Each non-default verdict triggers a **post-converge cross-label challenge**: the report has to argue why the two nearest-neighbour verdicts don't fit. Refutations live in *Choice*, making the verdict defensible at a glance.

## Install

### Claude Code (plugin channel)

```sh
/plugin install feature-torture@bastien-gallay/feature-torture
```

### Claude Desktop or claude.ai web (`.skill` archive)

Download the latest `feature-torture-vX.Y.Z.skill` from the [Releases page](https://github.com/bastien-gallay/feature-torture/releases) and:

- **Desktop**: double-click — `.skill` is OS-registered and opens an *Add to library* dialog.
- **Web**: Settings → Capabilities → Skills → upload.

### vercel-labs/skills

```sh
npx skills install bastien-gallay/feature-torture
```

## Quickstart

1. **Bootstrap a config** for your project. Copy the closest example from `examples/` to `.feature-torture.md` at your repo root, edit the placeholders.

   ```sh
   cp ~/.claude/skills/feature-torture/examples/lucid-lint.md .feature-torture.md
   $EDITOR .feature-torture.md
   ```

2. **Invoke the skill** in Claude Code:

   ```
   /feature-torture
   ```

   The agent reads the config, picks one unstarted feature at random, runs the start-state check, and writes a report to `{{REPORTS_DIR}}/<F-id>.md`.

3. **Read the report.** If the verdict is 👍 or ✂️ with a spec stub, hand the spec to the next pair-programming session. If it's ⏸ / 🧬 / 👎 / 🤷, the report is the deliverable.

## Configuration

`feature-torture` reads project config from, in order:

1. `.feature-torture.md` at project root.
2. `.personal/feature-torture/config.md` (gitignored variant).
3. Inferred values from `AGENTS.md` / `CLAUDE.md` / `README.md`.

The config supplies six placeholders: `PROJECT_NAME`, `ROADMAP_PATH`, `PICK_SCRIPT` (optional), `ARCH_DOCS_PATH`, `REPORTS_DIR`, and `UNSTARTED_RULE`. The `examples/` directory ships five real configs covering five different roadmap shapes:

- **`lucid-lint.md`** — Rust + bilingual docs project; markdown table with `☐` / `🚧` / `✅` glyphs.
- **`secondary-corpus.md`** — Python audit toolchain; `- [ ]` task list.
- **`daily-ops.md`** — Python CLI + Claude adapters; `- [ ]` task list with `- [/]` in-progress glyph.
- **`jira_analytics-FR-non-checkbox.md`** — French-language internal tool; H3-headings with `✅` on title for done, `**Statut : Corrigé**` inline for done-but-not-glyphed (multi-axis).
- **`rhetorix-suggestion-list.md`** — Suggestion-list document with no status concept; every numbered H3 is a candidate, status comes from grep.

## Session output

A typical report has 11 required sections, ~500–900 words, ≥ 1 finding per 80 words. The shape is:

1. Roman thumb verdict (one of the 6 labels)
2. TL;DR (≤ 3 sentences)
3. Make me dream (≤ 80 words; skip if 👎)
4. Job to be done
5. Adjacency map (3 named neighbours)
6. Roadmap-placement challenge (with quantitative comparison)
7. ADR core (Problem / Current state / Options / Choice / Consequences)
8. Output type (decision-only or decision+spec)
9. Spawned children (0 to 5)
10. Open questions (2 to 5, domain-tagged)
11. Techniques used (2–4 used + the rest rejected)

## Repo layout

```
feature-torture/
├── README.md                       # this file
├── CHANGELOG.md
├── LICENSE
├── CLAUDE.md                       # repo conventions
├── justfile                        # release automation
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   └── feature-torture/
│       ├── SKILL.md                # the canonical prompt with frontmatter
│       ├── tests.md                # corpus criteria
│       └── improvements-queue.md   # backlog of prompt-edit candidates
├── examples/                       # 5 real config fixtures
├── docs/
│   └── RELEASING.md
└── dist/                           # populated by `just release` / `just package`
```

## Acknowledgements

Process design pulled from:

- **[ADR](https://adr.github.io/)** for the report spine.
- **Kniberg Kata** (Current → Target → Next Concrete Step) for the next-step closing move.
- **SCAMPER** and **Devil's Advocate** as stress techniques.
- **Kano model**, **RICE / ICE**, **T-shirt sizing**, **Crazy-8s** as the PM-flavor branch.

Distribution scaffolding mirrors **[bfw](https://github.com/bastien-gallay/bfw)** — same justfile, same release flow, same `.skill` package shape.

## License

MIT — see [LICENSE](./LICENSE).
