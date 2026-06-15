# Contributing to claude-personas

Thanks for your interest in contributing. This repo is a Claude Code plugin: a
set of **read-only expert personas** (agents) plus two orchestrating skills
(`/personas:audit`, `/personas:panel`). Contributions that add a useful persona,
sharpen an existing one, or improve the orchestration are all welcome.

## Ground rules for personas

Every persona must stay true to the design that makes this plugin trustworthy:

- **Read-only.** Personas never edit, write, or commit. They discover and report.
- **Repo-agnostic.** Anything repo-specific lives in the consuming repo's
  `.claude/personas.config.json` binding — never hard-code paths, URLs, or
  stacks into a persona. See [`docs/binding.md`](docs/binding.md).
- **High-confidence findings only.** Auditors report findings they're confident
  about (>80%) and skip nits a linter/formatter already covers. A wrong
  "looks fine" is worse than a missed nit.
- **Actionable output.** Each finding is `file:line` + severity + a one-line
  *why it matters* + a concrete fix.

## Repo layout

See the [Layout section of the README](README.md#layout). In short:

```
agents/      # one Markdown file per persona (auto-discovered)
skills/      # the /personas:audit and /personas:panel orchestrators
docs/        # binding docs
```

## Adding a new persona

1. Create `agents/<your-persona>.md` with YAML frontmatter:

   ```markdown
   ---
   name: my-auditor
   description: One-line summary of the lens this persona applies.
   ---

   # My Auditor

   You are a <role with credible expertise>...
   ```

2. Follow the structure the existing personas use (read
   [`agents/python-fastapi-auditor.md`](agents/python-fastapi-auditor.md) as the
   reference):
   - **Voice / role** — who this expert is and what they care about.
   - **What to check** — the concrete checklist, grouped by area, with
     guardrails that prevent false positives.
   - **Protocol** — scope boundaries, read-only reminder, how to read files.
   - **Output** — grouped findings, `file:line` + severity + fix, a closing
     verdict.

3. If the persona should be part of an orchestrator's default panel, wire it
   into the relevant skill (`skills/audit/SKILL.md` for code auditors,
   `skills/panel/SKILL.md` for browser critics). Browser critics run
   **sequentially** — Playwright is one shared browser.

## Editing a skill

Skills are Markdown with frontmatter (`name`, `description`, `user-invocable`,
`argument-hint`). Keep the `description`'s `USE WHEN ...` clause accurate — it's
how the skill gets matched to a request.

## Testing locally

Install your working copy as a local marketplace instead of the GitHub source:

```bash
# from a repo where you want to try the personas
claude plugin marketplace add /path/to/your/claude-personas
claude plugin install personas@danieljoffe
```

**Restart your Claude Code session after installing or after changing an
agent/skill** — plugins register their skills and agents at session start.

Then exercise it against a real (or sample) repo:

```
/personas:audit                  # code auditors
/personas:panel http://localhost:3000 --type site   # browser critics
```

Verify your persona produces grounded `file:line` findings and no false
positives on a known-good codebase.

## Pull requests

- Keep PRs small and focused — one persona or one concern per PR.
- Use Conventional Commit messages, matching the existing history
  (`feat(panel): ...`, `fix(python-auditor): ...`, `docs: ...`).
- Bump `version` in [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json)
  when you change plugin behavior.
- Describe what you tested and against what kind of repo/surface.

## Questions

Open an issue — bug reports and new-persona proposals both have templates under
[`.github/ISSUE_TEMPLATE`](.github/ISSUE_TEMPLATE).
