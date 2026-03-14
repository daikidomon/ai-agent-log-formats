# Zed AI

## Basic Information

| Item | Details |
|------|---------|
| Type | Editor built-in AI (Rust-based editor) |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.config/zed/conversations/` |
| Linux | `~/.config/zed/conversations/` |

Note: Zed currently supports macOS / Linux only.

## File Structure (Estimated)

```
~/.config/zed/
├── settings.json                  # Editor settings
├── conversations/
│   └── {conversation-id}.json     # AI conversation logs
└── prompts/
    └── *.md                       # Custom prompts
```

## Data Format

Conversation logs are in JSON format (estimated):

```json
{
  "id": "conversation-uuid",
  "messages": [
    {
      "role": "user",
      "content": "User's prompt"
    },
    {
      "role": "assistant",
      "content": "AI response"
    }
  ]
}
```

## Retrieval Method

```python
import json
from pathlib import Path

zed_conversations = Path.home() / ".config" / "zed" / "conversations"
if zed_conversations.exists():
    for conv_file in sorted(zed_conversations.glob("*.json"),
                            key=lambda p: p.stat().st_mtime, reverse=True):
        with open(conv_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        for msg in data.get("messages", []):
            if msg.get("role") == "user":
                print(f"[zed] {msg.get('content', '')}")
```

## Notes

- **Requires hands-on investigation**: The conversation log storage path and data structure are estimated
- Zed's AI features are under rapid development, and the storage structure is likely to change
- Windows version has not been released
- User-defined custom prompts are stored in `prompts/`
