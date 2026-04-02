# Skill: Document Analysis

## Analyze Uploaded Document

When the user uploads or references a document for project creation or update:

1. Read the document (supports PDF, text, markdown, RTF, HWP/HWPX, images, Word/Excel)
2. Extract structured information:

### For Meeting Notes / Transcripts
- Meeting date and attendees
- Key discussion points
- Decisions made
- Action items with assignees and deadlines

### For Project Specs / Requirements
- Project name and description
- Objectives and goals
- Team members and roles
- Key dates and milestones
- Technical requirements
- Deliverables

### For General Documents
- Document type classification (meeting/supplementary/report/qa)
- Key facts and data points
- Relevance to existing projects

## Store Document

After analysis, store the document:

1. Classify the type: `meeting` | `supplementary` | `report` | `qa`
2. Copy to appropriate project folder:
   - meeting → `projects/{id}/meetings/`
   - supplementary, qa → `projects/{id}/docs/`
   - report → `projects/{id}/reports/`

## Extract Project from Documents

When creating a project from uploaded documents:

1. Analyze all provided documents
2. Extract and consolidate:
   - Project name
   - Description / objectives
   - Team members with roles
   - Key milestones with dates
   - Any noted risks or constraints
3. Present the extracted info to the user for confirmation
4. Use the `project-crud` skill to create the project with confirmed data
