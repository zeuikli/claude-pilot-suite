# Releasing claude-pilot-suite

`zeuikli/claude-pilot-suite` is the standalone source of truth. All development and releases happen directly in this repo.

## Version bump checklist

Before tagging a release, complete these steps **in order**:

1. Update `version` in `.claude-plugin/plugin.json`
2. Add a `## [x.y.z] - YYYY-MM-DD` section to `CHANGELOG.md`
3. Commit with message `chore(plugin): bump claude-pilot-suite to vX.Y.Z`
4. Tag **that commit** (see below)

## Tagging a release

Use an **annotated tag** so git records the tagger, date, and a summary message:

```bash
git tag -a vX.Y.Z -m "vX.Y.Z — <one-line summary of what changed>"
git push origin vX.Y.Z
```

Then create a GitHub Release so users see formatted release notes:

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z — <one-line summary>" \
  --notes "<paste key CHANGELOG bullet points here>"
```

> **Why annotated, not lightweight?**  
> Annotated tags carry author + date metadata and are treated as first-class objects by `git describe`, GitHub Releases, and plugin tooling. Lightweight tags are just pointers with no metadata.

## Branch workflow

- Develop on a feature branch
- Open a PR targeting `main`
- Merge via squash
