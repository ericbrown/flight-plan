# Flight Plan

A workflow system for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that adds persistent memory, structured planning, auto-triggering skills, and Linear integration.

---

## What's in here

| Component | What it does |
|-----------|--------------|
| **Commands** | 20 slash commands for planning, verification, git workflow, and session management |
| **Skills** | Auto-triggering behaviors (TDD, systematic debugging, verification gates) |
| **Memory Bank** | Persistent project context that survives across sessions |
| **Hooks** | Pre/post tool-use guards (destructive op protection, auto-formatting) |
| **Linear sync** | Automatic issue creation and progress updates |

---

## Install

```bash
git clone https://github.com/ericbrown/flight-plan.git ~/flight-plan
cd ~/flight-plan
./install.sh
```

Safe to re-run. Backs up existing config, merges learned patterns, preserves machine-specific settings.

**Update anytime:**
```bash
cd ~/flight-plan && git pull && ./install.sh
```

---

## Getting started

### First time in a project

```bash
cd /your/project
claude
/memory-init       # one-time: creates .claude/memory/ for persistent context
```

### Every session

```bash
/session-start     # loads context, shows project state
# ... do your work ...
/session-end       # saves context, syncs to Linear
```

---

## Core concepts

### Memory Bank

Each project gets `.claude/memory/` — persistent context that Claude reads at session start:

| File | Purpose |
|------|---------|
| `projectContext.md` | Architecture, stack, key entry points |
| `activeContext.md` | Current goal, approach, what to try next |
| `progress.md` | Done / in progress / up next |
| `decisionLog.md` | Architecture decisions with rationale |
| `conventions.md` | Project-specific patterns (auto-updated) |

Run `/memory-init` once per project to create these.

### Skills (auto-triggering)

No commands needed — these activate based on context:

| Skill | Activates when... |
|-------|------------------|
| `tdd` | Implementing new functions, classes, or endpoints |
| `systematic-debugging` | Tests fail or build errors |
| `verification-before-completion` | About to mark a task done |
| `finishing-a-branch` | All tasks complete, ready for PR |
| `worktrees` | Running parallel sessions on the same repo |

### Planning pipeline

For complex tasks, use the 3-stage planning pipeline:

```
/plan:context    →  scope + affected files (approval gate)
/plan:tasks      →  tasks in batches of 5 (annotation gate)
/execute         →  one subagent per task, Linear updated throughout
```

Or use `/boris <task>` to run all three stages with approval gates.

### Modes

Switch Claude's behavior for different phases of work:

| Mode | Behavior |
|------|----------|
| `/mode architect` | Read-only design — no file edits |
| `/mode code` | Full implementation (default) |
| `/mode debug` | Investigation, hypothesis-driven |
| `/mode review` | Strictly read-only review |
| `/mode audit` | Security scanning with logging |

---

## All commands

### Session
| Command | What it does |
|---------|-------------|
| `/session-start` | Load Memory Bank, orient to project |
| `/session-end` | Save context, update Linear, capture lessons |
| `/context` | Show context window usage + Memory Bank status |
| `/handoff` | Save cognitive state for cross-session handoff |
| `/memory-init` | Initialize Memory Bank (once per project) |

### Planning
| Command | What it does |
|---------|-------------|
| `/boris <task>` | Full workflow: plan → execute → verify → PR |
| `/plan:context` | Stage 1: scope + affected files |
| `/plan:tasks` | Stage 2: tasks in batches, Linear issues created |
| `/execute` | Stage 3: run tasks via subagents |

### Quality
| Command | What it does |
|---------|-------------|
| `/verify` | typecheck → tests → lint → build |
| `/test-and-fix` | Run tests, fix failures iteratively |
| `/simplify` | Clean up code after implementation |
| `/review-changes` | Pre-commit review of uncommitted changes |

### Git
| Command | What it does |
|---------|-------------|
| `/task-branch <name>` | Create feature branch + task context |
| `/task-done` | Verify, PR, clean up task context |
| `/commit-push-pr` | Stage, commit, push, create PR |
| `/checkpoint [name]` | Create named save point |
| `/rollback [target]` | Restore checkpoint or go back N commits |
| `/undo` | Revert last commit safely |

### Linear
| Command | What it does |
|---------|-------------|
| `/linear-update` | Add progress comment to current issue |
| `/linear-update <ID>` | Comment on specific issue |
| `/linear-update plan` | Push plan to all issues |

---

## Hooks

| Hook | Trigger | What it does |
|------|---------|--------------|
| `destructive-ops-guard` | Before `git reset --hard`, `rm -rf`, force-push | Creates checkpoint, stashes dirty files |
| `post-edit-formatter` | After file edits | Auto-formats by file type |

---

## Linear integration

When using the planning commands, every task automatically gets a Linear issue:

- Issues created during `/plan:tasks`
- Moved to In Progress when `/execute` starts each task
- Progress comments at each stage
- Session summary posted by `/session-end`
- Issues are never auto-closed — only you close them

Linear team/project IDs are saved to `.claude/project-config.json` after first use.

---

## Parallel sessions

Run multiple Claude Code sessions using git worktrees:

```bash
# Create parallel worktrees
git worktree add ~/<project>-2 -b feature/new-api
git worktree add ~/<project>-3 -b fix/some-bug

# Each worktree gets its own terminal + Claude session
# No conflicts, no stashing needed
```

See the `worktrees` skill for full setup.

---

## Syncing learned patterns

Lessons accumulate in `~/.claude/CLAUDE.md` as you correct Claude. Sync them across machines:

```bash
cd ~/flight-plan
./sync-lessons.sh              # local → repo
git add CLAUDE.md && git commit -m "sync lessons" && git push

# On another machine:
git pull && ./sync-lessons.sh --pull
```

---

## File structure

```
flight-plan/
  CLAUDE.md              # Global instructions + learned patterns
  install.sh             # Installer (safe to re-run)
  sync-lessons.sh        # Lesson sync across machines
  settings.base.json     # Hook config + permissions

  commands/              # Slash commands (20)
  skills/                # Auto-triggering behaviors (5)
  agents/                # Subagent definitions
  hooks/                 # Lifecycle hooks
  memory-template/       # Starter files for /memory-init
```

---

## License

MIT
