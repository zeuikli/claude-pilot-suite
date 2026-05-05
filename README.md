# Claude Pilot Suite

> **Version**: 0.1.1 (2026-05-05)
> **License**: MIT
> **Validated on**: Claude Sonnet 4.6 + Haiku 4.5 + Opus 4.7 (4-iteration benchmark, n=10 task corpus)

A portable, opinionated execution playbook for Claude Code that gets:

- **Haiku 4.5 quality close to Opus 4.7** (Haiku Pilot mode) — with 70–85% cost savings
- **Sonnet 4.6 quality on par with Opus 4.7** (Sonnet Pilot mode) — with 55–65% cost savings

Built on the empirical finding that **prompt structure + sub-agent delegation often matters more than raw model capability**. Documented in two studies:

> AgentOpt (arxiv 2604.06296): On HotpotQA, Opus alone = 31.71%; Ministral planner + Opus solver = 74.27%.
> Augment Code AGENTS.md eval (2026-04): Best-quality docs ≈ one model tier upgrade. Bad docs are worse than none.

---

## What's in this suite

```
claude-pilot-suite/
├── .claude-plugin/
│   └── plugin.json                  Plugin manifest (loadable via /plugin install)
├── CHANGELOG.md
├── rules/                          # Auto-loaded baseline discipline (6 files)
│   ├── core.md                       Language / Git / surgical changes / production safety
│   ├── subagent-strategy.md          Sub-agent delegation criteria + advisor mode
│   ├── context-management.md         Context window monitoring + compact strategy
│   ├── output-discipline.md          Plain-text formatting rules (no fluff)
│   ├── haiku-pilot.md                Haiku mode declaration + escalation gates
│   └── sonnet-pilot.md               Sonnet mode declaration + escalation gates
├── skills/                         # On-demand playbooks (loaded only when triggered)
│   ├── haiku-pilot/SKILL.md          Haiku-first execution + 4-step pre-flight
│   └── sonnet-pilot/SKILL.md         Quality-first Sonnet execution + 5-step pre-flight
├── examples/
│   └── CLAUDE.md.template            Sample CLAUDE.md to integrate the suite
├── README.md
├── INSTALL.md
├── RELEASING.md
└── LICENSE
```

---

## Two modes, one suite

### Haiku Pilot (cost-first, default)

**Triggers**: `haiku`, `haiku-pilot`, `haiku mode`, `cost-efficient run` — or `/haiku-pilot` skill

**Philosophy**: weak planner + strong delegation > strong planner alone. Default to Haiku 4.5, dispatch sub-agents aggressively, escalate only when quantitative gates fire.

**Per-session pre-flight** (4 steps):
1. **Reference Pattern** — point to an existing file, not abstract descriptions
2. **Think-Before-Coding** — restate requirement + list assumptions before implementation (compensates for Haiku's tendency to skip this)
3. **Diff-Review Pledge** — `git diff --stat` before declaring done; explain each changed file
4. **Content-First Structure** — question nature determines structure; no template-filling

**Escalation gates**:

| Condition | Escalate to |
|-----------|-------------|
| Same problem fails ≥ 3 times | Sonnet 4.6 |
| Task spans ≥ 10 independent files or cross-module design | Sonnet 4.6 |
| Keywords: "architecture decision" / "security review" / "threat modeling" | Opus 4.7 or `advisor()` |
| Code-gen > 300 LoC unprompted | Stop; redispatch to Sonnet `implementer` |
| User explicitly says "use Opus" / "use Sonnet" | Comply |

**Known Haiku gotchas** (mitigated by the SKILL):
- Skips assumption disclosure → Pre-flight #2 enforces it
- Writes 1.5–2× more comments → `output-discipline.md` caps unnecessary comments
- Long-context degradation > 300K tokens → compact at 30–35%, not 70%
- Multi-file inconsistency → ≥ 5-file edits auto-upgrade to Sonnet
- Template-first anti-pattern → Pre-flight #4 prevents it

**Expected savings**: 70–85% vs full-Opus baseline (80/15/5 task distribution: Haiku/Sonnet/Opus).

---

### Sonnet Pilot (quality-first)

**Triggers**: `sonnet`, `sonnet-pilot`, `sonnet mode`, `quality-first`, `Sonnet matches Opus`, `near Opus` — or `/sonnet-pilot` skill

**Philosophy**: deep context engineering + Opus-on-demand via `advisor()` ≈ full Opus quality.

**Per-session pre-flight** (5 mandatory steps — Sonnet's high confidence biases toward skipping these):
1. **Reference Pattern** — concrete file reference narrows design to "variant of correct pattern" instead of reinvention
2. **Decision-Log** — every non-obvious choice outputs: `Choice: <what> / Rejected: <option> — Reason: <one sentence>`
3. **Reasoning Chain Before Code** — restate task + list assumptions + implementation plan (≤ 5 bullets) + list options if ambiguous
4. **Self-Review Loop** — `git diff --stat` before done; changes > 30 LoC → dispatch `general-purpose` review; auth/payment/user-data → security-focused review; architecture → `Plan` agent before done
5. **Intermediate Checkpoint** — for ≥ 3 independent steps, pause after each: "Completed Step N/M: \<one-sentence result\>"; confirm before proceeding

**Escalation gate to Opus** (stricter than Haiku's — Opus is 5× the cost):

| Condition | Escalate to |
|-----------|-------------|
| Real global architecture decision affecting multiple services | Opus 4.7 |
| Security architecture design (designing auth, not reviewing code) | Opus 4.7 |
| Same problem fails ≥ 3 times and it's not a context issue | Opus 4.7 |
| `Plan` agent explicitly says "this needs Opus" | Opus 4.7 |
| User explicitly says "use Opus" | Opus 4.7 |

**Stay on Sonnet if** any one is true: task ≤ 19 files, < 3 prior attempts, no architecture/security keywords, single response ≤ 500 LoC, or reachable via `Plan` agent.

**Known Sonnet gotchas** (mitigated by the SKILL):
- Over-engineering trap — "elegantizes" unnecessarily → Decision-Log forces rejected-option rationale
- Confidence blindness — high confidence paradoxically skips self-review → Pre-flight #4 is mandatory
- Ambiguity self-resolve — guesses intent instead of listing options → Pre-flight #3 mandates listing
- Refactor-while-fixing — tidies surrounding code during bug fixes → `git diff --stat` scopes each change

**Expected savings**: 55–65% vs full-Opus baseline (80% Sonnet tasks, 20% escalate to `Plan` with Opus).

---

## The six rules (auto-loaded)

| File | What it enforces |
|------|-----------------|
| `core.md` | Reply in user's language; Git after every change; assumption disclosure before implementation; surgical-change scope (bug fix ≤ 50 LoC, feature ≤ 300 LoC); production safety confirmation gate |
| `subagent-strategy.md` | Delegate when reading ≥ 10 files, > 20 tool calls, or ≥ 3 independent subtasks; dispatch table mapping task types to `Explore` / `general-purpose` (Haiku or Sonnet) / `Plan` (Opus) agents |
| `context-management.md` | Compact at 70% general / 60% beginner / 30–35% long agentic; proactive compact every 300–400K tokens; context rot threshold awareness |
| `output-discipline.md` | No preamble ("Sure", "Of course"); no question restatement; plain-text ≤ 150 words; minimal comments; banned filler words |
| `haiku-pilot.md` | Mode declaration; self-check table (word count ≤ 800/question, anchors ≥ 3/question, escalation gate); soft threshold: total anchors ≥ 15 |
| `sonnet-pilot.md` | Mode declaration; self-check table (word count ≤ 800/question, anchors ≥ 5/question, Sonnet→Opus gate); soft threshold: total anchors ≥ 25 |

---

## Install

This bundle is **hybrid**: SKILLs load via Claude Code's plugin mechanism, while rules load via `@-import` from your CLAUDE.md.

See `INSTALL.md` for all four methods. Quick version (symlink — recommended for ongoing updates):

```bash
# Clone once to a shared location
cd ~/code-shared
git clone https://github.com/zeuikli/claude-pilot-suite.git

# Symlink into your project
cd ~/your-project
mkdir -p .claude/rules .claude/skills

ln -s ~/code-shared/claude-pilot-suite/rules/core.md              .claude/rules/core.md
ln -s ~/code-shared/claude-pilot-suite/rules/subagent-strategy.md .claude/rules/subagent-strategy.md
ln -s ~/code-shared/claude-pilot-suite/rules/context-management.md .claude/rules/context-management.md
ln -s ~/code-shared/claude-pilot-suite/rules/output-discipline.md .claude/rules/output-discipline.md
ln -s ~/code-shared/claude-pilot-suite/rules/haiku-pilot.md       .claude/rules/haiku-pilot.md
ln -s ~/code-shared/claude-pilot-suite/rules/sonnet-pilot.md      .claude/rules/sonnet-pilot.md

ln -s ~/code-shared/claude-pilot-suite/skills/haiku-pilot  .claude/skills/haiku-pilot
ln -s ~/code-shared/claude-pilot-suite/skills/sonnet-pilot .claude/skills/sonnet-pilot
```

Then add to your project's `CLAUDE.md`:

```markdown
## Pilot Suite Rules

- @.claude/rules/core.md
- @.claude/rules/subagent-strategy.md
- @.claude/rules/context-management.md
- @.claude/rules/output-discipline.md
- @.claude/rules/haiku-pilot.md
- @.claude/rules/sonnet-pilot.md
```

See `examples/CLAUDE.md.template` for a complete project CLAUDE.md example.

To update: `cd ~/code-shared/claude-pilot-suite && git pull`.

---

## What this is NOT

- **Not a model selector**: assumes you already have Haiku 4.5, Sonnet 4.6, and Opus 4.7 via Claude Code. It doesn't change which models are available.
- **Not a billing tool**: cost claims are calculated from public pricing. Actual savings depend on cache hit rate, input/output ratio, and task mix. Use a separate cost-tracking tool to measure real savings.
- **Not a code reviewer**: the Self-Review Loop dispatches `general-purpose` if you have it configured; otherwise the loop is just a `git diff --stat` checklist.

---

## Compatibility with other Claude Code skills

- **`citation-discipline`** (separate skill): enforces structured anchors when citing external sources. Both pilot SKILLs reference it (`§ Pre-flight #4`) but degrade gracefully if not installed.
- **`harness-eval`**: orthogonal — scores overall harness health (rule layer); this suite executes within that harness.
- **`output-discipline.md`** (included): plain-text formatting baseline referenced by both pilot SKILLs.

---

## Acknowledgments

- AgentOpt research (arxiv 2604.06296)
- Augment Code AGENTS.md evaluation (2026-04)
- Anthropic Claude Code conventions (`AGENTS.md` cross-tool root manifest)
- 4-iteration internal benchmark (5/3 baseline → v1 mode-declaration → v2 asymmetric → v3 symmetric)

Source repository: <https://github.com/zeuikli/claude-pilot-suite>. See `RELEASING.md` for the release flow.

---

## Contributing

Issues / PRs welcome:

- Workspace adaptation walk-throughs (Rails / Django / Next.js / Go / etc.)
- Updated cost calculations (when Anthropic pricing changes)
- New benchmark data points (if you A/B test the suite vs full-Opus)
- Bug reports with reproducible task transcripts

License: MIT (see `LICENSE`).
