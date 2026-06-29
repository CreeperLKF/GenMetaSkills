# Contributing

## Adding a New Skill

1. Create a directory under `skills/` named with hyphens (e.g., `skills/my-skill/`)
2. Add a `SKILL.md` with YAML frontmatter and markdown body:

```yaml
---
name: my-skill
description: Use when [triggering conditions and context]
---

# Skill Title

[Skill content...]
```

3. Optionally add a `README.md` in the skill directory for human-readable documentation
4. Add supporting files (templates, scripts, references) alongside `SKILL.md` as needed
5. Update the skill table in the root `README.md`

## Skill Format

- `name`: letters, numbers, and hyphens only
- `description`: starts with "Use when...", describes triggering context (not what the skill does), max ~500 characters
- Body: markdown sections scaled to complexity. See existing skills for examples.

## Supporting Files

Place heavy references (100+ lines), templates, or scripts in the skill directory alongside `SKILL.md`. Keep inline anything under ~50 lines.
