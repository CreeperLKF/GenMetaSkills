# nutshell-develop

A reference skill for the NutShell two-layer project structure.

## What is NutShell?

A project organization pattern where:

```
ProjectDev/          # Outer: personal dev wrapper
  Project/           # Inner: public project (submodule)
    docs/
    README.md
    .gitignore
    ...
  docs/              # Dev docs, agent state
  .claude/
  .vscode/
  README.md
  .gitignore
  ...
```

- **Outer layer** holds personal development files (agent docs, editor configs, dev notes) and is for individual developer workflow
- **Inner layer** holds the actual project (source, build configs, user-facing docs) and has its own git remote as a submodule

## Usage

Invoke the skill when setting up a new NutShell workspace or when you need the agent to understand the two-layer conventions. On first use, the skill will verify the directory structure and write a `CLAUDE.md` so the agent stays aware throughout the session.
