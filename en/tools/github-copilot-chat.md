# GitHub Copilot Chat

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code Extension |
| Log Format | SQLite (`state.vscdb`) |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Code/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Code/User/workspaceStorage/*/state.vscdb` |

## Data Format

SQLite database. Chat history is stored as JSON strings under specific keys in the `ItemTable` table.

### Table Structure

```sql
CREATE TABLE ItemTable (key TEXT UNIQUE ON CONFLICT REPLACE, value BLOB);
```

### Primary Keys

| Key | Description |
|-----|-------------|
| `memento/interactive-session` | Prompt history (primary) |
| `interactive.sessions` | Chat session index |
| `chat.ChatSessionStore.index` | Alternative key (varies by version) |

### Data Structure (`memento/interactive-session`)

```json
{
  "history": {
    "panel": [
      {
        "text": "User's prompt",
        "result": "Copilot's response..."
      }
    ],
    "inline": [
      {
        "text": "Prompt entered inline"
      }
    ]
  }
}
```

## Retrieval Method

```python
import json
import sqlite3
from pathlib import Path

def get_workspace_storage():
    """Return the workspaceStorage path based on the OS"""
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Code" / "User" / "workspaceStorage"

storage = get_workspace_storage()
for vscdb_path in storage.glob("*/state.vscdb"):
    conn = sqlite3.connect(str(vscdb_path))
    cursor = conn.cursor()

    cursor.execute(
        "SELECT value FROM ItemTable WHERE key = 'memento/interactive-session'"
    )
    row = cursor.fetchone()
    if row and row[0]:
        data = json.loads(row[0])
        history = data.get("history", {})
        for mode_key, entries in history.items():
            if isinstance(entries, list):
                for entry in entries:
                    text = entry.get("text", "").strip()
                    if text:
                        print(f"[{mode_key}] {text}")
    conn.close()
```

### Checking with sqlite3 Command

```bash
# List available keys
sqlite3 "<path>/state.vscdb" "SELECT key FROM ItemTable WHERE key LIKE '%chat%';"

# Session index
sqlite3 "<path>/state.vscdb" "SELECT value FROM ItemTable WHERE key = 'interactive.sessions';"
```

## Notes

- A separate `state.vscdb` exists for each workspace
- Chat messages themselves may not contain timestamps (use the file modification time as a substitute)
- The `sqlite3` command is required
- Key names may change depending on the Copilot version
