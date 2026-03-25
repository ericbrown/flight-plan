---
description: Save Memory Bank, update README, post Linear session summaries, capture learnings. Run at end of every session.
---

Close out the session cleanly. Execute each step in order.

## Step 1 — Cognitive Briefing

Before writing anything to disk, synthesize in your head:
- What was the main goal this session?
- What approach was taken and why?
- What failed or was abandoned, and why?
- What is the current state — what works, what's incomplete?
- What should the next session do first?
- Any open questions or deferred decisions?

This becomes the `activeContext.md` content. Capture THINKING, not just files changed.

---

## Step 2 — Update Memory Bank

Write or update each file in `.claude/memory/`:

**`activeContext.md`** — Replace entirely (this is the "latest state" snapshot for session-start):
```markdown
# Active Context
Last updated: [date]

## Session goal
[one sentence]

## Approach taken
[2–3 sentences]

## Current state
[what works, what's incomplete, what's broken]

## What failed / was abandoned
[specific — future Claude needs this to avoid repeating the same mistakes]

## Resume prompt
[One paragraph written to a fresh Claude instance: "You're working on X. Last thing
that happened was Y. Your next step is Z. Watch out for W."]
```

**`.claude/memory/sessions/[YYYY-MM-DD-HHmm].md`** — Create a NEW dated file (never overwrite previous sessions):
```markdown
# Session — [date and time]

## Goal
[one sentence]

## What happened
[2-5 bullets: what was built, fixed, or decided]

## What failed / was abandoned
[specific — prevents future sessions from retrying dead ends]

## Decisions made
[architectural or approach decisions with rationale]

## Resume prompt
[same as activeContext — written for a fresh Claude instance]
```
This preserves a complete history of every session. `activeContext.md` is the "latest" pointer; the sessions directory is the full log.

**`progress.md`** — Update status. Move completed items to the Completed section with dates (never delete them):
```markdown
## Completed
- [x] [task] — [one-line result] (completed [date])

## In progress
- [ ] [task] — [current state]

## Up next
- [ ] [task]
```
Items move DOWN the file as they progress: Up next → In progress → Completed. Never remove completed items — they're the project's work history.

**`sessionHistory.md`** — Append (never replace):
```
[date] — [one paragraph: what was done, outcome, what's next]
```

**`conventions.md`** — For every user correction this session, add:
```markdown
### [Plain-language pattern name]
[What happened. What was wrong. What the correct behavior is.]
```
Then evaluate: universal or project-specific?
Universal → also add to "Learned Patterns" in `~/.claude/CLAUDE.md`.

**`decisionLog.md`** — For every significant architectural or approach decision made this session, append:
```markdown
## [date] — [decision title]
**Context**: [why this decision was needed]
**Decision**: [what was decided]
**Rationale**: [why this option over alternatives]
**Alternatives**: [what else was considered and rejected]
```
Skip trivial decisions. Log only things a future Claude instance would need to understand "why is it built this way?"

---

## Step 3 — Update Linear issues

Find all Linear issue IDs in `plan.md` (format: `<!-- linear:PROJ-123 -->`).
Also check `.claude/task-context.md` for any additional issue IDs from this branch.

For each issue touched this session (started, in-progress, or failed):
Spawn `linear-sync` subagent (via Task tool) with:
```
Operation: ADD_SESSION_COMMENT
issue_id: PROJ-123
session_goal: [from Step 1]
current_state: [from Step 1]
resume_prompt: [from Step 1 resume prompt]
```

**Do NOT change any issue status. Issues stay In Progress until you close them.**

---

## Step 4 — Update branch task context

If on a feature branch, update `.claude/task-context.md`:
```markdown
# Task Context: [branch]
Last updated: [date]

## Objective
[one sentence]

## Plan
[paste plan.md Tasks section — checked + unchecked]

## Key decisions
[architectural decisions made this session]

## Current progress
[what's done, what's next]

## Resume here
[same resume prompt as activeContext.md — written to a fresh Claude instance]
```

Stage and commit:
`git add .claude/task-context.md && git commit -m "chore: update task context [date]"`

---

## Step 5 — Update README

Check if any of these changed this session:
- New features or commands added
- Configuration or setup steps changed
- New dependencies added
- Architecture changed

If yes: update `README.md`. Commit alongside the memory update.

---

## Step 6 — Verify nothing uncommitted

Run `git status`. If there are uncommitted changes:
- WIP code → offer to stash: `git stash push -m "wip: session end [date]"`
- Complete changes → offer to commit
- Memory/context files → stage and commit them

---

## Step 7 — Session summary

Output:
```
## Session End

Memory Bank:   saved (activeContext, progress, sessionHistory, conventions)
Linear:        session-end comments added to [N] issues (still open — close when ready)
Conventions:   [N] new / "none"
README:        updated / "no changes needed"
Branch ctx:    saved + committed / "not on feature branch"
Git:           clean / [N] changes stashed or committed

Next session: /session-start
```

---

## Step 8 — Lesson sync reminder

Output:
```
To sync learned patterns across machines:
  cd ~/Documents/claude-workflow
  ./sync-lessons.sh
  git add CLAUDE.md && git commit -m "sync lessons" && git push
```
