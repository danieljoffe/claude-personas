# Per-repo binding

The auditor personas are **repo-agnostic**. What's specific to each repo — where the
code lives, where migrations live, what's out of scope — goes in a small
`personas.config.json` in the consuming repo's `.claude/` directory. This is what
lets one persona audit any repo without edits.

## Where it goes

```
<consuming-repo>/.claude/personas.config.json
```

If it's absent, `/personas:audit` falls back to inference (Python + `pyproject.toml`
→ python; `supabase/migrations/**` or `*.sql` → db; entry points/auth/config →
security) and tells you what it assumed. A config makes runs deterministic.

## Fields

| Field         | Meaning                                                                 |
| ------------- | ----------------------------------------------------------------------- |
| `stack`       | Hints for which auditors are relevant (e.g. `python`, `fastapi`, `supabase-postgres`). |
| `paths.code`  | Glob(s) for application source the python + security auditors read.      |
| `paths.migrations` | Glob(s) for SQL migrations the db auditor reads.                    |
| `paths.tests` | Glob(s) for test files (test-coverage findings).                        |
| `outOfScope`  | Globs to exclude from every auditor (vendored code, lockfiles, caches). |

See `personas.config.example.json` at the plugin root for a working example.

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
