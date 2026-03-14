# JetBrains AI Assistant

## Basic Information

| Item | Details |
|------|---------|
| Type | Built-in IDE feature (all JetBrains products) |
| Log Format | XML / JSON |
| Local Logs | Yes |

## Storage Location

Stored in the JetBrains IDE settings directory:

| OS | Path |
|----|------|
| Windows | `%APPDATA%\JetBrains\<IDE><version>\` |
| macOS | `~/Library/Application Support/JetBrains/<IDE><version>/` |
| Linux | `~/.config/JetBrains/<IDE><version>/` |

`<IDE>` is the product name:
- `IntelliJIdea` (IntelliJ IDEA Ultimate)
- `IdeaIC` (IntelliJ IDEA Community)
- `PyCharm` / `PyCharmCE`
- `WebStorm`
- `GoLand`
- `RubyMine`
- `CLion`
- `Rider`
- `DataGrip`

## File Structure (Estimated)

```
JetBrains/<IDE><version>/
├── options/
│   └── ai-assistant.xml           # AI Assistant settings
├── ai-assistant/
│   └── chats/
│       └── {session-id}.json      # Chat sessions
└── log/
    └── idea.log                   # IDE log (may contain AI-related logs)
```

## Data Format

Chat history is in JSON format:

```json
{
  "id": "session-uuid",
  "messages": [
    {
      "role": "user",
      "content": "User's prompt",
      "timestamp": "2025-06-15T10:30:00Z"
    },
    {
      "role": "assistant",
      "content": "AI's response"
    }
  ]
}
```

## Retrieval Method

```python
import json
from pathlib import Path

def get_jetbrains_dirs():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        base = Path(os.environ["APPDATA"]) / "JetBrains"
    elif system == "Darwin":
        base = Path.home() / "Library" / "Application Support" / "JetBrains"
    else:
        base = Path.home() / ".config" / "JetBrains"
    return list(base.iterdir()) if base.exists() else []

for ide_dir in get_jetbrains_dirs():
    chats_dir = ide_dir / "ai-assistant" / "chats"
    if not chats_dir.exists():
        continue
    for chat_file in chats_dir.glob("*.json"):
        with open(chat_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        for msg in data.get("messages", []):
            if msg.get("role") == "user":
                print(f"[{ide_dir.name}] {msg.get('content', '')}")
```

## Notes

- **Requires hands-on investigation**: The chat history storage path and data structure are estimated
- A separate directory is created for each IDE version
- Since AI Assistant is provided as a plugin, data may also exist in the plugin's settings directory
- If linked with a JetBrains account, history may also be stored on the cloud side
