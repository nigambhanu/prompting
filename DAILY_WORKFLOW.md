# Daily Dev Workflow & Standards

> Quick reference for every team member. Bookmark this.

---

## Morning Standup Prep (2 min with AI)

```
Open Ask mode → paste:

"Check Jira board [PROJECT] for my in-progress tickets.
Summarise what I was working on and suggest my standup update:
- What I did yesterday
- What I'm doing today
- Any blockers"
```

---

## Starting a New Ticket

```
1. Pull ticket from Jira (Ask mode + Jira MCP)
2. Ask: "Break PROJ-NNN into subtasks and identify the affected layer"
3. Open Plan mode: "Plan implementation for subtask 1"
4. Review plan → switch to Agent mode: "Proceed"
5. Write/review tests
6. Commit with conventional format (see below)
```

---

## Commit Message Convention

```
<type>(<scope>): <short description>

Types:    feat | fix | chore | refactor | test | docs | perf
Scopes:   etl | api | ui | hydra | infra | ci

Examples:
  feat(etl): add daily_revenue_summary Bob job
  fix(api): return 400 instead of 500 for invalid date param
  feat(ui): add MetricsCard component with date selector
  refactor(api): extract revenue logic into service layer
  test(etl): add edge case tests for null value transform
```

AI auto-generate commit message:
```
Ask mode: "Given this git diff, write a conventional commit message:
[paste diff]"
```

---

## PR Description Template

````markdown
## What
[1-2 sentences: what does this PR do?]

## Why
[Link to Jira ticket: PROJ-NNN]
[Brief context on why this change is needed]

## How
[Key implementation decisions, non-obvious choices]

## Checklist
- [ ] Follows layer architecture (ETL/API/UI skill MD)
- [ ] Unit tests added/updated
- [ ] No hardcoded secrets or URLs
- [ ] Logging added for key operations
- [ ] Hydra schema unchanged (or migration noted)

## Testing
[How to test this change locally]
````

AI auto-generate PR description:
```
Ask mode: "Write a PR description for this git diff following our template:
[paste diff]"
```

---

## Code Review Checklist — Reviewer

**Before approving, verify:**

- [ ] Layer separation not violated (ETL/API/UI)
- [ ] Response envelope on all new API endpoints
- [ ] No `any` in TypeScript
- [ ] All async state has loading/error/empty handling
- [ ] ETL transforms are pure functions
- [ ] Tests cover the new code
- [ ] No secrets / hardcoded URLs

AI-assisted review:
```
Ask mode (Code Reviewer agent): "Review this PR diff for our team standards:
[paste diff]"
```

---

## Debugging Workflow with AI

```
1. Capture the error: full stack trace + relevant log lines
2. Ask mode:
   "I have this error in our [ETL/API/UI] layer:
   [paste error]
   
   Relevant code:
   [paste function/component]
   
   Diagnose the root cause and suggest a fix."

3. If data issue: "What Hydra query would confirm this hypothesis?"
4. If API issue: "What curl command would reproduce this?"
5. If UI issue: "What browser console check would confirm this?"
```

---

## Refactoring with AI

```
Ask mode (always Plan first):

"I want to refactor [describe current state] to [describe goal].
Current code: [paste]
Constraints: must not change API contract, must keep existing tests passing.

Plan the refactor steps without coding yet."
```

---

## Quick AI Shortcuts — Paste These

| Task | One-liner |
|------|-----------|
| Explain unfamiliar code | "Explain this code in plain English, then list any risks:" |
| Find bugs | "Find potential bugs and edge cases in this function:" |
| Improve error handling | "Add proper error handling to this function following our patterns:" |
| Add types | "Add TypeScript types to this code. No `any`. Use strict mode." |
| Write docstring | "Write a Google-style docstring for this function:" |
| Optimise query | "This Hydra read is slow. Suggest optimisations:" |

---

## When NOT to Use AI Agent Mode

- Modifying authentication or security logic → pair program with tech lead
- Changing Hydra table schemas → ADR required first
- Adding new npm/pip packages → get team approval
- Anything touching production config → manual review gate
- Debugging production incidents → human-led, AI as reference only
