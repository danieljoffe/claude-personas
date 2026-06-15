## What this changes

A short description of the persona/skill added or changed and the lens it
applies.

## Checklist

- [ ] Persona stays **read-only** (no edits/writes/commits) and **repo-agnostic**
      (no hard-coded paths/URLs/stacks — repo specifics go in the binding).
- [ ] Findings are high-confidence and actionable (`file:line` + severity + fix).
- [ ] If it joins a default panel, the relevant skill (`audit`/`panel`) is wired up.
- [ ] `version` bumped in `.claude-plugin/plugin.json` if plugin behavior changed.
- [ ] Conventional Commit messages (`feat(...)`, `fix(...)`, `docs:`).

## How I tested

The command(s) run and the kind of repo/surface tested against. Note any
false-positive checks on known-good code.
