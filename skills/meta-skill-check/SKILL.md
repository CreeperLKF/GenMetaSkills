---
name: meta-skill-check
description: >-
  Vet whether a candidate skill is general and portable enough to be reused beyond the
  context it was born in — the gate before admitting a skill to a shared marketplace or
  repo. Skills are usually distilled from messy production sessions and quietly carry
  host-specific assumptions, dangling references to files/scripts/tools that won't exist
  elsewhere, over-prescribed outputs (e.g. always writing a file when the task didn't ask
  for one), or triggers that fire too broadly. Use this skill whenever the user asks to
  check, vet, evaluate, or "battle-test" a skill, judge whether a skill is too specific or
  reusable enough, assess cross-session / cross-workspace / cross-task / cross-harness
  (Claude Code ↔ Codex) portability, review a skill before adding it to a marketplace, or
  asks "will this skill work for other people or projects". It reads the skill statically
  AND runs an isolated dry-run via a subagent on a mocked task context, then judges five
  lenses and returns a per-lens report ending in an overall Admit / Revise / Reject
  verdict. Convergent trigger: this is a deliberate review step, so reach for it when a
  skill is explicitly the subject of evaluation — not during ordinary skill authoring.
---

# Meta Skill Check

A skill for judging whether *another* skill is general enough to be reused. Use it as the
admission gate for a shared skill marketplace, or any time someone needs to know whether a
skill will survive being dropped into a context other than the one it was written in.

## Why this exists: skills inherit the noise of their birth

A skill is almost always distilled from one real, messy session. The session had a
specific host, a specific repo layout, specific tools loaded, and a specific problem half-
solved through trial and error. When that gets crystallized into a skill, the genuinely
reusable **signal** (the procedure, the judgment, the failure modes) arrives fused to
incidental **noise** from that one context: a hard-coded path, a script that only existed
in that repo, an assumption that `tmux` is present, a habit of writing a report file
because *that* task wanted one.

The noise is invisible from inside the originating context — everything it references
*does* exist there. It only bites when the skill runs somewhere else. Your job is to find
that noise before the skill is admitted, and decide whether it can be cleaned out (Revise)
or whether the skill is inseparable from its origin (Reject). This is the same signal-vs-
noise lens a clean-context filter applies to documents, pointed at skills.

## The five lenses

Judge a candidate against these. Each maps to a concrete failure the skill could cause
when reused.

| # | Lens | What you're really asking | Red flags |
|---|------|---------------------------|-----------|
| 1 | **Assumptions & constraints** | What does the skill silently assume about the environment, tools, paths, prior state? Are those assumptions reasonable, stated, and overridable? | Unstated dependence on a tool/OS/shell; baked-in hostnames, ports, usernames, absolute paths; assumes prior steps already happened |
| 2 | **Dangling references** | In a *clean* context with only those assumptions, does the skill point at things that won't exist? | References a `scripts/foo.py`, runbook, alias, or service that lives only in the origin repo; cites a file it never creates |
| 3 | **Portability** | Does it survive a new session, workspace, task, and harness (Claude Code ↔ Codex)? | Hard-codes one harness's tool names with no generic mapping or adapter; relies on this session's conversation history; only works on the one repo it came from |
| 4 | **Output specificity** | Is the produced result scoped to what the task needs, or over-prescribed? | *Always* writes a file / commits / opens a PR when the task didn't ask; forces a rigid template onto tasks that don't fit it; side effects the user didn't request |
| 5 | **Trigger scope** | Is the `description` trigger appropriately convergent? | Fires on broad everyday phrasing it shouldn't own; or so narrow it never fires; keyword-matches adjacent tasks that need something else |

A skill does not need to be perfect on all five. The point is to surface each issue with
evidence and a fix, then judge whether the whole is admittable.

## How to run a check

### Step 0 — Read the skill

Read the candidate's `SKILL.md` (and any `references/`, `scripts/`, `assets/`). Note its
declared `name`, `description`, and what it claims to do. Keep file:line handy — findings
must cite where in the skill the issue lives.

**Include the README in scope.** If the skill ships a `README.md` (or other bundled docs),
read and judge it under the same five lenses, not just as background. Documentation drifts
from the skill it describes and quietly introduces its own assumptions, conditions, and
constraints — install steps, an assumed directory layout, example commands, claims about
what the skill produces. A README that says "run `make test`" or "this lives in your
monorepo's `packages/` dir" is asserting an environment exactly the way the SKILL.md body
does, and it can dangle or over-constrain on its own. Cite README findings with the same
file:line discipline, and flag any place the README and SKILL.md disagree.

### Step 1 — Static pass

Go lens by lens *by reading*. Extract the assumption set explicitly (write it down: "this
skill assumes X, Y, Z"). Flag every concrete path, tool name, env var, or sibling-file
reference and ask whether a fresh host would have it. Much of lenses 1, 2, and 5 can be
settled here. Portability across harnesses (lens 3) is also largely a static call — look
for Claude-Code-only tool names with no generic mapping or adapter file.

### Step 2 — Isolated dry-run (the decisive test)

Static reading catches what you can predict. The dry-run catches what you can't: it shows
how a real agent **behaves** when handed only the skill and a task, with none of the
origin context to lean on. This is where dangling references and bad assumptions reveal
themselves, because the agent will reach for things that aren't there.

The mechanism leans on a property of subagents: **they don't inherit this conversation's
history.** A freshly spawned subagent loads only the host workspace's own context plus the
prompt you give it — exactly the "clean context" you want to test against. So:

1. **Construct a mock task context.** Write a realistic user request that *should* trigger
   the skill — the kind of thing a real user in a different project would type. Make it
   concrete but do **not** smuggle in the origin repo's files or conventions. If the skill
   needs some workspace to act on, create a minimal throwaway sandbox dir (empty, or
   stubbed with only what a generic project would plausibly have) — deliberately *not*
   pre-stocked with the production files the skill might assume.

2. **Spawn an isolated subagent.** Give it: the path to the candidate skill, the mock task
   prompt, and the sandbox working directory. Tell it plainly that this is a **dry run** —
   before spawning, explicitly select the runtime's isolated-context mode (for example,
   `fork_context: false` when that option exists) so it does not inherit this conversation.
   it should carry out the task as it normally would so its behavior is observable, but
   work only inside the sandbox and avoid irreversible or outward-facing side effects (no
   real network calls to production, no destructive commands). Ask it to **report what it
   did, what it reached for, and anything it expected to exist but couldn't find.**

3. **(Optional) Run a second, divergent dry-run** to probe portability directly — a
   different mock workspace (different layout, missing the tool the skill likes, or a task
   at the edge of the trigger). One clean happy-path run is the minimum; a divergent run
   sharpens lens 3.

4. **Capture the transcript and outputs**, not just the final answer. The interesting
   evidence is in the middle: the file it tried to read that didn't exist, the assumption
   it stated aloud, the report file it wrote unprompted.

> If subagents aren't available (e.g. running under a harness without them), fall back to
> a static-only check and say so explicitly — note that the dynamic evidence is missing
> and the verdict is lower-confidence.

### Step 3 — Judge from behavior, then write the report

Re-walk the five lenses, now armed with both the static reading and what the agent
actually did. Behavior outranks intent: if the skill *says* it's portable but the dry-run
agent immediately tried to `cat runbooks/deploy.md` that doesn't exist, lens 2 fails on
the evidence. Quote the transcript.

## Output format

Default to printing the report in the conversation. Do **not** write it to a file unless
the user asks — eat our own dog food on lens 4. Use this shape:

```markdown
# Meta-Skill Check: <skill-name>

## Verdict: Admit | Revise | Reject
<one sentence: the core reason>

## Lens findings
### 1. Assumptions & constraints — Pass | Concern | Fail
- <finding with file:line and, where relevant, dry-run evidence>
### 2. Dangling references — Pass | Concern | Fail
- ...
### 3. Portability (session / workspace / task / harness) — Pass | Concern | Fail
- ...
### 4. Output specificity — Pass | Concern | Fail
- ...
### 5. Trigger scope — Pass | Concern | Fail
- ...

## Recommended fixes
1. <concrete, minimal edit that would lift a Concern/Fail — quote the before/after where useful>

## Dry-run evidence
- Run A (clean context): <what the agent did; key moments>
- Run B (divergent), if run: <...>
```

## Verdict rubric

- **Admit** — no Fail; at most minor Concerns. The skill's signal is cleanly separable
  from its origin and it will plausibly work elsewhere as written.
- **Revise** — one or more issues, but each has a concrete, local fix (state an assumption,
  delete a dangling reference, make a file-write conditional, tighten or loosen the
  trigger). This is the common, healthy outcome. List the fixes so the author can act.
- **Reject** — the skill's value is inseparable from its originating context; making it
  general would mean rewriting it into a different skill. Say what the irreducible coupling
  is, so the decision is legible.

## Keep this skill honest

Two habits keep the check from rotting:

- **Cite evidence, don't assert.** Every Concern/Fail should point at a line in the skill
  or a moment in the dry-run. A verdict with no evidence is just an opinion.
- **Don't over-prescribe in your own output.** This skill judges other skills for forcing
  unnecessary file writes and rigid templates; it would be ironic to do the same. Print the
  report, scale its length to the findings, and only write files when asked.
