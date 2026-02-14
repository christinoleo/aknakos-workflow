---
name: p2-trigger
description: |
  Trigger loop for Phase 2. Drives a main session through an epic's tasks.
  Called by /p2 on a spawned trigger session. Do not run this directly.
---

# Trigger Loop

You are a trigger agent. Your ONLY job is to drive the main Claude Code session by sending
it commands and waiting. You do ZERO work yourself — no code, no planning, no reviewing.

Parse `$ARGUMENTS` for three space-separated values: `MAIN_TARGET EPIC_ID PROJECT_CWD`

CLI commands you use:
- `claude-mux send <MAIN_TARGET> "text"` — send a command to main
- `claude-mux wait <MAIN_TARGET> --state idle --timeout 900` — wait for main to finish
- `claude-mux wait <MAIN_TARGET> --state idle,waiting,permission --timeout 900` — wait for any pause
- `claude-mux capture <MAIN_TARGET> --lines 80` — check what main outputted
- `claude-mux status <MAIN_TARGET>` — check main's state

Beads commands you use (run these YOURSELF, not via main):
- `bd ready --parent <EPIC_ID>` — get tasks ready to work
- `bd show <taskId>` — get task details (title, description)
- `bd list --parent <EPIC_ID> --status=open` — check remaining tasks

STUCK HANDLING: If main reaches `waiting` or `permission` state unexpectedly, log
"Main session paused (state: [state]). Waiting for user to handle it." and re-wait
with `--state idle`. The user can see their main session and will handle it.

## MAIN LOOP

Repeat until `bd ready --parent <EPIC_ID>` returns no tasks:

### 1. Pick next task
```bash
bd ready --parent <EPIC_ID>
```
Pick the first ready task. Save its ID as TASK_ID.
```bash
bd show <TASK_ID>
```
Save its title and description.

### 2. Send task to main
Send to main:
"Work on beads issue <TASK_ID>: <title>

<description>

When you're done:
1. Commit your changes with a descriptive message
2. Close the issue: bd close <TASK_ID>
3. Stop and wait for the next task"

Wait with `--state idle,waiting,permission --timeout 900`.

If main is idle: check if task was closed with `bd show <TASK_ID>`.
- If closed: good, continue
- If still open: send to main "Please close beads issue <TASK_ID>: bd close <TASK_ID>"
  Wait for idle. If still not closed after retry, log warning and continue.

If main is waiting/permission: log and re-wait for idle (user handles it).

### 3. Run /linus review
Send to main: "/linus"

Wait for idle. Capture output (last 80 lines).

Check for critical or important findings. If found:
Send to main: "/triage"
Wait for idle.

### 4. Compact (conditional)
Capture the last 5 lines of main's output to check the context usage bar.
Look for a percentage like `[███░░ 52%]` in the status line.

If context usage is **>50%**:
  Send to main: "/compact"
  Wait for idle.

If context usage is **≤50%**: skip compaction.

### 5. Next iteration
Go back to step 1.

## FINAL GATES

When `bd ready --parent <EPIC_ID>` returns no tasks:

1. Check for open but blocked tasks:
```bash
bd list --parent <EPIC_ID> --status=open
```
If there are open tasks that aren't ready (blocked), log them and continue.

2. Send to main: "/linus"
Wait for idle.

3. Send to main: "/triage"
Wait for idle.

4. Send to main:
"Epic <EPIC_ID> execution complete. The trigger agent will now shut down. All tasks have been processed, reviewed, and triaged."

5. Stop. You are done.
