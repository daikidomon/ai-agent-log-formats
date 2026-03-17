# Cursor

## Basic Information

| Item | Details |
|------|---------|
| Type | AI-specialized editor (VS Code fork) + CLI agent |
| Log Format | JSONL (transcripts) + SQLite (chat state) |
| Local Logs | Yes |

## Storage Location

### Cursor Agent CLI

| OS | Path |
|----|------|
| macOS | `~/.cursor/` |
| Linux | `~/.cursor/` |
| Windows | `%USERPROFILE%\.cursor\` |

Override with environment variables `CURSOR_CONFIG_DIR` or `CURSOR_DATA_DIR`.

### Cursor GUI (Editor)

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Cursor\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Cursor/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Cursor/User/workspaceStorage/*/state.vscdb` |

## File Structure (CLI Agent - Verified)

```
~/.cursor/
├── cli-config.json                                        # CLI settings + auth info
├── statsig-cache.json                                     # Feature flag cache
├── chats/
│   └── <project-md5>/                                     # MD5 hash of project absolute path
│       └── <session-uuid>/
│           └── store.db                                   # SQLite: chat state (Merkle tree blobs)
├── projects/
│   └── <encoded-path>/                                    # Path separators (/) → - (no leading -)
│       ├── repo.json                                      # Project metadata (id)
│       ├── worker.log                                     # Worker/indexing log
│       ├── worker.sock                                    # Worker Unix socket
│       ├── .workspace-trusted                             # Trust marker
│       └── agent-transcripts/
│           └── <session-uuid>/
│               └── <session-uuid>.jsonl                   # Conversation transcript (JSONL)
└── skills-cursor/                                         # Built-in skills
    ├── .cursor-managed-skills-manifest.json
    ├── create-rule/SKILL.md
    ├── create-skill/SKILL.md
    ├── create-subagent/SKILL.md
    ├── shell/SKILL.md
    └── update-cursor-settings/SKILL.md
```

### Path Encoding Rules

#### Project Path (`projects/`)

Path separators `/` are converted to `-`, **without** a leading `-`:

```
/home/ubuntu/my-project → home-ubuntu-my-project
```

#### Chat Directory (`chats/`)

Directory name is the **MD5 hash** of the project absolute path:

```
/home/ubuntu/ai-agent-log-formats → cda24b2b5f52729dbc12e2aaada6b933
```

## Data Format (CLI Agent)

### Agent Transcript JSONL (Primary Conversation Log)

Located at `~/.cursor/projects/<encoded-path>/agent-transcripts/<session-uuid>/<session-uuid>.jsonl`

One JSON object per line:

```json
{
  "role": "user",
  "message": {
    "content": [
      {
        "type": "text",
        "text": "<user_query>\nUser's input text\n</user_query>"
      }
    ]
  }
}
```

```json
{
  "role": "assistant",
  "message": {
    "content": [
      {
        "type": "text",
        "text": "Assistant's response"
      }
    ]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `role` | string | `"user"` or `"assistant"` |
| `message.content` | array | Array of content blocks |
| `message.content[].type` | string | `"text"` |
| `message.content[].text` | string | Message text (user input wrapped in `<user_query>` tags) |

### store.db (SQLite Chat State)

Located at `~/.cursor/chats/<project-md5>/<session-uuid>/store.db`

#### Schema

```sql
CREATE TABLE blobs (id TEXT PRIMARY KEY, data BLOB);
CREATE TABLE meta (key TEXT PRIMARY KEY, value TEXT);
```

#### meta Table

Stores session metadata as hex-encoded JSON:

```json
{
  "agentId": "b172f4ff-d714-4b6d-9caf-a3f2a12a0188",
  "latestRootBlobId": "5c5994a555062ffea...",
  "name": "New Agent",
  "mode": "default",
  "createdAt": 1773719337438
}
```

#### blobs Table

Stores conversation data in a Merkle tree structure. Blob IDs are SHA-256 hashes. Blobs contain:
- User messages with `providerOptions` and `requestId`
- Assistant responses with content arrays
- Tree structure nodes linking message sequences

### cli-config.json

```json
{
  "version": 1,
  "authInfo": {
    "email": "user@example.com",
    "displayName": "User Name",
    "userId": 123456789,
    "authId": "github|user_..."
  },
  "permissions": {
    "allow": ["Shell(ls)"],
    "deny": []
  },
  "approvalMode": "allowlist",
  "sandbox": {
    "mode": "disabled",
    "networkAccess": "user_config_with_defaults"
  },
  "editor": {
    "vimMode": false
  },
  "attribution": {
    "attributeCommitsToAgent": true,
    "attributePRsToAgent": true
  }
}
```

## Retrieval Method (CLI Agent)

```python
import json
from pathlib import Path

cursor_dir = Path.home() / ".cursor"

# Read agent transcripts (JSONL)
projects_dir = cursor_dir / "projects"
for project_dir in projects_dir.iterdir():
    transcripts_dir = project_dir / "agent-transcripts"
    if not transcripts_dir.exists():
        continue
    for session_dir in transcripts_dir.iterdir():
        for jsonl_file in session_dir.glob("*.jsonl"):
            with open(jsonl_file, "r", encoding="utf-8") as f:
                for line in f:
                    entry = json.loads(line.strip())
                    if entry.get("role") == "user":
                        for part in entry.get("message", {}).get("content", []):
                            text = part.get("text", "")
                            # Strip <user_query> wrapper
                            text = text.replace("<user_query>\n", "").replace("\n</user_query>", "")
                            print(text)
```

## Data Format (GUI Editor)

The Cursor GUI editor (VS Code fork) uses `state.vscdb` (SQLite) for workspace-level storage.

```python
import json
import sqlite3
from pathlib import Path

def get_cursor_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Cursor" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Cursor" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Cursor" / "User" / "workspaceStorage"

storage = get_cursor_storage()
for vscdb_path in storage.glob("*/state.vscdb"):
    conn = sqlite3.connect(str(vscdb_path))
    cursor = conn.cursor()
    cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%'")
    keys = [row[0] for row in cursor.fetchall()]
    for key in keys:
        cursor.execute("SELECT value FROM ItemTable WHERE key = ?", (key,))
        row = cursor.fetchone()
        if row and row[0]:
            try:
                data = json.loads(row[0])
                print(f"[{key}] {json.dumps(data, ensure_ascii=False)[:200]}")
            except (json.JSONDecodeError, TypeError):
                print(f"[{key}] (non-JSON data, {len(row[0])} bytes)")
    conn.close()
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CURSOR_API_KEY` | API key for authentication |
| `CURSOR_CONFIG_DIR` | Override config directory |
| `CURSOR_DATA_DIR` | Override data directory |
| `CURSOR_WORKTREES_ROOT` | Worktree root directory |
| `AGENT_CLI_LOG_PATH` | Debug log output path |

## Notes

- Cursor Agent CLI stores data in `~/.cursor/` separately from the GUI editor
- CLI transcripts are plain JSONL (easy to parse); GUI uses `state.vscdb` (SQLite)
- Chat directory uses **MD5 hash** of project path; project directory uses **dash-encoded** path
- `store.db` uses a Merkle tree blob structure for conversation state
- User messages in transcripts are wrapped in `<user_query>` tags
- The GUI editor's `state.vscdb` key patterns may vary by version
- Skills are stored as managed SKILL.md files under `skills-cursor/`
