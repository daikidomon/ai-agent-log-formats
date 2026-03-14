# OpenCode

## Basic Information

| Item | Details |
|------|---------|
| Type | TUI Editor |
| Log Format | SQLite |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%USERPROFILE%\.local\share\opencode\` |
| macOS | `~/.local/share/opencode/` |
| Linux | `~/.local/share/opencode/` |

If the environment variable `XDG_DATA_HOME` is set, `$XDG_DATA_HOME/opencode/` is used.

## File Structure

```
~/.local/share/opencode/
├── opencode.db                # Main database
└── opencode-<channel>.db     # Per-channel database (if present)
```

## Data Format

SQLite database. Primary tables:

| Table | Description |
|-------|-------------|
| `project` | Project information (`worktree`, `name`, etc.) |
| `session` | Per-session metadata |
| `message` | Per-message data. JSON in the `data` column |
| `part` | Individual parts of a message. JSON in the `data` column |

### message.data Structure

```json
{
  "role": "user",
  "metadata": {}
}
```

### part.data Structure

```json
{
  "type": "text",
  "text": "User's prompt",
  "synthetic": false,
  "ignored": false
}
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
        s.parent_id AS session_parent_id,
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
    if part_data.get("synthetic") is True or part_data.get("ignored") is True:
        continue

    text = part_data.get("text", "").strip()
    project = Path(row["project_worktree"]).name if row["project_worktree"] else "unknown"
    if text:
        print(f"[{project}] {text}")

conn.close()
```

## Notes

- `session.parent_id IS NULL` targets only parent sessions (excludes sub-agent child sessions)
- `part.data.synthetic = true` indicates system-generated parts (should be skipped)
- `part.data.ignored = true` indicates parts ignored by the user (should be skipped)
- Timestamps are in `message.time_created` (Unix epoch in milliseconds)
- Project name is the basename of `project.worktree`
- `opencode.db` takes priority; if not found, search for `opencode-*.db`
