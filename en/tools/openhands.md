# OpenHands (formerly OpenDevin)

## Basic Information

| Item | Details |
|------|---------|
| Type | Autonomous agent (OSS) |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

```
workspace/
└── .openhands/
    └── sessions/
        └── {session-id}/
            ├── events.json        # Event log
            └── state.json         # Session state
```

Or under the directory specified by the `WORKSPACE_BASE` environment variable.

## Data Format

### events.json

An event-driven log. Each event is a single object:

```json
[
  {
    "id": 1,
    "timestamp": "2025-06-15T10:30:00Z",
    "source": "user",
    "action": "message",
    "args": {
      "content": "User's prompt"
    }
  },
  {
    "id": 2,
    "timestamp": "2025-06-15T10:30:05Z",
    "source": "agent",
    "action": "run",
    "args": {
      "command": "ls -la"
    }
  },
  {
    "id": 3,
    "timestamp": "2025-06-15T10:30:06Z",
    "source": "agent",
    "observation": "CmdOutputObservation",
    "content": "..."
  }
]
```

| Field | Type | Description |
|-------|------|-------------|
| `source` | string | `"user"` / `"agent"` |
| `action` | string | Action type (`"message"`, `"run"`, `"edit"`, etc.) |
| `args.content` | string | User's message |
| `observation` | string | Execution result type |

## Retrieval Method

```python
import json
from pathlib import Path

sessions_dir = Path("workspace/.openhands/sessions")
if sessions_dir.exists():
    for session_dir in sessions_dir.iterdir():
        events_file = session_dir / "events.json"
        if not events_file.exists():
            continue

        with open(events_file, "r", encoding="utf-8") as f:
            events = json.load(f)

        for event in events:
            if event.get("source") == "user" and event.get("action") == "message":
                text = event.get("args", {}).get("content", "").strip()
                if text:
                    print(f"[{session_dir.name}] {text}")
```

## Notes

- When running inside a Docker container, logs are saved at paths within the container
- The workspace location can be changed using the `WORKSPACE_BASE` environment variable
- Event logs include all agent actions (command execution, file editing, etc.)
- When using via the Web UI, session data is also stored on the server side
