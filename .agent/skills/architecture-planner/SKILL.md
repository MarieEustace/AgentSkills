---
name: architecture-planner
description: >
  Validates architectural choices, library selections, and error handling strategies 
  before implementation begins. Use when starting a new project, scaffolding, or adding major features.
---

# Architecture Planner

## When to use this skill

Activate when the user asks to start a new project, build a new system from scratch, or introduce a major new component/library to an existing codebase.

## What it should do

1. **Library Validation**: For every major dependency or library proposed, list it out and explicitly explain *why* it is a robust, safe, and appropriate choice for this specific project.
2. **Dependency Check**: Explicitly verify that no unverified, deprecated, or bleeding-edge experimental dependencies are being introduced.
3. **Error Handling Strategy**: Explicitly write out the error handling strategy. What layers will catch errors? How will they be formatted? Ensure there is no silent dropping of data.
4. **Data Flow**: Briefly map out how data will flow through the new components.
5. **Awaiting Validation**: You MUST halt and ask the user to validate the architectural choices and library selections before generating any implementation code.

## Key instructions

- **Do Not Implement Yet**: This skill is strictly for planning. Do not write the actual source code until the user approves the architecture.
- **Explicit Assumptions**: State clearly any assumptions you are making about the execution environment or scaling requirements.
- **Factorization**: If this architecture connects to existing code, propose how to factorize and keep the system coherent.
