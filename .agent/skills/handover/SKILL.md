---
name: handover
description: Generate a HANDOVER.md shift-change report capturing the full session context — what was done, what worked, what didn't, decisions made, lessons learned, next steps, and a file map — so the next Claude session can resume without losing anything.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(find:*), Bash(cat:*), Bash(head:*), Bash(tail:*), Bash(wc:*)
disable-model-invocation: true
---

# Handover Report Generator

You are writing a **shift-change report** called `HANDOVER.md`. Its sole purpose is to let the next Claude session pick up exactly where this one left off — with zero context loss.

## Step 1 — Gather Context

Before writing anything, silently collect project state:

- Git status: !`git status --short`
- Recent commits this session: !`git log --oneline -15`
- Changed files: !`git diff --stat`
- Unstaged changes: !`git diff --name-only`
- Project structure (top-level): !`find . -maxdepth 2 -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/dist/*' -not -path '*/__pycache__/*' | head -80`

## Step 2 — Review the Conversation

Look back through **everything** we discussed and did in this session. Identify:

1. The original goal / task the user came in with
2. Every action taken (files created, edited, deleted, commands run)
3. Problems encountered and how they were resolved
4. Decisions made and the reasoning behind them
5. Things that were tried but didn't work (and why)
6. Anything that was discussed but NOT yet implemented
7. Open questions or ambiguities

## Step 3 — Write HANDOVER.md

Generate `HANDOVER.md` in the project root using this structure:

```markdown
# Handover — [Brief Project/Task Name]

**Session date:** [today's date]
**Session summary:** [1-2 sentence TL;DR of what happened]

---

## What We Were Working On

[Describe the goal, motivation, and scope of the work in this session. Give enough context that someone with no prior knowledge understands the "why".]

## What Got Done

[Concrete list of accomplishments. Reference specific files, functions, configs changed. Be precise.]

## What Didn't Work (and How We Fixed It)

[Bugs, errors, failed approaches. Include the root cause and the fix. This is one of the most valuable sections — don't skip it.]

## Key Decisions & Rationale

[Architecture choices, library picks, trade-offs, naming conventions — anything where a judgment call was made. Explain WHY, not just what.]

## Lessons Learned & Gotchas

[Surprising behaviors, footguns, environment quirks, undocumented API behavior, things the next Claude should watch out for.]

## Next Steps

[Explicit, actionable tasks remaining. Each item should be clear enough to execute without asking follow-up questions. Include priority or order if it matters.]

- [ ] Task 1 — why and how
- [ ] Task 2 — why and how
- [ ] ...

## Important Files Map

[Key files that were created or modified, with a one-line description of each file's role.]

| File           | Role                          |
| -------------- | ----------------------------- |
| `path/to/file` | What it does / why it matters |

## Current State

[Is the project in a runnable state? Are there uncommitted changes? Broken tests? Anything the next session needs to handle immediately before moving forward?]
```

## Guidelines

- **Be exhaustive.** Assume the next session starts from scratch with ONLY this file as context.
- **Be specific.** File paths, function names, error messages, command outputs — concrete details over vague summaries.
- **Preserve the "why".** Decisions without rationale are useless to the next session.
- **Flag landmines.** If something is fragile, half-done, or likely to confuse, call it out explicitly.
- **Keep it scannable.** Use the headers. Don't bury critical info in walls of text.
- Write the file to the project root as `HANDOVER.md`.