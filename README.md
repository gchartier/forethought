# Forethought

A four-stage skill pipeline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that turns vague ideas into implemented code — with structured thinking at every step.

Instead of jumping straight to building, Forethought walks you through defining the problem, making architectural decisions, planning the work, and then executing task by task. Each stage produces a durable artifact that the next stage consumes, so nothing gets lost between "I have an idea" and "it's built."

## The four stages

```
/explore  →  /resolve  →  /blueprint  →  /next
 define       decide       plan          build
```

- **`/explore`** — Asks clarifying questions to turn a vague request into a sharp problem statement. Challenges your assumptions, surfaces constraints, and saves a discovery spec.
- **`/resolve`** — Maps out architectural decisions and works through them one by one. Each decision is recorded as an ADR (Architectural Decision Record) that captures the tensions, alternatives, and rationale.
- **`/blueprint`** — Reads the discovery spec and ADRs, then produces a concrete task breakdown. Every task traces back to a specific decision or requirement — nothing is invented.
- **`/next`** — Picks up the next unblocked task, loads its context, implements it, verifies acceptance criteria, and updates the plan. Run it repeatedly until done.

For a deeper look at how the stages connect, see [pipeline-overview.md](pipeline-overview.md).

## Installation

Forethought skills need to live where Claude Code can find them. There are two ways to set this up.

### Option A: Install globally (recommended)

This makes the skills available in every project on your machine.

1. Clone this repo anywhere you like:
   ```
   git clone https://github.com/gchartier/forethought.git ~/forethought
   ```

2. Create the Claude skills directory if it doesn't exist:
   ```
   mkdir -p ~/.claude/skills
   ```

3. Symlink each skill into that directory:
   ```
   ln -s ~/forethought/explore ~/.claude/skills/explore
   ln -s ~/forethought/resolve ~/.claude/skills/resolve
   ln -s ~/forethought/blueprint ~/.claude/skills/blueprint
   ln -s ~/forethought/next ~/.claude/skills/next
   ```

That's it. The skills are now available whenever you use Claude Code.

### Option B: Install for a single project

This keeps the skills scoped to one project.

1. Clone this repo anywhere you like:
   ```
   git clone https://github.com/gchartier/forethought.git ~/forethought
   ```

2. From your project directory, symlink or copy the skills:
   ```
   mkdir -p .claude/skills
   ln -s ~/forethought/explore .claude/skills/explore
   ln -s ~/forethought/resolve .claude/skills/resolve
   ln -s ~/forethought/blueprint .claude/skills/blueprint
   ln -s ~/forethought/next .claude/skills/next
   ```

## Usage

Start Claude Code in your project, then invoke each stage as a slash command.

### 1. `/explore` — when you have a fuzzy idea

Just describe what you want to do. Explore will ask you questions — answer them until the problem is sharp. It saves the result to `.claude/plans/discovery.md`.

### 2. `/resolve` — when you're ready to make decisions

Start with MAP mode to survey the full landscape of decisions, then work through them individually. Each decision is saved as an ADR in `.claude/adrs/`.

### 3. `/blueprint` — when decisions are made

Blueprint reads everything produced so far and generates a task breakdown in `.claude/plans/blueprint.md`. Review it, adjust if needed, and you're ready to build.

### 4. `/next` — when you're ready to build

Run `/next` to start the first task. When it's done, run `/next` again for the next one. Each invocation picks up where the last left off.

### What gets created in your project

```
your-project/
└── .claude/
    ├── plans/
    │   ├── discovery.md    ← from /explore
    │   └── blueprint.md    ← from /blueprint
    └── adrs/
        ├── backlog.md      ← from /resolve (MAP mode)
        ├── index.md        ← from /resolve
        └── 001-*.md        ← individual decisions from /resolve
```

These are working files for your planning process. Add `.claude/` to your `.gitignore` if you don't want them in version control, or commit them if you want the decision history alongside your code.

## License

[GNU GPL v3.0](LICENSE.md)
