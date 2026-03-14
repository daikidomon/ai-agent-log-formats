# Continue

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code / JetBrains extension (OSS) |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%USERPROFILE%\.continue\` |
| macOS | `~/.continue/` |
| Linux | `~/.continue/` |

## File Structure

```
~/.continue/
├── config.json                    # Configuration file
├── config.yaml                    # Configuration file (YAML format)
├── sessions/
│   └── {session-id}.json          # Per-session conversation logs
├── dev_data/
│   └── *.jsonl                    # Development data logs
└── index/                         # Code index
```

## Data Format

### sessions/{session-id}.json

```json
{
  "sessionId": "uuid-string",
  "title": "Session title",
  "dateCreated": "2025-06-15T10:30:00.000Z",
  "history": [
    {
      "message": {
        "role": "user",
        "content": "User's prompt"
      },
      "editorState": {},
      "contextItems": []
    },
    {
      "message": {
        "role": "assistant",
        "content": "Assistant's response"
      }
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `sessionId` | string | Session UUID |
| `title` | string | Session title (auto-generated) |
| `dateCreated` | string | ISO 8601 |
| `history[].message.role` | string | `"user"` / `"assistant"` |
| `history[].message.content` | string | Message body |
| `history[].contextItems` | array | Attached context information |

### dev_data/*.jsonl

Development data records (one JSON per line). Model inputs/outputs and metadata are recorded:

```json
{"timestamp": 1718441400000, "type": "chat", "prompt": "...", "completion": "..."}
```

## Retrieval Method

```python
import json
from pathlib import Path

continue_dir = Path.home() / ".continue"
sessions_dir = continue_dir / "sessions"

if sessions_dir.exists():
    for session_file in sorted(sessions_dir.glob("*.json"),
                                key=lambda p: p.stat().st_mtime, reverse=True):
        with open(session_file, "r", encoding="utf-8") as f:
            session = json.load(f)

        title = session.get("title", "untitled")
        for entry in session.get("history", []):
            message = entry.get("message", {})
            if message.get("role") == "user":
                text = message.get("content", "").strip()
                if text:
                    print(f"[{title}] {text}")
```

## Notes

- `~/.continue/` is centralized (logs from all projects in one location)
- `contextItems` may contain file contents or code snippets
- Both VS Code and JetBrains use the same directory
- `dev_data/` is only generated when Continue has anonymous usage data collection enabled
