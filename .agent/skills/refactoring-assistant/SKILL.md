---
name: refactoring-assistant
description: >
  Identifies and refactors duplicated logic into pure functions and extracts interfaces, 
  focusing on keeping code coherent and factored.
---

# Refactoring Assistant

## When to use this skill

Activate when the user asks to clean up code, factorize logic, fix technical debt, or improve the coherence of an existing module or file. Do NOT activate this automatically during standard feature implementation unless explicitly requested.

## What it should do

1. **Analyze Duplication**: Scan the provided files or modules for repeated code blocks, copied logic, or identical structural patterns.
2. **Propose Factorization**: Suggest specific functions, classes, or interfaces that could replace the duplicated code. Always favor pure functions where possible.
3. **Check Coherence**: Ensure that the proposed refactor aligns with the overall module's naming conventions and data flow.
4. **Strict Typing Check**: Ensure that any new extracted functions or interfaces use strict types and absolutely no `any` types. Ensure non-null assertions (`!`) are avoided in favor of proper null checks.

## Key instructions

- **Incremental Steps**: Present the refactoring steps clearly so the user can review the proposed changes before they are applied. 
- **Explain Trade-offs**: Briefly explain why the new factored code is better (e.g., easier to test, less prone to bugs) and note any potential risks.
- **Minimal Diffs**: When applying the refactor, provide targeted code replacements rather than full file rewrites.
