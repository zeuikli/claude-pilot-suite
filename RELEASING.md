# Releasing claude-pilot-suite

> The source of truth lives at `cc-workspace/dist/claude-pilot-suite/`. The
> standalone repo `zeuikli/claude-pilot-suite` is a `git subtree`-mirrored
> publish target. Re-run this flow whenever cc-workspace ships changes that
> should reach the standalone repo.

## One-time setup

```bash
# In your local cc-workspace clone:
git remote add pilot-standalone https://github.com/zeuikli/claude-pilot-suite.git
```

## Each release

```bash
# 1. From cc-workspace root, on the branch holding the latest plugin changes:
git subtree split --prefix=dist/claude-pilot-suite -b pilot-suite-export

# 2. Push the split branch to standalone repo's main:
git push pilot-standalone pilot-suite-export:main

# 3. (Optional) Tag the release on the standalone side:
git push pilot-standalone <commit-sha-from-step-1>:refs/tags/v0.1.1

# 4. Clean up local branch:
git branch -D pilot-suite-export
```

## Version bump checklist

Before running the flow above, in `dist/claude-pilot-suite/`:

1. Update `version` in `.claude-plugin/plugin.json`
2. Add a `## [x.y.z] - YYYY-MM-DD` section to `CHANGELOG.md`
3. Commit with message `chore(plugin): bump claude-pilot-suite to vX.Y.Z`

## First-time publish (initial bootstrap)

If `zeuikli/claude-pilot-suite` does not yet exist on GitHub:

1. Create the empty repo on GitHub (no README — we'll push our own)
2. Run `git subtree split` per "Each release" Step 1
3. Push with `git push pilot-standalone pilot-suite-export:main`
4. Verify the published tree on GitHub matches `dist/claude-pilot-suite/` locally

> Force-push (`--force-with-lease`) is acceptable on the standalone `main` if
> the subtree split produces a divergent history during a future re-base. The
> standalone repo's `main` is treated as a publish channel, not a development
> branch — local clones of the standalone repo should not commit directly.
