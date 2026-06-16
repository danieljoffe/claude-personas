---
name: harness
description: Audit a repo's Claude Code harness — CLAUDE.md + .claude/ (rules, agents, skills, docs, hooks, settings, memory) — for redundant/dead config, prose guardrails that should be enforced, always-loaded token bloat, contradictions, and feedback-loop gaps. USE WHEN the user says "audit my claude setup", "review my .claude config", "is my CLAUDE.md bloated", "clean up my agent harness", or wants a meta-review of how the AI is steered.
user-invocable: true
argument-hint: '[target-path] [--area dedup|guardrails|context|hygiene|feedback]'
---

# /personas:harness

Orchestrate the `harness-auditor` persona over a repo's Claude Code harness and return one consolidated, deduplicated report. This is the **meta** audit: it critiques the config that steers the agent, not the application code (`/personas:audit`) or the live UI (`/personas:panel`).

## Inputs

- **Target**: the path argument, else the current working directory.
- **Binding**: read `<target>/.claude/personas.config.json` → `harness` block (see this plugin's `personas.config.example.json`). It declares `claudeMd`, `configRoot`, `alwaysLoaded`, and `outOfScope`. If absent, infer: root + nested `CLAUDE.md`, `.claude/{rules,agents,skills,docs,hooks}`, `settings*.json`, and the memory index; exclude volatile dirs (`sessions`, `scratch`, `worktrees`, `agent-memory`, caches). Tell the user which binding you used.
- **`--area`** (optional): restrict to a single lens — `dedup`, `guardrails`, `context`, `hygiene`, or `feedback`.

## Steps

1. **Scope.** Resolve the binding. Build a manifest of harness files (respect `outOfScope`). In the **same** `ctx_batch_execute`, also build a **ground-truth inventory** of what the repo actually contains — top-level + `apps/` listing, `nx show projects` (if Nx), `git branch -r`, key config files — so the auditor can validate every path / app / branch / file the harness references against reality instead of guessing. Use context-mode for all of this so raw config stays off-prompt. Never silently truncate — if the surface is large, say what you're sampling.
2. **Cost the budget.** Split the manifest into **always-loaded** (root + nested `CLAUDE.md`, every `@`-imported doc, `SessionStart`-injected files, large static reminders) vs **path-scoped** (`.claude/rules/*` with globs) vs **on-demand** (skills/docs pulled only when invoked). The always-loaded set is paid every turn — flag its heavy hitters.
3. **Wiring map.** Cross-reference every declared skill / agent / persona / hook against where it's actually invoked or registered (commands, manifests, `settings*.json`, orchestrators, `@`-imports). Note orphans (referenced nowhere) and duplicates (defined in two places — e.g. a plugin persona **and** a local `.claude/agents` copy). Keep raw grep output in context-mode; pass the auditor the distilled map.
4. **Run the auditor.** Spawn `harness-auditor` (matching `subagent_type`) with: the target path, the file manifest, the always-loaded-vs-lazy split, the wiring map, and the **ground-truth inventory** (so it validates references against what actually exists, not assumptions). Discovery-only. For a large harness, fan out in parallel by lens (`dedup` / `guardrails` / `context` / `hygiene` / `feedback`) and tell the user you did.
5. **Consolidate.** Merge findings; dedupe by path (keep the highest severity + most specific fix; note when more than one lens flagged the same file).
6. **Verdict.** Derive from severity counts: **CRITICAL** (any HIGH), **NEEDS-ATTENTION** (MEDIUMs only), **HEALTHY** (none). (Not "BLOCK" — a harness audit gates nothing; it advises.)
7. **Log.** Append a one-line entry to `<target>/.claude/personas-harness-log.md`: `YYYY-MM-DD · <verdict> · H:<n> M:<n> L:<n> · areas:<...> · <one-line summary>`.

## Output

Lead with the **verdict** and a small table (lens × HIGH/MED/LOW). Then findings grouped HIGH → MEDIUM → LOW, each `**<path>** [lens] — issue — fix`. Put the single highest-leverage change first. Close with a short **"what's solid"** list. Keep raw config/tool dumps out of the final report — synthesize.

## Notes

- Read-only: this skill and the auditor never edit or commit. Offer to apply fixes (delete an orphan, convert a prose guardrail to a hook, path-scope a rule) only if the user asks afterward.
- Token budget: route every multi-file read, glob, and grep through context-mode; pass the auditor the manifest + maps, not raw file contents.
- Scope line: harness only. Application bugs → `/personas:audit`. UX/visual craft → `/personas:panel`.
