# orchestrate-teammates

A reference skill for coordinating multiple agent sessions as one working team.

## What are teammates?

Teammates are independent agent sessions coordinated by a lead session:

```
Lead session
  teammate-a     # owns one job, hypothesis, or review lens
  teammate-b     # works in parallel with its own context window
  teammate-c     # reports findings back through the message channel
```

- **Lead session** orients on the workspace, partitions work, briefs teammates, and synthesizes results.
- **Teammates** run in parallel, own distinct work, and can be steered mid-flight through an inter-agent message channel.
- **Workspace rules** still come from the host repo, such as `CLAUDE.md`, `AGENTS.md`, `README`, or `CONTRIBUTING`.

## Usage

Invoke the skill when a task benefits from coordinated parallel agent work: fan-out builds or benchmarks, multi-angle review, competing-hypothesis debugging, or cross-cutting implementation with clear file ownership.

The main `SKILL.md` uses Claude Code agent teams as the reference implementation. For Codex, read `references/codex.md` for the corresponding `multi_agent_v1` interface and behavior differences.
