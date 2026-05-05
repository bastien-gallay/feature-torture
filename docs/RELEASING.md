# Releasing feature-torture

Maintainer-facing notes on how a release is cut. Mirrors `bfw`'s flow exactly. Not linked from the public README — internal reference.

## What a release actually ships

There is no build step in the repo itself. A release is:

1. A bumped `version` in `.claude-plugin/plugin.json`.
2. A bumped `metadata.version` AND `plugins[0].version` in `.claude-plugin/marketplace.json` (both must match — see `just check-versions`).
3. A commit on `main` with the above changes.
4. An annotated git tag `vX.Y.Z` on that commit.
5. A `CHANGELOG.md` entry describing what changed.
6. A `feature-torture-vX.Y.Z.skill` bundle attached to the GitHub Release — a zip archive with the `.skill` extension. Built by `just package` (locally or in CI when the tag is pushed).

Users pick up the release via:

- **Claude Code plugin channel** — `/plugin marketplace update feature-torture` reads the new `plugin.json` version and refreshes its cache. Skipping the version bump is a silent freeze: users stay on the cached old code forever. **This is the single most important invariant the release process protects.**
- **vercel-labs/skills channel** — `npx skills update` re-clones the repo. No version field drives it; it always pulls latest `main`.
- **Claude Desktop / claude.ai channel** — users download `feature-torture-vX.Y.Z.skill` from the GitHub Release page. Updates are manual (re-download + re-install).

## Current process (manual)

### Prerequisites

- `just` and `jq` on `PATH`.
- Working tree is clean and on `main`.
- The CHANGELOG.md "Unreleased" section is renamed to `## vX.Y.Z` for the version you're cutting.

### Steps

```sh
# 1. Cut the release locally
just release 0.2.0

# 2. Push commit + tag
git push && git push --tags

# 3. Build the .skill artefact
just package

# 4. Create the GitHub Release and upload dist/feature-torture-v0.2.0.skill
gh release create v0.2.0 dist/feature-torture-v0.2.0.skill \
  --title "v0.2.0" \
  --notes-from-tag
```

### Failure modes the process protects against

- **Forgot to bump `plugin.json`.** `just release` is the only way to bump versions; it edits both manifests atomically. If you ever bump by hand, run `just check-versions` to confirm alignment.
- **CHANGELOG drift.** `just release X.Y.Z` refuses to run unless `## vX.Y.Z` is present in `CHANGELOG.md`. No more "released v0.2.0 with v0.1.0 in the changelog."
- **Dirty working tree.** `just release` aborts if the tree is dirty.
- **Tag re-use.** Aborts if the tag already exists.

## Future automation

When this repo gains a `.github/workflows/release.yml`, the workflow will:

1. Trigger on `push` of a `v*` tag.
2. Run `just package`.
3. Create the GitHub Release with the `.skill` artefact attached.
4. Run `just check-versions` as a sanity gate.

Until that workflow exists, the manual `gh release create` step above is required.
