---
description: Sonnet Pilot mode declaration + quality gates (auto-loaded when triggered)
tier: auto
---

# Sonnet Pilot — Mode Declaration

## Mode Overview

`sonnet-pilot` and `haiku-pilot` are coexisting execution modes; they do not conflict:

| Mode | Activation | Default Model | Quality Target |
|------|------------|---------------|----------------|
| **Haiku mode** (haiku-pilot) | default (auto-loaded) | Haiku 4.5 | Approach Opus, maximum cost savings |
| **Sonnet mode** (this SKILL) | trigger phrases (see below) | Sonnet 4.6 | Match Opus, quality-first |

**Conflict resolution**: whichever mode was most recently and explicitly activated wins. With no explicit activation → haiku-pilot is the default.

> **Note**: activation is via natural-language trigger phrases that the SKILL auto-loads on; there is **no** `/sonnet-pilot` or `/haiku-pilot` slash command.

## Trigger Phrases (auto-switch to Sonnet mode for this session)

- `sonnet`, `Sonnet`, `Sonnet mode`, `sonnet-pilot`
- Additional natural-language triggers: `full Sonnet`, `Sonnet matches Opus`, `Sonnet Pilot`, `quality-first`, `approach Opus`

## Behavior After Activation

1. Primary model for this session switches to **Sonnet 4.6**
2. Overrides the Haiku default in `haiku-pilot.md`
3. Immediately load the full playbook: `.claude/skills/sonnet-pilot/SKILL.md`
4. Run Per-Session Pre-flight (Decision-Log, Reasoning Chain Before Code, Self-Review Loop are mandatory)
5. **Citation Anchor** (required for wiki/ref citation tasks): every answer paragraph needs ≥ 3 structured anchors (`Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Concrete Numbers` / `Decision Tree` / `Gotcha [Source]`); vague phrasing like "the wiki mentions" or "the wiki shows" does **not** count as an anchor. Same baseline as `haiku-pilot.md` §Pre-flight #4.
6. **Mid-write Checkpoint** (required for architecture decisions / counter-factuals / synthesis tasks): pause around the 200-word mark and answer three questions:
    (i) Still within the question's scope? (Not drifting into adjacent topics)
    (ii) Estimated final word count ≤ 800? (Project from `current word count × estimated remaining sections`; > 800 → trim outline)
    (iii) Of the N required numbers / required sub-questions listed in the prompt, how many are covered so far? (List and tick off)
   Any failure → reset the outline before continuing; do not push through to the end. **Empirical basis**: v0.2.1 benchmark Q18 — sonnet-pilot wrote 750 words but still missed 2 of 3 required numbers (−28.64%, #33→#5), losing to Opus by −6 points. The checkpoint catches the missing citations at 200 words and lets you shrink the outline in time.

## Self-check Template (required for wiki citation tasks — **single table, no expansion**, symmetric to haiku-pilot)

| Check | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|-------|----|----|----|----|----|------|
| Word count (`wc -w`, ≤ 800/question) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor density (≥ 3 anchors/100w; per-question word count via `wc -w`) | ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Sonnet→Opus gate (hit/no-hit + one-line reason) | ___ | ___ | ___ | ___ | ___ | — |

**Soft threshold**: per-question density ≥ 3 anchors/100w; any question below threshold → **trigger retry** (re-run the question, add numbered citations).
**Prohibited**: expanding into multiple H3 sections, appending prose commentary, rewriting question context. **Fill only the table above.** Symmetric to `haiku-pilot.md` to avoid token-efficiency regression.

## Task-type Fast-path (skip self-check when overhead > value)

| Task type | Pre-flight self-check | Reasoning chain |
|-----------|:---------------------:|:---------------:|
| **Easy fact-extraction** (≤ 100w answer; single source; pure recall / definition) | **skip** (replace with a single line `fast-path: <reason>`) | inline 1-sentence note |
| Comparison / Application / Critique / Design (medium) | **mandatory** (full single-table) | full chain |
| Architecture / Counter-factual / Synthesis (hard) | **mandatory** + Mid-write Checkpoint (see §Behavior After Activation #6) | full chain |

**Trigger criteria** (any one → easy fast-path):
- Question asks "list / write out / define" with ≤ 5 facts requested
- Expected answer has no tables, decision trees, or cross-section comparison
- All needed source evidence is visible within the first 2 Reads

When fast-path applies, the self-check collapses to a single line:
```
fast-path: <one-sentence reason>
```

**Empirical basis**: 2026-05-06 v0.2.1 benchmark — Q01 sonnet-pilot lost −4 to vanilla on a simple 6-component recall task (self-check overhead exceeded value). Fast-path eliminates these net-negative cases.

## Boundary with haiku-pilot.md

- **Escalation gates**: the Haiku→Sonnet escalation conditions in haiku-pilot **do not apply** in Sonnet mode (already on Sonnet).
- **Opus escalation**: Sonnet mode has its own Sonnet→Opus gate (see `SKILL.md` §Escalation Gates).
- **Sub-agent dispatch**: both modes share `subagent-strategy.md`; do not duplicate.

## Known Gotchas

- **Don't activate both modes in the same turn**: if both `Haiku mode` and `Sonnet mode` triggers appear in one message, the last one wins.
- **Sonnet mode is not "always use Opus"**: Opus only appears when a quantified gate fires; the goal of Sonnet mode is to reach Opus-level quality with Sonnet, not to replace Opus.
- **Does not carry over after the session ends**: a new session reverts to the haiku-pilot default; every quality-first session must be re-activated.
