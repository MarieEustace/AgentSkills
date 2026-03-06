# Agent Configuration & Skills Guide

This repository contains custom instructions and skills designed to standardize and enhance the behavior of AI coding agents, specifically **Antigravity** (Google DeepMind) and **Claude Code** (Anthropic).

## 1. Using the Global Prompt

The core behavioral rules (tone, syntax restrictions, token efficiency, environment constraints) are housed in `GEMINI.md`.

### For Antigravity
*   **In-Project Configuration**: Antigravity will automatically read `GEMINI.md` if it is present in the root of your project workspace.
*   **Global Configuration**: To apply these rules across *all* projects, place them in `~/.gemini/GEMINI.md` (or your Antigravity Global User Rules settings).

### For Claude Code
*   **In-Project Configuration**: Claude Code specifically looks for a `CLAUDE.md` file in the project root. To use these rules, simply symlink or rename the file:
    ```bash
    ln -s GEMINI.md CLAUDE.md
    ```
*   **Global Configuration**: To apply these rules globally for Claude Code, place them in `~/.claude/CLAUDE.md` (or configure `~/.claude.json` directly).

---

## 2. Using Custom Skills

Skills are distinct, functional modules that the agent can read and execute for specific workflows (e.g., writing a commit message, reviewing code against a spec). 

### For Antigravity
Antigravity natively supports reading skills and workflows.
*   **In-Project Settings**: Place the skills in your project root using the exact path structure: `.agent/skills/<skill-name>/SKILL.md`. Antigravity will automatically discover and list them as available tools.
*   **Global Settings**: Global skills can typically be placed in `~/.gemini/skills/` or `.agents/skills/` located in your system's home directory depending on your specific Antigravity version configuration.

### For Claude Code
Claude Code does not natively auto-load `.agent/skills/` directories in the same way. To make Claude Code aware of these skills:
1.  Ensure you have a `CLAUDE.md` file in the project root.
2.  Add the following directive to your `CLAUDE.md`:
    ```markdown
    ## Available Custom Skills
    You have access to custom workflow instructions located in the `.agent/skills/` directory.
    When asked to perform a specific workflow (e.g., "be a code reviewer"), you MUST first use your file-reading tool to read `.agent/skills/<skill-name>/SKILL.md` and follow its instructions exactly.
    ```

---

## 3. Creating a New Skill (Template)

If you want to create a new skill to force the agent to follow a specific workflow, create a new folder `.agent/skills/<your-new-skill-name>/` and structure it like the following example.

**Skill Directory Architecture:**
```text
your-new-skill-name/
├── SKILL.md      # Required: Instructions + YAML metadata
├── references/   # Optional: Checklists, templates, style guides
├── scripts/      # Optional: Executable scripts the agent can run
└── assets/       # Optional: Example specs, image assets
```

**IMPORTANT**: The `SKILL.md` file *must* contain the YAML frontmatter blocks at the top so the agent can understand its name and purpose before opening it.

```markdown
---
name: your-skill-name
description: >
  A concise, 1-2 sentence description of what the skill does and 
  exactly WHEN the agent should decide to use it.
---

# Your Skill Name

## When to use this skill
Define the exact triggers for this skill. (e.g., "Activate this when the user asks to write documentation.")

## What it should do
Provide a numbered, step-by-step list of actions the agent MUST take.
1. First, check X.
2. Then, run tool Y.
3. Finally, format the output as Z.

## Key instructions
List the hard rules, constraints, or edge cases.
- **Tone**: Be strict and analytical.
- **Output Constraint**: Never output code, only markdown text.
- **Edge cases**: If the folder is empty, stop and warn the user.
```
