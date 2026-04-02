# Skill: Project CRUD

## Create Project

When the user asks to create a new project:

1. Generate a UUID for the project ID (use `uuidgen` command)
2. Create the folder structure:
   ```
   projects/{id}/
     project.json
     meetings/
     docs/
     reports/
   ```
3. Write `project.json` with all provided info. Use today's date for `created_at` and `updated_at`
4. If the user provides documents (meeting logs, specs), analyze them first using the `analyze-document` skill to extract structured info

### Required Fields
- `name` (string)
- `description` (string)

### Optional Fields
- `status` (default: "active")
- `start_date`, `end_date` (YYYY-MM-DD)
- `members` (array, default: [])
- `milestones` (array, default: [])

### Example

User: "Create a project called 'Website Redesign' for redesigning our corporate website. Start date April 1, end date June 30."

Action:
```bash
uuidgen  # → e.g. a1b2c3d4-...
mkdir -p projects/a1b2c3d4/meetings projects/a1b2c3d4/docs projects/a1b2c3d4/reports
```
Then write `projects/a1b2c3d4/project.json`.

## Read Project

When the user asks about a project:

1. If they specify a project name, search through `projects/*/project.json` files to find it
2. Read and parse the `project.json`
3. Present a summary including: name, status, timeline, member count, milestone progress

To list all projects:
```bash
ls projects/
```
Then read each `project.json` for summary info.

## Update Project

When the user asks to edit project info:

1. Read the existing `project.json`
2. Merge the changes (only update specified fields)
3. Update `updated_at` to current ISO timestamp
4. Write back the full `project.json`

**Never** remove fields that weren't explicitly asked to be removed.

## Delete Project

When the user asks to delete a project:

1. **Always confirm** with the user before deleting
2. Show the project name and a brief summary of what will be deleted
3. Only after confirmation, remove the entire project folder:
   ```bash
   rm -rf projects/{id}
   ```
