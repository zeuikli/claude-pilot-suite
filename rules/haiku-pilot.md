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

- **Haiku mode** (this rule): default, cost-first. Trigger words: `Haiku mode`, `cost-efficient run`, `/haiku-pilot`
- **Sonnet mode**: quality-first, overrides this default. Trigger words: `Sonnet mode`, `quality-first`, `/sonnet-pilot`
- Both modes coexist; whichever was most recently and explicitly activated wins. Sonnet playbook: `.claude/skills/sonnet-pilot/SKILL.md`

## Escalation Gates (quantified — not gut feeling)

| Trigger condition | Escalate to |
|-------------------|-------------|
| Same problem fails ≥ 3 times | Sonnet 4.6 |
| Task spans ≥ 10 independent files OR cross-module design | Sonnet 4.6 |
| Trigger words: "architecture decision" / "design review" / "security review" / "threat modeling" | Opus 4.7 / `advisor()` |
| Code-gen returns > 300 LoC unprompted | Stop; re-dispatch to Sonnet `implementer` |
| User explicitly says "use Opus" / "use Sonnet" | Comply |

**Outside the gates** → stay on Haiku, delegate per `subagent-strategy.md`.

## Per-Task Pre-flight (≤ 4 checks)

1. **Reference Pattern**: can you point to an existing file rather than describing in prose? (`src/auth/login.ts` > "something like login")
2. **Think-Before-Coding**: restate understanding in 1–2 sentences + list key assumptions before implementing (compensates for Haiku's tendency to skip this step)
3. **Diff-Review pledge**: run `git diff --stat` before declaring done and explain each change (catches unintended edits)
4. **Citation Anchor** (required for wiki/ref citation tasks): every answer paragraph needs ≥ 3 structured anchors (`Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Concrete Numbers` / `Decision Tree` / `Gotcha [Source]`); vague phrasing like "the wiki mentions" or "as the wiki shows" does **not** count as an anchor. Source: 2026-05-04 benchmark — Haiku 8 vs Sonnet 45 anchor lines, a 5.6× gap.

## Self-check Template (required for wiki/ref citation tasks — **single table, no expansion**)

| Check | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|-------|----|----|----|----|-----|------|
| Word count (`wc -w`, ≤ 800/question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor count (≥ 3 per question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Escalation gate (hit/no-hit + one-line reason) | ___ | ___ | ___ | ___ | ___ | — |

**Soft threshold**: total anchors ≥ 15; < 12 → **trigger retry** (re-run the question, add numbered citations).
**Prohibited**: expanding into multiple H3 sections, appending prose commentary, rewriting question context. **Fill only the table above.**
Source: 2026-05-04 v3 Haiku token-efficiency regression (self-check expanded into 5 H3 blocks, 2.24× word-count inflation).

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
