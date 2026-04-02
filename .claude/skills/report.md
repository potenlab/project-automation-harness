# Skill: Report Generation

## Project Status Report

When the user asks for a project report or status update:

1. Read `project.json` for core data
2. Read all meeting logs in `meetings/` for recent activity
3. Read any documents in `docs/` and `reports/` for context
4. Generate a report in `projects/{id}/reports/` with naming:
   ```
   YYYY-MM-DD-status-report.md
   ```

### Report Template

```markdown
# Project Status Report: {Project Name}

**Report Date**: YYYY-MM-DD
**Project Status**: Active / On Hold / Completed
**Timeline**: Start Date → End Date

## Executive Summary
Brief 2-3 sentence overview of where the project stands.

## Team
| Name | Role | 
|------|------|
| Name | Role |

## Milestone Progress
| Status | Milestone | Target Date |
|--------|-----------|-------------|
| ✅ | Completed milestone | Date |
| 🔲 | Pending milestone | Date |

**Overall Progress**: X/Y milestones completed (Z%)

## Recent Activity
- Summary of recent meetings and key decisions
- Action items status

## Risks & Issues
- Any identified risks or blockers

## Next Steps
- Upcoming milestones and priorities
```

## Weekly Summary

Lighter-weight report focusing on:
- What was accomplished this week
- What's planned for next week
- Any blockers

## Custom Report

When the user asks for a specific type of analysis:
1. Gather all relevant project data
2. Generate a focused report based on the request
3. Save to `reports/` folder
