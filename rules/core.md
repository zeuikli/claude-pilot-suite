---
description: Core rules — language / Git / surgical-changes / quality (always-loaded)
tier: auto
---

# Core Rules

## Language

- If the user writes in a non-English language, reply in that language. Keep technical terms in English (e.g. `kubectl`, `git`, function/library names).
- If the user writes in English, reply in English.

## Git Workflow

- **IMPORTANT**: After every change is complete, YOU MUST:
  1. `git add <files>` (avoid `-A` to prevent staging sensitive files)
  2. `git commit` with a clear message
  3. `git push -u origin <branch>` (retry up to 4 times with exponential backoff: 2s/4s/8s/16s on network errors)
- Update `README.md` / `CHANGELOG.md` as needed.

## Think-Before-Coding (Assumption Disclosure)

- **IMPORTANT**: When receiving an implementation request, **state your understanding and assumptions** before writing code:
  1. Restate the requirement in 1–2 sentences (your interpretation, not parroting)
  2. List key assumptions (e.g. "I assume this function is only called server-side")
  3. If multiple reasonable interpretations exist, **list options for user confirmation** rather than picking one
- Skip this step only if the user explicitly says "just do it" / "skip explanation".

## Surgical Changes

- Touch only the minimum scope the task requires; bug fixes do NOT clean up surrounding code, one-off operations do NOT extract helpers.
- Seeing an out-of-scope bug or improvement → **note and report, do NOT auto-fix** (commit atomicity > "I'm already here").

| Don't | Do |
|-------|-----|
| Rename unused vars to `_var` | Delete them outright |
| Leave `// removed` comments | Delete and explain in commit message |
| Add "might need it later" params or abstractions | Solve the current need; file an issue for future needs |

- Commit granularity: "fix X" + "refactor Y" = two commits, never combined (keeps PR review / `git blame` clean, reduces rollback risk).
- **Quantitative bounds (soft)**: Bug fix ≤ 50 LoC, new feature ≤ 300 LoC, single file ≤ 500 LoC. If exceeded, surface proactively rather than splitting silently.

## Production Safety Red Line

- **IMPORTANT**: For any apply / deploy / delete operation against a production environment (whatever your workspace's "production" markers are — branch name, namespace, project ID, environment variable, etc.), YOU MUST first show the plan / diff and obtain explicit user confirmation before executing.
- *Customize this rule for your stack*: replace the abstract "production environment" definition with your concrete markers (e.g. `production` branch, `prod` namespace, project ID containing `-prod`, IaC workspace named `production`).

## Implementation Habits (official best practices)

- **Reference File Technique**: Point to existing code (e.g. `src/auth/login.ts`) instead of describing requirements verbally. "Follow the pattern in <file>" is more precise than prose.
- **Diff Review**: After changes, ask for "show diff + explain each change in one sentence" to catch unintended side edits.
- **Regression Hunter**: When debugging, ask "which of the last N commits might have caused this bug?" to use `git log / git diff` for root-cause analysis.

## Verification & Quality

- For code tasks, before declaring "done", YOU MUST run verification commands and **show first 5 / last 5 lines of output** (middle `...` elided). Never just say "tests pass".
- On test failure, **show the full error output**, not just "it failed".
- Workspace integrity: run your healthcheck script if you have one.
- Pre-commit: run a code-review pass (e.g. `/deep-review` skill if you've installed it).

## Known Gotchas

- **`git add <files>` is often misread as a single file**: it accepts multiple. Example: `git add file1 file2 file3`.
- **Missing commit-message metadata** (session URL / ticket reference): hard to trace back. If you have a pre-commit hook for this, enforce it there.
- **"Quantitative bounds" treated as hard limits**: they're soft; exceed and surface proactively rather than splitting PRs reflexively.
- **Production red-line vs branch-protection conflict**: if you lack force-push permission, you can't "apply after second confirmation". Pre-confirm permission or hand off to an admin.

---

> *Customization note*: this rule file is workspace-agnostic. Adapt the "Production Safety Red Line" section to match your concrete production markers, and feel free to add / remove "Known Gotchas" based on what your team has actually hit.
