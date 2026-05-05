---
description: Context window monitoring + 1M context GA + Prompt Caching (advanced session management see session-management.md)
tier: auto
---

# Context Window Management

## Theoretical Anchor

**Context Engineering** (Karpathy 2025-06-25) = "filling the context window with just the right information for the next step". It cuts both ways: too little / wrong format → underperformance; too much / irrelevant → rising cost and degraded quality. Archived: `research/tweets/2025-06-25-@karpathy-607626.md`.

## Monitoring

- `/usage` — view session token/cost in real time.
- **Three-tier compact triggers** (priority order: behavioral signal > numeric threshold > timer):
  1. **Behavioral signal**: model asks "please provide more context" / "what are you trying to do?" — lost-model symptoms → immediately `/rewind` or `/compact`
  2. **Numeric threshold**: general tasks **70%**; beginners ~**60%**; long agentic runs **30–35%** proactive compact
  3. **Timer**: complex agentic — proactive compact every **300–400K tokens** (for 200K-context users: 30–35% ≈ 60–70K tokens)
- `/compact <hint>` to continue the task; `/clear` to switch tasks.
- **Core rationale**: compact refocuses attention (*Lost in the Middle* U-curve) — it is **not** a cost-saving measure. Correction cost after drift ≈ 2–3× a fresh start.

## Compression-Level Decision Table

Adjust action by context %: 0–40% unrestricted, 40–70% stay focused, 70–85% proactive compact, 85–95% stop new tasks, 95%+ immediate clear. Full table and instructions: `.claude/refs/context-monitor-table.md`.

## Auto Memory (cross-session persistence)

- Set `"autoMemoryEnabled": true` in `.claude/settings.json`
- Claude accumulates memory at `~/.claude/projects/<project>/memory/`
- `/memory` to view or edit; `/clear` and `/compact` do NOT affect Auto Memory

> 1M context is now GA: **Max / Team / Enterprise** subscriptions include Opus 1M (Sonnet 1M requires extra usage fees); **Pro** plan requires extra payment for both; **API / pay-as-you-go** at standard rates (no surcharge above 200K). Compaction pressure is low.

> **Context rot threshold** (community observation, not officially confirmed): quality degradation begins around **300–400K tokens**, highly task-dependent (sources: Thariq @trq212, 2026-04-16; Wisely Chen AI, 2026-04-24). Use **30–35% proactive compact** as the operating baseline for complex agentic tasks; 70% reminder threshold remains for general tasks.

> **Advanced compact / rewind strategies**: load on demand from `.claude/refs/session-management.md`.

## Prompt Caching Architecture Rules

Full rules in `.claude/refs/prompt-caching-rules.md` (load on demand; trigger words: `cache`, `prompt caching`). Core principle: Static first, dynamic last; tool lists and model must not change mid-session.

## Known Gotchas

- **Compact does not always fix the problem**: if the model has already drifted when summarizing, the summary will be poor too. On "lost model" signals, `/rewind` immediately — don't wait for the context % threshold.
- **Context rot is not directly correlated with token count**: > 3,500 tokens does not imply rot; focus on context engineering (filling with "just the right information"), not on compressing line counts.
- **`@-import` does not save tokens**: `@` directives expand at load time, not lazily. Only put rules you genuinely need in auto-loaded files.
- **The 70% warning is easy to ignore**: treat 70% as the signal to start collecting summary material, not as a yellow light you can dismiss.
