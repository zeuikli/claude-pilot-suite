---
name: sonnet-pilot
description: |
  Sonnet-first quality playbook: through context engineering, sub-agent delegation,
  and forced self-review loops, get Sonnet 4.6 to produce code/doc quality on par
  with Opus 4.7. Escalate to Opus only when quantitative gates trigger.
  Triggers: "sonnet", "Sonnet", "Sonnet mode", "sonnet-pilot".

  Do NOT use for: pure cost optimization, file-count cognitive heuristic,
  agent dispatch table, CLAUDE.md / rules audit, harness health check,
  default Haiku execution (use haiku-pilot for that).
  This SKILL is a Sonnet quality-enhancement router, not a generic decision tree.
allowed-tools: Read, Grep, Glob, Bash, TodoWrite
---

# Sonnet Pilot — Quality-First Execution Playbook

## Thesis

**Deep context engineering + precise delegation = quality floor protection + predictable escalation.**

> Augment Code eval (2026-04): Best-quality AGENTS.md provides performance gain equivalent to one model tier (Haiku→Sonnet or Sonnet→Opus effective). Bad documentation is worse than no documentation, by ~30%.
> AgentOpt (arxiv 2604.06296): On HotpotQA, strongest planner alone (Opus full stack) = 31.71%; weak planner + strong solver (layered delegation) = 74.27%. **Individual model capability does NOT predict ensemble performance.**

**What this SKILL actually does** (empirically validated, A/B/C benchmark, n=5 wiki tasks, Opus 4.7 graded):
- Sonnet + SKILL = 283/300; Sonnet Plain = 283/300; Opus = 286/300
- **SKILL net gain on routine extraction = 0**: on structured wiki tasks, Sonnet already performs near-Opus without this playbook
- **SKILL value = floor protection on hard tasks**: complex tasks with ambiguous requirements, multi-step agentic, cross-module design — where Sonnet without structure tends to miss assumptions, skip self-review, or produce over-engineered output

→ Think of this SKILL as **insurance**: no observable premium on easy days; prevents 10–30% quality loss on hard days.
→ If your tasks are mostly "extract facts from a structured document" → you may not see measurable gain; activate this for complex implementation and agentic tasks.
→ Sonnet as planner + advisor()/reviewer (Opus) on-demand > Opus full-stack for most tasks.

> Cost ratio: Sonnet ($3/$15) vs Opus ($15/$75). Quality first does NOT mean always-Opus; **Opus only fires when quantitative gates trigger.**

---

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

**Why it matters**: Opus naturally does this in reasoning; Sonnet tends to skip the intermediate decision record and emit results directly, making reasoning quality undiffable and review hard. Decision-log is a synthetic substitute for Opus's deeper thinking, externalized.

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
- Any change > 30 LoC → dispatch `quick-code-reviewer` sub-agent for post-implementation review
- Touches auth/payment/user-data → dispatch `security-reviewer`
- Architecture or cross-module design → call `advisor()` before declaring done

**Sonnet-specific**: Sonnet's high confidence is a double-edged sword — easy to skip self-review. This pledge is the externally bolted-on "doubt mechanism".

### 5. Intermediate Checkpoint (multi-step tasks)

For tasks with ≥ 3 independent steps, **pause and verify** after each:
- Output: "Completed Step N / M: <one-sentence result>"
- Confirm intermediate output matches expectation before proceeding
- If anything unexpected → stop and ask, don't power through

> Empirical basis: AgentOpt experiments show step roles in multi-step pipelines matter more than overall model capability; Sonnet's failure mode is "reasoning without verification checkpoints".

### 6. Mid-Write Outline Verification (architecture / counter-factual / synthesis tasks)

**Trigger**: tasks tagged Architecture / Counter-factual / Synthesis (per § Task-Type Fast-Path table).

**Procedure** (run once at ~200 words drafted, before continuing):

| Check | Action on fail |
|-------|----------------|
| Still inside the question's scope? | Drop tangential paragraphs |
| Projected total ≤ 800w? (current_wc × est_remaining_sections) | Cut outline; merge sections |
| All N mandatory citations / subquestions covered? (list them; checkmark) | Add missing items first; do not write more body until covered |

**Why this exists**: Pre-flight #4 (Self-Review Loop, `git diff --stat`) fires *after* the answer is written. By that point, runaway answers are already 700–800 words. Mid-write checkpoint catches scope drift / missed mandatory citations *while still cheap to fix*.

**Empirical basis**: 2026-05-06 v0.2.1 benchmark Q18 — sonnet-pilot wrote 750 words on a CLAUDE.md refactor plan but **omitted 2 of 3 required verbatim numbers** (−28.64%, #33→#5). A 200-word checkpoint listing required citations would have surfaced the omission with 550 words still to write. Net loss: −6 vs Opus baseline.

**Skip when**: Easy recall, Wiki/ref extraction, or Code implementation tasks (where the gate adds overhead with no expected catch).

### 7. Source-Verify (Line-Level, required for paper / wiki / spec citation tasks)

After drafting any answer that cites numbers, model names, verbatim quotes, or paper conclusions, run this 4-step loop **before declaring done**:

1. **Identify**: list every numeric / proper-noun / verbatim citation in your draft.
2. **Locate**: for each, run `grep -in "<term>" <source-path>` to obtain the actual line number.
3. **Annotate**: replace inline anchor with `(P0X §Y.Z:LineN)` format — line number is the key Sonnet-Pilot upgrade vs haiku-pilot's section-only verification.
4. **Verify**: re-read the source at that line; if the context contradicts your phrasing → rewrite the citation, do not append a hedge.

**Hard gate**: every numeric citation in a hard-task answer must include `:LineN`. Citations without line numbers fail D1 audit and require a retry.

> Empirical basis: 2026-05-08 6-agent 10Q benchmark — A3 (Opus+Pilot) earned D1 = 10/10 because every citation included grep-verified line numbers; A2 (Sonnet+Pilot) earned D1 = 9.5/10 because Source-Verify was section-level only. Closing this 0.5-point gap is the highest-ROI Sonnet-Pilot lever per § Verification analysis.

---

## Per-Task Router

> This router does NOT duplicate the full sub-agent dispatch table (see `subagent-strategy.md`). It only flags the "Sonnet quality mode" deltas.

### Task-Type Fast-Path

Identify task type first — skip inapplicable pre-flight steps to reduce overhead:

| Task type | Required pre-flight | Skip |
|-----------|--------------------|----|
| **Easy recall** (≤ 100w answer; ≤ 5 facts; single source; pure definition / number lookup) | None mandatory; replace self-check with single-line `fast-path: <reason>` | All — direct answer |
| **Wiki/ref extraction** (summarize, extract, list from a document) | #1 Reference Pattern, G-02 Upgrade Eval, G-03 Anti-pattern Check, Self-check template | #3 Reasoning Chain, #5 Intermediate Checkpoint |
| **Code implementation** (bug fix, feature, refactor) | #2 Decision-Log, #3 Reasoning Chain, #4 Self-Review Loop | G-02, G-03, Self-check template |
| **Multi-step agentic** (> 3 independent steps) | All pre-flight; especially #5 Intermediate Checkpoint | — |
| **Simple Q&A / lookup** | None mandatory; use judgment | All |

This fast-path prevents applying wiki-citation overhead to code tasks and vice versa.

### Quality Enhancements (overlay on subagent-strategy.md)

| Task type | Standard delegation | Quality-mode addition |
|-----------|---------------------|----------------------|
| Cross-module implementation | `implementer` (Sonnet) | After implementation → `quick-code-reviewer` review |
| Architecture / tech selection | `reviewer` (Opus) | Use `advisor()` first to confirm problem framing |
| Bug fix | `bugfix` skill (if installed) | Add `git log -10 --oneline` regression check |
| Test writing | `test-writer` (Sonnet) | After tests → `quick-code-reviewer` for boundary coverage |
| Pre-commit | `/deep-review` | Same; Sonnet mode enforces this (rule already exists, this is reminder) |

### When to call advisor() directly (instead of via reviewer sub-agent)

- Need a second opinion before "complete" (architecture decisions, security design, major refactors)
- Stuck > 2 attempts; suspect direction itself is wrong
- About to commit but unsure if heading the right way

### Forced Per-Question Upgrade Evaluation (wiki/ref citation tasks)

After completing each question in a wiki/ref citation task, output one line:

```
Upgrade eval Q<N>: <hit/no-hit> — <one-sentence reason>
```

Example: `Upgrade eval Q3: no-hit — task is information extraction, no cross-service design decision.`
Example: `Upgrade eval Q4: hit — question requires security architecture judgment; escalating to advisor().`

This is **not optional** on citation tasks — D4 (Upgrade Awareness) in the benchmark rubric specifically scores whether the model demonstrates awareness of its own limits per question. Missing this evaluation cost 13/50 points on D4.

---

## Wiki Anti-pattern Fidelity Check

For any task that extracts from a wiki/reference document (e.g., "summarize gotchas", "list anti-patterns", "extract best practices"):

**Before declaring the answer complete:**
1. Re-read the source section for any `Anti-pattern`, `Gotcha`, `Don't`, `❌`, or warning-flavored heading
2. Verify each one maps to a named anchor in your answer (e.g., `Anti-pattern #1: ...`, `Gotcha [Source]: ...`)
3. If any anti-pattern from the source is missing from your answer → add it before completing

**Why this matters**: pilot-benchmark D2 (Detail Density) showed Sonnet tends to over-represent positive patterns and under-represent anti-patterns/gotchas from source material. A complete wiki extraction requires symmetric coverage of both.

This check takes < 30 seconds and prevents the most common source of D2 point loss.

---

## Escalation Gate (Sonnet → Opus)

> **Sonnet covers most tasks. Opus is 5× the cost; this gate is stricter than haiku-pilot's.**

**Stay on Sonnet** (any one true):
- Task touches ≤ 19 independent files
- Same problem attempted < 3 times
- No trigger keywords ("architecture decision", "design review", "security review", "threat modeling", "tech selection")
- Code-gen single response ≤ 500 LoC
- Reachable via `advisor()` (Sonnet session + Opus advisor ≈ full Opus quality)

**Escalate to Opus 4.7** (any one true):
- Real global architecture decision (affects multiple services' design)
- Security architecture design (not just security code review — designing the auth system)
- **Q type tagged Architecture / Counter-factual / Synthesis (hard)** per § Task-Type Fast-Path table — auto-escalate; do not classify hard-tasks as gate "no-hit". Source: 2026-05-08 6-agent 10Q benchmark Q10 (architecture hard) — A2 (Sonnet+Pilot) self-classified as no-hit and lost −5 to A3 (Opus+Pilot).
- Same problem failed ≥ 3 times AND not a context issue
- `advisor()` explicitly says "this needs Opus"
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
          Still fails (2nd attempt) → call advisor() to verify direction
              ↓
          advisor() advised but still fails (3rd attempt)
              ↓
          Escalate to Opus 4.7 (do NOT skip advisor(); advisor is the bridge from Sonnet session to Opus opinion)
```

---

## Expected Effects

> Assumption: 80% of tasks complete in Sonnet session, 20% trigger advisor()/reviewer (Opus).

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
Quality target: Sonnet + good context ≈ Opus baseline; when Opus does fire, it surpasses the baseline.

---

## Known Gotchas (Sonnet 4.6 specific)

- **Over-engineering trap**: Sonnet sees "elegantization opportunity" and adds an abstraction layer, violating `core.md` § "Surgical Changes". Mitigation: Pre-flight #2 Decision-Log forces "Rejected: ..." rationale, making over-engineering reviewable.
- **Confidence blindness**: Sonnet has higher confidence than Haiku, ironically less likely to self-doubt — tends to skip `git diff --stat` and self-review. Mitigation: Pre-flight #4 Self-Review Loop is mandatory; "feels fine" is not an exemption.
- **Multi-step reasoning without checkpoints**: Sonnet can do multi-step reasoning, but doesn't auto-pause to verify mid-stream — easily "right direction, one step wrong, runs forward anyway". Mitigation: Pre-flight #5 Intermediate Checkpoint forces per-step status output.
- **Ambiguity self-resolve**: Sonnet's high capability biases toward "guess user intent and implement" rather than listing options. Violates `core.md` § "list options for user confirmation". Mitigation: Pre-flight #3 mandates listing.
- **Error handling for impossible cases**: Sonnet adds guards for cases that can't happen (e.g. internal function null check), violating `core.md` § "don't code for impossible scenarios". Mitigation: Decision-Log forces "why this handler".
- **Explanation tail**: After code, Sonnet tends to write long explanations, violating `output-discipline.md` § "no question restatement / no fluff". Mitigation: max 3 bullets after code block; do NOT restate code already written.
- **Refactor-while-fixing syndrome**: Sonnet "tidies up" surrounding code during a bug fix or feature add, mixing intent in diffs and breaking `core.md` § commit atomicity. Mitigation: `git diff --stat` then verify each change is in scope.

---

## Verification

| Hypothesis | How to verify |
|-----------|---------------|
| Sonnet + good context ≈ Opus baseline quality | A/B run 5 typical tasks: sonnet-pilot vs Opus full-stack; compare with `/deep-review` scoring |
| Escalation gate accurate | After 1 week, audit Opus escalations: how many fit a gate, how many were advisor() recommendations? |
| 55–65% savings? | Use your cost tracking tool; compare monthly |
| Decision-Log compliance | Grep completion answer files: `grep -c "Choice:\|Rejected:" <output_file>` ≥ 1 per non-obvious decision |

> Don't trust LLM self-evaluation. External tool verification is the basis.

---

## Relationship to Other Infrastructure

- `haiku-pilot` = cost-first mode, default Haiku; this SKILL = quality-first mode, default Sonnet. Switch via trigger phrases (`Haiku mode` / `haiku-pilot` or `Sonnet mode` / `sonnet-pilot`); there is no slash command.
- `subagent-strategy.md` rule = dispatch table (this SKILL adds "quality enhancements" overlay only; doesn't rewrite dispatch)
- `output-discipline.md` rule = output formatting baseline
- `advisor()` = bridge from Sonnet session to Opus opinion (haiku-pilot uses rarely; this SKILL uses frequently)
- (Optional) `harness-eval` skill = harness health audit
- Citation enforcement is **built into this SKILL** (Pre-flight #4 mirrored from haiku-pilot). Per-source-type anchor vocabulary: shared reference `anchor-dictionary.md` (if available alongside the suite).

This SKILL is the **execution layer**, not the decision layer.

---

## Sub-Agent Inheritance

Sub-agents do NOT inherit this SKILL's triggers. When dispatching, the parent prompt should include:

> "Run in Sonnet Pilot Mode: read `.claude/rules/sonnet-pilot.md` first, apply decision-log requirement and intermediate checkpoints."
