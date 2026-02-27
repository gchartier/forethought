---
name: blueprint
description: >-
  Translates a discovery spec and resolved ADRs into a concrete, ordered
  implementation plan. Use when architectural decisions are complete and the
  user is ready to decompose the project into buildable tasks. Requires both
  .claude/plans/discovery.md and resolved ADRs in .claude/adrs/. Do NOT use
  before /explore and /resolve have produced their outputs.
disable-model-invocation: true
---

# Blueprint

Transforms architectural artifacts into an actionable build plan. Reads the
discovery spec and resolved ADRs, extracts implementation requirements, and
produces a structured task breakdown the user can work through.

## Prerequisites

**Both** of these must exist before running:

1. `.claude/plans/discovery.md` — the discovery spec from `/explore`
2. `.claude/adrs/` — at least one accepted ADR, plus `backlog.md` or `index.md`

If either is missing, stop and tell the user:
*"Blueprint needs both a discovery spec and resolved architectural decisions.
Run `/explore` first to define the problem, then `/resolve` to work through
the hard trade-offs. Blueprint turns those outputs into a build plan."*

## Workflow

```
Step 1: Intake — read and synthesize all artifacts
Step 2: Extract — pull implementation requirements from each source
Step 3: Propose structure — phased layers vs. vertical slices
Step 4: Draft tasks — concrete, traceable, dependency-aware
Step 5: Iterate — walk through with user, refine section by section
Step 6: Write — save to .claude/plans/blueprint.md
```

### Step 1 — Intake

Read all artifacts:

- `.claude/plans/discovery.md`
- `.claude/adrs/backlog.md` (if it exists)
- `.claude/adrs/index.md` (if it exists)
- Every ADR file in `.claude/adrs/NNN-*.md`

Present a summary to the user:

*"I've read the discovery spec for [project name] and [N] architectural
decisions. Here's what I'm working with:"*

- **Scope**: one-line summary of what's being built (from discovery)
- **Decisions made**: list each ADR by number and title
- **Key constraints**: the 3-5 most implementation-relevant constraints
- **Tech stack**: extracted from discovery + ADRs
- **Unresolved tensions**: any "new tensions" from ADR resulting contexts
  that haven't been addressed by subsequent ADRs

Ask via `AskUserQuestion`:

**Question — Intake check** (header: "Intake")
> "Does this capture the current state? Anything changed since these were written?"
- **Looks right** — The artifacts reflect the current plan
- **Something has changed** — A decision was revised or a constraint shifted
- **There are more ADRs to resolve first** — Not ready for blueprint yet

If the user says something changed, gather the update, note it as a
deviation from the written artifacts, and incorporate it. If they say more
ADRs are needed, suggest running `/resolve` first.

### Step 2 — Extract

Mine each artifact for implementation requirements. Build a raw requirements
list organized by source:

**From discovery spec:**
- Each item under "Core Scope" becomes one or more implementation tasks
- Tech stack choices become setup/configuration tasks
- Constraints become acceptance criteria on tasks
- "Deferred" items are explicitly excluded (but noted for boundary clarity)

**From each ADR:**
- The **Therefore** section: the concrete decision to implement
- The **diagram**: data models, component topology, service boundaries
- The **Resulting context — what becomes easier**: features that come "for
  free" from the decision (still need wiring, but less work)
- The **Resulting context — what becomes harder**: tasks that need extra
  care or workarounds
- The **Resulting context — new tensions**: items that may need attention
  during implementation even though they weren't separate ADRs

**From the backlog:**
- Any "resolved inline" or "dissolved" items that still imply implementation
  work (e.g., "Jazz IS the WebSocket layer" still means you need to set up
  Jazz)
- The resolution order and dependency graph inform task ordering

Present the raw extraction to the user as a bulleted list grouped by source.
Don't organize into tasks yet — just show what was extracted. Ask:

**Question — Extraction check** (header: "Extraction")
> "Here's what I pulled from your artifacts. Anything missing or wrong?"
- **Looks complete** — Everything relevant is captured
- **Missing something** — There's an implementation requirement not in the artifacts
- **Over-extracted** — Some of these aren't relevant for the first build

### Step 3 — Propose structure

Based on the project shape, propose how to organize the build plan. Consider:

- **Phased (horizontal layers)** when: foundational infrastructure gates
  everything else, the tech stack needs setup before features make sense, or
  there's a clear "can't do B without A" dependency chain.

- **Vertical slices** when: the project has semi-independent features that
  each deliver end-to-end value, the user wants to ship incrementally, or
  early user feedback matters.

- **Hybrid** when: a thin foundation phase is needed, then features can
  proceed as vertical slices on top of it.

Present the recommendation with reasoning via `AskUserQuestion`:

**Question — Structure** (header: "Structure")
> "[Reasoning for recommendation]. How should we organize the build?"
- **[Recommended option]** — [one-line description]
- **[Alternative]** — [one-line description]
- **[Alternative]** — [one-line description]

Once the user picks a structure, organize the extracted requirements into
that structure. Name the phases or slices. Show the grouping and ask for
confirmation before drafting individual tasks.

### Step 4 — Draft tasks

For each phase/slice, draft concrete tasks. Each task has:

- **Title**: imperative, specific (e.g., "Define Jazz CoValue schema for
  rooms and messages")
- **Deliverable**: what exists when this task is done
- **Source**: traceability marker — `← ADR NNN`, `← discovery`, or
  `← backlog FN`
- **Depends on**: other tasks that must complete first (by title or ID)
- **Acceptance criteria**: 2-4 bullets defining "done" — drawn from
  discovery constraints and ADR resulting context
- **Notes**: implementation hints extracted from ADR diagrams, resulting
  context, or "what becomes harder" warnings

**Task granularity**: each task should be completable in a single focused
Claude session. If a task feels like it needs sub-tasks, it's too big —
split it. If two tasks would always be done together, merge them.

Present the full draft as a numbered list within phases/slices, showing all
fields. Use a fenced code block for the dependency graph:

```
Phase 1: Foundation
  T1 ──▶ T2 ──▶ T4
            └──▶ T3 ──▶ T5
```

### Step 5 — Iterate

Walk through the draft section by section. For each phase/slice, ask:

**Question — Phase review** (header: "[Phase/slice name]")
> "Here's [phase/slice name]. How does this look?"
- **Looks good** — Tasks are right-sized and correctly ordered
- **Tasks are too big** — Need to split some of these
- **Tasks are too small** — Some should be merged
- **Missing tasks** — Something isn't covered
- **Wrong order** — Dependencies need rearranging

When the user requests changes, show a diff of the affected section
(same format as `/explore` — `+` for additions, `-` for removals) and
re-confirm before moving to the next section.

After all sections are reviewed, show the complete dependency graph and
ask for final confirmation:

**Question — Final check** (header: "Final check")
> "Here's the complete blueprint. Ready to save?"
- **Save it** — The plan is ready
- **One more pass** — Need to revisit a section

### Step 6 — Write

Save to `.claude/plans/blueprint.md` using the format below. If the file
already exists, ask before overwriting.

After saving, suggest next steps:
*"The blueprint is saved. You can start working through the tasks in order.
Each task is scoped for a single Claude session — pick one, give Claude the
blueprint for context, and go."*

## Blueprint Format

```markdown
# [Project Name] — Blueprint

**Generated from:** discovery.md, ADR 001, ADR 002, ...
**Date:** YYYY-MM-DD
**Structure:** [Phased / Vertical slices / Hybrid]

## Summary

[2-3 sentences: what's being built, key architectural decisions, and
the organizing principle for this plan.]

## Constraints (from discovery)

- [Constraint 1]
- [Constraint 2]
- ...

## Dependency Graph

\```
[ASCII graph showing task dependencies]
\```

---

## [Phase/Slice 1 Name]

### T1: [Task title]

**Deliverable:** [What exists when done]
**Source:** ← ADR NNN · [title], ← discovery
**Depends on:** —
**Acceptance criteria:**
- [Criterion 1]
- [Criterion 2]

**Notes:** [Implementation hints, warnings from ADR resulting context]

### T2: [Task title]

...

---

## [Phase/Slice 2 Name]

...

---

## Deferred (from discovery)

Items explicitly out of scope for this blueprint:
- [Deferred item 1]
- [Deferred item 2]

## Open Questions

Tensions or uncertainties surfaced during blueprint creation that may
need attention during implementation:
- [Question 1]
- [Question 2]
```

## Principles

- **Extract, don't invent.** Every task traces back to a discovery item or
  ADR. If something feels necessary but isn't in the artifacts, surface it
  as a gap — don't silently add it.
- **Granularity = one session.** Each task should be completable in a single
  focused Claude Code session. This is the unit of work.
- **Dependencies are real.** Only mark a dependency when task B literally
  cannot start without task A's output. "Would be nice to do first" is not
  a dependency.
- **Constraints are acceptance criteria.** Discovery constraints don't
  disappear — they become the "done" definition on tasks.
- **Deferred means deferred.** Items marked deferred in discovery do not
  appear as tasks. They appear in the "Deferred" section as a reminder of
  scope boundaries.
- **The blueprint is a living document.** During implementation, tasks may
  need splitting, reordering, or adding. That's expected. The blueprint is
  a starting point, not a contract.
