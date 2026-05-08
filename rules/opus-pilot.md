---
description: Opus-first ceiling-elevation playbook (auto-loaded; activates on trigger words)
tier: auto
---

# Opus Pilot — Mode Declaration

## Mode Overview

`opus-pilot`, `sonnet-pilot`, and `haiku-pilot` are coexisting execution modes — they do not conflict:

| Mode | How to activate | Default model | Quality target |
|------|-----------------|---------------|----------------|
| **Haiku mode** (haiku-pilot) | default / `/haiku-pilot` | Haiku 4.5 | Near-Opus quality, maximum cost efficiency |
| **Sonnet mode** (sonnet-pilot) | `/sonnet-pilot` or trigger words | Sonnet 4.6 | Opus-level quality, quality-first |
| **Opus mode** (this rule) | trigger word(s) below | Opus 4.7 | **Beyond vanilla Opus** — ceiling elevation via harness/context engineering |

**Conflict resolution**: whichever mode was most recently and explicitly activated wins. No explicit activation → haiku-pilot is the default.

## Trigger Phrases (auto-switch to Opus mode for this session)

- `opus`, `Opus`, `Opus mode`, `opus-pilot`
- Additional natural-language triggers: `Opus Pilot`, `ceiling-elevation`, `beyond Opus`

**Sub-mode modifiers** (append after a base trigger to layer on extra capabilities):

| Modifier | Behavior delta |
|----------|----------------|
| `Opus 1M`, `1M context`, `extended context` | Base + 1M context window + episodic memory consolidation + proactive compact at 70% |
| `Opus xhigh`, `xhigh thinking`, `extended thinking` | Base + extended-thinking (xhigh) on synthesis/architecture/counter-factual steps |
| `Opus 1M xhigh`, `Opus 1M + xhigh` | Combines both modifiers above |

## Behavior After Activation

1. Primary model for this session switches to **Opus 4.7** (via `/model` or `claude --model claude-opus-4-7`)
2. Overrides Haiku/Sonnet defaults; locks downward escalation (Opus is the ceiling — no auto-upgrade exists; only sub-agent **down-delegation** to Sonnet/Haiku is allowed for cost optimization)
3. Immediately load full playbook: `.claude/skills/opus-pilot/SKILL.md`
4. Execute Per-Session Pre-flight (Reverse-Advisor pledge, Decision-Log Externalization, Start-Simple-First pledge, Ambiguity Surfacing — all mandatory)
5. **Citation Anchor + Source-Verify** (required for wiki/ref citation tasks): every answer paragraph needs ≥ 3 structured anchors (`P0X §section` / `Step N` / `Anti-pattern #N` / `Mechanism #N` / `CAR.<C|A|R>`); every numeric / proper-noun / verbatim claim must pass `grep -i` against the cited source. Same hard gate as haiku-pilot.md §Pre-flight #5 — Opus is **not** exempt.

## Self-check Template (required for wiki/ref citation tasks — **single table, no expansion** — symmetric with haiku-pilot / sonnet-pilot)

| Check | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|-------|----|----|----|----|-----|------|
| Word count (`wc -w`, ≤ 800/question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor density (≥ 3 anchors/100w; per question via `wc -w`) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Source-verify (every cited number `grep -i` locatable) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Down-delegation eval (delegated/no-delegate + one-line reason) | ___ | ___ | ___ | ___ | ___ | — |

**Soft threshold**: per-question density ≥ 3 anchors/100w; any miss → **trigger retry** (re-run with numbered citations). **Hard threshold**: any cited number that fails `grep -i` in source → **must retry**; Opus is **not** allowed to ship fabrications under the assumption "Opus's reasoning is correct".
**Prohibited**: expanding into multiple H3 sections, appending prose commentary, rewriting question context. **Fill only the table above.** Symmetric with haiku-pilot.md / sonnet-pilot.md to prevent token-efficiency regression.

## Boundary with sonnet-pilot.md / haiku-pilot.md

- **Escalation gates do not apply upward**: Opus is the ceiling. Opus mode replaces "Haiku/Sonnet → Opus" gates with "Opus → sub-agent down-delegation" decisions.
- **Mandatory pre-flights differ**: Opus does NOT need Sonnet's Decision-Log enforcement (Opus does this naturally), but DOES need its own anti-patterns (Reverse-Advisor, Ambiguity Surfacing, Start-Simple-First — all Opus-specific failure modes).
- **Sub-agent dispatch shared**: all three modes use `subagent-strategy.md`. Opus mode adds: explicit down-delegation to Sonnet/Haiku for cost-optimal sub-tasks even when Opus is capable.

## Known Gotchas

- **Activating multiple modes at once**: entering `/haiku-pilot /sonnet-pilot /opus-pilot` in one message → the last trigger wins; do not interleave.
- **"Opus mode = always Opus"**: Opus mode does NOT mean every sub-agent is Opus. Down-delegation to Sonnet/Haiku for routine sub-tasks is encouraged (AgentOpt: weak planner + delegation > strong solo). Use Opus where Opus's reasoning depth pays off.
- **Does not persist across sessions**: new sessions revert to haiku-pilot default; re-activate Opus mode at the start of each session that needs it.
- **1M / xhigh availability is plan-dependent**: 1M context requires Max / Team / Enterprise (or pay-as-you-go API); xhigh extended thinking budget is enabled at session start, not per-prompt — confirm via `/model` and effort settings before relying on these modes.
