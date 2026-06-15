---
name: panel
description: Run the experiential persona critics (accessibility, visual design, UX first-impression) against a live URL via the browser and produce one consolidated read. USE WHEN the user says "run the panel", "persona panel", "critique this site/storybook", or wants an experiential/visual/a11y critique of a live surface.
user-invocable: true
argument-hint: '<url> [--type storybook|site] [--personas a11y,design,ux]'
---

# /personas:panel

Drive a panel of embodied persona critics over a **live web surface** and return one consolidated, in-character read. Experiential/visual/a11y — the browser counterpart to `/personas:audit` (which reads code).

## Inputs

- **URL**: the first argument, else `web.url` / `web.baseUrl` from `<repo>/.claude/personas.config.json`.
- **Binding**: read the `web` block of `personas.config.json` — `viewport` (default `1440x900`), `targetType` (`storybook` | `site`), and `audience` / `goal` / `context` for the UX persona. `--type` overrides `targetType`.
- **`--personas`** (optional): restrict the panel (e.g. `a11y,design,ux`).

## Persona selection

Default panel (works on any target):
- `accessibility-specialist` — WCAG 2.1 AA
- `design-reviewer` — visual craft
- `ux-first-impression` — audience snap-judgment (pass it `audience` / `goal` / `context` from the binding)

The component-library critics `design-systems-engineer` / `consuming-app-engineer` ship in a later release and only apply when `targetType: storybook`.

## Steps

1. **Resolve** URL + binding. Confirm the URL is reachable with a quick `browser_navigate`.
2. **Run the personas SEQUENTIALLY — never in parallel.** Playwright MCP is a single shared browser; concurrent persona subagents fight over one tab and corrupt each other's navigation and findings. Spawn one persona subagent (matching `subagent_type`), let it fully finish, then spawn the next. Pass each: the URL, viewport, `targetType`, and (for `ux-first-impression`) the `audience` / `goal` / `context`.
3. **Consolidate** the in-character verdicts. Group findings by severity; when two personas independently flag the same thing, surface that as a strong signal (do not silently drop it as a duplicate).
4. **Verdict** — an overall read derived from the panel (e.g. ship-ready / needs-work / blocked), keeping each persona's voice in its own section.
5. **Log** a one-line entry to `<repo>/.claude/personas-panel-log.md`: `YYYY-MM-DD · <url> · <personas> · <one-line takeaway>`.

## Output

Lead with the panel verdict, then a section per persona (their voice, ranked findings), then a short cross-cutting "what worked" and the top 3 things to fix. Synthesize — no raw screenshots/snapshots/console dumps.

## Notes

- Read-only: personas never edit or commit.
- Sequential execution is mandatory (shared browser). If you need speed, cut the persona list — do not parallelize.
- Each persona observes the live surface directly, so findings are grounded in what's rendered — keep claims to what was actually seen.
