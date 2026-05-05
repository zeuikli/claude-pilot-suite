---
name: haiku-pilot
description: |
  Haiku-first execution playbook: through deliberate prompt structure, sub-agent
  delegation, and quantitative escalation gates, get Haiku 4.5 to produce
  near-Opus quality on most tasks. Escalate to Sonnet/Opus only when gates
  trigger.
  Triggers: "haiku", "haiku-pilot", "haiku mode".

  Do NOT use for: cost-only token optimization, file-count cognitive heuristic,
  agent dispatch table, CLAUDE.md / rules audit, harness health check.
  This SKILL is a runtime router + escalation gate, not a decision tree
  or directory.
allowed-tools: Read, Grep, Glob, Bash, TodoWrite
---

# Haiku Pilot — Default-Haiku Execution Playbook

## Per-Session Pre-flight (run once per task)

### 1. Reference Pattern Technique

| ❌ Verbal description | ✅ Point to a file |
|---------------------|-------------------|
| "Write a login-like function" | "Follow the pattern in `src/auth/login.ts`" |
| "Add a hook similar to useUser" | "Pattern: `hooks/useUser.ts:42-78`" |

**Why it matters**: Haiku diverges easily from abstract descriptions; pointing to a concrete file = strongest possible context.

### 2. Think-Before-Coding (compensates Haiku tendency)

Before implementation, output:
- 1–2 sentence **task understanding restatement** (your interpretation, not parroting)
- Key assumptions (≥ 1, prefixed with "Assumption: ...")
- If multiple interpretations are reasonable → **list options for user confirmation**, don't pick one yourself

> Source: `core.md` § "Think-Before-Coding". Haiku tends to skip this step; this SKILL enforces it.

### 3. Diff-Review Pledge

Before declaring "complete", run:
```bash
git diff --stat
git status
```

**Explain each change in one sentence**. Catches Haiku quietly modifying out-of-scope files (surgical-changes discipline).

### 4. Content-First Structure (not Template-First)

**Forbidden**: filling every question with a fixed sub-heading template; restating in prose what the table already says.

**Use instead**: the question's nature determines the structure:
- Single decision → bulleted list
- ≥ 3 anti-patterns → 1 combined table (with "trigger / solution" columns), NOT N separate tables
- Multi-option comparison → table lists only "final decision + exclusion rationale"
- Emoji budget: ≤ 3 per section (✅/❌/🔴); neutral statements get none

---

## Per-Task Router

### Coding tasks

| Task | Sub-agent | Model |
|------|-----------|-------|
| Single-file ≤ 30 LoC surgical edit | `general-purpose` | Haiku 4.5 |
| Cross-module / > 30 LoC / requires design | `general-purpose` | Sonnet 4.6 |
| Bug with failing test | `general-purpose` | Haiku 4.5 |
| Bug without test | `general-purpose` | Haiku → Sonnet if stuck |
| Test writing | `general-purpose` | Sonnet 4.6 |

### Research tasks

| Task | Sub-agent | Model |
|------|-----------|-------|
| ≥ 10-file fact-finding / locate symbols | `Explore` | Haiku 4.5 |
| Codebase structure inventory | `Explore` | Haiku 4.5 |

### Review tasks

| Task | Sub-agent | Model |
|------|-----------|-------|
| Single-file / single-function review | `general-purpose` | Sonnet 4.6 |
| Architecture decision / tech selection | `Plan` | Opus 4.7 |
| Auth / payment / user-data touchpoints | `general-purpose` | Sonnet 4.6 |
| OWASP / proactive threat modeling | `general-purpose` | Sonnet 4.6 |

### Docs tasks

| Task | Sub-agent | Model |
|------|-----------|-------|
| README / CHANGELOG / API docs | `general-purpose` | Haiku 4.5 |
| Major doc rewrite | `general-purpose` | Sonnet 4.6 |

> Add custom agents in `.claude/agents/` to replace `general-purpose` rows with tighter-scoped agents for lower token cost.

---

## Escalation Gates (quantitative, not "when complex")

> **Decompose first, escalate second**: try sub-agent decomposition before deciding to upgrade model. Decomposable = stay Haiku + delegate; non-decomposable = upgrade.

**Stay on Haiku** (any one true):
- Task touches ≤ 9 independent files
- Same problem attempted < 3 times
- No trigger keywords ("architecture decision", "design review", "security review", "threat modeling")
- Code-gen single response ≤ 300 LoC
- User hasn't explicitly requested upgrade

**Escalate to Sonnet 4.6**:
- Same problem failed ≥ 3 times
- Task touches ≥ 10 independent files OR cross-module design
- Test writing (boundary cases + mocks)
- User said "use Sonnet"

**Escalate to Opus 4.7 (or `Plan` agent)**:
- Trigger keywords: "architecture decision", "design review", "tech selection", "security review", "threat modeling"
- Multiple Sonnet attempts haven't converged (rare)
- User explicitly says "use Opus"

> Escalation isn't linear: context pollution → try `/compact` first; do NOT directly upgrade the model. See `context-management.md`.

---

## Failure Diagnostic Flow (when Haiku gets stuck)

```
Haiku fails 1×
    ↓
 Is it a context problem? (symptoms: misrestating requirements, lost-in-middle questions, ignoring given files)
    ├── Yes → /compact or /rewind, retry (still Haiku)
    └── No  → model insufficient
              ↓
          Try once more (redo pre-flight: Reference Pattern + assumption disclosure)
              ↓
          Still fails (3rd attempt)
              ↓
          Escalate to Sonnet (do NOT jump straight to Opus; 3 Haiku failures ≠ Opus needed)
```

---

## Expected Cost Savings

> Assumption: 80% of tasks fit Haiku conditions, 15% upgrade to Sonnet, 5% upgrade to Opus.

Reference pricing (2026-04 public, per 1M tokens, input/output):
- Haiku 4.5: $1 / $5
- Sonnet 4.6: $3 / $15
- Opus 4.7: $15 / $75

Weighted average vs full-Opus baseline:
```
Haiku  0.80 × ($1 + $5) / 2  = $2.40
Sonnet 0.15 × ($3 + $15) / 2 = $1.35
Opus   0.05 × ($15 + $75) / 2= $2.25
Total                         = $6.00 / 1M token (rough — input >> output in practice)

vs full-Opus baseline: ($15 + $75) / 2 = $45
Savings: 1 - 6/45 = 86.7%
```

**Conservative estimate: 70–85% savings** (depends on input/output ratio and cache hit rate).

---

## Known Gotchas (Haiku 4.5 specific)

- **Skips assumption-disclosure**: Haiku tends to write code directly, violating `core.md` § "Think-Before-Coding". Mitigation: this SKILL § Pre-flight #2 enforces it.
- **Verbose comments**: Haiku writes 1.5–2× more comments than Sonnet. Mitigation: `output-discipline.md` § "minimal comments" + default no comments.
- **Misses regression check**: when debugging, doesn't proactively ask "which of the last N commits could have caused this?". Mitigation: explicitly require it at debug task start.
- **Long-context degradation**: > 300K tokens, Haiku degrades more visibly than Sonnet. Mitigation: compact at 30–35% context, not at 70%.
- **Over-reliance on examples**: Haiku copies a pattern blindly, including anti-patterns. Mitigation: reference must be a "positive example"; put counter-examples in a separate `Don't` block.
- **Inconsistency on multi-file changes**: rename / API changes across 5+ files often miss spots. Mitigation: ≥ 5-file mechanical change → upgrade to Sonnet via `general-purpose` OR use `grep -rn` to enumerate first, then surgical-edit each file.
- **Template-first anti-pattern**: Haiku defaults to filling fixed sub-headings, leading to table + prose redundancy. See § Pre-flight #4 (Content-first principle).
- **Emoji inflation**: Haiku tends to use ✅❌🔴 to emphasize every point. Mitigation: ≤ 3 emotion symbols per section; neutral statements get none.

---

## Relationship to Other Rules

- `output-discipline.md` — no fluff in plain text
- `subagent-strategy.md` — who delegates to whom
- `context-management.md` — when to compact
- `sonnet-pilot` SKILL — quality-first counterpart

This SKILL is the **execution layer**, not the decision layer.

---

## Sub-Agent Inheritance

Sub-agents do NOT inherit this SKILL's triggers. When dispatching, the parent prompt should include:

> "Run in Haiku Pilot Mode: read `.claude/rules/haiku-pilot.md` first, apply pre-flight, dispatch downstream per the router."
