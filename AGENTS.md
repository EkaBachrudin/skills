# AGENTS.md

Authoring guide for this repository. Each skill is a self-contained directory under `skills/`.

## Layout

```text
skills/<skill-name>/
  SKILL.md          # required
  WORKFLOW.md       # optional companion
  CHECKLIST.md      # optional companion
  EXAMPLES.md       # optional companion
```

## Rules

- `name` in frontmatter must equal the directory name, lowercase kebab-case.
- `version` is semver; start new skills at `1.0.0` and bump on every behavior change.
- `description` must use the `>-` folded block scalar. Front-load capability, trigger phrases, and sibling disambiguation. Target 200-500 characters; never exceed 1024.
- `SKILL.md` should stay at or under 100 lines. Move detail into companions.
- Link companions with relative paths from `SKILL.md`.
- Skills are self-contained. Do not reference files outside the skill directory.
- Do not add `SKILL.md` at the repository root; it can shadow the skill catalog.
- Keep examples, drafts, and archived skills outside `skills/`.
- Reports belong under `.backend-skills/reports/<run-id>/`, never inside the installed skill directory.
- Skills are read-only by default. Destructive tests, load tests, or security probes require explicit authorization.
- No fabricated package names, API endpoints, or benchmark numbers. Mark unverified claims as unverified.

## Validation

```bash
npx skills add . --list
```

The output must list exactly the intended skills. Fix any skipped-skill warnings before committing.
