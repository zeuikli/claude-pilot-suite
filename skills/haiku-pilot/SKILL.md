---
name: haiku-pilot
description: |
  Haiku-first execution playbook: through deliberate prompt structure, sub-agent
  delegation, and quantitative escalation gates, get Haiku 4.5 to produce
  near-Opus quality on most tasks. Escalate to Sonnet/Opus only when gates
  trigger.
  Triggers: "Haiku mode", "haiku-pilot", "use Haiku", "save tokens with quality",
  "Haiku 模式", "全力 Haiku", "升級判斷", "節費執行".

  Do NOT use for: cost-only token optimization, file-count cognitive heuristic,
  agent dispatch table, CLAUDE.md / rules audit, harness health check.
  This SKILL is a runtime router + escalation gate, not a decision tree
  or directory.
allowed-tools: Read, Grep, Glob, Bash, TodoWrite
---

# Haiku Pilot — Default-Haiku Execution Playbook

## Thesis

**Weak planner + strong delegation > strong planner doing everything.**

> AgentOpt (arxiv 2604.06296): On HotpotQA, Opus alone = 31.71%; Ministral 3 8B planner + Opus solver = 74.27%.
> Augment Code eval (2026-04): Best-quality AGENTS.md provides performance gain equivalent to one model tier (Haiku → Sonnet effective).

→ Default parent session to **Haiku 4.5**; aggressively delegate to sub-agents; **escalate only on quantitative gate trigger**.

> "Documentation / process design improvements ≥ model upgrade" is the engineering basis. Blindly upgrading models in multi-stage pipelines often *reduces* performance (planner doesn't delegate).

---

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

### 4. Citation Anchor Enforcement

When citing wiki / docs / spec for numbers, conclusions, or anti-patterns, attach a structured anchor:
- `(<source> Step N)` — wiki step number
- `(anti-pattern #N)` — wiki anti-pattern number
- `(<file>:line-range)` — code / config line ref

If no precise anchor available → tag `[unverified]`. Never fake a source.
Violation: retry that paragraph; do NOT declare done.

> Empirical basis: anchor density discrepancy is the primary driver of citation accuracy variance between Haiku and Sonnet/Opus on benchmark tasks. See `citation-discipline` skill if installed.

### 5. Content-First Structure (not Template-First)

**Forbidden**: filling every question with a fixed sub-heading template; restating in prose what the table already says.

**Use instead**: the question's nature determines the structure:
- Single decision → bulleted list
- ≥ 3 anti-patterns → 1 combined table (with "trigger / solution / Lab vs Prod" columns), NOT N separate tables
- Multi-option comparison → table lists only "final decision + exclusion rationale", does NOT enumerate all rejected options
- Emoji budget: ≤ 3 per section (✅/❌/🔴); neutral statements get none

> Empirical basis: template-first wastes ~30% tokens AND lowers detail density vs content-first.

---

## Per-Task Router (decision table → existing skills)

> This router does NOT duplicate the full skill directory. See your workspace's skill resolver / index for the full list.

### Coding tasks

| Task | Sub-agent (suggested) | Model |
|------|----------------------|-------|
| Single-file ≤ 30 LoC surgical edit | `haiku-implementer` (or your equivalent) | Haiku 4.5 |
| Cross-module / > 30 LoC / requires design | `implementer` | Sonnet 4.6 |
| Bug with failing test | `bugfix` skill (if installed) | Haiku 4.5 |
| Bug without test | `debug` skill (if installed) | Haiku → Sonnet if stuck |
| Test writing | `test-writer` | Sonnet 4.6 |

### Research tasks

| Task | Sub-agent | Model |
|------|----------|-------|
| ≥ 10-file fact-finding | `researcher` | Haiku 4.5 |
| Codebase structure inventory | `architecture-explorer` | Haiku 4.5 |

### Review tasks

| Task | Sub-agent | Model |
|------|----------|-------|
| Pre-commit multi-dimension review | `/deep-review` | mixed (parallel) |
| Single-file / single-function review | `quick-code-reviewer` | Sonnet 4.6 |
| Cross-module architecture / tech selection | `reviewer` | Opus 4.7 |
| Auth / payment / user-data touchpoints | `security-reviewer` | Sonnet 4.6 |
| OWASP / proactive threat modeling | `security-auditor` | Sonnet 4.6 |

### Docs tasks

| Task | Sub-agent | Model |
|------|----------|-------|
| README / CHANGELOG / API docs | `doc-writer` | Haiku 4.5 |
| Major doc rewrite | `doc-writer` | Sonnet 4.6 |

> *Customize for your workspace*: replace agent names with the agents that exist in your `.claude/agents/` directory. If you don't have specialized agents yet, the `general-purpose` agent + a model override works as a fallback.

---

## Escalation Gates (quantitative, not "when complex")

> **Decompose first, escalate second**: before deciding "should I upgrade to Sonnet/Opus?", **try sub-agent decomposition** (researcher / implementer / test-writer / reviewer parallel fan-out). Decomposable = stay Haiku + delegate; non-decomposable = upgrade.

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

**Escalate to Opus 4.7 (or `advisor()`)**:
- Trigger keywords: "architecture decision", "design review", "tech selection", "security review", "threat modeling"
- Multiple Sonnet attempts haven't converged (rare)
- User explicitly says "use Opus" / "consult advisor"

> Escalation isn't linear: context pollution → try `/compact` first; do NOT directly upgrade the model. See `context-management.md`.

**Escalation Decision Tree**:
```
New task arrives
    ├── Can it be decomposed into ≥3 independent sub-tasks?
    │   ├── Yes → fan-out to sub-agents (stay Haiku as planner)
    │   └── No  → continue below
    ├── Touches ≥10 files OR cross-module design?
    │   ├── Yes → Sonnet implementer
    │   └── No  → stay Haiku
    ├── Failed ≥3 times on same problem?
    │   ├── Yes → Sonnet (not context issue confirmed)
    │   └── No  → retry / compact
    └── Architecture/security/design keyword?
        ├── Yes → Opus / advisor()
        └── No  → stay Haiku
```

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
- **Inconsistency on multi-file changes**: rename / API changes across 5+ files often miss spots. Mitigation: ≥ 5-file mechanical change → upgrade Sonnet `implementer` OR use `grep -rn` to enumerate first, then surgical-edit each file.
- **"3 failures before escalation" anti-pattern**: users get impatient quickly. Mitigation: at first failure, announce "I'll try once more; if it fails again I'll escalate to Sonnet" — gives user opt-out.
- **Template-first anti-pattern**: Haiku defaults to filling fixed sub-headings, leading to table + prose redundancy. See § Pre-flight #5 (Content-first principle).
- **Emoji inflation**: Haiku tends to use ✅❌🔴 to emphasize every point, wasting 5–10% tokens and lowering info density. Mitigation: ≤ 3 emotion symbols per section; neutral statements get none.
- **Anti-pattern count free-listing**: Haiku adds new items beyond the source's actual list to seem complete. Mitigation: if source lists N → answer with N. Derived inferences go in a separate `[derived]` section.
- **Citation semantic drift**: Haiku uses vague phrasing ("the wiki mentions X", "as the wiki example shows") instead of anchored citations (`Anti-pattern #N: ...`, `Step N: ...`, `Gotcha [Source]: ...`). 2026-05-04 benchmark measured anchor density at 0.75/100w vs Sonnet 3.43/100w (4.6× gap). Mitigation: vague phrases trigger a retry — every claim about a source document must cite a specific anchor, not just describe it.

---

## Verification

| Hypothesis | How to verify |
|-----------|---------------|
| Haiku + this SKILL ≈ Opus baseline quality | A/B run 5 typical tasks; compare with `/deep-review` scoring |
| Escalation gate quantitative thresholds are accurate | After 1 week, audit escalation events: how many fit a gate, how many were gut feel? |
| Actually saving 70–85%? | Use your cost tracking tool; compare to baseline |
| Citation Anchor (Pre-flight #4) compliance | Pre-completion grep on Haiku-completed answer files: `grep -cE "Step [0-9]+\|anti-pattern #[0-9]+\|:[0-9]+-[0-9]+" <answer_file>` ≥ 1 per 200 words. If not, retry to fix. |

> Don't trust LLM self-evaluation. External tool verification is the basis.

---

## Relationship to Other Infrastructure

- `output-discipline.md` rule = "no fluff in plain text"
- `subagent-strategy.md` rule = "who delegates to whom"
- `context-management.md` rule = "when to compact"
- This SKILL = "Haiku-default router + escalation gate"
- `sonnet-pilot` SKILL = quality-first counterpart (the same suite covers both modes)
- (Optional) `harness-eval` skill = harness health audit
- (Optional) `citation-discipline` skill = cross-source citation enforcement (this SKILL § Pre-flight #4 is wiki-default; cross-source needs the dedicated skill)

This SKILL is the **execution layer**, not the decision layer.

---

## Sub-Agent Inheritance

Sub-agents do NOT inherit this SKILL's triggers. When dispatching, the parent prompt should include:

> "Run in Haiku Pilot Mode: read `.claude/rules/haiku-pilot.md` first, apply pre-flight, dispatch downstream per the router."
