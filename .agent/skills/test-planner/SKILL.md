---
name: test-planner
description: >
  Generates a test plan from the spec before any code is written.
  Test: give a spec, ask for a test plan first.
---

# Test Planner

## When to use this skill

Activate when the user provides a specification or feature request and requires a test plan *before* implementation code is written.

## What it should do

1. Read the specification thoroughly and identify every single testable requirement.
2. For each requirement, clearly define:
   - What needs to be tested
   - The specific input and **setup data** (initial state, database records, etc.) required before the test runs (the "Arrange" phase).
   - The expected output
   - Any relevant edge cases
3. **Dependencies & Mocking**: Explicitly identify any external dependencies (e.g., databases, third-party APIs, time functions) and specify what parts need to be mocked.
4. Categorize the test cases into clearly defined sections:
   - Happy path
   - Errors / Error handling
   - Boundaries (edge cases limit testing)
   - Security
5. Output the final test plan strictly as a structured checklist so that it can be used during the code review phase.

## Key instructions

- **Primary Directive**: "Generate the test plan from the SPEC, not from the code. Tests should verify what was specified, not what was implemented."
- **Missing Spec Handling**: If the spec lacks needed details (e.g., exact error message format), DO NOT invent them. Flag them explicitly as "Requires Clarification from User" in the test plan.
- **AAA Pattern**: Structure each test case using the Arrange (Given), Act (When), Assert (Then) pattern.
- **No Implementation Yet**: Do not write the actual test code (like Jest, PyTest) unless explicitly asked. Stop at providing the structured test plan.
