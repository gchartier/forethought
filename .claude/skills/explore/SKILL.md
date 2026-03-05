---
name: explore
description: >-
  Asks the right clarifying questions to flesh out vague or under-specified
  requests. Use when a user gives a broad, ambiguous, or general "I want to
  do X" request that needs refinement before execution. Also use when the user
  explicitly asks for help thinking through a problem, scoping a project, or
  defining requirements. Do NOT use for requests that are already specific and
  actionable.
---

# Explore

When a request is vague or under-specified, probe the problem space — not just the solution space. Most vague requests exist because the person hasn't fully articulated the problem to themselves yet. The best questions help them think, not just gather inputs.

## Core principles

**Separate the problem from the solution.** Most requests arrive pre-packaged as a solution: "I want to buy a house," "I want to build an app," "I want to hire a designer." Before refining the solution, extract the underlying problem. The solution they stated may not be the best one — or even a good one — for their actual situation.

**Challenge the frame, not just the details.** If someone says "I want X," don't just ask "what kind of X?" Ask whether X is the right category at all. Generate 2-4 genuinely different approaches to the underlying problem, including ones the person likely hasn't considered. This isn't about being contrarian — it's about ensuring they've seen the full option space before committing.

**Ask only the 2-3 questions that most reduce ambiguity** for this specific situation. Skip anything inferable from context. Asking everything on a checklist feels like an interrogation.

## Dimensions to probe

Evaluate which of these dimensions are under-specified, then ask only about those:

**Motivation & Goals**
- Why do you want to do this? What's the underlying need?
- What does success look like? How will you know you've achieved it?
- Is this the goal itself, or a means to a deeper goal?

**Current State & Constraints**
- What have you already tried or considered?
- What resources do you have (time, money, skills, people)?
- Hard constraints — deadlines, budget, non-negotiables?
- Starting point / current experience level with this?

**Scope & Specifics**
- What exactly do you mean by [the ambiguous term]? (Almost every request has at least one word that could mean several things.)
- How big is this? One-time or ongoing? For yourself or others?
- Intended audience or context?

**Risks & Tradeoffs**
- What would make this go wrong?
- What are you willing to sacrifice — speed vs quality, cost vs convenience?
- What have you ruled out, and why?

**Assumption Check — is this a solution or a problem?**
- Detect when the request contains a pre-chosen solution (buy, build, hire, use X tool, etc.)
- Extract the underlying need: what problem does this solution solve for them?
- Generate 2-4 genuinely divergent alternatives that address the same need through different means — not variations on the same theme, but structurally different approaches (e.g., buy vs rent vs build vs join a community vs do nothing)
- Include at least one option the person almost certainly hasn't considered
- Present these as a lightweight menu early in the conversation, before drilling into details of any single path

## Triage — Guided Entry

Before probing, run a quick deterministic questionnaire using `AskUserQuestion` to orient the refinement. Ask all three questions in a single call:

**Question 1 — Nature** (header: "Nature")
> "What are you trying to do?"
- **Build something new** — Greenfield; needs scope, audience, and architecture decisions
- **Change or improve something existing** — Needs context discovery and impact analysis
- **Fix or solve a problem** — Needs reproduction, root cause, and success criteria
- **Explore options / think through an idea** — Needs divergent framing before any commitment

**Question 2 — Clarity** (header: "Clarity")
> "Where are you in your thinking?"
- **I know the solution, need help specifying details** — Jump to constraint extraction; minimal frame-challenging
- **I have a direction but I'm not confident it's right** — Challenge the frame early, then refine
- **I know the problem but not the solution** — Heavy divergent exploration; generate 3-4 structurally different approaches before narrowing
- **I'm still figuring out what the actual problem is** — Start with motivation/goals; everything else is premature

**Question 3 — Scope** (header: "Scope")
> "How big does this feel?"
- **Small and contained** — A function, a config, a single component
- **Medium — a few moving parts** — A feature, a module, a handful of files
- **Large or cross-cutting** — Touches multiple systems, layers, or teams
- **Not sure yet — that's part of what I need help with** — Scope discovery becomes the first probe

### Triage → Refinement Mode

Map the answers to a refinement strategy. This determines *how* the iterative loop behaves — which dimensions to probe first, how aggressively to challenge the frame, and when to exit.

| Clarity \ Nature | Build | Change | Fix | Explore |
|---|---|---|---|---|
| **Know the solution** | Constraint extraction — enumerate specs, validate scope | Impact scoping — what changes, what breaks | Likely already actionable — confirm and exit, or skip `/explore` entirely | Contradiction — "exploring" but "know the solution"? Gently challenge |
| **Have a direction** | Frame check — is this the right thing to build? Then scope | Trade-off surfacing — is the change worth it? Alternatives? | Diagnosis validation — is the suspected cause correct? | Option mapping — flesh out the direction + 2-3 alternatives |
| **Know the problem** | Divergent exploration — 3-4 approaches, then narrow | Root cause first — is the "change" fixing the right thing? | Solution generation — what are the possible fixes? | Pure problem framing — make the problem crisp before exploring |
| **Still figuring out** | Start with "why" — what need does building serve? | Start with "what's wrong" — name the pain point | Start with symptoms — what's actually happening? | Start with motivation — what triggered this exploration? |

For **scope**: if the user said "not sure yet," make scope discovery the first question in the iterative loop. If they said "small and contained" but the triage suggests cross-cutting concerns, flag the tension as an early misfit.

### Early Exit

If triage reveals the request is already actionable (e.g., Nature=Fix + Clarity=Know the solution + Scope=Small), say so and offer the choice via `AskUserQuestion`:

**Question — Continue?** (header: "Continue?")
> "This sounds specific enough to act on directly. What would you prefer?"
- **Skip exploration and go** — The request is clear enough to execute now
- **Think something through first** — There's an aspect worth exploring before starting

---

## Process — Iterative Constraint Discovery

After triage, work in a loop. Each iteration: ask one well-chosen question, then show the updated picture. This forces you to actually listen and adapt, rather than front-loading a checklist. **Use the refinement mode from triage to guide which questions you ask and in what order.**

**All user-facing questions MUST use `AskUserQuestion`** — never ask a question and wait for free-text. Formulate 2-4 concrete options for every question. The tool automatically provides an "Other" escape hatch for answers that don't fit, so focus options on the most likely or structurally distinct responses. This includes probe questions, constraint confirmations, approach selections, and exit confirmations.

**Each iteration:**

1. **Ask 1 question via `AskUserQuestion`** — the single question that most reduces ambiguity *given what you know so far*. Formulate 2-4 options that represent the most likely or structurally distinct answers. Include a one-line description on each option explaining its implications. The `AskUserQuestion` tool automatically provides an "Other" option for free-text input, so you don't need to include a catch-all — focus the options on the substantive choices. If the request contains a pre-chosen solution and this is the first iteration, the options should include the user's assumed frame *and* at least one alternative frame.

2. **After the user answers**, output the full updated constraint list.

   **Every constraint and misfit is numbered** (e.g., `1.`, `2.`, `3.`). Numbers are stable across iterations — a constraint keeps its number even when reworded. When a constraint is removed, its number is retired (not reused). New constraints get the next available number. This lets the user refer to items by number (e.g., "update #3", "remove #5 and #7").

   **On iteration 1**, output the list in regular markdown under the heading **Constraints / Misfits**. Number each item.

   **From iteration 2 onward**, output the full list as a single fenced code block with the `diff` language tag. Every constraint appears with its number — unchanged lines get a space prefix, removed/replaced lines get `-`, and new/updated lines get `+`. This is not a minimal diff; it is the *complete* list with change markers inline:

   ````
   ```diff
     1. Unchanged constraint
   - 2. Old version of a refined constraint
   + 2. New version of that constraint
     3. Another unchanged constraint
   + 8. Brand-new constraint from this iteration
     4. Yet another unchanged constraint
   ```
   ````

   Rules:
   - Always show the **full list** — every constraint, not just the changes
   - Every line includes the item's stable number
   - Unchanged lines: space prefix
   - Removed or replaced: `-` prefix
   - New or reworded: `+` prefix (keeps the same number when refining an existing item)
   - When a constraint is *refined*, show the old `-` and new `+` lines adjacent with the same number
   - If nothing changed, output the list in regular markdown instead (no diff block)

   Content rules (apply to all iterations):
   - One numbered line per concrete constraint, requirement, preference, or boundary
   - Mark anything that conflicts or creates tension as a **misfit** (e.g., "5. Misfit: wants cheap + wants Mendocino — these pull against each other")
   - Include inferred constraints (from context, not just explicit statements) and mark them as *inferred*

3. **Confirm the list via `AskUserQuestion`** — Ask the user to verify the constraints and misfits before continuing. Do NOT proceed to the next question until the user confirms.

   **Question — Confirm** (header: "Confirm")
   > "Does this capture things accurately? (You can reference items by number, e.g. 'update #3' or 'remove #5 and #7')"
   - **Looks good** — The constraints and misfits are correct as listed

   Only provide the single "Looks good" option. The tool's built-in "Other" option lets the user type corrections or additions by number. If the user types something via "Other," incorporate their feedback, re-display the updated list (retiring numbers of removed items, preserving numbers of updated items), and confirm again before proceeding.

4. **Then ask the next question** — chosen based on the confirmed constraint list. Prioritize questions that resolve misfits, close the biggest remaining ambiguity, or test an assumption you've been making. Always use `AskUserQuestion` as described in step 1.

**Repeat** until one of these exit conditions:
- The request is specific and actionable enough to execute well
- You've identified 2-4 structurally different approaches worth presenting — present them via `AskUserQuestion` with each approach as an option (label = short name, description = one-line summary of what it involves), then continue the loop on whichever the user picks
- The user signals they're ready to move forward

**On exit:**

1. Synthesize a clear problem statement + final constraint list. Confirm with the user via `AskUserQuestion`:

   **Question — Final check** (header: "Final check")
   > "Here's the final synthesis. Ready to lock this in?"
   - **Looks good, save it** — The problem statement and constraints are accurate

   The user can type adjustments via "Other" if something needs changing.

2. **Always save the final synthesis** to `.claude/plans/discovery.md` in the current project directory (create the directory if it doesn't exist). The file should be a clean, well-structured markdown document — not a transcript of the conversation, but the distilled output: problem statement, core scope, deferred features, constraints, and audiences.

3. **Generate the architecture backlog.** Review the final constraints and misfits and identify which ones imply architectural decisions — choices with genuine tension where multiple reasonable approaches exist. Then:
   - Group the candidate decisions into natural categories (e.g., Foundational → Structural → Operational → Deferred)
   - Name dependency relationships: which decisions gate others?
   - For each item, note known tensions, relevant constraints from the discovery, and any reference sources surfaced during exploration
   - Write to `.claude/adrs/backlog.md` (create the directory if needed) using this structure:

   ```markdown
   # Architecture Backlog

   **Project:** [name]
   **Date:** YYYY-MM-DD
   **Source:** discovery.md

   ## Framing

   [1-2 sentences: what's being built and the key constraints shaping it]

   ## Reference Sources

   - [Any significant resources surfaced during exploration]

   ## Backlog

   ### Foundational

   - **F1: [Decision name]** — [one-line tension]. Depends on: —. Sources: [if any]
   - **F2: [Decision name]** — [one-line tension]. Depends on: F1. Sources: [if any]

   ### Structural

   - **S1: [Decision name]** — [one-line tension]. Depends on: F1, F2. Sources: [if any]

   ### Operational

   - **O1: [Decision name]** — [one-line tension]. Depends on: S1. Sources: [if any]

   ### Deferred

   - **D1: [Decision name]** — [why deferred]
   ```

   Not every exploration will produce a backlog — if the project is small or the constraints don't imply architectural decisions, skip it and note that. Use `AskUserQuestion` to confirm the backlog before saving:

   **Question — Backlog** (header: "Backlog")
   > "Here's the decision landscape I see. Does this capture what needs deciding?"
   - **Looks good, save it** — The backlog captures the right decisions
   - **Skip the backlog** — This project doesn't need one

4. After saving, suggest `/resolve` to begin working through individual decisions: *"The discovery spec is saved [and the architecture backlog maps N decisions to work through]. Want to run `/resolve` to start resolving them?"*

## Example — Full Flow

User: "I want to get a cheap house within 30 min of Mendocino CA. I live in my RV with my wife and cat, limited income, want space for the RV as an in-law suite."

---

**Triage**

*The skill presents three questions via `AskUserQuestion`:*

> **Nature:** What are you trying to do?
> → **Build something new**
>
> **Clarity:** Where are you in your thinking?
> → **I have a direction but I'm not confident it's right**
>
> **Scope:** How big does this feel?
> → **Large or cross-cutting**

*Triage maps to: **Frame check** — the user has a direction (buy a house) but isn't confident. Large scope. Strategy: challenge whether "buy a house" is the right frame, then scope the real constraints.*

---

**Iteration 1 — Probe**

*The skill challenges the frame via `AskUserQuestion`:*

You said "cheap house" — but the core need seems to be *stable, affordable space near Mendocino with room for the RV*. A house is one path, but not the only one.

> **Motivation** (header): What's the primary driver behind buying a house specifically?
> - **Stability / permanence** — Tired of moving; want a fixed home base
> - **Building equity** — Want an appreciating asset, not just a place to live
> - **Both equally** — Stability and equity matter about the same
> - **Something else** *(user selects "Other" to explain)*

→ User selects: **Both equally**

**Constraints / Misfits**
1. Near Mendocino, within 30 min drive
2. Low income — needs to be cheap
3. Lives in RV with wife and cat
4. Needs space to park RV (in-law suite use)
5. Primary motivation: stability + building equity (weighted equally)
6. **Misfit:** "cheap" + "within 30 min of Mendocino" + "buy" — Mendocino is expensive; these three pull hard against each other
7. *Inferred:* may not qualify for a conventional mortgage given limited income

*The skill confirms via `AskUserQuestion`:*

> **Confirm** (header): Does this capture things accurately?
> - **Looks good** — The constraints and misfits are correct as listed
> *(user can also type corrections/additions via "Other")*

→ User selects: **Looks good**

---

**Iteration 2 — Probe**

*The skill asks the next highest-ambiguity question via `AskUserQuestion`:*

"Cheap" is doing a lot of work. Pinning down the budget reshapes which options are realistic.

> **Budget** (header): What does "cheap" mean in numbers for you?
> - **Under $100k** — Land-only territory; very constrained
> - **$100k–$150k** — Possible for land + basic structure in outer counties
> - **$150k–$200k** — Opens up manufactured homes and some fixer-uppers
> - **Not sure — more of a monthly payment concern** — Focused on affordability, not sticker price

→ User selects: **$100k–$150k**

*The skill follows up with another `AskUserQuestion` since "down payment" is tightly coupled:*

> **Down payment** (header): Do you have savings for a down payment?
> - **Yes, $10k–$20k** — Typical for owner-financed or land purchases
> - **Yes, more than $20k** — Opens up more conventional options
> - **Little to none** — Would need zero-down or very low-down options

→ User selects: **Yes, $10k–$20k**

```diff
  1. Near Mendocino, within 30 min drive
- 2. Low income — needs to be cheap
+ 2. Budget: $100k–$150k, with $10k–$20k down payment
+ 8. Low income — needs affordable monthly payments
  3. Lives in RV with wife and cat
  4. Needs space to park RV (in-law suite use)
  5. Primary motivation: stability + building equity (weighted equally)
- 6. Misfit: "cheap" + "within 30 min of Mendocino" + "buy" — Mendocino is expensive; these three pull hard against each other
+ 6. Misfit: $150k ceiling within 30 min of Mendocino is very tight for a house — but feasible for raw land or owner-financed land with a structure
- 7. Inferred: may not qualify for a conventional mortgage given limited income
+ 7. Inferred: conventional mortgage may be difficult; owner financing or land purchase more realistic
```

*The skill confirms via `AskUserQuestion`:*

> **Confirm**: Does this capture things accurately?
> - **Looks good**

→ User selects: **Looks good**

---

**Approach selection**

Given the budget tension, here are structurally different paths. *The skill presents them via `AskUserQuestion`:*

> **Approach** (header): Which direction interests you?
> - **Buy raw land** — ~$30–80k. Park the RV, add utilities. Cheapest path to equity. Bastrop/Elgin/Lockhart.
> - **Owner-financed property** — Avoids bank qualification. Modest house + acreage under $150k in rural TX.
> - **RV/tiny-home community lot** — Buy a lot in a community; gives stability without a house.
> - **Manufactured home on land** — $150k can cover land + new manufactured home in outer ring counties.

→ User selects: **Owner-financed property**

*(Loop continues, probing details of the chosen path...)*

## When NOT to refine

Proceed directly when:
- The request is specific and actionable as stated
- Context provides enough information to infer reasonable defaults
- The user has explicitly said "just do it" or similar
- Asking questions would delay a trivially simple task
