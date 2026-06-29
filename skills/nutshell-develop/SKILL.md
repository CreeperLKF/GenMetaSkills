---
name: nutshell-develop
description: Use when the user asks about project organization, where to put files or docs, or wants to initialize a NutShell-style two-layer development workspace with inner project submodule and outer dev wrapper
---

# NutShell Development Structure

A two-layer project organization where an outer "dev wrapper" repo contains an inner "public project" repo as a submodule.

## Initialization

When invoked, verify the workspace and bootstrap agent awareness:

1. **Check structure** - Confirm the current working directory is the outer layer (should end with `Dev` or similar convention). Verify that:
   - The inner project directory exists (e.g., if CWD is `ProjectDev/`, check `Project/` exists)
   - `docs/` exists in the outer layer
   - If anything is missing, warn the user to set up the structure manually. Do NOT create directories or repos.

2. **Write CLAUDE.md** - If the structure is valid, write a `CLAUDE.md` at the outer repo root (or `AGENTS.md` if not in Claude Code) with this template, substituting actual directory names:

```markdown
# NutShell Structure

This is a NutShell-structured project.

## Layout
- `{OuterName}/` (this repo) — personal dev files: agent docs, editor configs, dev notes
  - `docs/` — developer-authored documentation, agent state documents
  - `{InnerName}/` — public project (submodule, separate remote)

## Conventions
- Outer repo: commit documentation and config updates
- Inner repo: commit source code changes driven by development needs
- Inner `.gitignore` excludes editor files, agent state, and IDE configs
- Place agent-maintained docs in outer `docs/`, user-facing docs in `{InnerName}/docs/`
```

After writing, confirm to the user that the agent is now aware of the structure.

## Structure Reference

### Outer Layer (`xxxDev/`)

The outer repo holds everything related to personal development workflow:

- **`docs/`** — developer notes, architecture decisions, agent-maintained session docs, project state tracking
- **`.claude/`** — Claude Code configuration and memory (gitignored by inner, may be tracked by outer)
- **`.vscode/`** — editor settings (gitignored by inner, optionally tracked by outer)
- **`README.md`** — personal dev notes for this workspace
- **`.gitignore`** — excludes OS files; selectively tracks editor/agent configs for multi-device sync

The outer repo's primary purpose is maintaining documentation versions and enabling multi-device development. What exactly goes in `docs/` is not prescribed, but files should be placed in the correct layer.

### Inner Layer (`xxx/`)

The inner repo holds everything intended for public/shared development:

- Source code, build configs (`package.json`, `Cargo.toml`, etc.)
- User-facing documentation (`README.md`, `docs/` if applicable)
- Project-level `.gitignore` that **excludes** editor files, agent state, IDE configs

The inner repo has its own git remote and is tracked as a submodule by the outer repo. It contains nothing specific to any individual developer's setup.

### Commit Discipline

- **Outer repo** — commit after documentation work: session summaries, updated agent docs, config changes. Example: after a coding session where the agent updates its state docs, commit the outer repo.
- **Inner repo** — commit driven by actual development: features, bug fixes, refactors. The inner repo's commit history should read like a normal project's history.

Both repos have independent commit timelines. A single development session might produce commits in both repos, or only one.
