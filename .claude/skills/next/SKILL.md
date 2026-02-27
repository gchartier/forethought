---
name: next
description: >-
  Session bootstrapper for blueprint tasks. Finds the next unblocked task from
  .claude/plans/blueprint.md, loads its ADR and discovery context, and manages
  task lifecycle (start, verify, close out). Use after /blueprint has produced
  a build plan and you're ready to implement. Do NOT use before a blueprint
  exists.
disable-model-invocation: true
---

# Next

Finds the next unblocked blueprint task, loads its context, guides
implementation, and closes out with verification and status update.

## Activation

Read `.claude/plans/blueprint.md`. If it doesn't exist, stop:
*"No blueprint found. Run `/blueprint` first to create a build plan."*

**If `$ARGUMENTS` names a task** (e.g., `T5`): target that task regardless
of order. If that task is blocked by incomplete dependencies, warn the user
and list what's blocking it — then ask whether to proceed anyway or pick the
next unblocked task instead.

**Otherwise**: find the first task where `**Status:**` is not `done` and all
tasks listed in its `**Depends on:**` field have `**Status:** done`. Tasks
with no `**Status:**` field are `pending`.

**If a task has `**Status:** in-progress`**: a previous session was
interrupted. Target that task — ask the user whether to resume where they
left off or start it fresh.

If no actionable task exists (all done, or all remaining are blocked), report
the full status and stop.

## Context Load

For the target task:

1. Read `.claude/plans/discovery.md` — note the constraints relevant to this
   task's acceptance criteria
2. Read each ADR cited in the task's `**Source:**` field (e.g., `← ADR 001`
   → find and read `.claude/adrs/001-*.md`)
3. If the task depends on completed tasks, scan those tasks for any
   `**Deviation:**` notes that affect this task's approach

Present a brief summary via `AskUserQuestion`:

> **Next up: T[N] — [title]**
>
> **Deliverable:** [from blueprint]
> **Key context:** [1-3 sentences synthesizing the relevant ADR decisions
> and discovery constraints that bear on this task]
> **Acceptance criteria:** [list from blueprint]
> **Warnings:** [anything from ADR resulting contexts, predecessor
> deviations, or blueprint open questions that affects this task]

**Question — Ready?** (header: "Ready?")
> "Context loaded. Ready to start?"
- **Start implementing** — Begin work on this task
- **Show me the full ADR context** — Display the complete ADR text before starting
- **Pick a different task** — Choose another task instead

After confirmation, update the task's `**Status:**` field in the blueprint
to `in-progress`.

## Implementation

Implement the task using the acceptance criteria as the definition of done.
Follow ADR decisions — don't re-decide what's been decided.

When making non-obvious implementation choices, briefly note *why* (one
line), citing the relevant ADR or constraint.

**If something doesn't work as planned:**

- **Task needs splitting** — Implement what you can. Note the remainder in
  the close-out.
- **ADR assumption was wrong** — Implement the pragmatic solution. Record
  the deviation.
- **New tension surfaces** — Note it. Don't stop to resolve it now.

## Close-out

When implementation is complete:

### 1. Verify

Walk through each acceptance criterion. For criteria that can be tested
(builds, runs, assertions), run them. Present results:

```
Acceptance criteria:
  [x] Criterion 1 — passed (how)
  [x] Criterion 2 — passed (how)
  [ ] Criterion 3 — partial (explanation)
```

If any criterion fails, ask via `AskUserQuestion`:

**Question — Incomplete criteria** (header: "Criteria")
> "[Criterion] didn't pass. How to proceed?"
- **Fix it now** — Address the issue before closing out
- **Accept as-is** — Mark done with a note about what's incomplete
- **Leave in-progress** — Keep the task open for next session

### 2. Update blueprint

Edit `.claude/plans/blueprint.md`:

- Set `**Status:** done` on the completed task
- If implementation deviated from the plan, add a `**Deviation:**` field
  with a one-line explanation below the task's existing fields
- If the task was partially completed and the remainder is non-trivial,
  add a new task section for the remaining work, with appropriate
  dependencies
- If new tensions or questions were discovered, append them to
  `## Open Questions`

### 3. Report

Output a brief summary:

> **Completed: T[N] — [title]**
> **Deviations:** [any, or "none"]
> **Next up:** T[M] — [title] (or "all tasks complete" or "remaining
> tasks are blocked by: ...")

## Status field convention

Tasks in the blueprint use a `**Status:**` field inserted after the task
title line:

```markdown
### T1: Task title

**Status:** done
**Deliverable:** ...
```

Valid values: `pending`, `in-progress`, `done`. Absence means `pending`.
When `/next` first runs on a blueprint without status fields, it adds
them as it goes — no need to backfill all tasks upfront.
