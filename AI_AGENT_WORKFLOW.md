# AI Agent Workflow Guide — Analytics Team

> How to get the most out of VSCode AI agents (Plan / Ask / Agent mode) with GPT-4/5, Claude Sonnet/Opus.

---

## The Golden Rule

> **Context in = quality out.**
> Before any agent session, paste the relevant skill MD files into context.

---

## Which Model for Which Task

| Task | Recommended Model | Mode |
|------|-----------------|------|
| Architecture decisions, ADRs | Claude Opus / GPT-4 | Ask |
| Generate a new ETL job | Claude Sonnet | Plan → Agent |
| Write a new API endpoint | Claude Sonnet | Plan → Agent |
| UI component from design | Claude Sonnet | Agent |
| Debug a complex data issue | Claude Opus | Ask |
| Write unit tests for existing code | Any Sonnet | Agent |
| Quick syntax / small fix | Any model | Agent (inline) |
| Jira ticket breakdown | Claude Sonnet | Ask |
| Code review assistant | Claude Opus | Ask |

---

## Session Startup Template

Paste this at the start of every agent session (edit the [brackets]):

```
## Session Context

I'm working on the [Bob ETL / API / UI] layer of our analytics platform.

Stack: Python ETL (Bob jobs) → Hydra file DB → Python Bottle/Tornado API → React TypeScript UI.

Relevant skill files:
<paste ARCHITECTURE.md content>
<paste relevant SKILL MD (HYDRA / ETL / API / UI)>

## Task
[Describe your task here]

## Constraints
- [e.g., must stay within existing file structure]
- [e.g., no new dependencies without discussion]
- [e.g., must have unit tests]
```

---

## Plan Mode — When to Use It

Use **Plan mode** BEFORE Agent mode for:
- New feature spanning >2 files
- New ETL job (it will scaffold the full EVTL structure)
- New API endpoint group
- Refactoring existing code

**Plan mode prompt template:**
```
Plan (don't code yet) how you would implement:
[feature description]

Given these constraints:
- Follow ARCHITECTURE.md conventions
- Use patterns from [SKILL FILE]
- Affected files: [list files you know about]

Output a numbered plan with file names and what changes in each.
I'll review and then say "proceed".
```

---

## Agent Mode — Best Practices

1. **One task per session** — don't chain unrelated changes
2. **Review each file diff before confirming** — don't auto-accept all
3. **After generation, always ask**: "Are there any edge cases you didn't handle?"
4. **For ETL jobs**: ask agent to also generate the test file
5. **For API routes**: ask agent to also update the route registry / app.py
6. **For UI components**: ask agent to also add the Storybook story (if used)

---

## Jira MCP Workflow

> Requires Jira MCP connected in VSCode.

### Ticket → Code session
```
1. In Ask mode: "Summarise Jira ticket [PROJ-123] and break it into dev subtasks"
2. Review the breakdown
3. Switch to Plan mode: "Plan the implementation for subtask 1: [paste subtask]"
4. Switch to Agent mode: "Proceed with the plan"
5. After code is done: "Write the Jira comment summarising what was implemented in [PROJ-123]"
```

### Auto-generate Jira comments
```
After completing work, ask (Ask mode):
"Given the git diff below, write a Jira comment for ticket [PROJ-123] 
summarising what was changed and why, in 3-5 bullet points.

[paste git diff]"
```

---

## Confluence MCP Workflow (Read-only)

```
Ask mode: "Search Confluence for our Hydra schema documentation and 
summarise the table definitions relevant to the daily_revenue job."
```

Use Confluence context to:
- Feed existing design docs into agent context before coding
- Verify naming conventions match documented standards
- Pull runbook steps before writing automation

---

## Custom Prompt Library — Save These

### "Generate ETL Job"
```
Using the ETL_BOB_SKILL.md patterns and ARCHITECTURE.md conventions,
generate a complete Bob ETL job that:
- Source: [describe source]
- Target: Hydra table `[table_name]`
- Schedule: [cron]
- Transform: [describe transform logic]

Include: job file, transform file, schema dict, unit tests.
```

### "Generate API Endpoint"
```
Using API_SKILL.md patterns, add a new endpoint:
- Method + path: [GET /api/something/<param>]
- Returns: [describe the data]
- Service logic: [describe business logic]

Generate: route handler, service method, model method.
```

### "Generate UI Feature"
```
Using UI_SKILL.md patterns, create a UI feature:
- Feature: [describe what user sees / does]
- API endpoint it calls: [method + path]
- API response shape: [paste JSON]

Generate: TypeScript type, API client function, hook, component.
```

### "Write Unit Tests"
```
Write comprehensive pytest / Jest unit tests for:
[paste code]

Cover: happy path, edge cases, error conditions.
Mock all I/O (Hydra, HTTP) at the boundary.
```

---

## Do Not Let AI Do These Without Human Review

- [ ] Schema changes to Hydra tables
- [ ] Changes to `config.py` / environment variables
- [ ] New third-party library additions
- [ ] Auth / security logic
- [ ] SQL or query rewrites touching large datasets
- [ ] Anything marked `# CRITICAL` in codebase comments
