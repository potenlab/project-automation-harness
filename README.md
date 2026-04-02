# AI Project Management Agent

A Claude CLI agent harness for managing projects through structured file-based operations. Used as the backend brain for the Claude Chat Server's PMO Assistant.

## Architecture Overview

```mermaid
graph TB
    subgraph "Claude Chat Server (Next.js + SQLite)"
        UI["PMO Chat UI<br/>Skill Cards · File Upload · Quick Actions"]
        PROJUI["Project Chat UI<br/>Chat-based Progress Updates"]
        API["API Routes<br/>POST /api/pm/chat<br/>POST /api/pm/projects/{id}/chat"]
        DB["SQLite Database<br/>projects · members · milestones<br/>chat sessions · messages"]
        SYNC["pm-sync.ts<br/>Bidirectional Mirror"]
        Runner["pm-claude-runner.ts<br/>spawn() Claude CLI"]
    end

    subgraph "This Repository (Work Directory)"
        CLAUDE["CLAUDE.md<br/>Agent Instructions"]
        subgraph "Skills (.claude/skills/)"
            S1["project-crud.md"]
            S2["meeting-log.md"]
            S3["milestone.md"]
            S4["member.md"]
            S5["report.md"]
            S6["analyze-document.md"]
        end
        subgraph "Project Data (projects/)"
            PJ["project.json"]
            MT["meetings/*.md"]
            DC["docs/*"]
            RP["reports/*.md"]
        end
        UL["uploads/<br/>Uploaded documents"]
    end

    CLI["Claude CLI<br/>--print --stream-json --verbose<br/>--dangerously-skip-permissions"]

    UI -->|"User message + files"| API
    PROJUI -->|"'AUT-01 is done!'"| API
    API -->|"spawn()"| Runner
    Runner -->|"Executes in cwd"| CLI
    CLI -->|"Auto-reads"| CLAUDE
    CLAUDE -->|"References"| S1 & S2 & S3 & S4 & S5 & S6
    CLI -->|"Read/Write"| PJ & MT & DC & RP
    CLI -->|"Reads uploaded"| UL
    CLI -->|"stream-json response"| Runner
    Runner -->|"SSE stream"| API
    API -->|"After response"| SYNC
    SYNC -->|"Disk → SQLite"| DB
    SYNC -->|"SQLite → Disk"| PJ
    DB -->|"Data for UI"| UI & PROJUI
```

## How It Works — Full Flow

```mermaid
sequenceDiagram
    participant User as PM / Dev User
    participant Chat as Chat Server
    participant CLI as Claude CLI
    participant FS as File System
    participant DB as SQLite
    participant Sync as pm-sync.ts

    User->>Chat: Upload meeting-log.pdf + "Create project from this"
    Chat->>FS: Save file to uploads/
    Chat->>CLI: spawn claude --print (message + file paths)
    
    Note over CLI: Auto-reads CLAUDE.md from cwd
    CLI->>FS: Read uploads/meeting-log.pdf
    CLI->>FS: Read .claude/skills/analyze-document.md
    CLI->>FS: Read .claude/skills/project-crud.md
    CLI->>FS: uuidgen | tr upper lower → abc123
    CLI->>FS: mkdir projects/abc123/{meetings,docs,reports}
    CLI->>FS: Write projects/abc123/project.json
    CLI->>FS: Write projects/abc123/meetings/2026-04-01-kickoff.md
    
    CLI-->>Chat: stream-json result
    Chat->>Sync: fullSync()
    Sync->>FS: Scan projects/*/project.json
    Sync->>DB: Upsert project + members + milestones + docs
    Sync->>FS: Write project.json for any DB-only projects
    Chat-->>User: "Created project 'Workflow Automator v1.0'"

    Note over User: Later, a Dev updates progress...
    User->>Chat: "AUT-01 is done!"
    Chat->>CLI: spawn claude --resume {session}
    CLI->>FS: Read .claude/skills/milestone.md
    CLI->>FS: Read projects/abc123/project.json
    CLI->>FS: Update milestone status → completed
    CLI->>FS: Write projects/abc123/project.json
    CLI-->>Chat: "✅ Marked AUT-01 as completed! Progress: 1/5"
    Chat->>Sync: syncFileToDb() + writeProjectJson()
    Sync->>DB: Update milestone status in SQLite
    Chat-->>User: Progress bar updates (1/5 → 20%)
```

## Skill Routing

```mermaid
flowchart LR
    MSG["User Message"] --> CLAUDE["CLAUDE.md<br/>Core Rules & Schema"]
    
    CLAUDE --> ROUTE{{"Intent Detection"}}
    
    ROUTE -->|"create/edit/delete<br/>project"| SK1["project-crud.md"]
    ROUTE -->|"meeting notes<br/>action items"| SK2["meeting-log.md"]
    ROUTE -->|"'task X is done'<br/>progress updates"| SK3["milestone.md"]
    ROUTE -->|"team members<br/>roles"| SK4["member.md"]
    ROUTE -->|"status report<br/>weekly summary"| SK5["report.md"]
    ROUTE -->|"uploaded PDF/RTF/doc<br/>extract info"| SK6["analyze-document.md"]
    
    SK1 --> FS1["projects/{id}/project.json"]
    SK2 --> FS2["projects/{id}/meetings/*.md"]
    SK3 --> FS1
    SK4 --> FS1
    SK5 --> FS3["projects/{id}/reports/*.md"]
    SK6 --> FS4["projects/{id}/docs/*"]
    
    style ROUTE fill:#4285F4,color:#fff
    style SK1 fill:#34A853,color:#fff
    style SK2 fill:#4285F4,color:#fff
    style SK3 fill:#FBBC04,color:#000
    style SK4 fill:#EA4335,color:#fff
    style SK5 fill:#8E24AA,color:#fff
    style SK6 fill:#00ACC1,color:#fff
```

## Bidirectional Sync

SQLite and project.json are always mirrors of each other.

```mermaid
flowchart LR
    subgraph "Agent writes"
        A1["Agent creates project"] --> PJ1["project.json on disk"]
        PJ1 -->|"fullSync()"| DB1["SQLite projects table"]
    end
    
    subgraph "Web UI writes"
        W1["User creates via form"] --> DB2["SQLite projects table"]
        DB2 -->|"writeProjectJson()"| PJ2["project.json on disk"]
    end
    
    subgraph "Sync Triggers"
        T1["After every agent chat response"]
        T2["After every web UI mutation"]
        T3["POST /api/pm/sync (manual)"]
    end
    
    style DB1 fill:#4285F4,color:#fff
    style DB2 fill:#4285F4,color:#fff
    style PJ1 fill:#34A853,color:#fff
    style PJ2 fill:#34A853,color:#fff
```

**Sync details:**
- UUIDs normalized to lowercase (agent's `uuidgen` may produce uppercase)
- Members: clear + re-insert with `user_id` preservation by name+email match
- Milestones: clear + re-insert within transaction
- Documents: scans meetings/docs/reports dirs for untracked files
- Uppercase folders renamed to lowercase after sync

## Role-Based Access

```mermaid
flowchart TB
    subgraph "PMO Mode (PM only)"
        direction TB
        PMO["Full access to all projects"]
        PMO_DO["Create projects · Manage teams<br/>Upload docs · Generate reports<br/>Update all data"]
    end
    
    subgraph "Project Mode — PM"
        direction TB
        PM["Full project access"]
        PM_DO["Edit project info · Manage members<br/>Add milestones · Upload docs<br/>Update progress"]
    end
    
    subgraph "Project Mode — Dev"
        direction TB
        DEV["Limited project access"]
        DEV_DO["Ask questions · View all data<br/>Update milestone progress via chat<br/>Cannot edit project info or members"]
    end
    
    style PMO fill:#34A853,color:#fff
    style PM fill:#4285F4,color:#fff
    style DEV fill:#FBBC04,color:#000
```

## Chat Features

| Feature | PMO Assistant | Project Chat |
|---------|--------------|--------------|
| Multi-session history | Yes (SQLite-backed) | Yes (per user per project) |
| Session titles | Auto from project name or first message | Auto from first message |
| File upload | Drag & drop + attach button | — |
| Supported file types | PDF, RTF, TXT, MD, CSV, JSON, images, HWP, DOC/DOCX, XLS/XLSX | — |
| Skill cards | 6 skill cards on empty state | — |
| Quick actions | List projects, Overall status, Recent meetings | — |
| Progress bar | — | Dev role only (compact header bar) |
| Progress updates via chat | — | "AUT-01 is done!" → auto-updates milestone |
| Markdown rendering | Yes (tables, code blocks, lists) | Yes |
| Chat persistence | Survives refresh/navigation | Survives refresh/navigation |
| Clear/delete sessions | Per session | Per session |

## Skill Details

### project-crud.md
| Operation | Reads | Writes | Confirms? |
|-----------|-------|--------|-----------|
| Create | — | `project.json`, folder structure | No |
| Read | `project.json` | — | No |
| Update | `project.json` | `project.json` (merged) | No |
| Delete | `project.json` | Removes entire folder | **Yes** |

### meeting-log.md
| Operation | Reads | Writes |
|-----------|-------|--------|
| Add Log | `project.json` | `meetings/YYYY-MM-DD-title.md` |
| List Logs | `meetings/*` | — |
| Analyze Transcript | Raw text | Structured `meetings/*.md` |

### milestone.md — Progress Updates
| Operation | Reads | Writes |
|-----------|-------|--------|
| Mark done | `project.json` | `project.json` (status → completed) |
| Reopen | `project.json` | `project.json` (status → pending) |
| Add | `project.json` | `project.json` (appended) |
| View Timeline | `project.json` | — (formatted output) |

Natural language triggers: "task X is done", "AUT-01 completed", "finished the Redis deployment", "mark Design Phase as completed"

### member.md
| Operation | Reads | Writes | Confirms? |
|-----------|-------|--------|-----------|
| Add | `project.json` | `project.json` (appended) | No |
| Update | `project.json` | `project.json` (field merge) | No |
| Remove | `project.json` | `project.json` (filtered) | **Yes** |

### report.md
| Operation | Reads | Writes |
|-----------|-------|--------|
| Status Report | `project.json`, `meetings/*`, `docs/*` | `reports/YYYY-MM-DD-status-report.md` |
| Weekly Summary | `project.json`, `meetings/*` | `reports/YYYY-MM-DD-weekly.md` |

### analyze-document.md
| Operation | Reads | Writes |
|-----------|-------|--------|
| Analyze | Uploaded document (PDF, RTF, Word, images, etc.) | — (returns structured data) |
| Store | Classified document | `meetings/`, `docs/`, or `reports/` |
| Extract Project | Multiple documents | Uses `project-crud` to create |

## Setup

This repository is the **work directory** for the Claude CLI agent. It is referenced by the chat server via the `CLAUDE_WORK_DIR` environment variable.

```bash
# In the chat server's .env.local:
CLAUDE_BIN=/path/to/claude
CLAUDE_WORK_DIR=/path/to/this/repo
```

The Claude CLI automatically reads `CLAUDE.md` from its `cwd`. Skills in `.claude/skills/` are referenced by the agent as needed.

## File Map

```
.
├── CLAUDE.md                          # Master agent instructions
├── .claude/
│   └── skills/
│       ├── project-crud.md            # Create, Read, Update, Delete
│       ├── meeting-log.md             # Meeting notes & transcripts
│       ├── milestone.md               # Progress updates & timeline
│       ├── member.md                  # Team member management
│       ├── report.md                  # Status reports & summaries
│       └── analyze-document.md        # Document analysis & extraction
├── uploads/                           # Uploaded documents (agent reads these)
└── projects/                          # Project data (gitignored)
    └── {uuid}/
        ├── project.json               # Source of truth (synced with SQLite)
        ├── meetings/                   # Meeting logs (.md)
        ├── docs/                       # Supplementary files
        └── reports/                    # Generated reports (.md)
```
