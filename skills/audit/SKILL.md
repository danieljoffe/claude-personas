---
name: audit
description: Run the expert auditor personas (Python/FastAPI, SQL/Postgres, security) across a target repo and produce a consolidated audit. USE WHEN the user says "audit this repo", "critique the codebase", "run the personas", or wants a domain audit of an existing codebase.
user-invocable: true
argument-hint: '[target-path] [--area python|db|security]'
---

# /personas:audit

Orchestrate the auditor personas over an existing codebase and return one consolidated, deduplicated report. Whole-codebase audit (not a PR diff).

## Inputs

- **Target**: the path argument, else the current working directory.
- **Binding**: read `<target>/.claude/personas.config.json` (see this plugin's `personas.config.example.json`). It declares `stack`, `paths` (`code`, `migrations`, `tests`), and `outOfScope`. If absent, infer: Python files + `pyproject.toml` → python; `supabase/migrations/**` or `*.sql` → db; entry points / auth / config / URL-fetching → security. Tell the user which binding you used.
- **`--area`** (optional): restrict to a single auditor.

## Steps

1. **Scope.** Resolve the binding. Build a file manifest of in-scope files (respect `outOfScope`). Use context-mode `ctx_batch_execute` for the listing/globs so raw output stays off-prompt. If the manifest is large (>~400 files), report what you're sampling and why — never silently truncate.
2. **Route.** Map files to auditors by category:
   - `python-fastapi-auditor` ← `paths.code` + `paths.tests` (`.py`)
   - `sql-db-auditor` ← `paths.migrations` + `*.sql` + DB-access code
   - `security-auditor` ← `paths.code` (entry points, auth, config, URL-fetching, env handling)
3. **Fan out in parallel.** Spawn the selected auditors as subagents in a single batch (use the matching `subagent_type`). Give each: the target path, its assigned file list, and the binding. Discovery-only.
4. **Consolidate.** Merge findings; dedupe by `file:line` (keep the highest severity + most specific fix; note when multiple auditors flagged the same line).
5. **Verdict.** Derive an overall verdict from severity counts: **BLOCK** (any HIGH), **NEEDS-ATTENTION** (MEDIUMs only), **HEALTHY** (none).
6. **Log.** Append a one-line entry to `<target>/.claude/personas-audit-log.md`: `YYYY-MM-DD · <verdict> · H:<n> M:<n> L:<n> · areas:<...> · <one-line summary>`.

## Output

Lead with the **verdict** and a severity table (area × HIGH/MED/LOW). Then findings grouped HIGH → MEDIUM → LOW, each `**<file>:<line>** [area] — issue — fix`. Close with a short "what's solid" list. Keep raw source/tool dumps out of the final report — synthesize.

## Notes

- Read-only: this skill and its auditors never edit or commit. Offer to apply fixes afterward only if the user asks.
- Token budget: route every multi-file read and glob through context-mode; pass auditors file lists + the binding, not file contents.
