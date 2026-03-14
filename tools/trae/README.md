# Trae

## Basic Information

| Item | Details |
|------|---------|
| Type | AI IDE (by ByteDance, VS Code based) |
| Log Format | SQLite / JSON |
| Local Logs | Yes (estimated) |

## Storage Location

Uses a similar directory structure to VS Code since it is VS Code based (estimated):

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Trae\User\` |
| macOS | `~/Library/Application Support/Trae/User/` |
| Linux | `~/.config/Trae/User/` |

## File Structure (Estimated)

```
Trae/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## Data Format

As a VS Code fork, it is based on the `state.vscdb` (SQLite) structure, but there may be additional storage for Trae's proprietary AI features.

## Retrieval Method

```python
import sqlite3
from pathlib import Path

def get_trae_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Trae" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Trae" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Trae" / "User" / "workspaceStorage"

storage = get_trae_storage()
if storage.exists():
    for vscdb_path in storage.glob("*/state.vscdb"):
        conn = sqlite3.connect(str(vscdb_path))
        cursor = conn.cursor()
        cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%ai%'")
        for row in cursor.fetchall():
            print(f"Key: {row[0]}")
        conn.close()
```

## Notes

- **Requires hands-on investigation**: The log format and storage location for Trae have not been confirmed
- Although VS Code based, the application name is `Trae`, so the directory name differs
- If linked to a ByteDance account, data may also be stored on the cloud side
