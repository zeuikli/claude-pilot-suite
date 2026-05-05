---
description: Sonnet Pilot mode declaration + escalation gates (auto-loaded)
tier: auto
---

# Sonnet Pilot — Mode Declaration

## Mode Overview

`sonnet-pilot` and `haiku-pilot` are coexisting execution modes — they do not conflict:

| Mode | How to activate | Default model | Quality target |
|------|-----------------|---------------|----------------|
| **Haiku mode** (haiku-pilot) | default / `/haiku-pilot` | Haiku 4.5 | Near-Opus quality, maximum cost efficiency |
| **Sonnet mode** (this rule) | `/sonnet-pilot` or trigger words | Sonnet 4.6 | Opus-level quality, quality-first |

**Conflict resolution**: whichever mode was most recently and explicitly activated wins. No explicit activation → haiku-pilot is the default.

## Trigger Words (auto-switch to Sonnet mode this session)

- `/sonnet-pilot`, `Sonnet mode`, `full Sonnet`
- `Sonnet matches Opus`, `Sonnet Pilot`, `sonnet-pilot`, `quality-first`, `near Opus`

## Behavior After Activation

1. Primary model for this session switches to **Sonnet 4.6**
2. Overrides the Haiku default from `haiku-pilot.md`
3. Immediately load full playbook: `.claude/skills/sonnet-pilot/SKILL.md`
4. Execute Per-Session Pre-flight (Decision-Log, Reasoning Chain Before Code, Self-Review Loop are all mandatory)
5. **Citation Anchor** (required for wiki/ref citation tasks): every answer paragraph needs ≥ 3 structured anchors (`Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Concrete Numbers` / `Decision Tree` / `Gotcha [Source]`); vague phrasing like "the wiki mentions" or "as the wiki shows" does **not** count as an anchor. Same baseline as haiku-pilot.md §Pre-flight #4.

## Self-check Template (required for wiki/ref citation tasks — **single table, no expansion** — symmetric with haiku-pilot)

| Check | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|-------|----|----|----|----|-----|------|
| Word count (`wc -w`, ≤ 800/question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor count (≥ 5 per question, higher Sonnet baseline) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Sonnet→Opus gate (hit/no-hit + one-line reason) | ___ | ___ | ___ | ___ | ___ | — |

**Soft threshold**: total anchors ≥ 25; < 20 → **trigger retry** (re-run the question, add numbered citations).
**Prohibited**: expanding into multiple H3 sections, appending prose commentary, rewriting question context. **Fill only the table above.** Symmetric with haiku-pilot.md to prevent token-efficiency regression.

## Boundary with haiku-pilot.md

- **Escalation gates**: haiku-pilot's Haiku→Sonnet gate conditions do **not apply** in Sonnet mode (already on Sonnet).
- **Opus escalation**: Sonnet mode has its own Sonnet→Opus gate (see SKILL.md §Escalation Gates).
- **Sub-agent dispatch**: both modes share `subagent-strategy.md` — no duplication here.

## Known Gotchas

- **Do not activate both modes simultaneously**: entering `/haiku-pilot /sonnet-pilot` in one message → the last trigger wins.
- **Sonnet mode is not "always use Opus"**: Opus only appears when a quantified gate fires; the goal of Sonnet mode is to reach Opus-level quality with Sonnet, not to replace Opus.
- **Does not persist across sessions**: new sessions revert to haiku-pilot default; re-activate quality-first mode at the start of each session that needs it.
