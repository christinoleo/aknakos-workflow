# aknakos-workflow

Personal Claude Code plugin marketplace — dev workflow commands and hooks.

## Commands

| Command | Description |
|---------|-------------|
| `/bcheck` | Pick up a beads issue, assess context sufficiency, start coding or enter plan mode |
| `/linus` | Linus Torvalds-style code review with sub-agent investigation |
| `/triage` | Walk through review findings point-by-point, decide and execute fixes |
| `/verify` | Browser-based verification of changes using Chrome DevTools |
| `/replan` | Audit plan coverage against requirements, rewrite to 100% |
| `/redelta` | Audit delivered work against requirements, produce delta to reach 100% |
| `/plan-to-beads` | Convert approved plan into beads epic with tasks and dependencies |
| `/p1` | Phase 1 orchestration: plan → replan → plan-to-beads (spawns trigger agent) |
| `/p1-trigger` | Trigger loop for Phase 1 (called by /p1, do not run directly) |
| `/p2` | Phase 2 orchestration: execute epic tasks with review between each (spawns trigger agent) |
| `/p2-trigger` | Trigger loop for Phase 2 (called by /p2, do not run directly) |
| `/pilot` | Launch and control any TUI app (lazygit, btop, ncdu, etc.) inside a tmux session |

## Hooks

**workflow-hints.sh** — PreToolUse hook that injects next-step context:
- After `EnterPlanMode`: reminds to run `/replan` before exiting plan mode
- After `/linus`: suggests `/triage`
- After `/triage`: suggests `/verify`
- After `/replan`: suggests `/plan-to-beads`

## Install

```bash
/plugin marketplace add aknakos/aknakos-workflow
/plugin install aknakos-workflow@aknakos-workflow
```

## Edit & Push

Edit commands directly in `plugin/commands/`, then:

```bash
cd ~/Projects/aknakos-workflow
git add -A && git commit -m "update: description" && git push
```

On other machines, the marketplace auto-updates on next Claude Code start (or run `/plugin marketplace update aknakos-workflow`).

## Structure

```
aknakos-workflow/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace catalog
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json        # Plugin manifest
│   ├── commands/               # All slash commands
│   │   ├── bcheck.md
│   │   ├── linus.md
│   │   ├── p1.md
│   │   ├── p1-trigger.md
│   │   ├── p2.md
│   │   ├── p2-trigger.md
│   │   ├── plan-to-beads.md
│   │   ├── redelta.md
│   │   ├── replan.md
│   │   ├── triage.md
│   │   ├── verify.md
│   │   └── pilot.md
│   └── hooks/
│       ├── hooks.json          # Hook configuration
│       └── workflow-hints.sh   # Workflow hint injection
└── README.md
```
