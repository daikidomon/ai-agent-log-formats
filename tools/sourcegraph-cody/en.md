# Sourcegraph Cody

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code / JetBrains / CLI extension |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

### VS Code Extension

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\sourcegraph.cody-ai\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/sourcegraph.cody-ai/` |
| Linux | `~/.config/Code/User/globalStorage/sourcegraph.cody-ai/` |

### JetBrains

Stored in the JetBrains plugin data directory (under the IDE settings directory).

## File Structure (Estimated)

```
sourcegraph.cody-ai/
├── chat/
│   └── {session-id}.json          # Chat sessions
└── state/
    └── ...
```

## Data Format

Chat history is stored in JSON format.

```json
{
  "id": "session-uuid",
  "interactions": [
    {
      "humanMessage": {
        "text": "User's prompt"
      },
      "assistantMessage": {
        "text": "Cody's response"
      }
    }
  ]
}
```

## Retrieval Method

```python
import json
from pathlib import Path

def get_cody_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"
    else:
        return Path.home() / ".config" / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"

storage = get_cody_storage()
# Search for chat history files
for json_file in storage.rglob("*.json"):
    try:
        with open(json_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        # Parse based on structure
        if "interactions" in data:
            for interaction in data["interactions"]:
                human_msg = interaction.get("humanMessage", {})
                text = human_msg.get("text", "").strip()
                if text:
                    print(f"[cody] {text}")
    except (json.JSONDecodeError, OSError):
        continue
```

## Notes

- **Requires hands-on investigation**: The structure above is estimated. It may vary depending on the Cody version
- Data may also be stored in VS Code's `state.vscdb`
- If using the Cody CLI (`cody`), the log path may be different
- When connected to a Sourcegraph server, some logs may be stored on the server side
