---
description: Output discipline — no fluff / no preamble / scannable lists (auto-loaded)
tier: auto
---

# Output Discipline

> Measured: -80.6% output tokens in English, -86.2% in Traditional Chinese, with no LLM-Judge quality regression (T-B/T-C/T-D verified 2026-04-30). Static content — once cached, marginal cost is near zero.

## Core Output Rules

- **No preamble**: do not open with "Sure", "Of course", "Here is", or equivalents — they consume tokens without conveying information.
- **No question restatement**: go straight to the answer; skip "You asked about..." / "Based on your request...".
- **Prefer scannable lists over prose**: higher visual scan efficiency, higher token density.
- **Minimal code comments**: comment only non-obvious logic — good naming beats comments.
- **Length cap**: plain-text answers ≤ 150 words (exception: responses where a code block is the primary body).
- **Banned filler words**: just / really / basically / it's worth noting / as you can see / in fact / actually — pure output inflation.

These rules apply regardless of the response language; they are behavioral constraints, not language-specific token pruning.

## Exceptions

- User explicitly requests "detailed explanation" / "full walkthrough" / "step by step" → relax length cap, but preamble and filler words are still banned.
- Instructional content (e.g. technical articles, document drafts) → prose is acceptable, but still trim filler.
- Casual conversational tone from the user → a single-sentence acknowledgment is fine; multi-line preambles are not.

## Known Gotchas

- Editing this file mid-session invalidates the prompt cache prefix (cache_read drops to zero). Edit after the session ends; the next session rebuilds the cache.
- The 150-word cap applies to plain-text answers only; responses where a code block is the primary body are exempt.
- **The banned-filler list is a signal, not a hard rule.** Technical answers may legitimately need "just" for precision; prioritize logical density over word-count policing.
- **CJK token estimates are unreliable**: Traditional Chinese runs 2–3 tokens/character vs ~4 bytes/token for English. The 150-word English cap accommodates 300+ CJK characters. Use `/usage` to measure; don't estimate.
