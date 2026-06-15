# claude-personas

A portable repertoire of **expert auditor personas** for Claude Code, distributed as
a plugin so you can run domain critiques and audits in any repo — not just the one
they were written in.

This repo is **both the marketplace and the plugin** (`marketplace.json` →
`"source": "./"`).

## What's in it

**Phase 1 — domain auditors (this release)**

| Persona / skill          | Lens                                                                      |
| ------------------------ | ------------------------------------------------------------------------ |
| `python-fastapi-auditor` | Async correctness, typing, FastAPI patterns, `uv` packaging, test gaps.   |
| `sql-db-auditor`         | Migration safety, RLS coverage + performance, FK indexes, function safety.|
| `security-auditor`       | Authz/IDOR, SSRF, secret/key handling, PII exposure, rate limiting.       |
| `/personas:audit`        | Orchestrator — fans the auditors out at a target, dedupes, emits a verdict.|

These cover the Python/SQL/security surface that's easy to under-review when your
strengths are elsewhere (e.g. front-end). They report HIGH-confidence findings only,
with `file:line` + severity + a concrete fix.

**Phase 2 — embodied/browser personas (planned)**

Port the experiential critics (design-reviewer, consuming-app-engineer,
accessibility-specialist, tech-recruiter) + a `/personas:panel` browser smoketest,
with a web-target binding (URLs).

## Install

From any repo where you want to run audits:

```bash
# add this repo as a marketplace, then install the plugin
claude plugin marketplace add danieljoffe danieljoffe/claude-personas
claude plugin install personas@danieljoffe
```

(Or interactively: `/plugin marketplace add danieljoffe/claude-personas` then
`/plugin install personas@danieljoffe`.)

> **Restart your session after installing.** A plugin's skills and agents register
> at session start, so `/personas:audit` and the `python-fastapi-auditor` /
> `sql-db-auditor` / `security-auditor` subagents won't be available until you do.

## Use

1. Drop a `personas.config.json` in the target repo's `.claude/` (see
   [`docs/binding.md`](docs/binding.md) and `personas.config.example.json`). This is
   the per-repo binding — it points the personas at your code/migrations/tests so
   they stay repo-agnostic.
2. Run:

```
/personas:audit                       # audit the current repo
/personas:audit apps/wyrdfold-api     # audit a subpath
/personas:audit --area db             # one auditor only
```

You get a verdict (BLOCK / NEEDS-ATTENTION / HEALTHY), a severity table, findings
grouped by severity, and a one-line entry appended to
`.claude/personas-audit-log.md`.

The auditors are **read-only** — they never edit or commit. Ask separately if you
want the findings fixed.

## Layout

```
.claude-plugin/
  plugin.json          # plugin manifest
  marketplace.json     # marketplace manifest (self-referential source)
agents/                # the auditor personas (auto-discovered)
skills/audit/SKILL.md  # the /personas:audit orchestrator
docs/binding.md        # how the per-repo binding works
personas.config.example.json
```
