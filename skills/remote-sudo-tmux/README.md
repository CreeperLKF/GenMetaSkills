# remote-sudo-tmux

A portable skill for remote privileged maintenance through a human-auditable tmux sudo handoff.

## What is Remote Sudo Tmux?

A workflow for cases where an agent needs to run privileged commands on a remote host, but password entry must stay human-controlled and sudo authorization is tied to an interactive TTY.

```
Operator terminal
  local tmux session
    ssh -tt <host>
      remote tmux session
        user runs sudo -v
        agent verifies sudo -n true
        agent continues with tmux send-keys
```

- **Human-controlled authorization** keeps sudo passwords, tokens, private keys, and credential-bearing URLs out of chat
- **Same-TTY execution** keeps follow-up privileged commands in the tmux pane where the user authorized sudo
- **Host overlays** keep aliases, ports, usernames, service paths, proxies, and runbook details in project docs instead of the skill

## Usage

Invoke the skill when remote server work needs sudo and direct non-interactive sudo is unavailable, unsafe, or inappropriate. The skill will first inspect reachability and host tools without changing state, then create a local tmux bridge to a remote tmux session, ask the user to run `sudo -v` in the attached terminal, verify with `sudo -n true`, and continue privileged work in that same pane.

Before using it, read any project runbooks or docs that describe SSH entrypoints, remote sudo, tmux handoffs, or host-specific operating constraints. Treat those docs as overlays on the portable flow.
