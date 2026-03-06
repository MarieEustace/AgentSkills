---
name: code-reviewer
description: >
  Reviews generated code against the original spec, flags deviations and scope creep.
  Test: generate code, then invoke the skill.
---

# Code Reviewer

## What it should do

1. Compare the generated or proposed code against the original specification, point by point.
2. Explicitly flag any spec requirements that are unimplemented or partially implemented.
3. Explicitly flag scope creep (code or features that were not requested in the original spec).
4. **Unclear Specs**: If the spec is ambiguous or implicit, flag it as "Ambiguous Spec" instead of guessing the intent.
5. **Missing Tests/Docs**: Flag the absence of tests or documentation if they were implicitly or explicitly required.
6. **Project Conventions**: Verify that the code aligns with existing architectural patterns, style guidelines, and **naming conventions** of the codebase.
7. Conduct checks for common issues:
   - Input validation (are edge cases handled?)
   - Hardcoded values (are configurations or secrets hardcoded?)
   - Error handling (are errors swallowed or handled properly?)
   - **Security** (are there obvious flaws like SQL injection or sensitive data exposure?)
   - **Performance** (are there performance traps like N+1 queries or memory leaks?)
   - **Type Checks** (is strict typing used correctly? Is `any` avoided? Are interfaces well-defined?)
   - **Dependencies** (are there any unsafe, unverified, deprecated, or overly recent "bleeding-edge" dependencies introduced?)

## Key instructions

- **Primary Directive**: "Do NOT just check if the code runs. Check if it matches what was asked for. Working code that doesn’t match the spec is still wrong."
- **Actionable References**: You MUST provide exact file paths and line numbers for every deviation, issue, or scope creep identified. Avoid generic descriptions.
- **Output format**: Group your review findings strictly under the following headings:
  - **Matches spec**: List the implemented requirements that align with the spec.
  - **Deviates from spec**: List the requirements that are implemented incorrectly or missed entirely.
  - **Not in spec**: List any scope creep, extra features, or unrequested changes.
  - **Security/Performance/Typing Issues**: List any flaws found that were not explicitly mentioned in the spec but violate best practices.
