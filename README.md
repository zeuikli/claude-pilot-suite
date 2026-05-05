# Claude Pilot Suite

> **Version**: 0.1.0 (initial portable release, 2026-05-04)
> **License**: MIT
> **Validated on**: Claude Sonnet 4.6 + Haiku 4.5 + Opus 4.7 (4-iteration benchmark, n=10 task corpus)

A portable, opinionated execution playbook for Claude Code that gets:

- **Haiku 4.5 quality close to Opus 4.7** (Haiku Pilot mode) — with 70–85% cost savings
- **Sonnet 4.6 quality on par with Opus 4.7** (Sonnet Pilot mode) — with 55–65% cost savings

Built on the empirical finding that **prompt structure + sub-agent delegation often matters more than raw model capability**. Documented in two arXiv-backed studies:

> AgentOpt (arxiv 2604.06296): On HotpotQA, Opus alone = 31.71%; Ministral planner + Opus solver = 74.27%.
> Augment Code AGENTS.md eval (2026-04): Best-quality docs ≈ one model tier upgrade. Bad docs are worse than none.

---

## What's in this suite

```
claude-pilot-suite/
├── .claude-plugin/
│   └── plugin.json                  Plugin manifest (loadable via /plugin install)
├── CHANGELOG.md
├── rules/                          # Auto-loaded baseline discipline
│   ├── core.md                       Language / Git / surgical changes / quality
│   ├── subagent-strategy.md          Sub-agent delegation criteria + advisor mode
│   ├── context-management.md         Context window monitoring + compact strategy
│   ├── output-discipline.md          Plain-text formatting rules (no fluff)
│   ├── haiku-pilot.md                Mode declaration + escalation gates (Haiku→Sonnet/Opus)
│   └── sonnet-pilot.md               Mode declaration + escalation gates (Sonnet→Opus)
├── skills/                         # On-demand playbooks (loaded only when triggered)
│   ├── haiku-pilot/SKILL.md          Default-Haiku execution + 5-step pre-flight
│   └── sonnet-pilot/SKILL.md         Quality-first Sonnet execution + 5-step pre-flight
├── examples/
│   └── CLAUDE.md.template            Sample CLAUDE.md to integrate the suite
├── README.md
├── INSTALL.md
└── LICENSE
```

---

## Two modes, one suite

### Haiku Pilot (cost-first, default)

Trigger: `/haiku-pilot` or "Haiku mode" / "全力 Haiku" / "save tokens with quality"

**Philosophy**: weak planner + strong delegation > strong planner alone. Default to Haiku, dispatch sub-agents aggressively, escalate only on quantitative gate triggers.

**Escalation gates**:
- Same problem failed ≥ 3 times → Sonnet
- Task touches ≥ 10 files → Sonnet
- Architecture / design / security keywords → Opus or `advisor()`
- Code-gen > 300 LoC unrequested → halt, redispatch to Sonnet `implementer`

**Expected savings**: 70–85% vs full-Opus baseline (assuming 80/15/5 task distribution).

### Sonnet Pilot (quality-first)

Trigger: `/sonnet-pilot` or "Sonnet mode" / "品質優先" / "approach Opus quality"

**Philosophy**: deep context engineering + Opus-on-demand via `advisor()` ≈ full Opus quality.

**Pre-flight enforcements** (Sonnet's high confidence biases toward skipping these):
1. Reference Pattern (point to a file, not abstract)
2. Decision-Log (every non-obvious choice → `Choice: ... / Rejected: ... — Reason: ...`)
3. Reasoning Chain Before Code
4. Self-Review Loop (`git diff --stat` + per-file explanation)
5. Intermediate Checkpoint (pause and verify per step)

**Escalation gate to Opus**: stricter than haiku-pilot's, because Opus is 5× the cost.

**Expected savings**: 55–65% vs full-Opus baseline.

---

## Why these specifically (vs raw model selection)

The suite is the synthesis of 4 benchmark iterations measuring how prompt structure affects answer quality across model tiers. Key findings:

| Finding | Implication |
|---------|-------------|
| Sub-agent delegation > model upgrade for multi-step tasks | Default to Haiku + delegate, not jump to Opus |
| Citation density correlates with reviewer-verifiable quality | Pre-flight #4 Citation Anchor enforcement |
| Anti-expansion self-check tables prevent 2.24× word inflation | Self-Review Loop uses single-table format |
| Sonnet's confidence bias = skip self-review unless externally enforced | Pre-flight #4 is mandatory in Sonnet Pilot |
| Haiku's tendency to skip assumption disclosure | Pre-flight #2 enforces it |

---

## Install

This bundle is **hybrid**: SKILLs load via Claude Code's plugin mechanism (`.claude-plugin/plugin.json`), while rules load via `@-import` from your CLAUDE.md (rules are not yet plugin-supported).

See `INSTALL.md` for the four installation methods (plugin install / clone+symlink / copy / git subtree).

Quick version:

```bash
# Clone next to your project, then symlink
git clone https://github.com/zeuikli/claude-pilot-suite.git
ln -s "$(pwd)/claude-pilot-suite/rules" ~/your-project/.claude/rules-pilot
ln -s "$(pwd)/claude-pilot-suite/skills/haiku-pilot" ~/your-project/.claude/skills/haiku-pilot
ln -s "$(pwd)/claude-pilot-suite/skills/sonnet-pilot" ~/your-project/.claude/skills/sonnet-pilot

# Then update your project's CLAUDE.md to @-import the rules:
# @.claude/rules-pilot/core.md
# @.claude/rules-pilot/subagent-strategy.md
# @.claude/rules-pilot/context-management.md
# @.claude/rules-pilot/output-discipline.md
# @.claude/rules-pilot/haiku-pilot.md
# @.claude/rules-pilot/sonnet-pilot.md
```

See `examples/CLAUDE.md.template` for a complete example.

---

## Customization required

This suite is a **starter pack**. Two files require workspace-specific customization:

### 1. `rules/core.md` § Production Safety Red Line

The default text uses abstract markers ("whatever your workspace's production markers are"). Replace with concrete markers for your stack:

- Branch: `main` / `production` / `master`
- Namespace: `prod` / `production`
- Project ID prefix: `*-prod` / `prod-*`
- IaC workspace: `production` / `prod`
- Environment variable: `ENV=production`

### 2. `rules/subagent-strategy.md` § Sub-Agent Dispatch Table

The default table uses generic agent names (`researcher`, `implementer`, `test-writer`, etc.). Replace with the agents in your `.claude/agents/` directory.

If you don't have specialized agents, the `general-purpose` agent (always available) accepts a `model` override and works as a fallback.

---

## What this is NOT

- **Not a model selector**: this suite assumes you already have access to Haiku 4.5, Sonnet 4.6, and Opus 4.7 via Claude Code. It doesn't change which models are available.
- **Not a billing tool**: the cost claims are calculated from public pricing; actual savings depend on cache hit rate, input/output ratio, and your task mix. Use a separate cost-tracking tool to measure real savings.
- **Not a code reviewer**: the Self-Review Loop dispatches `quick-code-reviewer` if you have one; otherwise the loop is just a `git diff --stat` checklist.

---

## Compatibility with other Claude Code skills

- **`citation-discipline`** (separate skill, also published as portable): enforces structured anchors when citing external sources. Both pilot SKILLs reference it (`§ Pre-flight #4`) but degrade gracefully if not installed.
- **`harness-eval`**: orthogonal — it scores your overall harness health (rule layer); this suite executes within that harness.
- **`output-discipline.md`** (included): plain-text formatting baseline; both pilot SKILLs reference it.

---

## Acknowledgments

- AgentOpt research (arxiv 2604.06296)
- Augment Code AGENTS.md evaluation (2026-04)
- Anthropic Claude Code conventions (`AGENTS.md` cross-tool root manifest)
- 4-iteration internal benchmark (5/3 baseline → v1 mode-declaration → v2 asymmetric → v3 symmetric)

Originally part of [`cc-workspace`](https://github.com/zeuikli/cc-workspace) — split out via `git subtree` into this standalone repo for plugin distribution. The `dist/claude-pilot-suite/` directory in cc-workspace is the source of truth; this repo is the published mirror.

---

## Contributing

Issues / PRs welcome:

- Workspace adaptation walk-throughs (different stacks: Rails / Django / Next.js / Go / etc.)
- Updated cost calculations (when Anthropic pricing changes)
- New benchmark data points (if you A/B test the suite vs full-Opus)
- Bug reports with reproducible task transcripts

License: MIT (see `LICENSE`).
