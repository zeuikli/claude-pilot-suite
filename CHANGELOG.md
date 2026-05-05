# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

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
