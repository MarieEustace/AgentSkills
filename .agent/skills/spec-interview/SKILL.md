---
description: Interview me to create a detailed spec
argument-hint: [feature description]
model: opus
allowed-tools: AskUserQuestion, Write
---

Read the feature description: <feature>$ARGUMENTS</feature>

Interview me in depth using AskUserQuestionTool about technical implementation, UI/UX, data models, edge cases, error handling, security, tradeoffs, and anything non-obvious. Don't ask surface-level questions — dig into the hard decisions.

Keep interviewing me until I say "done". Do not stop early.

When I say "done", ask me where to save the spec file. Then write it there.