---
name: plan-mode
description: Produce reviewable implementation plans without making changes. Use when operating in Plan Mode, when a user asks for planning before implementation, or when the active environment requires read-only analysis and approval before execution.
---

# Plan Mode (Read-Only)

Understand the requested outcome and produce a clear, reviewable plan. Do not execute the proposed changes while in this mode.

## Safety boundary

- Treat Plan Mode as strictly read-only. Never create, edit, move, rename, or delete files, or otherwise mutate the workspace or external state.
- Do not run installs, builds that write files, migrations, commits, pushes, deployments, or other state-changing commands. Avoid commands with uncertain side effects.
- You may include short illustrative code snippets in the plan, but never apply them.
- If asked to implement, explain that you are in Plan Mode and can prepare or revise the plan. Execution may begin only after the user approves and the mode is switched to an execution-capable mode.
- When uncertain whether an action is read-only, do not perform it; describe it as a proposed step instead.

## Allowed discovery

Read relevant files; list directories; search code and symbols; inspect configuration, dependencies, tests, documentation, repository status/history/diffs; and ask concise clarifying questions. Use read-only commands only. Do not infer that a command is safe solely from its name—consider its effects.

## Workflow

1. **Understand:** Restate the requested outcome in one or two sentences.
2. **Explore:** Inspect relevant source, tests, configuration, and documentation. Identify current behavior, constraints, and conventions. Keep exploration focused.
3. **Clarify:** If ambiguity or a meaningful trade-off affects the plan, ask concise questions before finalizing it. Otherwise state reasonable assumptions.
4. **Plan:** Provide the structure below. Make steps ordered, concrete, and independently reviewable where practical. Reference exact paths and symbols found during exploration.
5. **Request approval:** End by asking the user to approve, revise, or reject the plan. Do not begin implementation.

## Required plan format

### Goal
One or two sentences describing the desired outcome.

### Context
Summarize relevant files, current behavior, constraints, and existing conventions discovered during read-only exploration.

### Proposed steps
Provide an ordered numbered list. For every step include:
- **What:** the proposed change.
- **Where:** the affected file paths, modules, or systems.
- **Why:** rationale where it is not self-evident.
- **Priority:** high, medium, or low when order is flexible.

### Risks and open questions
Identify likely regressions, breaking changes, edge cases, assumptions, and decisions that require the user's input. If none, say so briefly.

### Testing and verification
Describe checks to run after approval, including relevant tests and manual validation. Do not run any check in Plan Mode if it can write or mutate state.

### Out of scope
List what the plan intentionally excludes, or state that nothing material is excluded.

## Style

Be concise and specific. Avoid generic filler. Prefer incremental steps that are easy to review and verify. Use exact file paths and symbol names when available. Do not invent repository findings; label unknowns and assumptions clearly.

## Client plan updates (ACP)

If the client supports ACP plan session updates, send the plan as a `plan` update with one entry per proposed step, and set every entry to `pending` while planning. Include the complete entry list on every update because the client may replace the whole plan. If feedback changes the plan, send a new complete update. Do not claim to have sent an update if the capability is unavailable.

## Exit criteria

Remain in Plan Mode until the user approves the plan and the environment explicitly switches to an execution-capable `agent` mode. Approval alone does not authorize execution while the mode remains read-only. Until both conditions are met, do not execute any proposed step.
