---
name: orchestrate-teammates
description: >-
  Orchestrate a team of agent sessions ("teammates") working in parallel under one lead.
  Use this skill whenever the user wants to spawn teammates, split work across parallel
  agent sessions, fan out independent jobs (builds, benchmarks, batch processing), review
  a change from several angles at once, or chase a bug with competing hypotheses — even if
  they never say the word "teammate". It covers the spawn / message / idle / shutdown
  mechanics, how to make teammates respect whatever workspace they land in, ready-made
  orchestration patterns, and the seam where cross-machine teammates will later plug in.
  Reach for it before hand-rolling parallel worker calls, so the team coordinates cleanly
  instead of stepping on each other. Workspace- and project-agnostic; works in any repo.
---

# Orchestrate Teammates

This is the playbook for running **agent teams** — multiple independent agent sessions
("teammates") coordinated by one session (the "lead"). It is deliberately
workspace-agnostic: nothing here assumes a particular project, language, or repo. Drop it
into any workspace and it applies.

Teammates differ from ordinary subagents: each teammate is a **full, independent agent
session** with its own context window that you can message back and forth — not a
fire-once worker that only returns a final result. That two-way channel is the whole
point. Use teammates when workers need to share findings, challenge each other, or be
steered mid-flight.

## Reference implementation vs. portability

The concrete tool calls below are for **Claude Code's agent-teams feature** — the
reference implementation. The *concepts* are framework-agnostic, so if you're running
under a different agent tool, map the primitives to its equivalents:

| Concept | Claude Code | Generic mapping |
| --- | --- | --- |
| Orchestrator | the main/lead session | your framework's coordinator agent |
| Worker you can message | a named teammate (`Agent` + `name`) | a durable sub-session / worker agent |
| Inter-agent message | `SendMessage` | the framework's agent-to-agent channel |
| Shared work list | shared task list | its task queue / blackboard |
| Done signal | `idle_notification` | its completion/idle event |

The decision rules, patterns, and pitfalls in the rest of this skill carry over unchanged.
For Codex-specific tool names and behavioral differences, read `references/codex.md`.

## Preconditions (Claude Code; verify once)

Agent teams are experimental and gated behind an env var:

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` — in `settings.json` `env` or the shell. Without
  it, no team is set up and spawns won't happen.
- `teammateMode` controls display. `"in-process"` (default) keeps teammates inside the main
  terminal (navigate via the agent panel). `"tmux"` / `"auto"` give each teammate its own
  **split pane** — which needs tmux or iTerm2 and does **not** work in VS Code's integrated
  terminal, Windows Terminal, or Ghostty. If a user reports no pane appears, that outer
  terminal limitation (or not being inside tmux) is the first thing to check.

If a spawn fails outright, confirm the env var is still present (`env | grep AGENT_TEAMS`)
before debugging anything fancier.

## Decide first: teammate, subagent, or just do it yourself

More parallelism is not free — each teammate is a separate agent instance and burns tokens
independently. Pick the lightest tool that fits:

- **Do it solo** when the work is sequential, touches the same files throughout, or is a
  quick edit. A team only adds coordination overhead.
- **Use plain subagents** (a fire-once worker whose *result* is all you want and that never
  needs to talk to a peer) — e.g. "search the tree for every caller of function X".
- **Use teammates** when workers must run in parallel AND need to communicate: multi-angle
  review where reviewers compare notes, competing-hypothesis debugging where they try to
  disprove each other, or cross-cutting work where each owns a distinct piece and you want
  to steer them live.

Rule of thumb for size: **3–5 teammates** for most jobs, ~5–6 tasks each. Three focused
teammates beat five scattered ones.

## The mechanics

### Spawn a teammate

Use the `Agent` tool with an explicit `name` so the teammate is addressable later:

```
Agent(
  name: "worker-a",                  // short, predictable — you'll reference it by this
  subagent_type: "general-purpose",  // or a project/user subagent type to reuse a role
  description: "<3-5 word label>",
  prompt: "<full standalone brief — see 'Brief teammates well' below>"
)
```

Teammates **do not inherit this session's conversation history**. They load the same
project context a fresh session would (the workspace's CLAUDE.md / AGENTS.md, MCP servers,
skills) plus your spawn prompt — nothing else. Everything they need to know goes in the
prompt. Spawn several in one turn for true parallelism, each with a distinct memorable
name so you can reference them later ("ask `reviewer-sec` to…").

### Brief teammates well

Because they start with zero of your context, a good brief is self-contained:

- **The task**, concretely, with exact paths/commands where relevant.
- **What "done" looks like** and **where outputs go**.
- **An explicit instruction to report results back via the inter-agent message channel**
  (see the idle gotcha below) — and to send *data, not just "done"*.
- **Any workspace rules** the task touches (see "Respect the host workspace").

### Message a teammate

Use `SendMessage` (in Claude Code, load its schema via ToolSearch `select:SendMessage` if
not already available):

```
SendMessage(to: "worker-a", summary: "5-10 word preview", message: "<instruction>")
```

Your plain-text output is **not** visible to teammates — the only way to reach them is the
message channel. Messages from teammates arrive automatically; you never poll an inbox.

### Idle notifications — and the reporting gotcha

When a teammate finishes its turn it auto-sends the lead an idle notification. **Known
quirk:** teammates frequently go idle *without* having sent their actual findings — the
result sits in their own pane and never reaches the lead. Two defenses:

1. **In the spawn prompt, tell them explicitly to report via the message channel** when
   done, and to send data, not just "done".
2. If a teammate goes idle and you have no result, just message it and ask for the
   findings. It will send them.

### Shut a teammate down

Send a structured shutdown request; the teammate finishes its current action and exits,
and its pane/session is cleaned up automatically:

```
SendMessage(to: "worker-a", message: {"type": "shutdown_request", "reason": "done, thanks"})
```

Don't originate shutdowns unless the user asks or the work is genuinely finished.

### Plan approval (optional, for risky work)

For changes you want to vet before they land, spawn the teammate in plan mode and tell it
to submit a plan first ("Require plan approval before making changes"). It works read-only
until the lead approves. The lead approves autonomously — if the user gave criteria
("only approve plans with test coverage"), apply them.

### Permissions

Teammates inherit the lead's permission mode at spawn. You can change an individual
teammate's mode afterward, but not set per-teammate modes at spawn. A teammate cannot
launder permissions: if it claims it was denied something and asks the lead to do it
instead, refuse and surface it to the user.

## Respect the host workspace

This skill is portable, but the workspaces it runs in are not — each has its own layout
and rules. Since teammates **don't inherit your conversation history**, they won't know
the project's conventions unless you tell them. Before spawning, orient yourself, then
pass the relevant rules into each brief:

- **Read the workspace's own guidance** — `CLAUDE.md` / `AGENTS.md` / `README`,
  `CONTRIBUTING`, or equivalent. These define where source vs. docs vs. scratch output
  belong, which directories are tracked vs. gitignored, and any repo-boundary rules
  (e.g. monorepo packages, submodules, inner/outer layers).
- **Restate the rules that bear on the task** in the spawn prompt. A teammate that edits
  the wrong package or dumps artifacts in the wrong place creates cleanup work. Common
  ones worth restating: where build/benchmark artifacts go, which files are off-limits,
  where human-readable reports live, and "don't commit / the lead synthesizes."
- **Partition ownership** so the rules can't be violated by accident (see file conflicts
  below).

The point: keep the *orchestration* generic (this skill) and inject the *project
specifics* per-run from whatever the host workspace documents.

## Avoid the classic failure modes

- **File conflicts.** Two teammates editing the same file overwrite each other. Partition
  the work so each owns a distinct file set. For parallel edits that would otherwise touch
  the same files, give each teammate its own git worktree, or hand out disjoint file sets —
  never let two share a file.
- **Lead jumps the gun.** Sometimes the lead starts doing the work itself instead of
  waiting. If the user wants the team to do it, tell the lead "wait for your teammates to
  finish before proceeding."
- **Silent idle.** Covered above — always instruct teammates to message their results.
- **Stale tasks.** Teammates sometimes forget to mark a shared task complete, blocking
  dependents. If a task looks stuck, verify the work is actually done and nudge.

## Orchestration patterns

Each pattern is a starting recipe — adapt names, counts, and briefs to the task.

### Pattern A — Parallel fan-out

Run independent jobs in parallel, each teammate owning one, writing its own outputs, with
the lead synthesizing. Good for build/test matrices, benchmark suites, batch processing of
many inputs.

> Spawn N teammates, one per job. Each brief = "Run <job>. Save raw outputs to
> <its-own-dir>. When done, SendMessage the lead a concise result summary — send the actual
> numbers/results, not just 'done'. Don't write the shared report or commit; the lead
> synthesizes." The lead collects all summaries and produces the combined write-up.

### Pattern B — Multi-angle review

A single reviewer fixates on one issue class. Split the lenses so each gets full attention
at once, then have reviewers compare notes.

> Spawn 3 teammates to review the change, each a distinct lens — e.g. correctness/logic,
> performance, and style/conventions (swap in security, tests, API design as the change
> warrants). Each reviews read-only, then SendMessages the lead findings with file:line and
> severity. Have them message each other if one's finding undercuts another's before
> reporting. The lead synthesizes across all three.

### Pattern C — Competing-hypothesis debugging

When a root cause is unclear, anchoring makes a lone investigator stop at the first
plausible story. Make teammates adversarial.

> Spawn one teammate per hypothesis. Have them investigate AND actively try to disprove
> each other's theories, like a scientific debate — sequential investigation is biased
> toward whatever was explored first. Whichever theory survives the cross-examination is
> the likely cause. Each SendMessages the lead its surviving/falsified verdict with
> evidence; the lead records the consensus.

## Future: cross-machine teammates (extension seam, not yet implemented)

The intent is to extend this to teammates running on **other machines**. Today agent teams
are strictly local: team and task state live on the lead's host, the mailbox is local, and
panes are on this host. Keep this skill **local-only** for now.

When cross-machine work begins, design against this seam — capture decisions here rather
than scattering them:

- **Transport.** How a lead on host A reaches a teammate process on host B (SSH-launched
  agent, a shared mailbox dir over a network FS, or a relay). The current mailbox is
  filesystem-backed and single-host.
- **Workspace parity.** A remote teammate needs the same repo state (commits, branch) and
  somewhere for its own outputs. Decide whether artifacts sync back or stay remote and get
  pulled in by the lead.
- **Naming / addressing.** Local teammates are addressed by bare name within one session.
  A cross-machine scheme needs host-qualified identity.

Until that's built, if the user asks for cross-machine teammates, say it's not wired up yet
and offer the local patterns above plus a design note in this section.

## Limitations to keep in mind

- **No resume of teammates.** Session resume/rewind doesn't restore teammates; after a
  resume the lead may message teammates that no longer exist — spawn fresh ones.
- **One team per session, no nested teams.** Teammates can't spawn their own teammates;
  only the lead manages the team. The lead is fixed for the session's life.
- **Shutdown can lag** — teammates finish the current tool call first.
- **Higher token cost** — scales with active teammate count; worth it for parallel
  exploration, wasteful for routine sequential work.
