---
name: vibe-coding-planner
description: >
  Generates a structured build reference plan for AI-assisted (vibe) coding sessions.
  Use this skill whenever a user wants to plan, blueprint, or map out a software project
  before writing code. Trigger when the user says things like "I want to build X", "help me
  plan this app", "what should I build first", "create a build plan", "I'm about to start
  coding", "help me structure my project", "vibe coding plan", or describes a software idea
  and wants direction. Also trigger when the user wants to update their build plan after
  completing a phase, or when they are about to continue coding on an existing project and
  need to recover context. This skill runs in three modes: Init (new project), Update (phase
  complete), and Review (resume a session from an existing plan).
---

# Vibe Coding Planner

This skill produces a `build-plan.md` file — a living reference document the AI reads at the
start of every coding session to understand exactly what is being built, how, and where things stand.

It runs in three modes:

- **Init** — called once when starting a new project
- **Update** — called after completing a phase to append progress to the plan
- **Review** — called at the start of a coding session to recover context from an existing plan

---

## Mode Detection

Determine which mode to use:

- If there is no existing `build-plan.md` in the project → **Init mode**
- If the user says "phase done", "I finished phase X", "update the plan", or similar → **Update mode**
- If `build-plan.md` exists and the user is about to continue coding → **Review mode**
- If unsure, ask: "Are you starting a new project, continuing an existing one, or marking a phase complete?"

---

## Init Mode

Work through each section in order. Do not skip ahead. Do not generate a section that requires
user input until you have that input confirmed.

---

### Step 1 — Understand the Idea

Ask the user questions to gather as much context as possible before generating anything.
Do not guess. Do not assume. Keep asking until you have enough to work with confidently.

Cover all of these:

- What are you building? Describe it in plain language.
- Who is it for? Who will use it day to day?
- What problem does it solve, and what does the user do today without it?
- What does success look like in 30 days? In 6 months?
- Any existing work, prior attempts, references, or apps you're inspired by?
- Solo or team? What is your technical level?
- Any hard constraints — timeline, budget, must-use tools or platforms?
- What is explicitly out of scope? What will this NOT do?

Once you have sufficient answers, summarise your understanding back to the user in 2–3 sentences
and ask them to confirm before moving on.

---

### Step 2 — Project Identity

Generate this section entirely from the confirmed answers. No user input needed unless they
flag something as wrong.

Output:

- **Product name** — a working name if none given
- **One-line purpose** — what it does in under 15 words
- **Description** — one paragraph covering what it is, who it is for, and why it exists
- **Out of scope** — a short list of things this project will not do

---

### Step 3 — Stack Contract

Propose a stack with clear reasoning for each choice. Keep it lean and appropriate to the
user's skill level and constraints. Then ask the user to confirm, adjust, or override.

Cover:

- Frontend (framework, styling)
- Backend (language, framework, or serverless)
- Database (type, specific tool)
- Auth (approach and tool)
- Hosting / deployment
- Key third-party services or APIs (only if needed)

Lock in the stack only after the user confirms. Mark each choice as **confirmed**. Once confirmed,
treat these choices as a contract — future coding agents must not change them without explicit
user approval.

---

### Step 4 — Feature Breakdown

Propose a feature list based on the idea and confirmed scope. Divide into:

- **Must-have** — without these, the product doesn't function
- **Nice-to-have** — valuable but not blocking

Present to the user and ask them to add, remove, or move items before locking in.
Each feature gets a status tag. Default: `planned`.

---

### Step 5 — File and Folder Structure

Generate the full project structure from the confirmed stack and feature list.
No user input needed unless they flag an issue.

Show it as a tree. Be specific — include key files, not just folders.

---

### Step 6 — Coding Rules

Ask the user directly. Do not guess or use defaults without asking first.

Cover:

- Naming conventions (variables, files, components)
- Component or module patterns (if frontend)
- Error handling approach
- Anything to avoid (libraries, patterns, shortcuts)
- Any existing linter, style guide, or personal rules to follow

If the user has no strong preferences, suggest sensible defaults and ask them to confirm.

---

### Step 7 — Build Phases

Generate ordered build phases from the confirmed features and structure.
No user input needed unless they flag an issue.

Each phase should:

- Have a clear goal
- Be completable in one focused session
- List which features it covers
- Note its status: `planned`

Order phases by dependency — do not put auth after features that require auth, etc.

---

### Step 8 — Risk Flags

Generate this section fully. No user input.

Identify the parts of this project most likely to cause problems. Present as a table with three columns: the risk area, why it matters for this specific project, and a concrete mitigation.

Categories to consider:

- Technical complexity (e.g. real-time features, complex data models)
- Third-party dependencies that could break or change
- Auth and security edge cases
- Scope creep risks
- Areas where the user's skill level may create friction

Be honest and specific. Do not pad this with generic warnings.

---

### Step 9 — Current State Summary

Set this to:

```
## Current State

**Phases completed:** None

**What exists:** Nothing built yet.

**What is next:** Begin Phase 1.

**Known issues:** None.
```

---

### Output — Write build-plan.md

Once all sections are confirmed, write the full `build-plan.md` to the root of the project.

Use the template below exactly. Do not omit any section.

---

```markdown
# Build Plan — [Product Name]

> Last updated: [date]

---

## Project Identity

**Purpose:** [one-line purpose]

**Description:** [one paragraph]

**Out of scope:**
- [item]
- [item]

---

## Stack Contract

> These choices are locked. Future coding agents must not change them without explicit user approval.

| Layer | Choice | Status |
|---|---|---|
| Frontend | | confirmed |
| Backend | | confirmed |
| Database | | confirmed |
| Auth | | confirmed |
| Hosting | | confirmed |

---

## Feature Registry

| Feature | Priority | Status |
|---|---|---|
| [feature] | must-have | planned |
| [feature] | nice-to-have | planned |

---

## File and Folder Structure

```
[tree here]
```

---

## Coding Rules

- [rule]
- [rule]

---

## Build Phases

### Phase 1 — [Name]
**Goal:** [what this phase achieves]
**Features covered:** [list]
**Status:** planned

### Phase 2 — [Name]
...

---

## Risk Flags

| Risk | Why It Matters | Mitigation |
|---|---|---|
| [risk area] | [specific reason this matters for this project] | [concrete mitigation] |

---

## Current State

**Phases completed:** None

**What exists:** Nothing built yet.

**What is next:** Begin Phase 1.

**Known issues:** None.

---

## Progress Log

_Appended automatically after each phase is marked complete._
```

---

## Review Mode

Triggered when `build-plan.md` exists and the user is about to continue coding.

Read the plan and summarise back to the user:

- **Project identity** — what is being built and for whom
- **Stack Contract** — the locked technology choices
- **Active or next phase** — what the current focus should be
- **Relevant coding rules** — constraints to respect this session
- **Features not yet done** — remaining planned work
- **Known risks or blockers** — anything flagged that applies to the current phase

Keep the summary concise. Do not reprint the entire plan. Then proceed directly with the user's coding task, treating `build-plan.md` as the source of truth for all decisions.

---

## Update Mode

Triggered when the user signals a phase is complete.

Ask if not provided:
- Which phase was completed?
- What was actually built?
- Did anything break or deviate from the plan?
- Any blockers or notes for the next phase?

Then do all of the following in one step:

1. **Update the Feature Registry** — set all features covered in this phase to `done`
2. **Update the Build Phases** — set the completed phase status to `done`, next phase to `in progress`
3. **Update the Current State section** — rewrite it with what now exists, what is next, and any known issues
4. **Append to the Progress Log** — add a timestamped entry at the bottom. Never overwrite previous entries.

Progress log entry format:

```markdown
### [Date] — Phase [N] Complete: [Phase Name]

**Built:**
- [what was built]

**Deviations from plan:**
- [any changes made vs. what was planned, or "None"]

**Known issues:**
- [anything broken or incomplete, or "None"]

**Next:** Phase [N+1] — [Phase Name]
```

Write the updated file back to `build-plan.md`. Confirm to the user that the file has been updated.

---

## Principles

- The `build-plan.md` is the AI's source of truth. Treat it as such.
- Never overwrite existing progress log entries. Append only.
- Keep each section current — a stale plan is worse than no plan.
- If the user deviates from the plan mid-build, note it in the log, do not silently update the original plan without flagging it.
- When reading the file at the start of a session, summarise the current state to the user before doing anything else.
