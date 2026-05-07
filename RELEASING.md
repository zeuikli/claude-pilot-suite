# Releasing claude-pilot-suite

`zeuikli/claude-pilot-suite` is the standalone source of truth. All development and releases happen directly in this repo.

## Pre-flight

Before starting a release, make sure your local `main` is in sync with `origin/main`:

```bash
git fetch origin main
git status                          # expect "up to date with 'origin/main'"
```

If `origin/main` has commits you don't (e.g. a recently-merged PR), rebase your work onto it before bumping the version:

```bash
git rebase origin/main
```

Resolve any conflicts before continuing. **Do not** force-push or use a merge commit — keep `main` history linear.

## Version bump checklist

Before tagging a release, complete these steps **in order**:

1. Update `version` in `.claude-plugin/plugin.json`
2. Update the **Version** header line in `README.md` (line 3) so it matches: `> **Version**: x.y.z (YYYY-MM-DD — <one-line summary of what changed>)`
3. Add a `## [x.y.z] - YYYY-MM-DD` section to `CHANGELOG.md` with `### Added` / `### Changed` / `### Fixed` subsections as appropriate
4. Commit the three files together with message `chore(plugin): bump claude-pilot-suite to vX.Y.Z`
5. Push the commit to `origin/main` (`git push origin main`) **before** tagging — the tag must point at a commit that already exists on the remote
6. Tag **that commit** (see below)

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

## Post-release verification

After `gh release create` returns the release URL, verify the release renders correctly:

```bash
gh release view vX.Y.Z --web    # opens the release page in your browser
```

Check that: (a) the tag points at the bump commit, (b) the release notes match the new `CHANGELOG.md` section, (c) the release is not flagged as draft or pre-release, (d) `git log --oneline origin/main` shows the bump commit at HEAD.

## Branch workflow

- Develop on a feature branch
- Open a PR targeting `main`
- Merge via squash
