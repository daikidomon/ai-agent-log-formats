# Cursor

## Basic Information

| Item | Details |
|------|---------|
| Type | AI-specialized editor (VS Code fork) |
| Log Format | SQLite (`state.vscdb`) + proprietary storage |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Cursor\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Cursor/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Cursor/User/workspaceStorage/*/state.vscdb` |

### Global Storage

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Cursor\User\globalStorage\` |
| macOS | `~/Library/Application Support/Cursor/User/globalStorage/` |
| Linux | `~/.config/Cursor/User/globalStorage/` |

## File Structure

```
Cursor/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb              # Per-workspace state DB
├── globalStorage/
│   └── storage.json                 # Global settings
└── History/                         # File edit history
```

## Data Format

As a VS Code fork, it primarily uses `state.vscdb` (SQLite).

### state.vscdb

```sql
-- Search for keys related to chat history
SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%' OR key LIKE '%cursor%';
```

Cursor-specific features:
- **Composer**: Multi-file editing sessions
- **Chat**: Side panel chat
- **Cmd+K**: Inline edits

Logs for each feature may be stored under different keys.

### Expected Key Patterns

| Key (estimated) | Description |
|-----------------|-------------|
| `composer.*` | Composer sessions |
| `cursorChat.*` | Chat history |
| `aiChat.*` | AI chat sessions |

## Retrieval Method

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

    # First, check the list of keys
    cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%'")
    keys = [row[0] for row in cursor.fetchall()]
    print(f"Keys found: {keys}")

    # Check the value of each key
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

## Notes

- Although it is a VS Code fork, the chat functionality is Cursor's own implementation
- Composer / Chat / Cmd+K may store data in different locations
- The storage structure changes frequently with version updates
- Additional proprietary DB files may exist beyond `state.vscdb`
- **Requires hands-on investigation**: The key patterns above are estimated. Verification in an actual environment is recommended
