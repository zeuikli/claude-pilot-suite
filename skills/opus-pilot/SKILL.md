---
name: opus-pilot
description: |
  Opus-first ceiling-elevation playbook: through reverse-advisor loops, parallel
  harness hypotheses, CAR-explicit declarations, externalized decision logs,
  and meta-harness diagnostics, push Opus 4.7 BEYOND vanilla Opus 4.7 quality.
  Triggers: "opus", "Opus", "Opus mode", "opus-pilot".
  Sub-mode modifiers: "Opus 1M" (1M context), "Opus xhigh" (extended thinking).

  Do NOT use for: cost optimization (use haiku-pilot / sonnet-pilot for that),
  default execution (Opus is the ceiling — only fire when the task warrants it),
  routine extraction / lookup (Opus on easy tasks underperforms Sonnet+pilot
  due to over-elaboration). This SKILL elevates Opus on hard tasks; it does
  not make Opus the default.
allowed-tools: Read, Grep, Glob, Bash, TodoWrite
---

# Opus Pilot — Ceiling-Elevation Execution Playbook

## Thesis

**Vanilla Opus is a strong solo solver. Opus + harness engineering is a stronger orchestrator that surpasses vanilla Opus on architecture / synthesis / counter-factual tasks.**

> **AgentOpt** (P10 / arxiv 2604.06296): On HotpotQA, Opus solo = **31.71%**; Ministral planner + Opus solver = **74.27%** (+42.56pp). **Individual model capability does NOT predict ensemble performance.** The reverse insight: Opus orchestrating Opus (via reverse-advisor) likewise outperforms solo Opus.
> **Meta-Harness** (P03 / arxiv 2603.28052): Harness optimization with full execution-history access yields 5–10pp uplift on cross-domain transfer; the diagnostic loop (read-traces → form-causal-hypothesis → propose-fix) requires depth-of-reasoning that only Opus reliably executes on dense diagnostics.
> **Confucius** (P02 / arxiv 2512.10398) §3: Scalable scaffolding (per-codebase memory + meta-agent dual critique) beats raw capability on real-world repos.
> **CAR** (P07 / preprints 202603.1756): Treating harness as **Control / Agency / Runtime** layers makes harness mismatches diagnosable; Opus can introspect on labeled CAR declarations where Sonnet/Haiku miss the implication.

**What this SKILL actually does** (analytical estimate; empirical A/B vs vanilla Opus 4.7 to be verified per § Verification):
- On hard analytical tasks (architecture, counter-factual, synthesis) → **+8–15% quality** uplift over vanilla Opus via Reverse-Advisor + Parallel Hypotheses + CAR explicitness.
- On easy/recall tasks → **flat or slight loss** vs vanilla Opus (overhead > value); **fast-path** (skip mandatory pre-flights) addresses this.
- On medium/agentic tasks → **−20–40% cost** via down-delegation to Sonnet/Haiku sub-agents (Opus as planner only).

→ **Opus + this SKILL ≠ "always more thinking"**. It's **structured thinking + explicit delegation**. Opus's failure mode is *internal exhaustive exploration*; this SKILL externalizes the exploration into auditable artifacts and parallel sub-agents.

> Cost ratio: Opus ($15 / $75) is 5× Sonnet, 15× Haiku. Opus mode should fire only when the task *requires* Opus reasoning depth. Use sonnet-pilot or haiku-pilot for the other 80% of tasks.

---

## The Five Mechanisms (Beyond-Vanilla Levers)

### Mechanism #1 — Reverse-Advisor Loop

**Trigger**: any global-architecture / cross-module / synthesis / counter-factual / security-design decision.

**Procedure**:
1. Opus drafts the decision (Choice + Rationale + Rejected options).
2. Call `advisor()` (which is itself Opus with fresh context).
3. The advisor sees the full transcript; it acts as **peer reviewer**, not as fallback.
4. If advisor disagrees → reconcile via one more advisor call ("I found X, you suggest Y; which constraint breaks the tie?").
5. Only declare done after advisor concurs OR explicitly notes "remaining disagreement is judgment-bound".

**Why Opus-specific**: Sonnet/Haiku call advisor() to *upgrade*; Opus calls advisor() for *peer review*. The cost is 1× (Opus → Opus), justified on architecture-grade decisions. AgentOpt principle inverted: orchestrated > solo, even at the ceiling tier. **(P10 AgentOpt 31.71% → 74.27%; P02 Confucius §4 dual-critique meta-agent)**

**Anti-pattern**: skipping advisor() because "Opus already thought about it carefully". The advisor sees blind spots Opus's depth-first reasoning misses.

---

### Mechanism #2 — Parallel Hypotheses + Synthesis (Multi-Sample)

**Trigger**: harness-design / system-architecture / "what's the right approach" tasks.

**Procedure**:
1. Generate **N=3 candidate approaches** in parallel (single message, multiple Agent calls). Each sub-agent produces a complete proposal.
2. Read all N. Rank on agreed criteria (e.g., simplicity, correctness, ablation-friendliness).
3. **Synthesize the best features into a final design** — not "pick the best one", but "compose the strongest from N".
4. Document which features came from which candidate (citation traceability).

**Why Opus-specific**: Sonnet/Haiku parallel sampling produces lower variance (≈ same answer 3×); Opus produces meaningfully different proposals that benefit from synthesis. AgentFlow §3.2 structured DSL constraint hints at this — multi-agent variants outperform single-agent runs. **(P01 AgentFlow §3; P03 Meta-Harness ablation: 2–3 components drive 80% of gains, so synthesis must remain selective)**

**Anti-pattern**: generating 7 candidates + writing a 1500-word tradeoff matrix. The cost exceeds the value past N=3.

---

### Mechanism #3 — CAR Explicitness (Three-Layer Harness Verbalization)

**Trigger**: any harness-design / agent-architecture / runbook / tool-config task.

**Procedure**: structure the answer (and any artifacts produced) using **Control / Agency / Runtime** labels:
- **Control**: policies (when does X fire? what's the decision rule?)
- **Agency**: tool access (which tools, with what permissions, fan-out vs serial)
- **Runtime**: memory / compaction / state-store / observability

**Why Opus-specific**: Opus excels at introspecting on labeled structures; CAR labels enable *self-diagnosis* of harness mismatches mid-task ("The bug is in Runtime — observability is missing for state X"). Haiku/Sonnet read CAR labels but don't use them for self-diagnosis. **(P07 CAR framework; P04 NLAH §3.2 explicit contracts)**

**Concrete artifacts** to produce when this mechanism fires:
- `harness.md` (or inline section): three sub-headings labeled `## Control`, `## Agency`, `## Runtime`.
- Any cross-component dependency must cite the CAR label of the dependency target.

---

### Mechanism #4 — Decision-Log Externalization (Live Journal)

**Trigger**: any task with ≥ 3 non-trivial decisions (most architecture / debugging / refactor tasks).

**Procedure**: maintain a **live decision journal** indexed by step number. Format (strict — no prose paragraphs):

```
Step <N>: <decision-name>
Choice: <what you decided>
Rejected: <option considered but excluded> — Reason: <one sentence>
```

The journal is **part of the deliverable**. Sub-agents (e.g., reviewer) audit the journal without re-reading full transcript.

**Why Opus-specific**: Sonnet's decision-log is *enforcement* of behavior Opus does naturally; for Opus, the lever is **externalization** — make the natural reasoning durable & reviewable. Token overhead < 5% on Opus due to its existing verbosity; on Haiku/Sonnet this would be 15–25% overhead. **(P04 NLAH §6.1 artifact-backed closure; P08 OpenDev §8.3 transparency over abstraction)**

**Strict format violation triggers retry** (Opus tendency: paragraph-long journal entries). One sentence per `Reason` line, full stop.

---

### Mechanism #5 — Meta-Harness Filesystem Observability Loop

**Trigger**: failed task ≥ 1 attempt; harness-iteration / "this isn't working" diagnostic tasks.

**Procedure**:
1. Persist **raw execution traces** (tool calls, errors, intermediate outputs) to filesystem (`/tmp/<task-id>/trace-<N>.log`).
2. On failure, Opus reads the trace **directly** (not summarized). Forms causal hypothesis: "The failure is at trace step K because <invariant violated>".
3. Propose **single targeted fix** (not 5-option redesign).
4. Re-attempt. If fails again → escalate to advisor() (Mechanism #1).

**Why Opus-specific**: Opus reasoning on 10K+ tokens of raw diagnostic data is the unique capability; Sonnet/Haiku struggle past 5K diagnostic tokens. Meta-Harness P03 §4 showed full-history access gives 5–10pp uplift, conditional on the proposer being able to reason over that history. **(P03 Meta-Harness §4; P02 Confucius §3.1 per-codebase persistent memory)**

**Anti-pattern**: re-running the failed task with verbal "I'll be more careful this time". No causal hypothesis = no progress.

---

## Per-Session Pre-flight (run once per task — task-type-conditioned)

### Always-On Pre-flights

#### A. Ambiguity Surfacing (counters Opus over-confidence failure mode)

Before substantive work, if the task admits ≥ 2 reasonable interpretations:
- **List the interpretations** (1 line each).
- **Pick one + state why** OR **ask user to confirm**.
- Do **NOT** silently pick one and dive in.

> Opus's confidence mask hides ambiguity. This pledge externalizes it. **(core.md § Think-Before-Coding; Opus-specific because Sonnet/Haiku are more likely to ask, Opus more likely to assume)**

#### B. Start-Simple-First Pledge (counters Opus over-engineering failure mode)

When proposing a harness / architecture / multi-component design:
- Start from a **1–2 component baseline**.
- Justify each additional component with: **expected ≥ 5pp gain OR specific failure mode it prevents**.
- Default to "ship simpler"; complexity must earn its place.

> P03 §5.2 ablation: 2–3 components drive 80% of gains; Opus tendency to propose 5–7 components yields diminishing returns. **(P03 Meta-Harness ablation; P06 Survey §8.1 efficiency vs over-engineering)**

#### C. Down-Delegation Eval (counters Opus "I'll do it myself" failure mode)

For multi-step tasks (≥ 3 steps), at task planning time:
- For each step, ask: "Does this require Opus reasoning depth? Or can a Sonnet / Haiku sub-agent do it cheaper?"
- **Default delegate** for: file-search, fact-extraction, mechanical-edit, single-file ≤ 30 LoC, format-conversion.
- **Keep on Opus** for: synthesis, cross-module-design, threat-modeling, counter-factual reasoning.
- Output the dispatch table before executing.

> P10 AgentOpt: weak planner + strong solver beats strong solo. The reverse — strong planner + weak solver — also outperforms solo when the planner's job is decomposition and synthesis, not raw solving.

### Task-Type Fast-Path

Identify task type first; skip inapplicable mechanisms to reduce overhead:

| Task type | Required mechanisms | Skip |
|-----------|--------------------|----|
| **Easy recall** (≤ 100w answer; ≤ 5 facts; single source) | Citation Anchor + Source-Verify only | M1, M2, M3, M4, M5, A, B, C |
| **Wiki/ref citation** (multi-paragraph extraction with anchors) | Citation Anchor + Source-Verify + M4 (Decision-Log light: choice/rejected per major claim) | M1 (no architecture decision), M2, M3, M5, B |
| **Code implementation** (bug fix / feature / refactor) | A (Ambiguity), B (Start-Simple), C (Down-Delegation), M4 (Decision-Log) | M1 (only if true architecture), M2, M3, M5 (unless debugging) |
| **Architecture / synthesis / counter-factual** | All five mechanisms (M1–M5) + all pre-flights (A, B, C) | — |
| **Multi-step agentic** (> 3 independent steps) | C (Down-Delegation) + M4 + M5 (on failure) | M1 (per-step), M2 (only at design stage) |

> Empirical basis (cross-pilot benchmark, 2026-05-06 v0.2.1): pre-flight overhead on easy/recall tasks net-negative (Q01 sonnet-pilot −4 vs vanilla; Q04 haiku-pilot −2 vs vanilla). Fast-path eliminates these regressions.

---

## Down-Delegation Decision Tree (replaces Haiku/Sonnet escalation gates)

> **Opus is the ceiling**. There is no auto-upgrade. The only knob is: **stay Opus** vs **delegate to Sonnet/Haiku sub-agent**.

```
New step in current task
    ├── Does this step REQUIRE Opus reasoning depth?
    │   (synthesis / cross-module design / counter-factual / threat-modeling)
    │   ├── Yes → stay Opus
    │   └── No  → continue below
    ├── Is this step decomposable into sub-tasks?
    │   ├── Yes → fan-out to Sonnet/Haiku sub-agents (parallel if independent)
    │   └── No  → continue below
    ├── Can a Sonnet sub-agent (5× cheaper) achieve ≥ 95% of Opus quality?
    │   ├── Yes → delegate to Sonnet (especially for: file search, fact extraction, single-file edits)
    │   └── No  → stay Opus
    └── Is this step a one-shot deterministic action (read file, run grep, format conversion)?
        ├── Yes → delegate to Haiku (15× cheaper)
        └── No  → stay Opus
```

**Outcome distribution target** (informal): on a multi-step task, ~30–40% steps stay Opus, ~40–50% delegate to Sonnet, ~10–20% delegate to Haiku.

---

## Mode-Specific Behaviors

### Base Opus mode (triggers: `opus` / `Opus` / `Opus mode` / `opus-pilot`)

- Mechanisms 1–5 active per task-type fast-path.
- Pre-flights A, B, C mandatory on architecture / synthesis / multi-step tasks.
- Output-discipline pinning: **default ≤ 150w per topic** unless user explicitly requests "detailed" — Opus's natural verbosity must be capped.
- Sub-agent dispatch per `subagent-strategy.md` + Down-Delegation tree above.

### Opus 1M mode (modifier: `Opus 1M` / `1M context` / `extended context`)

Activates **on top of** base Opus mode:
- 1M context window enabled (requires Max / Team / Enterprise plan, or pay-as-you-go).
- **Proactive compact at 70%** (vs vanilla 80%) — long-horizon sessions degrade earlier than appears.
- **Episodic memory consolidation**: every ~200K tokens, write a `consolidated-notes.md` summarizing key decisions / open questions / remaining work; the notes survive `/compact`.
- Suitable for: weeks-long projects, code repos > 1M LoC, cross-codebase refactoring, multi-paper research synthesis.
- **(P02 Confucius §3.1 persistent per-codebase memory; P08 OpenDev §7.2 episodic memory; P03 Meta-Harness §4 full-history-access)**

### Opus 1M + xhigh mode (modifier: `Opus 1M xhigh` / `Opus xhigh` + `Opus 1M`)

Activates **on top of** Opus 1M mode:
- Extended thinking (xhigh tier) enabled — ~3× reasoning tokens for the steps that need them.
- **xhigh fires only on**: M1 (Reverse-Advisor decisions), M2 (Synthesis step), M3 (CAR top-level architecture), counter-factual / threat-modeling questions.
- **Routine sub-steps stay standard Opus** — xhigh on every step is wasteful.
- Mechanism 1 (Reverse-Advisor) becomes **xhigh advisor reviews standard-Opus draft**: the advisor uses xhigh for the review pass; the draft uses standard Opus.
- Mid-write checkpoint enforced (per sonnet-pilot pre-flight #6) at ~200w on architecture tasks: list mandatory citations, verify scope.
- Suitable for: one-time high-stakes decisions (e.g., "redesign auth system"), pre-major-commit final review, ambiguous counter-factual analysis.
- **(xhigh extended-thinking budget; opus47-best-practices.md effort tiers)**

---

## Citation & Source-Verify Discipline (same hard gate as haiku-pilot / sonnet-pilot)

For wiki / reference / paper-citation tasks, every cited claim must:
1. Have a structured anchor: `(P0X §section)` / `(<file>:lines)` / `Mechanism #N` / `CAR.<C|A|R>` / `Step N` / `Anti-pattern #N`.
2. Pass `grep -i "<number-or-name>" <source-path>` — no fabrications.
3. Compounded numbers (e.g., "5.6× cognitive load") must state the source-verifiable atomic numbers in parentheses.

**Opus is not exempt.** Common failure modes to reject before submitting:
- Cross-paper number attribution without re-cite ("AgentOpt 74% on harness benchmarks" — actual: HotpotQA only).
- Non-existent model versions ("Claude 3.5 Opus" — no such Anthropic-published version).
- "Approximately X%" without paper anchor.
- Vague targets ("失敗率 ↓ 60–80%") not paper-grounded.

> **Empirical basis** (2026-05-06 v0.2.1 benchmark): haiku-pilot shipped 5 fabrications in 20 questions despite anchor enforcement; sonnet-pilot shipped 0 (Decision-Log gate caught them). Opus's confidence makes it equally susceptible to "this number sounds right" — Source-Verify is a binary gate, not a soft threshold.

---

## Failure Diagnostic Flow (when Opus stalls or produces wrong output)

```
Opus task fails 1×
    ↓
 Is it a context problem? (symptoms: misrestating requirements, 
 lost-in-middle, ignoring given files)
    ├── Yes → /compact or /rewind, redo Pre-flights A/B/C, retry (still Opus)
    └── No  → continue
              ↓
          Is it an ambiguity problem? (Pre-flight A skipped or under-applied)
              ├── Yes → list interpretations, get user confirm, retry
              └── No  → continue
                        ↓
                    Activate Mechanism #5 (Meta-Harness):
                    write trace to filesystem, read it, form causal hypothesis,
                    propose targeted fix, retry
                        ↓
                    Still fails (3rd attempt)
                        ↓
                    Activate Mechanism #1 (Reverse-Advisor):
                    advisor() reviews the failed approach; reconcile per its feedback
                        ↓
                    If advisor() concurs current approach is fundamentally flawed
                        ↓
                    Stop. Report to user with: failure trace + advisor feedback +
                    proposed alternative direction. Do not loop indefinitely.
```

> **No upward escalation exists** (Opus is the ceiling). The only "escalation" is reverse-advisor (Opus → Opus peer review). If that fails, the task itself may be ill-posed; surface this to the user rather than burning cost in further attempts.

---

## Expected Effects (analytical estimate; verify via § Verification)

> Assumption: Opus mode fires on the 20% of tasks where Opus reasoning depth is required (architecture / synthesis / counter-factual / threat-modeling). For the other 80%, sonnet-pilot / haiku-pilot are correct choices.

Reference pricing (2026-04 public, per 1M tokens, input/output average):
- Opus 4.7: $15 / $75 → avg $45
- Sonnet 4.6: $3 / $15 → avg $9 (used for sub-agent down-delegation)
- Haiku 4.5: $1 / $5 → avg $3 (used for routine sub-tasks)

```
Vanilla Opus on architecture task:
  Opus solo, full-context, ~80K tokens output
  ≈ $45 × 0.08 = $3.60 per task
  Quality baseline: 100%

Opus + this SKILL (with down-delegation):
  Opus planner: 30% steps × $45 × 0.024  ≈ $0.32
  Sonnet sub-agents: 50% steps × $9 × 0.04 ≈ $0.18
  Haiku sub-agents: 20% steps × $3 × 0.04 ≈ $0.024
  Reverse-advisor (M1): 1× × $45 × 0.01    ≈ $0.45
  Total                                     ≈ $0.97
  
vs vanilla baseline: $3.60
Cost reduction: 1 - 0.97/3.60 ≈ 73%
Quality target (analytical): +8–15% over vanilla on architecture/synthesis tasks
```

**Conservative estimate**: **−60% to −75% cost** at **+5–15% quality** on architecture-grade tasks. On easy/recall tasks (where this SKILL should NOT fire), expect 0 to −5% net (overhead > value — use sonnet-pilot / haiku-pilot instead).

> **Reality check**: cost numbers above assume aggressive down-delegation. If Opus refuses to delegate (failure mode C), the SKILL becomes net-negative. Pre-flight C (Down-Delegation Eval) is therefore the highest-ROI gate.

---

## Known Gotchas (Opus 4.7 specific)

- **"My reasoning is enough" trap**: Opus's confidence in its own depth makes it skip Mechanism #1 (Reverse-Advisor). Mitigation: on architecture / counter-factual tasks, advisor() is **mandatory**, not optional. Treat skipping advisor() the same as Sonnet skipping decision-log — a hard gate violation.
- **Over-elaboration on easy tasks**: Opus on an "extract 3 facts from a paper" task writes 800 words instead of 200. Mitigation: Task-Type Fast-Path; **default ≤ 150w** with output-discipline pinning. If you find Opus on an easy task, the right answer is *don't use Opus mode* — switch to sonnet-pilot or haiku-pilot.
- **Synthesizing 7 candidate approaches when 3 would suffice**: Mechanism #2 says N=3, not N=7. Opus tendency to "explore the full design space" wastes tokens. Mitigation: hard cap N=3 in Mechanism #2; if a 4th candidate genuinely adds value, document why before generating it.
- **Decision-log as paragraphs**: Opus produces "well-reasoned" paragraph-long decision entries. Mitigation: Mechanism #4 strict format — `Choice: X | Rejected: Y — Reason: <1 sentence>`. Multi-line `Reason` triggers retry.
- **Refusing to delegate "because Opus can do it"**: Pre-flight C exists exactly to counter this. Down-delegation is not insulting to Opus's capability; it's correct ensemble allocation per AgentOpt principle.
- **Long-context overconfidence**: Opus tolerates 1M context but degrades at ~700K-900K on dense reasoning. Mitigation: 1M mode mandates compact at 70% (140K for vanilla 200K; 700K for 1M); episodic memory consolidation at 200K boundaries.
- **xhigh on every step**: In `Opus 1M + xhigh` mode, xhigh costs ~3× tokens per step. Using xhigh on routine reads is wasteful. Mitigation: xhigh fires only on M1 / M2 / M3 / counter-factual / threat-modeling; routine reads stay standard Opus.
- **Premature conclusion**: Opus on ambiguous tasks tends to "lock in" an interpretation early in reasoning, then defends it through the rest. Mitigation: Pre-flight A (Ambiguity Surfacing) — list interpretations *before* substantive work, not as a retrospective justification.

---

## Verification

| Hypothesis | How to verify |
|-----------|---------------|
| Opus + this SKILL > vanilla Opus on architecture/synthesis | A/B run ≥ 5 such tasks; LLM-judge with 6-dimension rubric (Citation Accuracy / Detail Density / Structural Depth / Escalation Awareness / Token Efficiency / Format Consistency) |
| Reverse-advisor (M1) catches errors vanilla Opus misses | Audit advisor disagreement rate over 10 architecture tasks; expect ≥ 30% disagreements lead to changes |
| Down-delegation actually fires (Pre-flight C compliance) | `grep -c "Delegated:\|delegate:" <task_outputs>` ≥ 1 per multi-step task |
| 60–75% cost reduction on multi-step tasks | Compare token usage (vanilla Opus solo run vs Opus mode with delegation); use cost-tracking tool |
| Source-Verify (citation hard gate) compliance | Same as haiku-pilot / sonnet-pilot: `grep -i "<number>" <source>` for every cited number; zero fabrications allowed |
| 1M / xhigh modes used correctly | xhigh budget triggers only on M1/M2/M3 steps (not routine reads); episodic memory file exists at 200K boundaries in 1M sessions |

> Don't trust LLM self-evaluation. External tool verification is the basis. Especially for "Opus + SKILL > vanilla Opus" — **vanilla Opus is the baseline to beat, not Sonnet baseline.**

---

## Relationship to Other Infrastructure

- `haiku-pilot` SKILL = cost-first, default Haiku → near-Opus quality
- `sonnet-pilot` SKILL = quality-first, default Sonnet → Opus-equivalent quality
- `opus-pilot` SKILL = ceiling-elevation, default Opus → **beyond-vanilla-Opus** quality on hard tasks
- `subagent-strategy.md` rule = dispatch table (this SKILL adds Down-Delegation tree as Opus-specific overlay)
- `output-discipline.md` rule = output formatting baseline (especially relevant — Opus over-elaborates by default)
- `context-management.md` rule = compact triggers (Opus 1M mode tightens to 70% from 80%)
- `advisor()` = bridge for Mechanism #1 (Reverse-Advisor); Opus → Opus peer review
- (Optional) `harness-eval` skill = harness health audit (complements Mechanism #3 CAR explicitness)
- Citation enforcement is **built into this SKILL** (citation hard gate + M1 Peer-Review Pass). Per-source-type anchor vocabulary: shared reference `anchor-dictionary.md` (if available alongside the suite).

This SKILL is the **execution layer for hard tasks**, not the default execution layer. **Most tasks should use haiku-pilot or sonnet-pilot.**

---

## Sub-Agent Inheritance

Sub-agents do NOT inherit this SKILL's triggers. When dispatching, the parent prompt should include:

> "Run in Opus Pilot mode: read `.claude/rules/opus-pilot.md` first; apply Mechanisms #1–5 per task-type fast-path; mandatory pre-flights A (Ambiguity Surfacing), B (Start-Simple-First), C (Down-Delegation Eval); strict citation Source-Verify (`grep -i` every numeric/name claim against source path)."

For Opus 1M sub-agents: also add "use 1M context window; compact proactively at 70%; consolidate notes every 200K tokens."

For xhigh sub-agents: also add "use extended thinking (xhigh) only on Mechanisms #1, #2, #3 and counter-factual / threat-modeling steps; routine steps remain standard Opus."
