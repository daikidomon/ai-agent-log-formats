# Cline

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code Extension |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\saoudrizwan.claude-dev\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/saoudrizwan.claude-dev/` |
| Linux | `~/.config/Code/User/globalStorage/saoudrizwan.claude-dev/` |

## File Structure

```
saoudrizwan.claude-dev/
├── state/
│   └── taskHistory.json              # Task history index
└── tasks/
    └── {task-id}/
        ├── api_conversation_history.json  # API conversation log (primary)
        ├── ui_messages.json               # UI display messages
        └── task_metadata.json             # Task metadata
```

## Data Format

### api_conversation_history.json

```json
[
  {
    "role": "user",
    "content": [
      {
        "type": "text",
        "text": "User's prompt"
      }
    ]
  },
  {
    "role": "assistant",
    "content": [
      {
        "type": "text",
        "text": "Cline's response..."
      }
    ]
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `role` | string | `"user"` or `"assistant"` |
| `content` | array/string | Array of content blocks or text |
| `content[].type` | string | `"text"`, `"image"`, etc. |
| `content[].text` | string | Text body |

### ui_messages.json

Messages displayed on the UI side. More human-readable than `api_conversation_history.json`, but differs from the content sent to the API.

## Retrieval Method

```python
import json
from pathlib import Path

def get_cline_tasks_dir():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        base = Path(os.environ["APPDATA"])
    elif system == "Darwin":
        base = Path.home() / "Library" / "Application Support"
    else:
        base = Path.home() / ".config"
    return base / "Code" / "User" / "globalStorage" / "saoudrizwan.claude-dev" / "tasks"

tasks_dir = get_cline_tasks_dir()
for task_dir in sorted(tasks_dir.iterdir(), key=lambda p: p.stat().st_mtime, reverse=True):
    history_file = task_dir / "api_conversation_history.json"
    if not history_file.exists():
        continue

    with open(history_file, "r", encoding="utf-8") as f:
        data = json.load(f)

    for msg in data:
        if msg.get("role") == "user":
            content = msg.get("content", "")
            if isinstance(content, list):
                text = " ".join(c.get("text", "") for c in content if c.get("type") == "text")
            else:
                text = str(content)
            if text.strip():
                print(text.strip())
```

## Notes

- Task IDs are in UUID format
- `api_conversation_history.json` also contains tool call results (long text with XML tags in `content`)
- Individual timestamps are not included in messages (use `task_metadata.json` or file modification time as a substitute)
- Roo Code and Kilo Code share the same structure (Cline forks)
