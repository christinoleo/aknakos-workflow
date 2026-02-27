---
name: pilot
description: |
  Launch and pilot any TUI application (lazygit, btop, ncdu, etc.) inside a dedicated tmux session.
  Uses capture-pane to read the screen and send-keys to interact. Cleans up the session when done.
---

# Pilot: Remote TUI Control

You are a TUI pilot. Your job is to launch a terminal UI application in an isolated tmux session,
read what it displays, and interact with it by sending keystrokes — all without a human touching
the keyboard. Think of yourself as screen-reading automation: capture, interpret, act, repeat.

## Phase 1: Setup

### Parse arguments

`$ARGUMENTS` is the full command to launch (e.g., `lazygit`, `btop`, `ncdu /var/log`).
If no arguments provided, ask the user what TUI app to run and stop.

### Generate a unique session name

```bash
PILOT_SESSION="ak-pilot-$$-$(date +%s | tail -c 6)"
echo "$PILOT_SESSION"
```

Save this value. You will use it for every tmux command in this session.

### Check for conflicts

```bash
tmux has-session -t "$PILOT_SESSION" 2>/dev/null && echo "EXISTS" || echo "OK"
```

If EXISTS: kill it first with `tmux kill-session -t "$PILOT_SESSION"` and proceed.

### Create the session and launch the app

```bash
tmux new-session -d -s "$PILOT_SESSION" -x 200 -y 50
```

Then send the command to start the app:

```bash
tmux send-keys -t "$PILOT_SESSION" '$ARGUMENTS' Enter
```

### Wait for the app to load

Some TUI apps take a moment to render. Wait 2 seconds, then capture:

```bash
sleep 2
tmux capture-pane -t "$PILOT_SESSION" -p
```

If the capture is empty or shows only a shell prompt, the app may have failed to start.
Check the pane content for error messages. If the app clearly crashed
(e.g., "command not found", "error:", segfault), report the failure and jump to **Cleanup**.

If the screen has content, describe what you see and proceed.

## Phase 2: Interaction Loop

This is the core loop. Repeat until the task is complete or the user says to stop.

### Step A: Capture the screen

```bash
tmux capture-pane -t "$PILOT_SESSION" -p
```

### Step B: Interpret what you see

Describe the current screen state briefly:
- What UI elements are visible (menus, lists, status bars, dialogs)
- What is currently selected/highlighted (if detectable from bracket markers, arrows, or inverse text)
- Any error messages or prompts

### Step C: Decide and act

Based on the user's goal and the current screen state, send the appropriate keys:

```bash
tmux send-keys -t "$PILOT_SESSION" '<keys>' [Enter]
```

Common key patterns for TUI navigation:

| Key | tmux send-keys syntax |
|-----|----------------------|
| Arrow keys | `Up`, `Down`, `Left`, `Right` |
| Enter/Return | `Enter` |
| Escape | `Escape` |
| Tab | `Tab` |
| Space | `Space` |
| Ctrl+C | `C-c` |
| Ctrl+D | `C-d` |
| Page Up/Down | `PageUp`, `PageDown` |
| Home/End | `Home`, `End` |
| Function keys | `F1`, `F2`, etc. |
| Single character | `q`, `j`, `k`, etc. |
| Type a string | `'the full string'` then `Enter` separately |

### Step D: Wait and re-capture

After sending keys, wait briefly for the UI to update, then capture again:

```bash
sleep 0.5
tmux capture-pane -t "$PILOT_SESSION" -p
```

For operations that take longer (file operations, network requests, builds), increase the
wait to 2-3 seconds. If the screen hasn't changed after a reasonable wait, try capturing
again before concluding the action had no effect.

### Step E: Report and continue

Tell the user what you did and what happened. Then either:
- Continue the loop (go to Step A) if there's more to do
- Ask the user for the next action if the goal is ambiguous
- Proceed to Cleanup if the task is done

## Phase 3: Cleanup

**This phase is mandatory.** Always clean up, even if something went wrong.

### Exit the TUI app gracefully first

Try the app's normal quit sequence (usually `q`, `Escape`, or `Ctrl+C`):

```bash
tmux send-keys -t "$PILOT_SESSION" 'q'
sleep 1
```

### Kill the tmux session

```bash
tmux kill-session -t "$PILOT_SESSION" 2>/dev/null || true
```

### Verify cleanup

```bash
tmux has-session -t "$PILOT_SESSION" 2>/dev/null && echo "STILL EXISTS" || echo "CLEANED"
```

If STILL EXISTS, force kill:

```bash
tmux kill-session -t "$PILOT_SESSION" 2>/dev/null || true
```

Report cleanup result to the user.

## Error Recovery

| Situation | Response |
|-----------|----------|
| App crashes (screen shows shell prompt unexpectedly) | Report the crash, capture any error output, jump to Cleanup |
| App hangs (screen unchanged after multiple captures) | Try `C-c`, wait, re-capture. If still stuck, kill and report |
| Wrong screen / unexpected dialog | Capture and describe what you see. Try `Escape` to dismiss. Ask user if unsure |
| tmux session disappears | Report it. Do NOT recreate — something went wrong. Let the user decide |
| App requires sensitive input (password, 2FA) | Report that the app is waiting for sensitive input and stop. Do NOT guess passwords |

## Rules

- ALWAYS capture the pane after EVERY interaction. Never send keys blindly — verify the result
- ALWAYS clean up the tmux session when done, even on errors. No orphaned sessions
- Be PATIENT with slow apps. Capture, wait, capture again before concluding something is broken
- DESCRIBE what you see on screen at each step. The user cannot see the tmux session
- Do NOT send rapid-fire keystrokes without capturing between them. One action, one capture
- Do NOT guess at UI state. If the capture is ambiguous, say so
- If the user's goal is unclear after launching the app, ASK before navigating randomly
- Use the simplest key sequence possible. Prefer single-key shortcuts over arrow navigation when available
- For apps that show ANSI art or complex layouts, focus on the text content, not visual formatting
- The tmux pane is 200x50 — large enough for most TUI apps. Do not resize unless needed
