---
description: Sub-agent strategy + advisor mode + delegation rules (auto-loaded)
tier: auto
---

# Sub-Agent Strategy & Advisor Mode

## Delegation Decision (single criterion)

Default: handle in the main conversation. Trigger sub-agent delegation if **any** of these hits:

| Condition | Reason |
|-----------|--------|
| Reading **≥ 10 files** | Research-type task; context rot risk is high |
| Expected tool calls **> 20** | Excessive tool noise pollutes the main conversation |
| Decomposable into **≥ 3 independent subtasks** | Parallel fan-out — dispatch all in one message |
| Task type ∈ {research, security review, architecture decision} | Type-based trigger, regardless of count |

Otherwise → handle in main conversation; switch to delegation if conditions become true mid-execution.

Main agent's responsibilities: plan division of labor (`TodoWrite`), coordinate sub-agents, return only **summaries** (not raw output).

## Advisor Mode

- **IMPORTANT**: Use `advisor()` before architecture decisions, before implementing core logic, and before declaring "done". The advisor sees the full transcript — no need to restate context.
- Recommended pattern: orient first (cheap reads / greps), then advisor before any substantive write.

## Model Selection: Cognitive Step Count

By independent file count:
- 0–1 file → Haiku
- 2–9 files → Sonnet
- 10+ files → Sonnet (with `advisor()` for tricky cases) or Opus

> If the count is borderline (e.g. 6–9), bias higher; downgrade only after you've seen the model handle similar work.

---

## Sub-Agent Dispatch Table

> Agent types below are Claude Code built-ins — always available without any `.claude/agents/` setup. Add custom agents to override rows as your workspace grows.

| Scenario | Suggested agent type | Suggested model |
|----------|---------------------|-----------------|
| Research / > 10 files / locate symbols | `Explore` | Haiku 4.5 |
| 3+ independent subtasks | dispatch multiple in one message | — |
| Surgical code edit (single file ≤ 30 LoC, explicit path, no design decision) | `general-purpose` | Haiku 4.5 |
| Cross-module / > 30 LoC / requires design judgment | `general-purpose` | Sonnet 4.6 |
| Test writing (boundary cases + mocks) | `general-purpose` | Sonnet 4.6 |
| Quick pre-commit security check | `general-purpose` | Sonnet 4.6 |
| OWASP / proactive threat modeling | `general-purpose` | Sonnet 4.6 |
| Architecture decision / implementation plan | `Plan` | Opus 4.7 |
| Pre-commit multi-dimension review | `/deep-review` skill | mixed |

**To add custom agents**: create `.claude/agents/<name>/agent.md` and replace the `general-purpose` rows with your specialized agents for tighter scope and lower token cost.

---

## Tool Design Mental Model

> Before creating new agents or skills, audit existing ones (use `harness-eval` skill if installed) and check whether the work belongs to an existing agent or genuinely needs a new one.

## Known Gotchas

- **Over-summoning the advisor**: frequent Opus consultations get expensive; only use for architecture decisions / complex reviews / stuck-points.
- **"Cognitive step count" boundary fuzziness**: 6–9 vs 10+ is hard to judge; bias conservative (pick the larger model) or try and adjust.
- **Over-worry about downgrading**: Opus → Sonnet is sufficient for most tasks; Sonnet often avoids Opus's over-engineering side effect.
- **Subagent fork vs cache conflict**: `--fork-session` with a large parent context breaks cache; compact periodically to keep parent lean.
