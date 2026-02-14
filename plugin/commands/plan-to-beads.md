---
name: plan-to-beads
description: |
  Convert an approved plan into a Beads epic with tasks. Detects parallel vs sequential
  phases from plan structure and sets dependencies accordingly. Run after /replan approval.
---

# Plan → Beads: Convert Plan to Epic + Tasks

Convert the current approved plan into a Beads epic with properly structured tasks and
dependencies. This bridges the gap between "we know what to do" and "let's track execution."

## Step 1: Find the Plan

Locate the plan file:
- Check the most recent plan: `ls -t ~/.claude/plans/*.md | head -1`
- If argument provided (`$ARGUMENTS`), use that path instead
- Read the plan file completely

If no plan file exists, check the conversation history for the most recently approved plan
(from ExitPlanMode). Use that content directly.

## Step 2: Parse the Plan

Extract from the plan:

1. **Title** — First heading or `# Plan:` line
2. **Summary** — Content under `## Summary` or the first paragraph
3. **Phases/Steps** — Each `### Phase N:`, `### N.`, or numbered section
4. **Files affected** — Any file paths mentioned (include in epic description)
5. **Dependency structure** — Determine what depends on what:

### Detecting Parallel vs Sequential

- **Sequential indicators:** "after", "then", "once X is done", "depends on", numbered order
  with no contrary signals
- **Parallel indicators:** "simultaneously", "independently", "can be done in parallel",
  tasks in different domains (e.g., frontend + backend), tasks touching unrelated files
- **When ambiguous:** Default to sequential, but flag it and ask the user

Build a dependency graph, not just a linear chain.

## Step 3: Create the Epic

```bash
bd create "[Plan Title]" -t epic -p 1 -d "[summary]. Files: [list affected files]" --json
```

Capture the epic ID from the JSON output.

## Step 4: Create Tasks

For each phase/step in the plan:

```bash
bd create "[Phase title]" -t task -p 2 -d "[first paragraph of phase content]" --json
```

- Use the first paragraph as the description (keep it scannable)
- Preserve any acceptance criteria or specific requirements in the description
- If a phase has sub-steps, include them as a checklist in the description

## Step 5: Set Dependencies

Based on the dependency analysis from Step 2:

```bash
# Sequential: each phase depends on the previous
bd dep add <phase2-id> <phase1-id>

# Parallel: independent tasks have no dependency between them
# but may share a common predecessor
bd dep add <phase3-id> <phase1-id>
bd dep add <phase4-id> <phase1-id>
# phase3 and phase4 can run in parallel after phase1

# All tasks are children of the epic
bd dep add <epic-id> <task-id>
```

## Step 6: Report

Output a clear summary:

```
Created from: [filename or "conversation plan"]

Epic: [title] ([epic-id])
  ├── [Phase 1] ([id]) — ready
  ├── [Phase 2] ([id]) — blocked by [phase1-id]
  ├── [Phase 3] ([id]) — ready (parallel with Phase 4)
  ├── [Phase 4] ([id]) — ready (parallel with Phase 3)
  └── [Phase 5] ([id]) — blocked by [phase3-id], [phase4-id]

Dependency graph:
  Phase 1 → Phase 2
  Phase 1 → Phase 3 ┐
  Phase 1 → Phase 4 ┤→ Phase 5
                     ┘

Total: [N] tasks ([M] ready, [K] blocked)
Run `bd ready` to start working.
```

## Rules

- Original plan file is preserved — never modify or delete it
- Task descriptions use first paragraph only unless there are critical details
- When in doubt about parallel vs sequential, ASK — wrong dependencies waste time
- If the plan has no clear phases (just a wall of text), break it into logical chunks and
  confirm the breakdown with the user before creating issues
