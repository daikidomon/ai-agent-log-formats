# OpenCode

## Basic Information

| Item | Details |
|------|---------|
| Type | TUI coding agent |
| Log Format | SQLite |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.local/share/opencode/` |
| Linux | `~/.local/share/opencode/` |
| Windows | `%USERPROFILE%\.local\share\opencode\` |

If the environment variable `XDG_DATA_HOME` is set, `$XDG_DATA_HOME/opencode/` is used.

### Config Locations (Searched in Order)

```
~/.config/opencode/config.json
~/.config/opencode/opencode.json
~/.config/opencode/opencode.jsonc
~/.opencode/opencode.jsonc
~/.opencode/opencode.json
```

## File Structure (Verified)

```
~/.local/share/opencode/
├── bin/                                  # Installed binary
│   └── opencode
├── log/                                  # Debug logs (timestamped)
│   └── YYYY-MM-DDTHHMMSS.log
├── opencode.db                           # Main SQLite database
├── snapshot/                             # Git bare repos for diff snapshots
│   └── <project-id>/                    # Bare git repository per project
└── storage/
    ├── migration                         # Migration marker
    └── session_diff/                     # Session diff JSON files
        └── <session-id>.json            # Diff data per session
```

## Data Format (Verified)

SQLite database (`opencode.db`). Managed by Drizzle ORM.

### Tables Overview

| Table | Description |
|-------|-------------|
| `project` | Project information (worktree path, VCS type, etc.) |
| `session` | Session metadata (title, slug, version, summary, permissions) |
| `message` | Messages with role and model info. JSON in `data` column |
| `part` | Individual parts of a message (text, tools, steps). JSON in `data` column |
| `todo` | Session-scoped todo items |
| `workspace` | Workspace branches and types |
| `session_share` | Session sharing metadata (URL, secret) |
| `account` | OAuth account credentials |
| `account_state` | Active account selection |
| `control_account` | Control plane account credentials |
| `permission` | Per-project permission rules |

### project Table

```sql
CREATE TABLE `project` (
  `id` text PRIMARY KEY,           -- SHA-1 hash
  `worktree` text NOT NULL,        -- Absolute path to project
  `vcs` text,                      -- "git" or null
  `name` text,
  `icon_url` text,
  `icon_color` text,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `time_initialized` integer,
  `sandboxes` text NOT NULL,       -- JSON array
  `commands` text
);
```

Example:

```json
{
  "id": "5d04e1f82490ad90bbb09a6241051203cd6cbb47",
  "worktree": "/home/ubuntu/ai-agent-log-formats",
  "vcs": "git",
  "sandboxes": "[]"
}
```

### session Table

```sql
CREATE TABLE `session` (
  `id` text PRIMARY KEY,              -- Prefixed: ses_...
  `project_id` text NOT NULL,
  `parent_id` text,                   -- For sub-agent sessions
  `slug` text NOT NULL,               -- Human-readable name (e.g., "brave-falcon")
  `directory` text NOT NULL,          -- Working directory path
  `title` text NOT NULL,              -- Auto-generated title
  `version` text NOT NULL,            -- OpenCode version (e.g., "1.2.27")
  `share_url` text,
  `summary_additions` integer,        -- Code change summary
  `summary_deletions` integer,
  `summary_files` integer,
  `summary_diffs` text,
  `revert` text,
  `permission` text,                  -- JSON: permission rules for session
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `time_compacting` integer,
  `time_archived` integer,
  `workspace_id` text,
  FOREIGN KEY (`project_id`) REFERENCES `project`(`id`) ON DELETE CASCADE
);
```

Example:

```json
{
  "id": "ses_305eeae76ffeZFzOzz903nD0J0",
  "project_id": "5d04e1f82490ad90bbb09a6241051203cd6cbb47",
  "slug": "shiny-wolf",
  "directory": "/home/ubuntu/ai-agent-log-formats",
  "title": "Greeting in conversation setup",
  "version": "1.2.27",
  "permission": "[{\"permission\":\"question\",\"pattern\":\"*\",\"action\":\"deny\"}]"
}
```

### message Table

```sql
CREATE TABLE `message` (
  `id` text PRIMARY KEY,              -- Prefixed: msg_...
  `session_id` text NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `data` text NOT NULL,               -- JSON string
  FOREIGN KEY (`session_id`) REFERENCES `session`(`id`) ON DELETE CASCADE
);
```

#### message.data (User)

```json
{
  "role": "user",
  "time": { "created": 1773721964971 },
  "summary": { "diffs": [] },
  "agent": "build",
  "model": {
    "providerID": "openai",
    "modelID": "gpt-4o-mini"
  }
}
```

#### message.data (Assistant)

```json
{
  "role": "assistant",
  "time": {
    "created": 1773721965043,
    "completed": 1773721968543
  },
  "parentID": "msg_cfa1151ab001QbI6kqQG7fwKae",
  "modelID": "gpt-4o-mini",
  "providerID": "openai",
  "mode": "build",
  "agent": "build",
  "path": {
    "cwd": "/home/ubuntu/ai-agent-log-formats",
    "root": "/home/ubuntu/ai-agent-log-formats"
  },
  "cost": 0.0014427,
  "tokens": {
    "total": 9585,
    "input": 9574,
    "output": 11,
    "reasoning": 0,
    "cache": { "read": 0, "write": 0 }
  },
  "finish": "stop"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `role` | string | `"user"` or `"assistant"` |
| `time.created` | integer | Unix epoch ms |
| `time.completed` | integer | Completion time (assistant only) |
| `parentID` | string | Parent message ID (assistant only) |
| `model.providerID` / `providerID` | string | Provider name (e.g., `"openai"`, `"anthropic"`) |
| `model.modelID` / `modelID` | string | Model name |
| `agent` | string | Agent mode (e.g., `"build"`) |
| `cost` | number | Request cost in USD (assistant only) |
| `tokens` | object | Token usage breakdown (assistant only) |
| `finish` | string | `"stop"` or `"tool-calls"` (assistant only) |

### part Table

```sql
CREATE TABLE `part` (
  `id` text PRIMARY KEY,              -- Prefixed: prt_...
  `message_id` text NOT NULL,
  `session_id` text NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `data` text NOT NULL,               -- JSON string
  FOREIGN KEY (`message_id`) REFERENCES `message`(`id`) ON DELETE CASCADE
);
```

#### Part Types

| Type | Description |
|------|-------------|
| `text` | Text content (user input or assistant response) |
| `step-start` | Beginning of an inference step (includes git snapshot hash) |
| `step-finish` | End of an inference step (includes cost, tokens, reason) |
| `tool` | Tool call with input, output, and execution metadata |

#### part.data — text

```json
{
  "type": "text",
  "text": "Hello! How can I assist you today?",
  "time": { "start": 1773721968509, "end": 1773721968509 },
  "metadata": {
    "openai": { "itemId": "msg_008bb2297fce..." }
  }
}
```

#### part.data — step-start

```json
{
  "type": "step-start",
  "snapshot": "c432666404def571f8762ae88063a9e6ffb1de81"
}
```

#### part.data — step-finish

```json
{
  "type": "step-finish",
  "reason": "stop",
  "snapshot": "c432666404def571f8762ae88063a9e6ffb1de81",
  "cost": 0.0014427,
  "tokens": {
    "total": 9585,
    "input": 9574,
    "output": 11,
    "reasoning": 0,
    "cache": { "read": 0, "write": 0 }
  }
}
```

#### part.data — tool

```json
{
  "type": "tool",
  "callID": "call_01nYsvZD3mzrD1MnqlCWp2Jz",
  "tool": "bash",
  "state": {
    "status": "completed",
    "input": {
      "command": "ls",
      "description": "Lists files in current directory"
    },
    "output": "README.ja.md\nREADME.md\ntools\n",
    "title": "Lists files in current directory",
    "metadata": {
      "output": "README.ja.md\nREADME.md\ntools\n",
      "exit": 0,
      "description": "Lists files in current directory",
      "truncated": false
    },
    "time": { "start": 1773722148816, "end": 1773722148855 }
  },
  "metadata": {
    "openai": { "itemId": "fc_0447f6e49c79..." }
  }
}
```

### todo Table

```sql
CREATE TABLE `todo` (
  `session_id` text NOT NULL,
  `content` text NOT NULL,
  `status` text NOT NULL,
  `priority` text NOT NULL,
  `position` integer NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  PRIMARY KEY(`session_id`, `position`),
  FOREIGN KEY (`session_id`) REFERENCES `session`(`id`) ON DELETE CASCADE
);
```

### workspace Table

```sql
CREATE TABLE `workspace` (
  `id` text PRIMARY KEY,
  `branch` text,
  `project_id` text NOT NULL,
  `type` text NOT NULL,
  `name` text,
  `directory` text,
  `extra` text,
  FOREIGN KEY (`project_id`) REFERENCES `project`(`id`) ON DELETE CASCADE
);
```

## ID Format

| Entity | Prefix | Example |
|--------|--------|---------|
| Session | `ses_` | `ses_305eeae76ffeZFzOzz903nD0J0` |
| Message | `msg_` | `msg_cfa1151ab001QbI6kqQG7fwKae` |
| Part | `prt_` | `prt_cfa1151ac001ncI58skuBtS8I8` |
| Project | (none) | SHA-1 hash |

## CLI Tools

```bash
# Run non-interactive session
opencode run "your message"
opencode run -m openai/gpt-4o-mini "say hello"

# Export session as JSON
opencode export <session-id>

# List/delete sessions
opencode session list
opencode session delete <session-id>

# Direct SQL queries
opencode db "SELECT * FROM session" --format json
opencode db path   # Print database path

# Token usage statistics
opencode stats

# List available models
opencode models [provider]
```

## Retrieval Method

```python
import json
import sqlite3
from pathlib import Path

db_path = Path.home() / ".local" / "share" / "opencode" / "opencode.db"
conn = sqlite3.connect(str(db_path))
conn.row_factory = sqlite3.Row
cursor = conn.cursor()

cursor.execute("""
    SELECT
        m.id AS message_id,
        m.time_created,
        m.data AS message_data,
        p.worktree AS project_worktree,
        s.slug AS session_slug,
        s.title AS session_title,
        pt.data AS part_data
    FROM message m
    JOIN session s ON s.id = m.session_id
    JOIN project p ON p.id = s.project_id
    JOIN part pt ON pt.message_id = m.id
    WHERE s.parent_id IS NULL
    ORDER BY m.time_created ASC, pt.time_created ASC
""")

for row in cursor.fetchall():
    message_data = json.loads(row["message_data"])
    if message_data.get("role") != "user":
        continue

    part_data = json.loads(row["part_data"])
    if part_data.get("type") != "text":
        continue

    text = part_data.get("text", "").strip()
    project = Path(row["project_worktree"]).name if row["project_worktree"] else "unknown"
    if text:
        print(f"[{project}/{row['session_slug']}] {text}")

conn.close()
```

## Notes

- `session.parent_id IS NULL` targets only parent sessions (excludes sub-agent child sessions)
- Timestamps are Unix epoch in milliseconds
- Project ID is a SHA-1 hash of the worktree path
- Session slugs are human-readable two-word names (e.g., "brave-falcon")
- The `snapshot/` directory contains bare git repos used for computing diffs between steps
- `storage/session_diff/` stores per-session diff data as JSON arrays
- Debug logs in `log/` are timestamped text files
- Schema is managed by Drizzle ORM with bundled migrations
- The `opencode export` command provides a structured JSON view of session data
- The `opencode db` command allows direct SQL queries against the database
