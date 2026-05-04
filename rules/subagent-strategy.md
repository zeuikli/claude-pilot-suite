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

## Sub-Agent Dispatch Table (TEMPLATE — customize for your workspace)

> **This is a starter template.** Replace agent names with the agents that exist in your `.claude/agents/` directory. The model column reflects our defaults; adjust based on your cost/quality preferences.

| Scenario | Suggested agent type | Suggested model |
|----------|---------------------|-----------------|
| Research / > 10 files | `researcher` / `architecture-explorer` | Haiku 4.5 |
| 3+ independent subtasks | dispatch multiple in one message | — |
| Surgical code edit (single file ≤ 30 LoC, explicit path, no design decision) | `haiku-implementer` | Haiku 4.5 |
| Cross-module / > 30 LoC / requires design judgment | `implementer` | Sonnet 4.6 |
| Test writing (boundary cases + mocks) | `test-writer` | Sonnet 4.6 |
| Quick pre-commit security check | `security-reviewer` | Sonnet 4.6 |
| OWASP / proactive threat modeling | `security-auditor` | Sonnet 4.6 |
| Architecture decision / tech selection | `reviewer` | Opus 4.7 |
| Pre-commit multi-dimension review | `/deep-review` skill | mixed |

**How to customize**:

1. List your actual agents: `ls .claude/agents/`
2. For each row above, replace the suggested agent name with one of yours (or note "no equivalent" and consider building one)
3. Adjust the model column if your team prefers a different cost/quality balance
4. Delete rows that don't apply to your work (e.g. no security focus → drop security rows)

If you don't have specialized agents yet, the `general-purpose` agent (always available) accepts a `model` override and works as a fallback for all rows.

---

## Tool Design Mental Model

> Before creating new agents or skills, audit existing ones (use `harness-eval` skill if installed) and check whether the work belongs to an existing agent or genuinely needs a new one.

## Known Gotchas

- **Over-summoning the advisor**: frequent Opus consultations get expensive; only use for architecture decisions / complex reviews / stuck-points.
- **"Cognitive step count" boundary fuzziness**: 6–9 vs 10+ is hard to judge; bias conservative (pick the larger model) or try and adjust.
- **Over-worry about downgrading**: Opus → Sonnet is sufficient for most tasks; Sonnet often avoids Opus's over-engineering side effect.
- **Subagent fork vs cache conflict**: `--fork-session` with a large parent context breaks cache; compact periodically to keep parent lean.
