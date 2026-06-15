# Per-repo binding

The personas are **repo-agnostic**. What's specific to each repo — where the code
lives, which URL to critique, who the audience is — goes in a small
`personas.config.json` in the consuming repo's `.claude/` directory. This is what
lets one persona work against any repo or surface without edits.

There are two binding sections:

- **`paths` / `stack` / `outOfScope`** — for the code auditors (`/personas:audit`).
- **`web`** — for the experiential browser critics (`/personas:panel`).

Either can be present on its own; a repo with no live UI only needs the code section,
and a static site with no backend only needs `web`.

## Where it goes

```
<consuming-repo>/.claude/personas.config.json
```

If it's absent, `/personas:audit` falls back to inference (Python + `pyproject.toml`
→ python; `supabase/migrations/**` or `*.sql` → db; entry points/auth/config →
security) and tells you what it assumed. A config makes runs deterministic.

## Fields — code auditors (`/personas:audit`)

| Field         | Meaning                                                                 |
| ------------- | ----------------------------------------------------------------------- |
| `stack`       | Hints for which auditors are relevant (e.g. `python`, `fastapi`, `supabase-postgres`). |
| `paths.code`  | Glob(s) for application source the python + security auditors read.      |
| `paths.migrations` | Glob(s) for SQL migrations the db auditor reads.                    |
| `paths.tests` | Glob(s) for test files (test-coverage findings).                        |
| `outOfScope`  | Globs to exclude from every auditor (vendored code, lockfiles, caches). |

## Fields — experiential critics (`/personas:panel`), under `web`

| Field         | Meaning                                                                 |
| ------------- | ----------------------------------------------------------------------- |
| `web.url`     | The URL the browser critics target (deployed Storybook, dev server, or live site). |
| `web.viewport`| Browser viewport, e.g. `1440x900` (default).                            |
| `web.targetType` | `storybook` (a component-library catalog) or `site` (a live web app/page). Steers persona behavior. |
| `web.audience` | For `ux-first-impression`: who the reader is (recruiter, customer, investor…). |
| `web.goal`    | For `ux-first-impression`: what that reader is trying to accomplish.     |
| `web.context` | For `ux-first-impression`: the claim/offer/positioning the site makes, to judge against. |

A `web` block with `targetType: storybook` and no `audience`/`goal` is fine — those
three only feed the UX persona. See `personas.config.example.json` at the plugin root
for a working example.

## Example (wyrdfold)

```json
{
  "stack": ["python", "fastapi", "supabase-postgres"],
  "paths": {
    "code": ["apps/*/app"],
    "migrations": ["supabase/migrations"],
    "tests": ["apps/*/tests"]
  },
  "outOfScope": ["**/node_modules/**", "**/.venv/**", "**/__pycache__/**", "**/*.lock"]
}
```
