# Backend Skills

Agent skills for reviewing and improving backend services: performance, security, and correctness.

Each skill is a self-contained directory under `skills/` with a `SKILL.md` and optional companion files. The repository follows the Agent Skills format, so it can be installed with the [`skills`](https://github.com/vercel-labs/skills) CLI into Claude Code, Codex, Cursor, OpenCode, Gemini CLI, GitHub Copilot, Windsurf, and other compatible agents.

## Install

Requires Node.js 22.20 or later for the CLI.

```bash
# List available skills without installing
npx skills add https://github.com/EkaBachrudin/skills.git --list

# Install interactively
npx skills add https://github.com/EkaBachrudin/skills.git

# Install all three skills globally
npx skills add https://github.com/EkaBachrudin/skills.git --all -g

# Install specific skills into specific agents
npx skills add https://github.com/EkaBachrudin/skills.git \
  -s backend-performance -s backend-security \
  -a claude-code -a opencode
```

SSH works too:

```bash
npx skills add git@github.com:EkaBachrudin/skills.git
```

## Skills

| Skill | Use it for | Report |
| --- | --- | --- |
| `backend-performance` | Slow endpoints, N+1 queries, missing indexes, caching, concurrency, memory/CPU growth, load-test results | `.backend-skills/reports/<run-id>/performance.md` |
| `backend-security` | Authentication, authorization/IDOR, injection, SSRF, secrets, input validation, rate limiting, dependency and trust-boundary issues | `.backend-skills/reports/<run-id>/security.md` |
| `backend-bugs` | Race conditions, error handling, edge cases, transaction boundaries, idempotency, resource leaks, API contract mismatches | `.backend-skills/reports/<run-id>/bugs.md` |

## Example prompts

- "Why is `GET /orders` slow? Profile it and give me the top three fixes."
- "Audit authorization on the invoice endpoints. Check for IDOR and missing tenant checks."
- "Find race conditions and partial-write bugs in the payment flow."

## Safety

- Skills are read-only by default. They report findings; they do not modify code unless you ask.
- Performance load tests and security probes must target an environment you are authorized to test.
- Reports must redact secrets, tokens, personal data, and production identifiers.

## Repository layout

```text
skills/
  <skill-name>/
    SKILL.md          # required: frontmatter + instructions
    WORKFLOW.md       # detailed procedure
    CHECKLIST.md      # review checklist
    EXAMPLES.md       # example findings and report shape
```

## Authoring

See [AGENTS.md](AGENTS.md). The short version:

- `name` in frontmatter equals the directory name.
- `description` uses `>-`, is trigger-rich, and stays roughly under 500 characters.
- `version` is semver.
- `SKILL.md` stays at or under 100 lines; details go into companions.
- Skills are self-contained; no file references outside the skill directory.
- Validate with `npx skills add . --list` before pushing.

## License

Add a license before distributing this repository publicly.
