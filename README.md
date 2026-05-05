# Claude Pilot Suite

> **Version**: 0.1.1 (2026-05-05)
> **License**: MIT
> **Validated on**: Claude Sonnet 4.6 + Haiku 4.5 + Opus 4.7 (4-iteration benchmark, n=10 task corpus)

A portable, opinionated execution playbook for Claude Code that gets:

- **Haiku 4.5 quality close to Opus 4.7** (Haiku Pilot mode) — with 70–85% cost savings
- **Sonnet 4.6 quality on par with Opus 4.7** (Sonnet Pilot mode) — with 55–65% cost savings

Built on the empirical finding that **prompt structure + sub-agent delegation often matters more than raw model capability**:

> AgentOpt (arxiv 2604.06296): Opus solo on HotpotQA = 31.71%; Ministral planner + Opus solver = 74.27%.
> Augment Code AGENTS.md eval (2026-04): Best-quality docs ≈ one model tier upgrade. Bad docs are worse than none.

---

## What's in this suite

```
claude-pilot-suite/
├── .claude-plugin/
│   └── plugin.json                  Plugin manifest (loadable via /plugin install)
├── CHANGELOG.md
├── rules/                          # 6 auto-loaded baseline rules (@-imported in CLAUDE.md)
│   ├── core.md                       Language / Git / surgical changes / production safety
│   ├── subagent-strategy.md          Sub-agent delegation criteria + advisor mode
│   ├── context-management.md         Context window monitoring + compact strategy
│   ├── output-discipline.md          Plain-text formatting rules (no fluff)
│   ├── haiku-pilot.md                Haiku mode declaration + escalation gates
│   └── sonnet-pilot.md               Sonnet mode declaration + escalation gates
├── skills/                         # On-demand playbooks (load only when triggered)
│   ├── haiku-pilot/SKILL.md          Haiku-first execution + 4-step pre-flight + task router
│   └── sonnet-pilot/SKILL.md         Quality-first Sonnet execution + 5-step pre-flight + task router
├── examples/
│   └── CLAUDE.md.template            Sample CLAUDE.md showing how to integrate the suite
├── README.md
├── INSTALL.md
├── RELEASING.md
└── LICENSE
```

---

## Two modes, one suite

### Haiku Pilot (cost-first, default)

**Trigger words**: `haiku`, `haiku-pilot`, `haiku mode`, `cost-efficient run`

**Philosophy**: weak planner + smart delegation > strong planner doing everything. Default to Haiku 4.5, delegate to sub-agents aggressively, escalate only when a quantitative gate fires.

**Per-task pre-flight** (4 checks, from `haiku-pilot.md` + `skills/haiku-pilot/SKILL.md`):

1. **Reference Pattern** — point to an existing file (`src/auth/login.ts`) instead of verbal description; Haiku diverges easily from abstract descriptions
2. **Think-Before-Coding** — restate task understanding + list key assumptions before writing code; compensates for Haiku's tendency to skip this step
3. **Diff-Review Pledge** — run `git diff --stat` before declaring done, explain each changed file; catches unintended scope creep
4. **Content-First Structure** — question nature determines structure; forbidden: filling fixed sub-heading templates; emoji budget ≤ 3 per section

**Escalation gates** (quantitative — not "when it feels complex"):

| Condition | Escalate to |
|-----------|-------------|
| Same problem fails ≥ 3 times | Sonnet 4.6 |
| Task spans ≥ 10 independent files or cross-module design | Sonnet 4.6 |
| Keywords: "architecture decision" / "design review" / "security review" / "threat modeling" | Opus 4.7 or `advisor()` |
| Code-gen > 300 LoC unprompted | Stop; redispatch to Sonnet implementer |
| User explicitly says "use Opus" / "use Sonnet" | Comply |

**Stay on Haiku if** any one is true: ≤ 9 independent files, < 3 prior attempts, no trigger keywords, single response ≤ 300 LoC, no explicit upgrade request.

**Known Haiku 4.5 gotchas** (mitigated in the SKILL):
- Skips assumption disclosure → Pre-flight #2 enforces it
- Verbose comments (1.5–2× more than Sonnet) → `output-discipline.md` caps unnecessary comments
- Long-context degradation > 300K tokens → compact at 30–35%, not 70%
- Inconsistency on multi-file changes (5+ files) → auto-upgrade to Sonnet
- Template-first anti-pattern → Pre-flight #4 prevents it

**Expected savings**: 70–85% vs full-Opus baseline (80% Haiku / 15% Sonnet / 5% Opus task distribution).

---

### Sonnet Pilot (quality-first)

**Trigger words**: `sonnet`, `sonnet-pilot`, `sonnet mode`, `full Sonnet`, `quality-first`, `Sonnet matches Opus`, `near Opus`

**Philosophy**: deep context engineering + `advisor()` on-demand ≈ full Opus quality. Sonnet mode does not mean "always use Opus"; the goal is Opus-level quality using Sonnet.

**Per-session pre-flight** (5 mandatory steps, from `skills/sonnet-pilot/SKILL.md`):

1. **Reference Pattern** — same as Haiku, but Sonnet given abstract descriptions tends to design from scratch; concrete file reference narrows to "variant of correct pattern"
2. **Decision-Log** — for every non-obvious technical decision, output before completion: `Choice: <what> / Rejected: <option> — Reason: <one sentence>`; Opus does this naturally; Sonnet must be forced
3. **Reasoning Chain Before Code** — restate task + list assumptions + implementation plan (≤ 5 bullets) + list options if ambiguous; Sonnet's high confidence paradoxically makes it more likely to skip this
4. **Self-Review Loop** — `git diff --stat` before done; changes > 30 LoC → dispatch `general-purpose` review; auth/payment/user-data → security-focused review; architecture → `Plan` agent before done
5. **Intermediate Checkpoint** — for ≥ 3 independent steps, pause after each: "Completed Step N/M: \<one-sentence result\>"; stop and ask if unexpected; don't power through

**Escalation gate to Opus** (stricter than Haiku's — Opus is 5× the cost):

| Condition | Escalate to |
|-----------|-------------|
| Real global architecture decision affecting multiple services | Opus 4.7 |
| Security architecture design (designing auth system, not just reviewing code) | Opus 4.7 |
| Same problem fails ≥ 3 times and it is not a context issue | Opus 4.7 |
| `Plan` agent explicitly says "this needs Opus" | Opus 4.7 |
| User explicitly says "use Opus" | Opus 4.7 |

**Stay on Sonnet if** any one is true: ≤ 19 independent files, < 3 prior attempts, no trigger keywords, single response ≤ 500 LoC, or reachable via `Plan` agent (Sonnet session + Opus Plan ≈ full Opus quality).

**Known Sonnet 4.6 gotchas** (mitigated in the SKILL):
- Over-engineering trap — adds abstraction layers; Decision-Log forces "Rejected: ..." rationale
- Confidence blindness — high confidence means it skips self-review; Pre-flight #4 is mandatory
- Ambiguity self-resolve — guesses intent instead of listing options; Pre-flight #3 mandates listing
- Multi-step reasoning without checkpoints — Pre-flight #5 forces per-step status output
- Refactor-while-fixing syndrome — `git diff --stat` scopes each change

**Does not persist across sessions**: new sessions revert to Haiku Pilot default; re-activate with a trigger word at session start.

**Expected savings**: 55–65% vs full-Opus baseline (80% Sonnet / 20% escalate to `Plan` with Opus).

---

## The six rules (auto-loaded)

All six load via `@-import` in your project CLAUDE.md. They apply in every session regardless of which pilot mode is active.

| File | What it enforces |
|------|-----------------|
| `core.md` | Reply in user's language; `git add → commit → push` after every change (retry up to 4×); state understanding + assumptions before coding; surgical change scope (bug fix ≤ 50 LoC, feature ≤ 300 LoC); production safety confirmation gate |
| `subagent-strategy.md` | Delegate when reading ≥ 10 files, > 20 tool calls, ≥ 3 independent subtasks, or task type ∈ {research, security review, architecture decision}; model selection by file count (0–1 → Haiku, 2–9 → Sonnet, 10+ → Sonnet/Opus); advisor() usage |
| `context-management.md` | Three-tier compact triggers: behavioral signal (model asks for context) → `/rewind` immediately; numeric (70% general / 60% beginner / 30–35% long agentic); timer (compact every 300–400K tokens); auto memory configuration |
| `output-discipline.md` | No preamble ("Sure", "Of course"); no question restatement; plain-text ≤ 150 words; minimal comments; banned filler words (just / really / basically / it's worth noting). Measured: −80.6% output tokens with no quality regression |
| `haiku-pilot.md` | Haiku mode declaration; self-check table for citation tasks (word count ≤ 800/question, anchors ≥ 3/question); soft threshold: total anchors ≥ 15 |
| `sonnet-pilot.md` | Sonnet mode declaration; self-check table for citation tasks (word count ≤ 800/question, anchors ≥ 5/question); soft threshold: total anchors ≥ 25; boundary rules with haiku-pilot |

---

## Sub-agent dispatch table (from `subagent-strategy.md`)

All agent types below are Claude Code built-ins — available without any `.claude/agents/` setup.

| Scenario | Agent type | Model |
|----------|-----------|-------|
| Research / > 10 files / locate symbols | `Explore` | Haiku 4.5 |
| 3+ independent subtasks | dispatch multiple in one message | — |
| Surgical code edit (single file ≤ 30 LoC, no design decision) | `general-purpose` | Haiku 4.5 |
| Cross-module / > 30 LoC / requires design judgment | `general-purpose` | Sonnet 4.6 |
| Test writing (boundary cases + mocks) | `general-purpose` | Sonnet 4.6 |
| Quick pre-commit security check | `general-purpose` | Sonnet 4.6 |
| OWASP / proactive threat modeling | `general-purpose` | Sonnet 4.6 |
| Architecture decision / implementation plan | `Plan` | Opus 4.7 |
| Pre-commit multi-dimension review | `/deep-review` skill | mixed |

Add custom agents in `.claude/agents/<name>/agent.md` to replace `general-purpose` rows with tighter-scoped agents.

---

## Install

This bundle is **hybrid**: SKILLs load via Claude Code's plugin mechanism; rules load via `@-import` from your CLAUDE.md. See `INSTALL.md` for all four methods and troubleshooting.

### Method 1: plugin install (quickest)

Run inside Claude Code:

```
/plugin install https://github.com/zeuikli/claude-pilot-suite
```

Or from a local clone:

```
/plugin install /path/to/claude-pilot-suite
```

The plugin manager installs both SKILLs automatically. Rules still need to be wired manually (see step 2 below) — this is a Claude Code limitation, not a bug.

**Step 2 — wire rules into your project's `CLAUDE.md`:**

```bash
# Copy rule files into your project
cd ~/your-project
mkdir -p .claude/rules
cp /path/to/claude-pilot-suite/rules/*.md .claude/rules/
```

Then add to your `CLAUDE.md`:

```markdown
## Pilot Suite Rules (auto-loaded)

@.claude/rules/core.md
@.claude/rules/subagent-strategy.md
@.claude/rules/context-management.md
@.claude/rules/output-discipline.md
@.claude/rules/haiku-pilot.md
@.claude/rules/sonnet-pilot.md
```

**Verify**: in a fresh Claude Code session, type `haiku-pilot` or `sonnet-pilot` — Claude should acknowledge the mode switch and load the SKILL playbook. Skills are also invocable directly as `/claude-pilot-suite:haiku-pilot` and `/claude-pilot-suite:sonnet-pilot`.

---

### Method 2: clone + symlink (recommended for ongoing updates)

```bash
# Clone once to a shared location
cd ~/code-shared
git clone https://github.com/zeuikli/claude-pilot-suite.git

# In each project that should use it:
cd ~/your-project
mkdir -p .claude/rules .claude/skills

ln -s ~/code-shared/claude-pilot-suite/rules/core.md               .claude/rules/core.md
ln -s ~/code-shared/claude-pilot-suite/rules/subagent-strategy.md  .claude/rules/subagent-strategy.md
ln -s ~/code-shared/claude-pilot-suite/rules/context-management.md .claude/rules/context-management.md
ln -s ~/code-shared/claude-pilot-suite/rules/output-discipline.md  .claude/rules/output-discipline.md
ln -s ~/code-shared/claude-pilot-suite/rules/haiku-pilot.md        .claude/rules/haiku-pilot.md
ln -s ~/code-shared/claude-pilot-suite/rules/sonnet-pilot.md       .claude/rules/sonnet-pilot.md

ln -s ~/code-shared/claude-pilot-suite/skills/haiku-pilot  .claude/skills/haiku-pilot
ln -s ~/code-shared/claude-pilot-suite/skills/sonnet-pilot .claude/skills/sonnet-pilot
```

Then add to your project's `CLAUDE.md`:

```markdown
## Pilot Suite Rules (auto-loaded)

@.claude/rules/core.md
@.claude/rules/subagent-strategy.md
@.claude/rules/context-management.md
@.claude/rules/output-discipline.md
@.claude/rules/haiku-pilot.md
@.claude/rules/sonnet-pilot.md
```

**Pull updates**: `cd ~/code-shared/claude-pilot-suite && git pull` — all projects pick up the new version on their next session.

**Verify**: in a fresh Claude Code session, type `haiku-pilot` or `sonnet-pilot` — Claude should acknowledge the mode switch and load the SKILL playbook.

See `examples/CLAUDE.md.template` for a complete project CLAUDE.md example.

---

## What this is NOT

- **Not a model selector**: assumes you already have Haiku 4.5, Sonnet 4.6, and Opus 4.7 via Claude Code. It does not change which models are available.
- **Not a billing tool**: cost claims are calculated from public pricing (2026-04). Actual savings depend on cache hit rate, input/output ratio, and task mix.
- **Not a code reviewer**: the Self-Review Loop dispatches `general-purpose` (Sonnet) for review; if not configured, the loop falls back to a `git diff --stat` checklist.

---

## Acknowledgments

- AgentOpt (arxiv 2604.06296): empirical basis for weak-planner + strong-delegation > strong-planner-alone
- Augment Code AGENTS.md evaluation (2026-04): empirical basis for docs quality ≈ model tier upgrade

Source repository: <https://github.com/zeuikli/claude-pilot-suite>. See `RELEASING.md` for the release flow.

---

## Contributing

Issues / PRs welcome:

- Workspace adaptation walk-throughs (Rails / Django / Next.js / Go / etc.)
- Updated cost calculations (when Anthropic pricing changes)
- New benchmark data points (if you A/B test the suite vs full-Opus)
- Bug reports with reproducible task transcripts

License: MIT (see `LICENSE`).
