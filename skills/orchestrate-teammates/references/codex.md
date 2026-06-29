# Codex Adapter

Use this reference when applying `orchestrate-teammates` inside Codex instead of
Claude Code. The orchestration patterns stay the same, but the callable interface and
some runtime guarantees differ.

## Interface mapping

| Skill concept | Claude Code reference implementation | Codex multi-agent interface |
| --- | --- | --- |
| Spawn teammate | `Agent(...)` | `multi_agent_v1.spawn_agent(...)` |
| Address teammate | `name` | returned `agent id`; nickname is optional and not the authority |
| Message teammate | `SendMessage(...)` | `multi_agent_v1.send_input(target: <agent id>, ...)` |
| Wait for result | idle notification | completion notification or `multi_agent_v1.wait_agent(...)` |
| Shut down teammate | structured `shutdown_request` message | `multi_agent_v1.close_agent(target: <agent id>)` |

Use `tool_search` for the current `multi_agent_v1` schema if the tools are not already
loaded.

## Spawn in Codex

Use `spawn_agent` only when the user explicitly asks for subagents, delegation, teammates,
or parallel agent work, or when the active Codex tool policy permits it. Do not infer that a
team is allowed only because the task could be parallelized.

Typical shape:

```text
multi_agent_v1.spawn_agent(
  agent_type: "worker",
  message: "<standalone teammate brief>",
  fork_context: false
)
```

Choose the agent type from the exposed schema:

- `explorer`: read-only, specific codebase questions.
- `worker`: bounded implementation or verification with explicit file/module ownership.
- `default`: general tasks when no sharper role fits.

Do not set `model`, `reasoning_effort`, or `service_tier` unless the user asks or the task
has a concrete reason to override the inherited defaults.

## Context and addressing differences

Codex teammates are addressed by the returned agent id. If you want human-friendly names
like `reviewer-sec`, keep a lead-local map from nickname to agent id; do not assume the
nickname can be used as the message target.

`fork_context` controls history inheritance:

- Omit it or set `false` to match the main skill's "fresh teammate" model. Put all needed
  task context, paths, constraints, and reporting instructions in the prompt.
- Set `true` only when the teammate genuinely needs the current conversation context.

Codex can pass structured `items` as well as plain `message`. Use `items` when handing a
skill, image, connector mention, or other structured reference to the teammate.

## Messaging and completion

Use `send_input` to steer an existing teammate:

```text
multi_agent_v1.send_input(
  target: "<agent id>",
  message: "<instruction>"
)
```

Set `interrupt: true` only when you need the teammate to stop its current direction and
handle the new instruction immediately.

Use `wait_agent` sparingly. Wait only when the lead is blocked on the result; otherwise keep
doing non-overlapping lead work while teammates run. When a teammate finishes but the final
message lacks data, send a follow-up asking for the concrete findings, paths, commands, or
outputs.

Close completed teammates with `close_agent` once their results are integrated so they do
not remain open unnecessarily.

## Codex-specific guardrails

- Give workers disjoint file or module ownership. Tell them they are not alone in the
  codebase and must not revert changes made by others.
- For edit tasks, ask workers to edit in their own workspace and list changed paths in the
  final answer.
- Do not use the lead to bypass a teammate's denied permission. If a teammate reports a
  permission block, surface it instead of laundering the action through the lead.
- Keep the tmux assumptions in the main Claude Code section as-is; this Codex adapter only
  maps Codex's multi-agent API and does not change terminal assumptions for Claude Code.
