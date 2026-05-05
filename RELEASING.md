# Releasing claude-pilot-suite

`zeuikli/claude-pilot-suite` is the standalone source of truth. All development and releases happen directly in this repo.

## Version bump checklist

Before tagging a release:

1. Update `version` in `.claude-plugin/plugin.json`
2. Add a `## [x.y.z] - YYYY-MM-DD` section to `CHANGELOG.md`
3. Commit with message `chore(plugin): bump claude-pilot-suite to vX.Y.Z`

## Tagging a release

```bash
git tag vX.Y.Z
git push origin vX.Y.Z
```

## Branch workflow

- Develop on a feature branch
- Open a PR targeting `main`
- Merge via squash
