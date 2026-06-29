---
name: remote-sudo-tmux
description: "Guides remote privileged maintenance through a human-auditable tmux handoff: inspect current host state, start a local tmux bridge to SSH, let the user enter sudo -v in the attached terminal, verify sudo in the same TTY, and continue with tmux send-keys. Use when remote server work needs sudo, direct non-interactive sudo is unavailable or unsafe, the user wants to authorize interactively, or an agent needs a portable pattern for TTY-scoped sudo authorization without asking for passwords in chat."
---

# Remote Sudo Tmux

## Purpose

Use this skill to coordinate privileged remote maintenance while keeping password entry human-controlled and auditable. The user enters the sudo password only inside an attached terminal; the agent continues in the same tmux pane after verifying that sudo is cached for that TTY.

## Core Rules

- Start with read-only reachability, identity, and state checks before privileged commands.
- Never ask the user to paste or reveal a sudo password, token, private key, or credential-bearing URL in chat.
- Treat sudo authorization as TTY-local unless proven otherwise. A successful `sudo -v` in one SSH or tmux pane may not authorize a separate SSH command.
- Keep privileged follow-up in the same tmux-attached remote TTY that the user used for `sudo -v`.
- Prefer short, inspectable commands. For longer operations, write a temporary script, show or checksum it if useful, then run it with `sudo -n`.
- Stop before changing state if the tmux pane is in an editor, at an unclear prompt, already running a command, or attached by another human who has not agreed to the operation.
- Keep host-specific aliases, ports, usernames, proxies, and service paths outside this skill. Load them from the current project docs, SSH config, runbooks, or user prompt.

## Before Sudo

Confirm the target and tools without changing state:

```zsh
ssh -o ClearAllForwardings=yes <host> 'hostname; id; uname -a; command -v tmux || true; command -v sudo || true'
```

Read any project runbooks or docs that describe remote sudo, SSH entrypoints, tmux handoffs, or host-specific operating constraints. Treat those docs as overlays on this portable procedure. For example, a repository might provide `runbooks/remote-sudo-via-local-tmux.md`.

If the task does not require sudo, do it without sudo. If local or remote `tmux` is missing, explain the missing dependency and use another auditable interactive terminal path only after the user agrees.

## Create The Handoff

Choose session names that are unique enough to avoid collisions and meaningful enough for a human to inspect, for example `<task>-<host>` locally and `<task>-handoff` remotely.

Start a local tmux session that owns the SSH connection and attaches to or creates a remote tmux session:

```zsh
tmux new-session -d -s <local-session> \
  "ssh -tt -o ClearAllForwardings=yes <host> 'TERM=xterm-256color tmux new-session -A -s <remote-session>'"
tmux attach -t <local-session>
```

Ask the user to run this command inside the attached terminal:

```zsh
sudo -v
```

Tell the user to confirm when it returns to the shell prompt. Do not ask for or handle the password in chat.

## Verify And Continue

After the user confirms, verify sudo from the same local tmux path:

```zsh
tmux send-keys -t <local-session>:0.0 'sudo -n true && echo SUDO_OK' C-m
```

Inspect the output before running privileged work:

```zsh
tmux capture-pane -p -t <local-session>:0.0 | tail -80
```

Run privileged commands through the same pane:

```zsh
tmux send-keys -t <local-session>:0.0 'sudo -n systemctl status <service> --no-pager' C-m
```

For multi-line operations, create a remote temporary script from inside the same pane, verify it, then execute it:

```zsh
cat >/tmp/<task>.sh <<'EOF'
set -euo pipefail
# commands here
EOF
chmod +x /tmp/<task>.sh
sudo -n /tmp/<task>.sh
```

Send the script block in small enough chunks that the terminal cannot swallow lines. Re-capture the pane before execution if there is any doubt about what was written.

## Failure Handling

- If `sudo -n true` asks for a password or fails with "a password is required", the user likely authorized a different TTY. Reattach to the intended tmux path and ask the user to run `sudo -v` there.
- If direct `ssh <host> 'sudo ...'` fails after tmux authorization, keep using tmux. This is expected on hosts with TTY-scoped sudo tickets.
- If the terminal reports `terminal does not support clear` or similar, recreate the bridge with `TERM=xterm-256color`.
- If the pane contains a partial paste, editor buffer, pager, or unexpected prompt, stop and recover the shell state before sending more commands.
- If a privileged command fails after partial changes, inspect the current state before retrying or rolling back.

## Cleanup

When the operation is complete and no human is still using the sessions, list and remove only the sessions created for this task:

```zsh
tmux list-sessions
ssh -o ClearAllForwardings=yes <host> 'tmux list-sessions'
tmux kill-session -t <local-session>
ssh -o ClearAllForwardings=yes <host> 'tmux kill-session -t <remote-session>'
```

Leave sessions running if they contain useful output, an active long-running job, or a human is still attached. Record the session names in the operation log when cleanup is deferred.
