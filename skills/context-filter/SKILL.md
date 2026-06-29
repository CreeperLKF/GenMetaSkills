---
name: context-filter
description: Use when the user asks to perform a "clean delivery", create an "unbiased start", filter out conversational noise from a document, or synthesize a clean, objective prompt/state for a new agent without historical trial-and-error baggage.
---

# Context Filter (Meta-Skill)

A meta-skill designed to perform "Clean Delivery" and establish an "Unbiased Start." Its purpose is to take a messy, historically polluted context (or document) and distill it into a clean, objective format without losing critical architectural constraints.

## Core Philosophy: Signal vs. Noise

During iterative development, context naturally accumulates noise: trial-and-error paths, local runtime hacks, and overly specific negative constraints ("don't do X"). When transitioning this context to a final document or passing it to a new agent, this noise acts as an anti-pattern, forcing the reader to think about rejected paths. 

Your objective is to identify and filter this noise, leaving only the true signal.

### 1. Identify and Filter Noise (What to Remove)
- **Ephemeral Workarounds:** Ignore steps taken to bypass transient bugs, local environment quirks, or accidental situations that are not meant to be systematically avoided.
- **Resolved States:** If the system has naturally converged to a steady state where a previously encountered "corner case" is no longer possible by design, remove the warnings about it.
- **Optional/Runtime Variables:** Remove negative prompts against specific tools or conditions if they were merely optional choices or specific to the previous user's hyper-local setup.

### 2. Preserve the Baseline (What to Keep)
- **Fundamental Constraints:** Retain negative prompts ONLY if the system has *not* reached a steady state regarding that issue, and a new agent/developer would naturally (but incorrectly) attempt that doomed path without explicit instruction (e.g., hard API limitations, immutable framework restrictions).
- **Core Domain Logic:** Keep the objective requirements, the final decided architecture, and the ultimate goals.

## Execution Protocol

When invoked to perform a context filter, execute the following workflow abstractly:

1. **Analyze:** Review the target context, document, or prompt. Map out the established absolute rules versus the developmental journey (the trial and error).
2. **Decouple:** Separate universal requirements from situational history. 
3. **Abstract & Rewrite:** Rewrite the target from a higher-level, objective perspective. Frame instructions around *what to do and how the system works*, strictly limiting *what not to do* to fundamental constraints.
4. **Deliver:** Provide the clean, unpolluted output. This could be a synthesized prompt for a subagent, a refactored clean document, or an actionable plan for reorganizing information.

*Note: Execute this skill concisely. Do not over-explain the filtering process to the user unless asked. Simply apply the philosophy and deliver the requested clean state based on the current contextual goal.*
