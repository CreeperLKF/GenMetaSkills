# meta-skill-check

A skill for vetting whether another skill is general enough to be reused — the admission
gate for a shared skill marketplace.

## What does it check?

Skills are usually distilled from one messy session, so they carry that session's noise:
host-specific assumptions, references to files that only existed there, outputs scoped to
that one task. This skill separates the reusable signal from that noise across five lenses:

```
Candidate skill
   │
   ▼ [static read]            extract assumptions, flag dangling references, judge trigger
   │
   ▼ [isolated dry-run]       spawn a subagent with ONLY the skill + a mock task, watch it
   │                          reach for things that aren't there
   ▼
Per-lens report → Admit | Revise | Reject
```

1. **Assumptions & constraints** — what it silently depends on, and whether that's reasonable
2. **Dangling references** — does it point at files/scripts/tools a clean host won't have
3. **Portability** — survives a new session, workspace, task, and harness (Claude Code ↔ Codex)
4. **Output specificity** — is the result scoped to the task, or over-prescribed (e.g. always writing a file)
5. **Trigger scope** — is the `description` trigger appropriately convergent

## Usage

Invoke it when a skill is the explicit subject of evaluation: "is this skill too specific?",
"will this work in other projects?", "vet this before I add it to the marketplace". The
check reads the skill, runs at least one isolated dry-run on a mocked task context, and
returns a per-lens report with an overall verdict. It prints the report in the conversation
by default and only writes a file if you ask — the same output discipline it judges other
skills for.
