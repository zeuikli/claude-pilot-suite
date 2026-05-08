# Claude Pilot Suite

> **Version**: 0.4.1 (2026-05-08 — opus-pilot added; 5 benchmark-driven fixes for haiku/sonnet-pilot)
> **License**: MIT
> **Validated on**: Claude Sonnet 4.6 + Haiku 4.5 + Opus 4.7 (6-agent × 10-question pilot-vs-vanilla benchmark)

A portable, opinionated execution playbook for Claude Code that provides:

- **Haiku 4.5 quality close to Opus 4.7** (Haiku Pilot mode) — with 70–85% cost savings
- **Sonnet 4.6 quality floor protection + predictable escalation** (Sonnet Pilot mode) — with 55–65% cost savings
- **Opus 4.7 ceiling elevation** (Opus Pilot mode) — 5 structural mechanisms to exceed vanilla Opus baseline on hard analytical tasks

**What "quality floor protection" means**: on structured wiki-extraction tasks, Sonnet+SKILL ≈ Sonnet Plain (both ~283/300 vs Opus 286/300 in A/B/C benchmark). The SKILL's value is in *preventing quality drops* on complex tasks (ambiguous requirements, multi-step agentic, cross-module design) — not in boosting already-strong performance on routine extraction. Think of it as insurance: no premium on good days, prevents catastrophic failure on hard days.

Built on the empirical finding that **prompt structure + sub-agent delegation often matters more than raw model capability**. Documented in two arXiv-backed studies:

> AgentOpt (arxiv 2604.06296): On HotpotQA, Opus alone = 31.71%; Ministral planner + Opus solver = 74.27%.
> Augment Code AGENTS.md eval (2026-04): Best-quality docs ≈ one model tier upgrade. Bad docs are worse than none.

---

## What's in this suite

```
claude-pilot-suite/
├── rules/                          # Auto-loaded baseline discipline
│   ├── core.md                       Language / Git / surgical changes / quality
│   ├── subagent-strategy.md          Sub-agent delegation criteria + advisor mode
│   ├── context-management.md         Context window monitoring + compact strategy
│   ├── output-discipline.md          Plain-text formatting rules (no fluff)
│   ├── haiku-pilot.md                Mode declaration + escalation gates (Haiku→Sonnet/Opus)
│   ├── sonnet-pilot.md               Mode declaration + escalation gates (Sonnet→Opus)
│   └── opus-pilot.md                 Mode declaration for Opus ceiling-elevation (no upward escalation)
├── skills/                         # On-demand playbooks (loaded only when triggered)
│   ├── haiku-pilot/SKILL.md          Default-Haiku execution + 7-step pre-flight (incl. Source-Verify gate)
│   ├── sonnet-pilot/SKILL.md         Quality-first Sonnet execution + 7-step pre-flight (incl. Line-Level Source-Verify)
│   └── opus-pilot/SKILL.md           Ceiling-elevation playbook: 5 mechanisms + 3 Opus-specific pre-flights
├── examples/
│   └── CLAUDE.md.template            Sample CLAUDE.md to integrate the suite
├── README.md
├── INSTALL.md
└── LICENSE
```

---

## Three modes, one suite

### Haiku Pilot (cost-first, default)

Trigger: "haiku" / "Haiku" / "Haiku mode" / "haiku-pilot"

**Philosophy**: weak planner + strong delegation > strong planner alone. Default to Haiku, dispatch sub-agents aggressively, escalate only on quantitative gate triggers.

**Escalation gates**:
- Same problem failed ≥ 3 times → Sonnet
- Task touches ≥ 10 files → Sonnet
- Architecture / design / security keywords → Opus or `advisor()`
- Code-gen > 300 LoC unrequested → halt, redispatch to Sonnet `implementer`

**Expected savings**: 70–85% vs full-Opus baseline (assuming 80/15/5 task distribution).

### Sonnet Pilot (quality-first)

Trigger: "sonnet" / "Sonnet" / "Sonnet mode" / "sonnet-pilot"

**Philosophy**: deep context engineering + Opus-on-demand via `advisor()` ≈ full Opus quality.

**Pre-flight enforcements** (Sonnet's high confidence biases toward skipping these):
1. Reference Pattern (point to a file, not abstract)
2. Decision-Log (every non-obvious choice → `Choice: ... / Rejected: ... — Reason: ...`)
3. Reasoning Chain Before Code
4. Self-Review Loop (`git diff --stat` + per-file explanation)
5. Intermediate Checkpoint (pause and verify per step)
6. Mid-Write Outline Verification (architecture / counter-factual / synthesis tasks: at ~200w drafted, verify scope / projected total / mandatory-citation coverage — catches runaway answers while still cheap to fix)

**Escalation gate to Opus**: stricter than haiku-pilot's, because Opus is 5× the cost.

**Expected savings**: 55–65% vs full-Opus baseline.

### Opus Pilot (ceiling-elevation)

Trigger: "opus" / "Opus" / "Opus mode" / "opus-pilot"

Sub-mode modifiers (append to any base trigger):
- `Opus 1M` / `1M context` / `extended context` → activate 1M context window
- `Opus xhigh` / `xhigh thinking` / `extended thinking` → activate extended thinking budget

**Philosophy**: Opus is the ceiling — no upward escalation. The goal is to exceed vanilla Opus baseline on hard tasks through structural mechanisms, not by calling a stronger model.

**5 ceiling-elevation mechanisms**:
1. **Reverse-Advisor Loop** — Opus critiques its own draft before finalizing
2. **Parallel Hypotheses Synthesis** — generate N competing hypotheses, merge strongest elements
3. **CAR Explicitness** — every claim anchored to Context / Action / Result triples
4. **Decision-Log Externalization** — every non-obvious choice logged as `Choice: … / Rejected: … — Reason: …`
5. **Meta-Harness Filesystem Loop** — for agentic tasks, Opus writes a plan file and verifies after each step

**3 Opus-specific pre-flights** (fires before any hard task):
1. Ambiguity Surfacing — list all under-specified inputs before proceeding
2. Start-Simple-First — solve a stripped-down version first, then generalize
3. Down-Delegation Eval — judge whether sub-tasks can be delegated to Sonnet/Haiku before consuming Opus tokens

**Hard rule**: Opus is the ceiling. All escalation goes *down* (Sonnet/Haiku sub-agents), never up.

**Citation grounding**: 9 research papers (P01–P09) + P10 AgentOpt. See `skills/opus-pilot/SKILL.md` §References.

---

## Why these specifically (vs raw model selection)

The suite is the synthesis of benchmarks measuring how prompt structure affects answer quality across model tiers. Key findings:

| Finding | Source | Implication |
|---------|--------|-------------|
| Sub-agent delegation > model upgrade for multi-step tasks | AgentOpt arxiv 2604.06296 | Default to Haiku + delegate, not jump to Opus |
| Citation density correlates with reviewer-verifiable quality | v0.2.1 benchmark | Pre-flight #4 Citation Anchor enforcement |
| Anti-expansion self-check tables prevent 2.24× word inflation | v3 benchmark | Self-Review Loop uses single-table format |
| Sonnet's confidence bias = skip self-review unless externally enforced | A/B/C benchmark | Pre-flight #4 is mandatory in Sonnet Pilot |
| Haiku's tendency to skip assumption disclosure | haiku-pilot.md §Gotchas | Pre-flight #2 enforces it |
| Haiku fabricates numbers on hard citation tasks (5/20 cases) | v0.2.1 benchmark | Pre-flight #5 Source-Verify gate (`grep -i` every cited number against source) |
| Self-check overhead can exceed value on tiny answers (Q01/Q04 net-negative vs vanilla) | v0.4.1 benchmark | Task-type Fast-path: easy recall → one-line `fast-path: <reason>` |
| Haiku anchor density collapses on hard tasks (7/100w medium → 1/100w hard) | v0.4.1 benchmark | Pre-flight #4 hard-task threshold raised to ≥ 5 anchors/paragraph |
| Sonnet self-check word-count enforcement fails without `wc -w` gate (Q10 1115w, +39.4% over) | v0.4.1 benchmark | Hard-cap added: hard-task over 800w must rewrite, not apologize |
| Hard task type misclassification causes gate no-hit (architecture-hard scored as medium) | v0.4.1 benchmark | Escalation gate now auto-triggers on Q type tagged Architecture/Counter-factual/Synthesis |
| Section-level citation vs line-level: 0.5pt D1 gap (A2=9.5 vs A3=10.0) | v0.4.1 benchmark | Pre-flight #7 Source-Verify requires `(P0X §Y.Z:LineN)` on hard tasks |

---

## Install

See `INSTALL.md` for three installation methods (clone+symlink / copy / subtree).

Quick version:

```bash
# Clone next to your project, then symlink
git clone https://github.com/<your-fork>/claude-pilot-suite.git
ln -s "$(pwd)/claude-pilot-suite/rules" ~/your-project/.claude/rules-pilot
ln -s "$(pwd)/claude-pilot-suite/skills/haiku-pilot" ~/your-project/.claude/skills/haiku-pilot
ln -s "$(pwd)/claude-pilot-suite/skills/sonnet-pilot" ~/your-project/.claude/skills/sonnet-pilot
ln -s "$(pwd)/claude-pilot-suite/skills/opus-pilot"   ~/your-project/.claude/skills/opus-pilot

# Then update your project's CLAUDE.md to @-import the rules:
# @.claude/rules-pilot/core.md
# @.claude/rules-pilot/subagent-strategy.md
# @.claude/rules-pilot/context-management.md
# @.claude/rules-pilot/output-discipline.md
# @.claude/rules-pilot/haiku-pilot.md
# @.claude/rules-pilot/sonnet-pilot.md
# @.claude/rules-pilot/opus-pilot.md
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
- **Opus Pilot is not the default**: use it only for architecture decisions, counter-factual reasoning, synthesis across multiple sources, or threat modeling where vanilla Opus is the explicit baseline to beat. Most tasks should run on haiku-pilot or sonnet-pilot.

---

## Compatibility with other Claude Code skills

- **`output-discipline.md`** (included): plain-text formatting baseline; all three pilot SKILLs reference it.

---

## Acknowledgments

- AgentOpt research (arxiv 2604.06296)
- Augment Code AGENTS.md evaluation (2026-04)
- Anthropic Claude Code conventions (`AGENTS.md` cross-tool root manifest)
- 4-iteration internal benchmark (5/3 baseline → v1 mode-declaration → v2 asymmetric → v3 symmetric)
- 6-agent × 10-question pilot-vs-vanilla benchmark (2026-05-08, v0.4.1 patch source)
- opus-pilot citation grounding: 9 papers (P01 AgentFlow / P02 Confucius / P03 Meta-Harness / P04 NLAH / P05 COMPOSITE-STEM / P06 Agent Harness Survey / P07 CAR / P08 OpenDev / P09 Skill Issue) + P10 AgentOpt

Originally part of `cc-workspace` — extracted as portable plugin pack for community use.

---

## Contributing

Issues / PRs welcome:

- Workspace adaptation walk-throughs (different stacks: Rails / Django / Next.js / Go / etc.)
- Updated cost calculations (when Anthropic pricing changes)
- New benchmark data points (if you A/B test the suite vs full-Opus)
- Bug reports with reproducible task transcripts

License: MIT (see `LICENSE`).
