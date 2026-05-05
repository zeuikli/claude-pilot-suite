# Installation

Four install methods, ordered by Claude Code idiomatic-ness.

> **Hybrid bundle note**: SKILLs (in `skills/`) load via Claude Code's plugin mechanism. Rules (in `rules/`) load via `@-import` from your project CLAUDE.md — rules are not yet plugin-supported, so they require copy/symlink. All four methods install both.

---

## Method 1: plugin install + rules copy (recommended for SKILLs)

Best for: trying the SKILLs first without committing to file-level integration; lets Claude Code's `/plugin` system manage SKILL lifecycle.

```bash
# 1. SKILLs via plugin install (manages skills/haiku-pilot + skills/sonnet-pilot)
/plugin install <path-or-url-to-claude-pilot-suite>

# 2. Rules still need copy (rules/ is not plugin-supported)
cd ~/your-project
mkdir -p .claude/rules
cp <path-to-claude-pilot-suite>/rules/*.md .claude/rules/

# 3. Wire rules into CLAUDE.md (see Wire rules section below)
```

> The plugin manifest at `.claude-plugin/plugin.json` registers `name: claude-pilot-suite` and the SKILLs are auto-discovered by Claude Code from the plugin's `skills/` directory.

---

## Method 2: clone + symlink (recommended)

Best for: ongoing updates, contributing back, having one canonical pilot suite shared across multiple projects.

```bash
# 1. Clone the suite somewhere persistent
cd ~/code-shared  # or wherever you keep shared tooling
git clone https://github.com/<your-fork>/claude-pilot-suite.git

# 2. In each project that should use it:
cd ~/your-project

# Symlink rules (each rule is a single file)
mkdir -p .claude/rules
ln -s ~/code-shared/claude-pilot-suite/rules/core.md              .claude/rules/core.md
ln -s ~/code-shared/claude-pilot-suite/rules/subagent-strategy.md .claude/rules/subagent-strategy.md
ln -s ~/code-shared/claude-pilot-suite/rules/context-management.md .claude/rules/context-management.md
ln -s ~/code-shared/claude-pilot-suite/rules/output-discipline.md .claude/rules/output-discipline.md
ln -s ~/code-shared/claude-pilot-suite/rules/haiku-pilot.md       .claude/rules/haiku-pilot.md
ln -s ~/code-shared/claude-pilot-suite/rules/sonnet-pilot.md      .claude/rules/sonnet-pilot.md

# Symlink SKILL directories (whole folder)
mkdir -p .claude/skills
ln -s ~/code-shared/claude-pilot-suite/skills/haiku-pilot   .claude/skills/haiku-pilot
ln -s ~/code-shared/claude-pilot-suite/skills/sonnet-pilot  .claude/skills/sonnet-pilot

# 3. Wire rules into CLAUDE.md (see examples/CLAUDE.md.template)
```

**Pull updates**:
```bash
cd ~/code-shared/claude-pilot-suite && git pull
```

All your projects pick up the new version on their next session — no per-project work.

---

## Method 3: copy (no auto-update)

Best for: starter usage, customizing without diverging from upstream concerns, projects that want vendoring.

```bash
cd ~/your-project
git clone https://github.com/<your-fork>/claude-pilot-suite.git /tmp/pilot-suite

# Copy what you need
cp /tmp/pilot-suite/rules/*.md            .claude/rules/
cp -r /tmp/pilot-suite/skills/haiku-pilot  .claude/skills/
cp -r /tmp/pilot-suite/skills/sonnet-pilot .claude/skills/
rm -rf /tmp/pilot-suite

# Wire rules into CLAUDE.md
```

You're free to modify any file. To pull future updates, manually re-copy the files you haven't customized.

---

## Method 4: git subtree (vendored, traceable)

Best for: monorepos, projects that want suite under their own VCS history without external dependency.

```bash
cd ~/your-project

# Add as a subtree under .claude/
git subtree add --prefix=.claude/pilot-suite \
  https://github.com/<your-fork>/claude-pilot-suite.git main --squash

# Then symlink within your .claude/ to standard locations
ln -s pilot-suite/rules/core.md              .claude/rules/core.md
ln -s pilot-suite/rules/subagent-strategy.md .claude/rules/subagent-strategy.md
ln -s pilot-suite/rules/context-management.md .claude/rules/context-management.md
ln -s pilot-suite/rules/output-discipline.md .claude/rules/output-discipline.md
ln -s pilot-suite/rules/haiku-pilot.md       .claude/rules/haiku-pilot.md
ln -s pilot-suite/rules/sonnet-pilot.md      .claude/rules/sonnet-pilot.md
ln -s pilot-suite/skills/haiku-pilot   .claude/skills/haiku-pilot
ln -s pilot-suite/skills/sonnet-pilot  .claude/skills/sonnet-pilot

# Wire rules into CLAUDE.md
```

**Pull updates**:
```bash
git subtree pull --prefix=.claude/pilot-suite \
  https://github.com/<your-fork>/claude-pilot-suite.git main --squash
```

---

## Wire rules into CLAUDE.md

After installing files, your CLAUDE.md must `@-import` the rules so they auto-load each session. Add to the end of your CLAUDE.md:

```markdown
## Pilot Suite Rules (auto-loaded)

- @.claude/rules/core.md
- @.claude/rules/subagent-strategy.md
- @.claude/rules/context-management.md
- @.claude/rules/output-discipline.md
- @.claude/rules/haiku-pilot.md
- @.claude/rules/sonnet-pilot.md
```

See `examples/CLAUDE.md.template` for a full example with explanations.

---

## Verify installation

In a fresh Claude Code session, type:

```
/haiku-pilot
```

You should see Claude acknowledge mode switch + load the SKILL playbook. Try a task; pre-flight should fire.

---

## Customization required

Two files need workspace-specific edits before production use. See `README.md` § "Customization required" for details:

1. **`rules/core.md` § Production Safety Red Line** — replace abstract markers with your concrete production environment markers
2. **`rules/subagent-strategy.md` § Sub-Agent Dispatch Table** — replace generic agent names with the agents in your `.claude/agents/` directory

If you skip customization, the suite still works but with degraded specificity (e.g. dispatch table won't reference real agents in your repo).

---

## Uninstall

For Method 1 (symlink):
```bash
rm .claude/rules/{core,subagent-strategy,context-management,output-discipline,haiku-pilot,sonnet-pilot}.md
rm -r .claude/skills/{haiku,sonnet}-pilot
# Remove the @-imports from CLAUDE.md
```

For Methods 2/3, delete the copied files / subtree.

---

## Troubleshooting

### Rules don't seem to load

Check:
1. Files exist: `ls -la .claude/rules/`
2. CLAUDE.md `@-import` lines are present and paths are correct
3. New session started after install (rules load at session start, not mid-session)

### "Triggers" don't activate the SKILL

The pilot SKILLs are on-demand (loaded by trigger phrase). Required:
- Trigger phrase is in the message (e.g. `/haiku-pilot` or "Haiku 模式")
- The SKILL directory exists at `.claude/skills/<skill-name>/SKILL.md`
- The SKILL frontmatter parses (no syntax errors)

If still not working, run a Claude Code skill list query to verify the skill is registered.

### My agents aren't in the dispatch table

Expected. The default table is a starter template. Edit `rules/subagent-strategy.md` to list your actual agents. See README § "Customization required".

### Cost savings less than promised

Cost claims assume an 80/15/5 task distribution (Haiku/Sonnet/Opus). Your actual distribution may differ:
- Heavy on architecture/security tasks → more Opus → savings closer to 50%
- Heavy on simple coding/research → more Haiku → savings could exceed 85%

Use a cost-tracking tool to measure your actual distribution and adjust expectations.
