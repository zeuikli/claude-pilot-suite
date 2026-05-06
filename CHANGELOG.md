# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [0.2.0] - 2026-05-06

### Fixed

- `rules/haiku-pilot.md` and `rules/sonnet-pilot.md`: Added missing YAML frontmatter so rules load correctly via `@-import` (#13)
- README.md and INSTALL.md: Corrected markdown content errors (broken links, wrong section headers, stale text)
- **G-01** `rules/sonnet-pilot.md`: Anchor threshold changed from fixed count to density ≥ 3 per 100 characters
- **G-06** `skills/haiku-pilot/SKILL.md`: Added citation semantic drift warning to prevent hallucinated citations on paraphrase

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
