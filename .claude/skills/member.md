# Skill: Team Member Management

## Add Member

When the user asks to add a team member:

1. Read `project.json`
2. Add to the `members` array:
   ```json
   {
     "name": "Full Name",
     "role": "PM | Developer | Designer | QA | Stakeholder",
     "email": "email@example.com"
   }
   ```
3. Update `updated_at` timestamp
4. Write back `project.json`

If the user provides a name without a role, ask what role to assign.

## Remove Member

1. Read `project.json`
2. Find member by name (case-insensitive partial match)
3. **Confirm** with the user before removing
4. Remove from the array
5. Update `updated_at` timestamp
6. Write back `project.json`

## Update Member

1. Read `project.json`
2. Find member by name
3. Update role or email as specified
4. Update `updated_at` timestamp
5. Write back `project.json`

## List Members

Read `project.json` and present members in a clear format:

```
Team for "Project Name" (4 members):

👤 John Kim — PM (john@example.com)
👤 Sarah Lee — Developer (sarah@example.com)
👤 Mike Park — Designer (mike@example.com)
👤 Jane Cho — QA (jane@example.com)
```
