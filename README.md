# GenMetaSkills

A personal marketplace of Claude Code skills and plugins.

## Skills

| Skill | Description |
|-------|-------------|
| [nutshell-develop](skills/nutshell-develop/) | Reference guide for the NutShell two-layer project structure (outer dev wrapper + inner project submodule) |
| [context-filter](skills/context-filter/) | Distills a messy, historically polluted context into a clean, unbiased prompt or document — keeping core signal, dropping trial-and-error noise |
| [remote-sudo-tmux](skills/remote-sudo-tmux/) | Runs remote privileged maintenance through a human-auditable tmux sudo handoff — the user enters `sudo -v` in an attached terminal, the agent continues in the same TTY |
| [orchestrate-teammates](skills/orchestrate-teammates/) | Coordinates a team of parallel agent sessions ("teammates") under one lead — spawn/message/idle/shutdown mechanics, orchestration patterns, and a Codex adapter |
| [meta-skill-check](skills/meta-skill-check/) | Vets whether a candidate skill is general/portable enough to admit to the marketplace — static read + isolated dry-run across five lenses, ending in an Admit/Revise/Reject verdict |

## Installation

### Automatic (recommended)

Add this repo as a plugin marketplace, then install the plugin from inside Claude Code:

```
/plugin marketplace add https://github.com/CreeperLKF/GenMetaSkills.git
/plugin install gen-meta-skills@gen-meta-skills
```

After installing, the skills become available automatically. Use `/plugin` to manage
or update them later.

### Manual

Clone the repo and reference a skill's `SKILL.md` directly:

```
git clone https://github.com/CreeperLKF/GenMetaSkills.git
```

Then either copy or symlink the skill directory into your Claude Code skills
directory, or point your own plugin configuration at `skills/<skill-name>/`.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a new skill.
