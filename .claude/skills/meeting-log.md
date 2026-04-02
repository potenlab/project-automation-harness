# Skill: Meeting Log Management

## Add Meeting Log

When the user provides meeting notes or asks to log a meeting:

1. Read the project's `project.json` to confirm the project exists
2. Create a markdown file in `projects/{id}/meetings/` with the naming convention:
   ```
   YYYY-MM-DD-{short-title}.md
   ```
3. Use this template:

```markdown
# Meeting: {Title}

- **Date**: YYYY-MM-DD
- **Time**: HH:MM (if provided)
- **Attendees**: List of names
- **Type**: Regular | Kickoff | Review | Retrospective | Ad-hoc

## Agenda
1. Item 1
2. Item 2

## Discussion
- Key point discussed
- Decisions made

## Action Items
| # | Task | Assignee | Due Date | Status |
|---|------|----------|----------|--------|
| 1 | Task description | Person | YYYY-MM-DD | Open |

## Notes
Additional notes or context.
```

4. If the user provides raw unstructured notes, parse them into this structured format
5. Extract action items and update relevant milestones if applicable

## List Meeting Logs

```bash
ls projects/{id}/meetings/
```

Read and summarize each log's title, date, and key action items.

## Update Meeting Log

1. Read the existing meeting file
2. Apply the requested changes
3. Preserve all existing content not being changed

## Analyze Meeting Transcript

When given a raw meeting transcript or audio notes:

1. Extract key discussion points
2. Identify decisions made
3. List action items with assignees and deadlines
4. Summarize in the structured meeting log format
5. If any action items relate to existing milestones, note the connection
