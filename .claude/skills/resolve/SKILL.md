---
name: resolve
description: >-
  Captures architectural decisions as Alexandrian-inspired decision records
  through guided conversation. Use when trade-offs are being weighed, a decision
  has just been made, or the user names a tension or misfit that needs resolving.
  Also suggest naturally when architectural reasoning is happening in conversation,
  e.g. "This feels like a decision worth capturing — want to run /resolve?"
---

# Resolve

You are a reasoning partner helping the user surface, sharpen, and record
architectural decisions. You use an Alexandrian-inspired format where decisions
emerge from named forces — not imposed, but resolved.

## Context Loading

**Always** check for `.claude/plans/discovery.md` and `.claude/adrs/backlog.md` at the start of any `/resolve` invocation. If they exist, read them — the discovery spec provides the problem statement, scope, and constraints; the backlog maps the decision landscape. Use both as foundational context for any decision being made.

If either exists, acknowledge it briefly: *"I found the discovery spec and backlog for [project name]. I'll use them as context."*

## Entry Points

Detect which mode applies, or ask if unclear.

### DISTILL — mid-conversation

A decision is being debated right now. Your job:

1. Scan conversation context for the decision point, tensions, and alternatives
2. Name what you see: *"It looks like the main decision is X, with tension between Y and Z. The alternatives so far are A, B, C. Is that right?"*
3. Draft a complete ADR with status **Proposed**
4. Iterate with the user to sharpen it

### CAPTURE — post-decision

A decision was just made. Your job:

1. Scan conversation context for what was decided, why, and what was rejected
2. Draft a complete ADR with status **Proposed**
3. Iterate to confirm accuracy, especially the Consequences

### DELIBERATE — greenfield

The user has a tension or misfit to think through. Your job:

1. Ask them to describe the tension, the felt difficulty, the misfit
2. Help them name the forces — what pulls in each direction
3. Generate genuine alternatives (no strawmen)
4. Reason through options together, letting a resolution emerge
5. Draft a complete ADR with status **Proposed**

This mode is the most Alexandrian: feel the misfit, name the forces, let the
resolution emerge from them.

## Workflow

```
Step 1: Detect mode (DISTILL / CAPTURE / DELIBERATE)
Step 2: Gather material (scan context, ask, surface resources)
Step 3: Review sources (present compiled list, user confirms before drafting)
Step 4: Draft full ADR (draft aggressively, confirm explicitly)
Step 5: Iterate section by section (including diagram)
Step 6: Save as Proposed
Step 7: Feedback cycle (user annotates → agent resolves → repeat)
Step 8: Accept
```

### Step 1 — Detect mode

If the conversation contains active debate about an architectural choice,
default to DISTILL. If a decision was clearly just made, default to CAPTURE.
If the user invoked `/resolve` without prior context, default to DELIBERATE.

Ask to confirm: *"This looks like a [distill/capture/deliberate] situation — does that feel right?"*

### Step 2 — Gather material

**Always**: Read `.claude/adrs/backlog.md` to check if this decision matches a
backlog item. If it does, pull in the item's research notes, tensions, spec
refs, dependency information, and any listed reference sources as starting
context. Name the match:
*"This maps to backlog item F1 (Atomicity definition). Here's what we already know…"*

**Always**: Surface resources. If the backlog item lists research links, name
them: *"The backlog points to these resources: [list]. Have you read them?
Are there others that bear on this specific decision — articles, docs,
examples, prior art?"* If the user names resources, fetch and review them
before drafting. If the backlog item has no research links, still ask:
*"Are there any external resources — articles, reference implementations,
prior art — that should inform this decision?"*

**Delegating long sources:** When a source is too long to process inline
(large documents, lengthy web pages, extensive code files), do **not** try to
digest it yourself. Instead, spin up a **Task subagent** dedicated to that
source. In the subagent prompt:

1. State what the ADR is about — the decision being made and the key tensions
2. Tell the subagent which source to read/fetch — and tell it to **read the
   entire source**, not just search for keywords. Context matters; the
   subagent needs to understand the source as a whole before judging what's
   relevant
3. Tell it what to care about — the specific forces, options, or constraints
   that this source might speak to. This guides what it surfaces, but
   should not limit what it reads
4. Ask it to return a concise summary of the parts relevant to the decision,
   with direct quotes or specific references where useful. It should also
   flag anything surprising or unexpected it found — context the main agent
   might not have thought to ask about

Example prompt shape: *"We're drafting an ADR about [decision]. The key
tensions are [X vs Y]. Read [source path/URL] and extract anything relevant
to these tensions — recommendations, trade-offs, constraints, prior art.
Return a concise summary with specific references I can cite."*

Use `subagent_type: "general-purpose"` for these delegations. Launch
multiple source subagents in parallel when several long sources need
processing. Fold their results into your source compilation before
presenting the list in Step 3.

**DISTILL/CAPTURE**: Mine the conversation for:
- What is being decided (or was decided)
- What forces are in tension
- What alternatives were discussed and why they were accepted/rejected
- What constraints or context shaped the discussion

**DELIBERATE**: Ask the user to describe:
- The situation — what's true right now
- The tension — what's pulling in different directions
- Any options they've already considered

### Step 3 — Review sources

Before drafting, present the user with a consolidated list of every source
identified so far — from the backlog, conversation context, user-provided
links, fetched documentation, code files, and any other material surfaced
during Step 2. Format the list as it would appear in the ADR's Sources
section (numbered keys with paths/URLs and brief descriptions).

Use **AskUserQuestion** to confirm:

*"Here are the sources I've compiled that will inform this ADR:*

*[numbered list of sources]*

*Before I draft, I want to make sure this is complete. Anything to add, remove, or clarify?"*

Options: **Looks complete** / **I have sources to add** / **Some of these need clarification**

If the user adds sources, fetch and review them before proceeding. If the
user flags sources for clarification, resolve each one. Repeat until the
user confirms the source list is complete.

Only proceed to drafting once the user has signed off on the sources.

### Step 4 — Draft full ADR

Present a **complete draft** using the format below. A full document is easier
to react to than a blank template. The user's job is editing, not writing.

Include all sections, even if some are thin. Mark uncertain parts with
`[?]` so the user knows where to focus.

**Sources are mandatory.** Before drafting, compile the list of every document,
file, URL, or resource that informed the ADR — backlog items, other ADRs,
documentation, articles, code files, conversation-referenced links. Assign
each a numeric key and include them in the Sources section. Use those keys
as inline references throughout the draft wherever a claim or option draws
on a specific source.

### Step 5 — Iterate section by section

Walk through the draft using **AskUserQuestion** for each section. Ask one
section at a time so the user can focus:

- **Situation**: *"Does this capture the reality you're working in? Anything missing?"*
- **Forces**: *"Are these the real tensions? Is anything pulling that I haven't named?"*
- **Paths considered**: *"Are these honest alternatives? Would a reasonable person choose any of the rejected options?"*
- **Therefore**: *"Does this feel like it follows from the forces, or does it feel imposed?"*
- **Consequences**: Push here — *"What does this make harder? What options does it close? What new tensions does it create?"*
- **Diagram**: Present 2–3 options for what the diagram should show (e.g., system topology after the decision, force diagram, before/after flow, dependency graph). Generate ASCII for simplicity or mermaid when topology is complex. The diagram must show something the prose doesn't — not decorative.

Use AskUserQuestion with concrete options where possible (e.g., "Looks good" /
"Needs changes" / "Missing a force"), so the user can respond quickly when a
section is solid and elaborate only when needed.

### Step 6 — Save as Proposed

**Always save with status Proposed**, regardless of mode.

1. Read `.claude/adrs/` to find the highest existing number
2. Assign the next sequential number (zero-padded to 3 digits)
3. Slugify the title (lowercase, hyphens, no special characters)
4. Write to `.claude/adrs/NNN-slugified-title.md`
5. Update `.claude/adrs/index.md` (create if it doesn't exist)
6. If this ADR matches a backlog item, update its status in `.claude/adrs/backlog.md` to `→ ADR NNN`

After saving, tell the user: *"Saved as Proposed. Review the document and add
any feedback directly as markdown quotes (`> your note`) wherever you want
changes. Let me know when you're done annotating."*

### Step 7 — Feedback cycle

When the user signals they have annotated the document:

1. **Read** the saved ADR file
2. **Find all annotations** — lines starting with `>` that were not part of the
   original draft (the situation paragraph uses `>` by convention, so look for
   *new* quoted lines that appear adjacent to other sections)
3. **For each annotation**, use **AskUserQuestion** to clarify intent if needed,
   then resolve the feedback into the document text
4. **Save again** with status **Proposed**
5. Tell the user: *"Updated. Review again — add more `> notes` if anything else
   needs work, or let me know it's ready to accept."*

Repeat this cycle as many times as needed. Each round: read → resolve
annotations → save as Proposed → ask if done.

### Step 8 — Accept

When the user indicates the ADR is final (no more feedback), use
**AskUserQuestion** to confirm:

*"Ready to mark this ADR as Accepted?"*

Options: **Accept** / **One more round of feedback**

On acceptance:
1. Remove any remaining `> annotation` lines from the document
2. Change status from **Proposed** to **Accepted**
3. Update the status in `.claude/adrs/index.md`
4. Save the file

## ADR Format

```markdown
# NNN · Decision Title

**Status:** Proposed | Accepted | Superseded by NNN
**Date:** YYYY-MM-DD
**Deciders:** ...

> A paragraph grounding you in the situation. Concrete, present-tense.
> Not "background" — the lived reality that makes this decision
> necessary. You are here, facing this.

**Sources:**

[1](relative/path/to/file.md) — Brief description of what this source is
[2](https://example.com/relevant-doc) — Brief description
[3](../adrs/001-prior-decision.md) — Brief description

If no sources were referenced: "No sources referenced."

---

**Forces:**

- **Force name** — what pulls in one direction, and why
- **Force name** — what pulls against it, and why

Forces are genuine tensions, not a pro/con list. Each one names
something real that resists a simple answer. Together they describe
why this is hard.

**Paths considered:**

- **Option A** — what it resolves, what it leaves unresolved
- **Option B** — ...

Honest. No strawmen. Each option should be one a reasonable
person might choose.

---

∴ **Therefore:**

The decision, stated as a resolution of the forces above. Active
voice, one to three sentences. It should feel like it *follows from*
the forces — not arbitrary, but the move that best resolves the
tensions named above.

---

[diagram: ASCII or mermaid]

---

**Resulting context:**

What is now true. What becomes easier. What becomes harder.
What new tensions emerge — these are seeds for future decisions.

**Linked decisions:** [NNN · Title](NNN-title.md), ...
```

### Format notes

- The **∴ Therefore** section is the structural heart. If it doesn't feel
  like it emerges from the forces, the forces section needs more work.
- **Resulting context** should name new tensions that seed future decisions.
  The resulting context of one ADR is the opening situation of the next.
- **Linked decisions** connect to prior ADRs that this supersedes, builds on,
  or tensions with. Check existing ADRs for relevant links.
- The **diagram** shows structural or relational consequences — not the
  decision itself, but what the world looks like after.

### Sources

Every ADR **must** have a Sources section after the situation paragraph. Each source gets a numeric key (`[N](path/or/url)`) used as the visible link text both in the Sources list and inline throughout the body — so the reader can click through from any reference point.

Sources include: backlog items, other ADRs, documentation, articles, code files, URLs — anything referenced during creation. Reference sources inline wherever a claim, force, or option draws on them: `([2](https://example.com/docs))`.

If no external sources were referenced (rare), include: `No sources referenced.`

## Index Format

Maintain `.claude/adrs/index.md`:

```markdown
# Decision Record Index

| #   | Title                  | Status   | Date       |
|-----|------------------------|----------|------------|
| 001 | Decision title here    | Accepted | 2025-01-15 |
| 002 | Another decision       | Proposed | 2025-01-20 |
```

## File Conventions

- **Location**: `.claude/adrs/`
- **Naming**: `NNN-slugified-title.md` (e.g., `001-use-postgresql.md`)
- **Index**: `.claude/adrs/index.md` — record of decisions made
- **Architecture backlog**: `.claude/adrs/backlog.md` — the decision landscape. Contains framing, reference sources, and categorized candidate decisions with research context and dependencies. Produced by `/explore`; updated as ADRs are made.
- **Numbering**: sequential, zero-padded to 3 digits, never reused

## When the material is thin

If the decision seems trivial or the forces are one-sided, do not refuse.
Instead offer alternatives:

1. *"This could work as a lightweight decision log entry — the forces are pretty one-sided. Want to do that instead?"*
2. *"Want to dig deeper? Sometimes a decision seems obvious until you name the forces."*
3. *"Go ahead with a full ADR anyway — even simple decisions benefit from a record sometimes."*

A lightweight log entry is a single paragraph in a running log file at
`.claude/adrs/log.md` with date, one-line decision, and brief rationale.

## Principles

- **Draft aggressively, confirm explicitly.** Full drafts, not blank templates.
- **Forces are tensions, not pros/cons.** They describe why the decision is hard.
- **The decision resolves forces.** If it doesn't follow from them, rework the forces.
- **Diagrams earn their place.** They show what words can't.
- **Consequences are honest.** Push for what gets harder, what closes, what's lost.
- **Resulting context seeds the next decision.** The chain of ADRs is a narrative.
