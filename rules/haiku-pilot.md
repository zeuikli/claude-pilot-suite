---
description: Haiku Pilot mode declaration + escalation gates (auto-loaded)
tier: auto
---

# Haiku Pilot — Default Haiku, Quantified Escalation

> Source: AgentOpt (arxiv 2604.06296, 2026-04) + Augment Code AGENTS.md eval (2026-04-22).
> Full playbook: `.claude/skills/haiku-pilot/SKILL.md` (trigger words: "Haiku mode", "full Haiku", "escalation gates", "Haiku Pilot").

## Thesis

**Weak planner + smart delegation > strong planner doing everything.** AgentOpt empirical result: Opus solo on HotpotQA 31.71%; Ministral planner + Opus solver 74.27%.
→ Parent session defaults to Haiku; aggressively delegate to sub-agents; **escalate only when a quantified gate fires**.

## Mode Switching

- **Haiku mode** (this rule): default, cost-first. Trigger phrases: `haiku`, `Haiku`, `Haiku mode`, `haiku-pilot`, `cost-efficient run`
- **Sonnet mode**: quality-first, overrides this default. Trigger phrases: `sonnet`, `Sonnet`, `Sonnet mode`, `sonnet-pilot`, `quality-first`
- Both modes coexist; whichever was most recently and explicitly activated wins. Sonnet playbook: `.claude/skills/sonnet-pilot/SKILL.md`
- Activation is via natural-language phrases the SKILL auto-loads on; there is **no** `/sonnet-pilot` or `/haiku-pilot` slash command.

## Escalation Gates (quantified — not gut feeling)

| Trigger condition | Escalate to |
|-------------------|-------------|
| Same problem fails ≥ 3 times | Sonnet 4.6 |
| Task spans ≥ 10 independent files OR cross-module design | Sonnet 4.6 |
| Trigger words: "architecture decision" / "design review" / "security review" / "threat modeling" | Opus 4.7 / `advisor()` |
| Code-gen returns > 300 LoC unprompted | Stop; re-dispatch to Sonnet `implementer` |
| User explicitly says "use Opus" / "use Sonnet" | Comply |

**Outside the gates** → stay on Haiku, delegate per `subagent-strategy.md`.

## Per-Task Pre-flight (≤ 5 checks)

1. **Reference Pattern**: can you point to an existing file rather than describing in prose? (`src/auth/login.ts` > "something like login")
2. **Think-Before-Coding**: restate understanding in 1–2 sentences + list key assumptions before implementing (compensates for Haiku's tendency to skip this step)
3. **Diff-Review pledge**: run `git diff --stat` before declaring done and explain each change (catches unintended edits)
4. **Citation Anchor** (required for wiki/ref citation tasks): every answer paragraph needs **≥ 3 anchors for easy/medium tasks; ≥ 5 anchors for hard tasks** (architecture / counter-factual / synthesis). Format must be `(P0X §Y.Z)` for paper-anchored citations (e.g. `(P03 §4.2)`), or one of `Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Gotcha [Source]` for non-paper sources. Vague phrasing like "the wiki mentions" or "as the wiki shows" does **not** count. Source: 2026-05-04 benchmark — Haiku 8 vs Sonnet 45 anchor lines (5.6× gap); 2026-05-08 6-agent 10Q benchmark — Haiku-Pilot hard-question anchor density collapsed from 7/100w to 1/100w, primary D2 gap vs vanilla Opus.
5. **Source-Verify** (required for citation tasks): every cited number / model name / verbatim quote must be locatable via `grep -i` in the cited source file. Compounded numbers ("5.6×") must state the source-verifiable atomic numbers in parentheses. Anti-patterns to reject before submitting: cross-paper number attribution without re-cite; non-existent model versions ("Claude 3.5 Opus"); "approximately X" without paper anchor; vague targets ("失敗率 ↓ 60–80%") not paper-grounded. Source: 2026-05-06 v0.2.1 benchmark — 5 fabrication cases in haiku-pilot responses (Q06, Q12, Q13, Q17, Q18) capped Haiku→Opus gap closure at 45%.

## Self-check Template (required for wiki/ref citation tasks — **single table, no expansion**)

| Check | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|-------|----|----|----|----|-----|------|
| Word count (`wc -w`, ≤ 800/question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor count (≥ 3 per question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Source-verify (every cited number `grep -i` locatable) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Escalation gate (hit/no-hit + one-line reason) | ___ | ___ | ___ | ___ | ___ | — |

**Soft threshold**: total anchors ≥ 15; < 12 → **trigger retry** (re-run the question, add numbered citations). **Hard threshold**: any cited number that fails `grep -i` in source → **must retry** (cannot ship with fabricated numbers).
**Hard cap (word count)**: hard-task questions (architecture / counter-factual / synthesis) with `wc -w` > 800 → **must rewrite to fit**, cannot append apology line. Self-check word-count row must be filled **after** running `wc -w` on the actual draft, not estimated. Source: 2026-05-08 6-agent 10Q benchmark — A1 Q10 self-reported within budget but actual 973w (+21.6% over); A2 Q10 1115w (+39.4%). Eyeball estimation without `wc -w` is the failure mode.
**Prohibited**: expanding into multiple H3 sections, appending prose commentary, rewriting question context. **Fill only the table above.**
Source: 2026-05-04 v3 Haiku token-efficiency regression (self-check expanded into 5 H3 blocks, 2.24× word-count inflation).

## Task-type Fast-path (skip mandatory self-check when overhead > value)

| Task signature | Pre-flight self-check | Reasoning chain |
|----------------|:---------------------:|:---------------:|
| **Easy fact-extraction** (≤ 100w answer; single source; numeric / definitional recall) | **skip** (1-line `fast-path: <reason>` instead) | inline 1-sentence note |
| Comparison / Application / Critique / Design (medium) | **mandatory** (full single-table) | full chain |
| Architecture / Counter-factual / Synthesis (hard) | **mandatory** + Source-Verify Loop (Pre-flight #5) | full chain |

**Trigger criteria** (any one → easy fast-path):
- Question asks "list / write / define" with ≤ 5 facts requested
- Expected answer has no tables, decision trees, or cross-section comparison
- All needed source evidence visible within first 2 Reads

When fast-path applies, replace the 4-row self-check table with a single line:
```
fast-path: <one-sentence reason>
```

**Empirical basis**: 2026-05-06 v0.2.1 benchmark — Q01 sonnet-pilot lost −4 to vanilla on simple 6-component recall (self-check overhead exceeded value); Q04 haiku-pilot lost −2 to vanilla on 4-number recall. Fast-path eliminates these net-negative cases.

## P+S (Prohibition + Solution)

| Don't | Do |
|-------|-----|
| Instinctively escalate to Opus for "complex" tasks | Run Haiku + pre-flight first; escalate only when a gate fires |
| Switch models after a single Haiku failure | Diagnose first — context issue (`/compact`) or true model ceiling (escalate at 3 failures) |
| Assume "Haiku is always worse" | SWE-bench Verified: Haiku 4.5 = 73.3%, only 5pp behind Sonnet, at 1/3 the cost |

## Known Gotchas (Haiku 4.5 specific)

- **Skipping assumption disclosure**: Haiku tends to jump straight to code; the rule in `core.md` §Think-Before-Coding enforces this step.
- **Verbose comments**: Haiku writes 1.5–2× more comments than Sonnet; `output-discipline.md` default is no comments unless non-obvious.
- **Missing regression check**: Haiku does not automatically ask "which of the last N commits might have caused this?" when debugging; use trigger words "regression" or "recent changes" to prompt it explicitly.
- **Citation semantic drift**: Haiku tends toward vague phrasing ("the wiki mentions", "the wiki example is clear") rather than structured anchors in citation tasks; 2026-05-04 benchmark measured citation density at 0.75/100w (vs Sonnet 3.43). Pre-flight #4 enforces anchor structure.

> Full per-task router, Haiku vs Sonnet comparison, and implementation checklist: load the `haiku-pilot` SKILL.
