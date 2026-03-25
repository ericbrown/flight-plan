# claude-workflow

Personal Claude Code workflow — persistent memory, Boris-style planning, Linear integration, and auto-skills.

**The main entry point:** `/boris <task>` — describe what to build, answer a couple of questions, say **"go"**. Everything else runs automatically.

---

## Install

```bash
git clone https://github.com/YOUR_USERNAME/claude-workflow.git ~/Documents/claude-workflow
cd ~/Documents/claude-workflow
chmod +x install.sh sync-lessons.sh
./install.sh
```

Safe to re-run. Backs up existing config, merges learned patterns, preserves machine-specific settings.

**Update anytime:**
```bash
cd ~/Documents/claude-workflow && git pull && ./install.sh
```

---

## First time in a project

```bash
cd /your/project
claude
/memory-init       # one-time setup: creates .claude/memory/ with persistent context
/session-start     # loads context, presents summary, enters plan mode
```

---

## Daily workflow

### The short version

```
/session-start               # load context
/boris implement <feature>   # plan + "go" + execute + verify + PR
/session-end                 # save everything
```

### The full picture

| Step | Command | What happens |
|------|---------|--------------|
| Start | `/session-start` | Reads Memory Bank, presents project summary, enters plan mode |
| Describe task | `/boris <task>` | Scopes the work, writes plan.md in batches, creates Linear issues |
| Review plan | *(annotate plan.md)* | Add `<!-- skip -->`, `<!-- use X instead -->` inline comments |
| Execute | say **"go"** | Runs each task via subagents, updates Linear, commits as it goes |
| Verify | *(auto-runs)* | typecheck → full tests → lint → build |
| Ship | *(auto-runs)* | Creates PR, posts PR link to Linear issues |
| End | `/session-end` | Saves Memory Bank, posts session summary to Linear, captures lessons |

---

## Why "go"?

`/boris` runs two approval gates before touching any code:

1. **After scope** — "Scope looks right? Any corrections?" → you review affected files
2. **After plan** — "Plan complete. Annotate then say 'go'" → you annotate, then say go

After "go" it runs autonomously: subagent per task, Linear updated at each stage, reverts on failure, never patches bad approaches.

---

## The planning hang fix

`/plan:tasks` writes tasks in **batches of 5** with an explicit stop after each batch. The model terminates cleanly at each batch boundary rather than trying to generate an entire plan in one unbounded pass — which is why superpowers' `/write-plan` hung on complex projects.

---

## Memory Bank

Each project gets `.claude/memory/` — run `/memory-init` once:

| File | Contents |
|------|----------|
| `projectContext.md` | What the project is, architecture, stack, key entry points |
| `activeContext.md` | Current state: goal, approach, what failed, resume prompt |
| `progress.md` | Tasks: done / in progress / up next |
| `decisionLog.md` | Architecture decisions with rationale |
| `conventions.md` | Project-specific corrections, auto-updated during sessions |
| `sessionHistory.md` | Rolling session summaries (append-only) |

Plus `.claude/task-context.md` on each feature branch — **committed to git** for cross-machine handoff. `git pull` on any machine → Claude has full context.

---

## Linear integration

Every task in every plan automatically gets a Linear issue. No manual issue creation needed.

| Event | What happens in Linear |
|-------|----------------------|
| `/plan:tasks` creates a task | Issue created with description + plan + acceptance criteria |
| `/execute` starts a task | Issue moved to **In Progress** + starting comment |
| Test written (red) | Comment: "Test written, confirmed failing" |
| Implementation written | Comment: "Implementation written, running verification" |
| Task passes verification | Comment: "Verified, committed [hash]" |
| Task blocked / fails | Comment: exact error output + "reverted" |
| PR created | Comment: PR link on all completed issues |
| `/session-end` | Comment: session summary + resume prompt on all active issues |
| You | **Only you close issues** — nothing in this workflow ever closes them |

Linear team + project IDs are resolved on first `/plan:tasks` run and saved to `.claude/project-config.json`.

---

## All commands

### Main
| Command | What it does |
|---------|-------------|
| `/boris <task>` | Full orchestrated workflow end-to-end |
| `/session-start` | Load Memory Bank, orient to project, enter plan mode |
| `/session-end` | Save context, update Linear, remind about lesson sync |

### Planning (3-stage pipeline)
| Command | What it does |
|---------|-------------|
| `/plan:context` | Stage 1: scope + affected files, approval gate |
| `/plan:tasks` | Stage 2: tasks in batches of 5, Linear issues created, annotation gate |
| `/execute` | Stage 3: subagent per task, Linear updated throughout |

### Quality
| Command | What it does |
|---------|-------------|
| `/verify` | typecheck → tests → lint → build — full suite |
| `/test-and-fix` | Run tests, fix failures iteratively until green |
| `/simplify` | Code-simplifier subagent — clean up after implementation |
| `/review-changes` | Read-only pre-commit review: correctness, coverage, conventions |

### Git
| Command | What it does |
|---------|-------------|
| `/task-branch <name>` | Create feature branch + task context |
| `/task-done` | Verify, PR, remove task context, update Memory Bank |
| `/commit-push-pr` | Stage, commit, push, create PR |
| `/checkpoint [name]` | Create named git tag save point |
| `/rollback [target]` | Restore checkpoint or go back N commits |
| `/undo` | Revert last Claude-made commit safely |

### Context & memory
| Command | What it does |
|---------|-------------|
| `/memory-init` | Initialize Memory Bank (run once per project) |
| `/context` | Context window usage + Memory Bank + plan status |
| `/handoff` | Cognitive briefing — saves mental model for cross-session handoff |

### Modes (Boris-style)
| Command | What it does |
|---------|-------------|
| `/mode architect` | Read-only design mode — no file edits |
| `/mode code` | Full implementation (default) |
| `/mode debug` | Investigation, hypothesis-driven, limited writes |
| `/mode review` | Strictly read-only code review |
| `/mode audit` | Security scanning with logging |

### Linear
| Command | What it does |
|---------|-------------|
| `/linear-update` | Add progress comment to current task's issue |
| `/linear-update PROJ-123` | Add comment to a specific issue |
| `/linear-update plan` | Push revised plan.md to all issues |

---

## Auto-triggering skills

No commands needed — these activate automatically:

| Skill | Activates when... |
|-------|------------------|
| `tdd` | Implementing any new function, class, or endpoint |
| `systematic-debugging` | A test fails or build errors out |
| `worktrees` | Running parallel sessions on the same repo |

---

## Hooks

| Hook | Trigger | What it does |
|------|---------|--------------|
| `destructive-ops-guard` | Before `git reset --hard`, `rm -rf`, force-push | Creates checkpoint tag, stashes dirty files |
| `post-edit-formatter` | After any file edit | Auto-formats: prettier / ruff / gofmt by extension |

---

## Parallel fleet (Boris-style)

Run 3–5 sessions simultaneously using git worktrees:

```bash
# From your main project directory:
# Example: create parallel worktrees for your project
git worktree add ~/<project>-2 -b feature/new-api
git worktree add ~/<project>-3 -b fix/some-bug

# Each worktree gets its own Claude Code session (separate terminal tab)
# Each tab works on a different branch — no conflicts, no stashing needed
```

Conventions: `~/[repo]-2`, `~/[repo]-3` etc. Each gets its own `.claude/task-context.md`.
See the `worktrees` skill for full setup.

---

## Syncing learned patterns across machines

Lessons accumulate in `~/.claude/CLAUDE.md` as Claude makes mistakes and you correct them.
Sync them so all machines benefit:

```bash
# After any work session:
cd ~/Documents/claude-workflow
./sync-lessons.sh                                      # local → repo
git add CLAUDE.md && git commit -m "sync lessons" && git push

# On another machine (local or Ubuntu server via Tailscale):
git pull && ./sync-lessons.sh --pull                   # repo → local

# Both directions at once:
./sync-lessons.sh --both
```

Deduplicates by `### heading` — safe to run repeatedly, never removes existing lessons.

---

## File structure

```
claude-workflow/
  CLAUDE.md              Global instructions + learned patterns (the brain)
  install.sh             Installer — safe to re-run
  sync-lessons.sh        Bidirectional lesson sync across machines
  settings.base.json     Hook wiring + permissions
  .gitignore

  commands/              Slash commands (19 total)
    boris.md             ← main entry point
    session-start.md
    session-end.md
    plan-context.md
    plan-tasks.md
    execute.md
    verify.md
    test-and-fix.md
    simplify.md
    review-changes.md
    task-branch.md
    task-done.md
    commit-push-pr.md
    checkpoint.md
    rollback.md
    undo.md
    context.md
    handoff.md
    mode.md
    linear-update.md

  agents/
    linear-sync.md       Linear MCP agent — all issue operations

  skills/                Auto-triggering skills (no command needed)
    tdd.md
    systematic-debugging.md
    worktrees.md

  hooks/                 Lifecycle hooks
    destructive-ops-guard.sh
    post-edit-formatter.sh

  memory-template/       Starter files used by /memory-init
    projectContext.md
    activeContext.md
    progress.md
    conventions.md
```
