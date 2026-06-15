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

**Phase 2 — experiential/browser personas (this release: the universal three)**

| Persona / skill          | Lens                                                                      |
| ------------------------ | ------------------------------------------------------------------------ |
| `accessibility-specialist` | Deep WCAG 2.1 AA — keyboard, focus management, ARIA, contrast.          |
| `design-reviewer`        | Visual craft — spacing rhythm, type scale, token taste, motion, cohesion. |
| `ux-first-impression`    | A configurable audience's snap first-pass — friction, bounce, advance/pass.|
| `/personas:panel`        | Orchestrator — runs the critics against a live URL (sequentially), synthesizes. |

These drive a real browser (Playwright MCP) against a live URL and return in-character
verdicts. They run **sequentially** — Playwright is one shared browser.

**Phase 2 (still to come):** the two component-library critics — `design-systems-engineer`
(API/system credibility) and `consuming-app-engineer` (adoption/DX) — which apply when
`targetType: storybook`.

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
> at session start, so the `/personas:audit` and `/personas:panel` skills and their
> subagents won't be available until you do.

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

For an experiential critique of a live surface (needs the `web` block in the config,
and Playwright MCP connected):

```
/personas:panel https://ui.example.com --type storybook
/personas:panel http://localhost:3000 --type site
/personas:panel --personas a11y,design          # subset, URL from the binding
```

You get a panel verdict plus a section per persona in their own voice, and a one-line
entry in `.claude/personas-panel-log.md`.

Every persona is **read-only** — they never edit or commit. Ask separately if you
want the findings fixed.

## Layout

```
.claude-plugin/
  plugin.json          # plugin manifest
  marketplace.json     # marketplace manifest (self-referential source)
agents/                # all personas (auto-discovered)
  python-fastapi-auditor.md  sql-db-auditor.md  security-auditor.md   # code auditors
  accessibility-specialist.md  design-reviewer.md  ux-first-impression.md  # browser critics
skills/
  audit/SKILL.md       # the /personas:audit orchestrator (reads code)
  panel/SKILL.md       # the /personas:panel orchestrator (drives the browser)
docs/binding.md        # how the per-repo binding works
personas.config.example.json
```
