# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.4.1] - 2026-05-08

### Fixed

5 corrections derived from the 2026-05-08 6-agent × 10-question pilot-vs-vanilla benchmark (`research/analysis/2026-05-08-pilot-vs-vanilla-6agent-10q-benchmark/` in the integrator repo, e.g. `zeuikli/cc-workspace`). Each correction targets a measurable gap observed when Sonnet+Pilot or Haiku+Pilot underperformed vanilla Opus 4.7 on hard-task subsets.

- **`rules/sonnet-pilot.md`** § Self-check Template — added **hard-cap word-count enforcement**: hard-task questions over 800w must rewrite (cannot append apology); word-count row must be filled after `wc -w` actually runs on the draft. Targets the A2 (Sonnet+Pilot) Q08–Q10 over-budget pattern (Q10 1115w / +39.4% over).
- **`rules/haiku-pilot.md`** § Self-check Template — symmetric hard-cap word-count enforcement. Targets A1 (Haiku+Pilot) Q10 actual 973w (+21.6% over budget) despite self-reporting within budget.
- **`rules/haiku-pilot.md`** + **`skills/haiku-pilot/SKILL.md`** § Pre-flight #4 — anchor count requirement strengthened: hard tasks now require **≥ 5 structured anchors per paragraph** (vs ≥ 3 for easy/medium); `(P0X §Y.Z)` format is the preferred form. Targets the Haiku-Pilot D2 anchor-density collapse from 7/100w (medium) to 1/100w (hard) observed in the benchmark.
- **`skills/sonnet-pilot/SKILL.md`** § Escalation Gate — added explicit trigger condition: "Q type tagged Architecture / Counter-factual / Synthesis (hard)" auto-escalates to Opus; prevents hard-task gate "no-hit" misclassification. Targets A2 Q10 misclassification (lost −5 to A3 on architecture-hard).
- **`skills/sonnet-pilot/SKILL.md`** § Per-Session Pre-flight — added new **#7 Source-Verify (Line-Level)** step mirroring haiku-pilot SKILL #5 but stricter: every numeric citation on hard tasks must include `(P0X §Y.Z:LineN)` (line-number granularity). Closes the 0.5-point D1 gap between A2 (D1=9.5) and A3 (D1=10.0) on the benchmark.

### Notes

- Patch release: no new SKILL files, no new triggers, no rule schema change. Existing v0.4.0 install paths (clone+symlink, copy, plugin install) continue to work without re-install.
- Verification methodology — see `research/analysis/2026-05-08-pilot-vs-vanilla-6agent-10q-benchmark/final-report-zh-TW.md` § 8 in the integrator repo for source-of-truth reasoning behind each correction.

## [0.4.0] - 2026-05-07

### Added

- **`rules/opus-pilot.md`** (new): mode declaration for Opus Pilot. Activates on triggers `opus` / `Opus` / `Opus mode` / `opus-pilot` (symmetric with haiku/sonnet 4-trigger pattern). Sub-mode modifiers: `Opus 1M` (1M context), `Opus xhigh` (extended thinking). Self-check template symmetric with haiku-pilot / sonnet-pilot to prevent token-efficiency regression.
- **`skills/opus-pilot/SKILL.md`** (new): ceiling-elevation playbook for Opus 4.7. Implements 5 mechanisms (Reverse-Advisor Loop / Parallel Hypotheses Synthesis / CAR Explicitness / Decision-Log Externalization / Meta-Harness Filesystem Loop) and 3 Opus-specific pre-flights (Ambiguity Surfacing / Start-Simple-First / Down-Delegation Eval). Task-type fast-path consistent with haiku/sonnet-pilot. **Hard rule**: Opus is the ceiling — no upward escalation; only sub-agent down-delegation to Sonnet/Haiku per `subagent-strategy.md`.
- **Citation grounding** for opus-pilot: 9 papers (P01 AgentFlow / arxiv 2604.20801, P02 Confucius / arxiv 2512.10398, P03 Meta-Harness / arxiv 2603.28052, P04 NLAH / arxiv 2603.25723, P05 COMPOSITE-STEM / arxiv 2604.09836, P06 Agent Harness Survey / preprints 202604.0428, P07 CAR / preprints 202603.1756, P08 OpenDev / arxiv 2603.05344, P09 Skill Issue blog / humanlayer.dev) + P10 AgentOpt as foundational orchestration insight (HotpotQA 31.71% solo → 74.27% orchestrated).
- **`README.md`** + **`INSTALL.md`** + **`examples/CLAUDE.md.template`**: synced to advertise the third pilot mode; install / uninstall paths updated.

### Notes

- `opus-pilot` is **not** the default. Most tasks should run on `haiku-pilot` (default) or `sonnet-pilot`. `opus-pilot` fires only on hard analytical tasks (architecture decisions, counter-factual reasoning, synthesis across multiple sources, threat modeling) where vanilla Opus is the baseline to beat.
- Verification methodology — see `research/analysis/2026-05-07-opus-pilot-skill-20q-benchmark/` in the integrator repo (e.g. `zeuikli/cc-workspace`).

## [0.3.1] - 2026-05-07

### Fixed

- **`rules/sonnet-pilot.md`**: Translated all rules to English (#15)
- **`skills/haiku-pilot/SKILL.md`** + **`skills/sonnet-pilot/SKILL.md`**: Removed false `/sonnet-pilot` and `/haiku-pilot` slash-trigger claims; trigger phrases are natural-language only, not slash commands (#16)
- **`examples/CLAUDE.md.template`**: Synced with v0.3.0 changelog and README; accurate trigger phrases and install instructions (#17)
- **`skills/haiku-pilot/SKILL.md`** § Compatibility: Removed non-existent skills (`/ultrathink`, `/deep-research`) from compatibility section (#18)

### Changed

- **`RELEASING.md`** + **`INSTALL.md`**: Codified v0.3.0 release lessons; synced trigger phrase documentation

## [0.3.0] - 2026-05-07

### Added

- **`rules/haiku-pilot.md`** + **`skills/haiku-pilot/SKILL.md`**: New Pre-flight #5 **Source-Verify gate** for citation tasks. Every cited number / model name / verbatim quote must be locatable via `grep -i` in the source file before completion. Self-check table gains a Source-verify row; soft threshold gains a hard threshold for fabricated numbers. Targets the 5 fabrication cases (Q06/Q12/Q13/Q17/Q18) caught in v0.2.1 benchmark; expected +20–30% Haiku→Opus gap closure.
- **`rules/haiku-pilot.md`** + **`rules/sonnet-pilot.md`** + **`skills/haiku-pilot/SKILL.md`** + **`skills/sonnet-pilot/SKILL.md`**: New **Task-type Fast-path** section. Easy fact-extraction tasks (≤ 100w answer, ≤ 5 facts, single source) skip the mandatory self-check and replace it with a single `fast-path: <reason>` line. Eliminates the net-negative cases observed in v0.2.1 benchmark (Q01 sonnet-pilot −4 vs vanilla; Q04 haiku-pilot −2 vs vanilla).
- **`rules/sonnet-pilot.md`** + **`skills/sonnet-pilot/SKILL.md`**: New Pre-flight #6 **Mid-Write Outline Verification** for architecture / counter-factual / synthesis tasks. At ~200 words drafted, pause and verify scope / projected total word count / mandatory-citation coverage. Targets the runaway pattern in benchmark Q17/Q18 (sonnet-pilot 750w with 2/3 required numbers missing, lost −6 to Opus).

### Changed

- **`skills/haiku-pilot/SKILL.md`**: Pre-flight § "Content-First Structure" renumbered from #5 to #6 to make room for Source-Verify Loop at #5.
- **`skills/haiku-pilot/SKILL.md`** § Verification: added Source-Verify (Pre-flight #5) compliance row.
- **`skills/haiku-pilot/SKILL.md`** § Known Gotchas: added "Number fabrication on hard tasks" entry pointing at Pre-flight #5 as the mitigation.

## [0.2.1] - 2026-05-06

### Added

- **`.claude-plugin/plugin.json`**: Added `triggers` field mapping each skill name to its valid trigger phrases, enabling plugin tooling to surface trigger discovery without reading individual SKILL.md files

## [0.2.0] - 2026-05-06

### Fixed

- `rules/haiku-pilot.md` and `rules/sonnet-pilot.md`: Added missing YAML frontmatter so rules load correctly via `@-import` (#13)
- README.md and INSTALL.md: Corrected markdown content errors (broken links, wrong section headers, stale text)
- **G-01** `rules/sonnet-pilot.md`: Anchor threshold changed from fixed count to density ≥ 3 per 100 characters
- **G-06** `skills/haiku-pilot/SKILL.md`: Added citation semantic drift warning to prevent hallucinated citations on paraphrase
- **`skills/haiku-pilot/SKILL.md`** and **`skills/sonnet-pilot/SKILL.md`**: Finalized trigger phrase list — `"haiku"`, `"Haiku"`, `"Haiku mode"`, `"haiku-pilot"` / `"sonnet"`, `"Sonnet"`, `"Sonnet mode"`, `"sonnet-pilot"`. Removed Chinese phrases and unrelated English phrases; added missing bare `"haiku"` / `"Haiku"` / `"sonnet"` / `"Sonnet"` that INSTALL.md documents as valid triggers

### Changed

- **SKILL trigger phrases** (`haiku-pilot`, `sonnet-pilot`): Simplified to natural-language forms that Claude Code recognises out of the box (#2)
- **Sub-agent names** (`rules/subagent-strategy.md`, `skills/haiku-pilot/SKILL.md`): Replaced fictional agent names with the real Claude Code built-in agent types (`Explore`, `code-reviewer`, `general-purpose`, etc.) (#5)
- **SKILL.md content** (`haiku-pilot`, `sonnet-pilot`): Removed sections unrelated to this repo; tightened scope to pilot-mode behaviour only (#7)
- **`examples/CLAUDE.md.template`**: Updated trigger phrases to match actual SKILL names (#8)
- **`plugin.json`**: Updated description, version, keywords, and fixed `author` object format (#10, #12)
- **README.md**: Full rewrite based on actual repo files and real use cases; added `/plugin install` one-liner and usage examples (#11, #12); repositioned thesis to quality floor protection framing (G-05a)
- **INSTALL.md**: Simplified "Verify installation" section (#4)
- **RELEASING.md**: Streamlined release checklist; removed obsolete multi-repo guidance (#5)
- **G-02** `skills/sonnet-pilot/SKILL.md`: Added forced per-question upgrade evaluation step
- **G-03** `skills/sonnet-pilot/SKILL.md`: Added wiki anti-pattern fidelity check
- **G-04** `skills/sonnet-pilot/SKILL.md`: Added task-type fast-path table for routing decisions
- **G-05b** `skills/sonnet-pilot/SKILL.md`: Rewrote thesis section with empirical benchmark scores (283/283/286)
- **G-08** `skills/haiku-pilot/SKILL.md`: Added ASCII escalation decision tree for clearer upgrade path

## [0.1.1] - 2026-05-05

### Changed

- Migrated to standalone repo `zeuikli/claude-pilot-suite` as the canonical source of truth.
- Updated all installation URLs to canonical `zeuikli/claude-pilot-suite`.
- Added `repository` field to `.claude-plugin/plugin.json` for plugin tooling discovery.
- Added `RELEASING.md` documenting the release flow.

### Fixed

- Plugin manifest `homepage` now points to standalone repo root.

## [0.1.0] - 2026-05-04

### Added

- Initial release of `haiku-pilot` SKILL with 5-step pre-flight checklist
- Initial release of `sonnet-pilot` SKILL with deep context engineering + Opus escalation gates
- 6 auto-load baseline rules: `core.md`, `subagent-strategy.md`, `context-management.md`, `output-discipline.md`, `haiku-pilot.md`, `sonnet-pilot.md`
- 4 installation methods: plugin install (SKILLs), clone+symlink, copy, git subtree
- Plugin manifest (`.claude-plugin/plugin.json`) for Claude Code's `/plugin install` mechanism

### Notes

- Derived from 4-iteration internal benchmark (5/3 baseline → v1 mode-declaration → v2 asymmetric → v3 symmetric)
- Portable starter pack — requires customization of `rules/core.md` Production Safety Red Line and `rules/subagent-strategy.md` Sub-Agent Dispatch Table before production use
- Hybrid bundle: SKILLs load via Claude Code plugin mechanism; rules load via `@-import` (rules not yet plugin-supported)
