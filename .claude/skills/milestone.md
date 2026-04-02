# Skill: Milestone Management

## Add Milestone

When the user asks to add a milestone:

1. Read the project's `project.json`
2. Determine the next milestone `id` (max existing id + 1, or 1 if none)
3. Add the milestone to the `milestones` array:
   ```json
   {
     "id": 1,
     "title": "Milestone title",
     "description": "Detailed description",
     "date": "YYYY-MM-DD",
     "status": "pending"
   }
   ```
4. Update `updated_at` timestamp
5. Write back `project.json`

## Update Milestone

1. Read `project.json`
2. Find the milestone by `id` or by `title` (fuzzy match)
3. Update only the specified fields
4. Update `updated_at` timestamp
5. Write back `project.json`

Common updates:
- Mark as completed: `"status": "completed"`
- Change date: `"date": "YYYY-MM-DD"`
- Update description

## Delete Milestone

1. Read `project.json`
2. Find the milestone
3. Remove it from the array
4. Update `updated_at` timestamp
5. Write back `project.json`

## View Timeline

When asked to show project timeline or progress:

1. Read `project.json`
2. Present milestones sorted by date
3. Show completion status:
   ```
   Timeline for "Project Name":
   
   ✅ [2026-03-01] Design Phase — Completed
   ✅ [2026-03-15] Prototype Review — Completed
   🔲 [2026-04-01] Development Sprint 1 — Pending
   🔲 [2026-04-15] QA Testing — Pending
   🔲 [2026-05-01] Launch — Pending
   
   Progress: 2/5 milestones completed (40%)
   ```
