# Google Antigravity

## Basic Information

| Item | Details |
|------|---------|
| Type | AI IDE / Agent |
| Log Format | Text (log files) |
| Local Logs | Yes (partial) |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%USERPROFILE%\.gemini\antigravity\brain\` |
| macOS | `~/.gemini/antigravity/brain/` + `~/.gemini/antigravity/conversations/` |
| Linux | `~/.gemini/antigravity/brain/` |

## File Structure

```
~/.gemini/antigravity/
├── brain/
│   └── {conversation-id}/
│       └── .system_generated/
│           └── logs/                  # Conversation logs (text format)
└── conversations/
    └── *.pb                           # Protocol Buffers format (not human-readable)
```

## Data Format

- Under `logs/`: Log files in text format
- `conversations/*.pb`: Protocol Buffers format (binary, not readable as text)

## Retrieval Method

```python
from pathlib import Path

brain_dir = Path.home() / ".gemini" / "antigravity" / "brain"
for log_dir in brain_dir.glob("*/.system_generated/logs"):
    for log_file in sorted(log_dir.rglob("*"), key=lambda p: p.stat().st_mtime, reverse=True):
        if not log_file.is_file() or log_file.suffix == ".pb":
            continue
        text = log_file.read_text(encoding="utf-8").strip()
        if text:
            print(f"[{log_file.parent.parent.parent.name[:12]}] {text[:200]}")
```

## Notes

- As a relatively new tool, the log format may change
- If the `.gemini/` folder is deleted, the conversation list remains but the content becomes unreadable (known bug)
- `.pb` files are in Protocol Buffers format and cannot be read as text, so they should be skipped
- Use file modification time as a substitute for timestamps
