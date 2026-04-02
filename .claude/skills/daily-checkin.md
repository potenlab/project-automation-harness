# Skill: Daily Check-in Reports

## Read Team Check-ins

When the PM asks about daily tasks, check-ins, or what the team is working on:

Phrases like:
- "What did the team do today?"
- "Show me today's check-ins"
- "What is Raka working on?"
- "Daily report"
- "Who's doing what?"

You must:

1. The check-in data is stored in the SQLite database at `../claude-chat-server/data/chat.db`
2. However, you **cannot** access the database directly
3. Instead, read the project data from `projects/*/project.json` to understand what milestones are pending/completed
4. Cross-reference with the team member list

## Summarize Team Activity

When asked for a team overview:

1. Read all `projects/*/project.json` files
2. For each project, list:
   - Pending milestones (what's being worked on)
   - Recently completed milestones (what was done)
   - Team members assigned
3. Present as a daily summary:

```
📋 Team Daily Summary — 2026-04-02

**Workflow Automator v1.0**
Members: Lead Developer, Frontend Engineer, DevOps Engineer
- ✅ AUT-01: Deploy Redis instance — Completed
- 🔲 AUT-02: Generate Discord Bot Token — Pending (due: Mar 30)
- 🔲 AUT-03: Integrate POST to /auth — Pending (due: Apr 01)

**JDR Tour**
Members: Raka (Developer)
- 🔲 Video generation fix — Pending
```

## Check Individual Progress

When PM asks about a specific developer:

1. Search `projects/*/project.json` for projects where that person is a member
2. Show their assigned milestones and completion status
3. Format:

```
👤 Raka's Tasks:

Project: JDR Tour
- 🔲 Video generation fix — Pending (due: Apr 03)

Project: Workflow Automator v1.0
- ✅ AUT-01: Deploy Redis — Completed
- 🔲 AUT-03: Integrate POST — Pending (due: Apr 01)

Progress: 1/3 tasks completed
```

## Track Overdue Tasks

When PM asks about overdue or at-risk tasks:

1. Read all project.json files
2. Find milestones where `date < today` AND `status = "pending"`
3. Flag them:

```
⚠️ Overdue Tasks:

Project: Workflow Automator v1.0
- AUT-02: Generate Discord Bot Token — Due: Mar 30 (3 days overdue)
  Assigned team: Lead Developer

No other overdue tasks found.
```
