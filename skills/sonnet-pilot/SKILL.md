---
name: sonnet-pilot
description: |
  Sonnet-first quality playbook: through context engineering, sub-agent delegation,
  and forced self-review loops, get Sonnet 4.6 to produce code/doc quality on par
  with Opus 4.7. Escalate to Opus only when quantitative gates trigger.
  Triggers: "sonnet", "sonnet-pilot", "sonnet mode".

  Do NOT use for: pure cost optimization, file-count cognitive heuristic,
  agent dispatch table, CLAUDE.md / rules audit, harness health check,
  default Haiku execution (use haiku-pilot for that).
  This SKILL is a Sonnet quality-enhancement router, not a generic decision tree.
allowed-tools: Read, Grep, Glob, Bash, TodoWrite
---

# Sonnet Pilot — Quality-First Execution Playbook

## Per-Session Pre-flight (run once per session)

### 1. Reference Pattern Technique

| ❌ Abstract description | ✅ Point to a file |
|----------------------|-------------------|
| "Write a validation module" | "Follow the pattern in `src/auth/validator.ts`" |
| "Add a hook similar to useUser" | "Pattern: `hooks/useUser.ts:42-78`" |
| "Reference the deploy flow we used last time" | "Follow `<your-playbook>.md` Step 3-6" |

**Sonnet-specific**: Sonnet given an abstract description tends to design from scratch; concrete reference = narrows design space to "variant of correct pattern", not "invent from zero".

### 2. Decision-Log Requirement (Opus has this built-in; Sonnet must enforce)

For every non-obvious technical decision, before completion output 2 lines:
```
Choice: <what you decided>
Rejected: <option considered but excluded> — Reason: <one sentence>
```

**Why it matters**: Opus naturally does this in reasoning; Sonnet tends to skip the intermediate decision record and emit results directly, making reasoning quality undiffable and review hard.

### 3. Reasoning Chain Before Code

Before implementation, output (don't skip):
1. **1-2 sentence task understanding** (your interpretation, not parroting)
2. **Key assumptions** (≥ 1, prefixed with "Assumption: ...")
3. **Implementation plan** (bullet list, ≤ 5 steps)
4. If multiple reasonable interpretations exist → **list options for user confirmation**, don't pick one yourself

> Source: `core.md` § "Think-Before-Coding". Sonnet has higher confidence than Haiku, paradoxically MORE prone to skipping assumption verification; this pre-flight enforces it.

### 4. Self-Review Loop (quality gate)

Before declaring "complete":

```bash
git diff --stat   # confirm change scope
git status        # confirm no unintended additions
```

**Explain each modified file in one sentence**. Then:
- Any change > 30 LoC → dispatch `general-purpose` (Sonnet) sub-agent for post-implementation review
- Touches auth/payment/user-data → dispatch `general-purpose` (Sonnet) with security focus
- Architecture or cross-module design → use `Plan` agent before declaring done

**Sonnet-specific**: Sonnet's high confidence is a double-edged sword — easy to skip self-review. This pledge is the externally bolted-on "doubt mechanism".

### 5. Intermediate Checkpoint (multi-step tasks)

For tasks with ≥ 3 independent steps, **pause and verify** after each:
- Output: "Completed Step N / M: <one-sentence result>"
- Confirm intermediate output matches expectation before proceeding
- If anything unexpected → stop and ask, don't power through

---

## Per-Task Router

> This router only flags the "Sonnet quality mode" deltas on top of `subagent-strategy.md`.

### Quality Enhancements (overlay on subagent-strategy.md)

| Task type | Standard delegation | Quality-mode addition |
|-----------|---------------------|----------------------|
| Cross-module implementation | `general-purpose` (Sonnet) | After implementation → `general-purpose` review pass |
| Architecture / tech selection | `Plan` (Opus) | Use `Plan` to confirm problem framing first |
| Bug fix | `general-purpose` (Haiku) | Add `git log -10 --oneline` regression check |
| Test writing | `general-purpose` (Sonnet) | After tests → verify boundary coverage |
| Pre-commit | `general-purpose` review | Sonnet mode enforces this before every commit |

### When to use the `Plan` agent directly

- Need a second opinion before "complete" (architecture decisions, security design, major refactors)
- Stuck > 2 attempts; suspect direction itself is wrong
- About to commit but unsure if heading the right way

---

## Escalation Gate (Sonnet → Opus)

> **Sonnet covers most tasks. Opus is 5× the cost; this gate is stricter than haiku-pilot's.**

**Stay on Sonnet** (any one true):
- Task touches ≤ 19 independent files
- Same problem attempted < 3 times
- No trigger keywords ("architecture decision", "design review", "security review", "threat modeling", "tech selection")
- Code-gen single response ≤ 500 LoC
- Reachable via `Plan` agent (Sonnet session + Opus Plan ≈ full Opus quality)

**Escalate to Opus 4.7** (any one true):
- Real global architecture decision (affects multiple services' design)
- Security architecture design (not just security code review — designing the auth system)
- Same problem failed ≥ 3 times AND not a context issue
- `Plan` agent explicitly says "this needs Opus"
- User explicitly says "use Opus"

> **Note**: context issue → try `/compact` first; doesn't count as model failure. Sonnet's long-context degradation kicks in around ~500K tokens (vs Haiku's ~300K), but still monitor.

---

## Failure Diagnostic Flow

```
Sonnet fails 1×
    ↓
 Context issue? (symptoms: misrestating requirements, ignoring given references, repeating prior errors)
    ├── Yes → /compact or /rewind; redo Pre-flight #1-3; retry (still Sonnet)
    └── No  → model or direction issue
              ↓
          Was Pre-flight #2 Decision-Log done? If not, do it and retry
              ↓
          Still fails (2nd attempt) → use Plan agent to verify direction
              ↓
          Plan agent advised but still fails (3rd attempt)
              ↓
          Escalate to Opus 4.7
```

---

## Expected Cost Savings

> Assumption: 80% of tasks complete in Sonnet session, 20% trigger `Plan` agent (Opus).

Reference pricing (2026-04 public, per 1M tokens, input/output average):
- Sonnet 4.6: $3 / $15 → avg $9
- Opus 4.7: $15 / $75 → avg $45

```
Sonnet  0.80 × $9   = $7.20
Opus    0.20 × $45  = $9.00
Total                = $16.20 / 1M token (rough — input >> output in practice)

vs full-Opus baseline: $45
Savings: 1 - 16.20/45 = 64%
```

**Conservative estimate: 55–65% savings** (depends on cache hit rate, input/output ratio).

---

## Known Gotchas (Sonnet 4.6 specific)

- **Over-engineering trap**: Sonnet sees "elegantization opportunity" and adds an abstraction layer, violating `core.md` § "Surgical Changes". Mitigation: Pre-flight #2 Decision-Log forces "Rejected: ..." rationale, making over-engineering reviewable.
- **Confidence blindness**: Sonnet has higher confidence than Haiku, ironically less likely to self-doubt — tends to skip `git diff --stat` and self-review. Mitigation: Pre-flight #4 Self-Review Loop is mandatory; "feels fine" is not an exemption.
- **Multi-step reasoning without checkpoints**: Sonnet can do multi-step reasoning, but doesn't auto-pause to verify mid-stream. Mitigation: Pre-flight #5 Intermediate Checkpoint forces per-step status output.
- **Ambiguity self-resolve**: Sonnet's high capability biases toward "guess user intent and implement" rather than listing options. Violates `core.md` § "list options for user confirmation". Mitigation: Pre-flight #3 mandates listing.
- **Error handling for impossible cases**: Sonnet adds guards for cases that can't happen, violating `core.md` § "don't code for impossible scenarios". Mitigation: Decision-Log forces "why this handler".
- **Explanation tail**: After code, Sonnet tends to write long explanations, violating `output-discipline.md` § "no fluff". Mitigation: max 3 bullets after code block.
- **Refactor-while-fixing syndrome**: Sonnet "tidies up" surrounding code during a bug fix, mixing intent in diffs and breaking `core.md` § commit atomicity. Mitigation: `git diff --stat` then verify each change is in scope.

---

## Relationship to Other Rules

- `haiku-pilot` SKILL — cost-first mode; switch with trigger `haiku-pilot`
- `subagent-strategy.md` — dispatch table (this SKILL adds quality enhancements only)
- `output-discipline.md` — output formatting baseline
- `context-management.md` — when to compact

This SKILL is the **execution layer**, not the decision layer.

---

## Sub-Agent Inheritance

Sub-agents do NOT inherit this SKILL's triggers. When dispatching, the parent prompt should include:

> "Run in Sonnet Pilot Mode: read `.claude/rules/sonnet-pilot.md` first, apply decision-log requirement and intermediate checkpoints."
