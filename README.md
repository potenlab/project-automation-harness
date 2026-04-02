# AI Project Management Agent

A Claude CLI agent harness for managing projects through structured file-based operations. Used as the backend brain for the Claude Chat Server's PMO Assistant.

## Architecture Overview

```mermaid
graph TB
    subgraph "Claude Chat Server (Next.js)"
        UI["PMO Chat UI<br/>Skill Cards & Quick Actions"]
        API["API Route<br/>POST /api/pm/chat"]
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
    end

    CLI["Claude CLI<br/>--print --stream-json<br/>--dangerously-skip-permissions"]

    UI -->|"User message"| API
    API -->|"spawn()"| Runner
    Runner -->|"Executes in cwd"| CLI
    CLI -->|"Auto-reads"| CLAUDE
    CLAUDE -->|"References"| S1 & S2 & S3 & S4 & S5 & S6
    CLI -->|"Read/Write"| PJ & MT & DC & RP
    CLI -->|"stream-json response"| Runner
    Runner -->|"SSE stream"| API
    API -->|"Streamed words"| UI
```

## How the Agent Harness Works

```mermaid
sequenceDiagram
    participant User as PM User
    participant Chat as Chat Server
    participant CLI as Claude CLI
    participant FS as File System

    User->>Chat: "Create project Website Redesign"
    Chat->>CLI: spawn claude --print --system-prompt "PMO mode..."
    
    Note over CLI: Auto-reads CLAUDE.md from cwd
    CLI->>FS: Read .claude/skills/project-crud.md
    CLI->>FS: uuidgen → abc123
    CLI->>FS: mkdir projects/abc123/{meetings,docs,reports}
    CLI->>FS: Write projects/abc123/project.json
    
    CLI-->>Chat: stream-json result
    Chat-->>User: "Created project 'Website Redesign'"
    
    User->>Chat: "Add meeting log for today"
    Chat->>CLI: spawn claude --resume {session_id}
    CLI->>FS: Read .claude/skills/meeting-log.md
    CLI->>FS: Read projects/abc123/project.json
    CLI->>FS: Write projects/abc123/meetings/2026-04-01-kickoff.md
    CLI-->>Chat: stream-json result
    Chat-->>User: "Added meeting log for 2026-04-01"
```

## Skill Routing

```mermaid
flowchart LR
    MSG["User Message"] --> CLAUDE["CLAUDE.md<br/>Core Rules & Schema"]
    
    CLAUDE --> ROUTE{{"Intent Detection"}}
    
    ROUTE -->|"create/edit/delete<br/>project"| SK1["project-crud.md"]
    ROUTE -->|"meeting notes<br/>action items"| SK2["meeting-log.md"]
    ROUTE -->|"add/update/complete<br/>milestones"| SK3["milestone.md"]
    ROUTE -->|"team members<br/>roles"| SK4["member.md"]
    ROUTE -->|"status report<br/>weekly summary"| SK5["report.md"]
    ROUTE -->|"analyze document<br/>extract info"| SK6["analyze-document.md"]
    
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

## Project Data Structure

```mermaid
graph TD
    subgraph "projects/"
        subgraph "Project A (uuid)"
            PJ_A["project.json<br/>─────────<br/>name, description<br/>status, dates<br/>members[], milestones[]"]
            
            subgraph "meetings/"
                M1["2026-03-15-kickoff.md"]
                M2["2026-03-22-weekly.md"]
                M3["2026-04-01-review.md"]
            end
            
            subgraph "docs/"
                D1["requirements.pdf"]
                D2["design-spec.md"]
            end
            
            subgraph "reports/"
                R1["2026-03-29-status-report.md"]
                R2["2026-04-01-status-report.md"]
            end
        end
        
        subgraph "Project B (uuid)"
            PJ_B["project.json"]
            MB["meetings/"]
            DB["docs/"]
            RB["reports/"]
        end
    end
    
    style PJ_A fill:#1a1a1a,color:#e8eaed,stroke:#4285F4
    style PJ_B fill:#1a1a1a,color:#e8eaed,stroke:#4285F4
```

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

### milestone.md
| Operation | Reads | Writes |
|-----------|-------|--------|
| Add | `project.json` | `project.json` (appended) |
| Update | `project.json` | `project.json` (field merge) |
| Complete | `project.json` | `project.json` (status → completed) |
| View Timeline | `project.json` | — (formatted output) |

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
| Analyze | Uploaded document | — (returns structured data) |
| Store | Classified document | `meetings/`, `docs/`, or `reports/` |
| Extract Project | Multiple documents | Uses `project-crud` to create |

## Two Operating Modes

```mermaid
flowchart TB
    subgraph "PMO Mode"
        direction TB
        PMO_IN["System Prompt:<br/>'You are in PMO mode'"]
        PMO_CWD["cwd: /project-manager/"]
        PMO_ACC["Access: All projects<br/>Full read/write"]
        PMO_IN --> PMO_CWD --> PMO_ACC
    end
    
    subgraph "Project Mode (PM Role)"
        direction TB
        PROJ_PM_IN["System Prompt:<br/>'Project assistant for X'<br/>+ context (members, milestones)"]
        PROJ_PM_CWD["cwd: /project-manager/"]
        PROJ_PM_ACC["Access: Full read/write<br/>Update data, manage team"]
        PROJ_PM_IN --> PROJ_PM_CWD --> PROJ_PM_ACC
    end
    
    subgraph "Project Mode (Dev Role)"
        direction TB
        PROJ_DEV_IN["System Prompt:<br/>'Project assistant for X'<br/>+ context (members, milestones)"]
        PROJ_DEV_CWD["cwd: /project-manager/"]
        PROJ_DEV_ACC["Access: Read-only<br/>Answer questions only"]
        PROJ_DEV_IN --> PROJ_DEV_CWD --> PROJ_DEV_ACC
    end
    
    style PMO_ACC fill:#34A853,color:#fff
    style PROJ_PM_ACC fill:#4285F4,color:#fff
    style PROJ_DEV_ACC fill:#FBBC04,color:#000
```

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
│       ├── milestone.md               # Timeline & milestone tracking
│       ├── member.md                  # Team member management
│       ├── report.md                  # Status reports & summaries
│       └── analyze-document.md        # Document analysis & extraction
└── projects/                          # Project data (gitignored)
    └── {uuid}/
        ├── project.json               # Source of truth
        ├── meetings/                   # Meeting logs
        ├── docs/                       # Supplementary files
        └── reports/                    # Generated reports
```
