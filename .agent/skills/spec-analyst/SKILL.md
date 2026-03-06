---
name: spec-analyst
description: >
  Reviews a user's specification for completeness and clarity before the agent
  begins planning. Use when a user provides a spec, project brief, or feature
  description and wants it reviewed before implementation begins.
---

# Spec Analyst

## When to use this skill

Activate when the user asks you to review, analyze, or check a specification.
Also activate when a user provides a spec and says "build this" — review the
spec FIRST, before generating a plan.

Only activate for specs with 3+ requirements or multi-component systems.
For trivial tasks (single function, one-step operation), skip the review.

## How to review a spec

1. Read the entire specification carefully
2. For each requirement, check:
   - Is the expected behavior clearly defined?
   - Are edge cases mentioned?
   - Are error scenarios covered?
   - Are there implicit assumptions that should be explicit?
3. Classify each gap as:
   - **Blocker** — cannot plan without an answer (missing auth model, undefined
     data flow, ambiguous core behavior)
   - **Clarification** — should be answered but a reasonable default exists
     (empty input handling, specific error messages, pagination limits)
4. Present gaps to the user one at a time, starting with blockers
   (see conversation flow below)
5. Do NOT proceed to planning until all blockers are resolved and the user
   confirms the spec is complete

## Conversation flow

- Start with a brief summary: how many blockers and clarifications you found
- Present the first blocker, explain what is missing, and ask a specific question
- Wait for the user to respond
- Incorporate their answer, then present the next gap
- After all blockers are resolved, present clarifications one by one
- For clarifications, suggest a reasonable default the user can accept or override
- When all gaps are addressed, summarize what was decided and ask:
  "Ready to proceed to planning?"

## What to check for

Adapt these criteria to the type of spec (API, UI, infrastructure, data
pipeline, etc.):

- Clear input/output definitions
- Authentication and authorization requirements
- Data validation rules (what is valid, what is rejected, what are the limits)
- Error handling (what errors can occur, what messages to show)
- Edge cases (empty inputs, duplicates, concurrent access, boundary values)
- Performance constraints (rate limits, pagination, size limits, timeouts)
- Dependencies on external systems or services

## What to flag

- Vague language: "should handle errors properly" (what errors? what's proper?)
- Missing constraints: no rate limits, no pagination, no size limits
- Implicit assumptions: "users can log in" (with what? JWT? session? OAuth?)
- Unspecified behavior: "admin can manage users" (create? delete? both?
  what permissions?)