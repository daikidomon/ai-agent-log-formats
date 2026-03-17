# Gemini CLI

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI |
| Log Format | JSON |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.gemini/` |
| Linux | `~/.gemini/` |
| Windows | `%USERPROFILE%\.gemini\` |

If the environment variable `GEMINI_CLI_HOME` is set, that path takes precedence.

## File Structure

```
~/.gemini/
├── settings.json                              # User configuration
├── GEMINI.md                                  # Global custom instructions
├── projects.json                              # Project path → slug mapping
├── history/
│   └── <project-slug>/                        # Per-project history directory
│       └── .project_root                      # Marker file with full project path
└── tmp/
    └── <project-slug>/                        # Slugified project basename (not a hash)
        ├── .project_root                      # Marker file with full project path
        ├── chats/
        │   └── session-<timestamp>-<id>.json  # Auto-saved session files
        ├── logs/
        │   └── session-<sessionId>.jsonl      # Activity logs
        ├── checkpoints/                       # Session checkpoints
        ├── shell_history                      # Shell command history
        └── <sessionId>/
            ├── plans/                         # Agent plans
            └── tasks/                         # Agent tasks
```

### Project Slug Naming

Project directories are named using a **human-readable slug** derived from the project directory basename (not a hash):

- `/home/ubuntu/my-project` → `my-project`
- `/home/ubuntu` → `ubuntu`
- Collisions are handled with counter suffixes: `my-project`, `my-project-1`, `my-project-2`

The mapping is stored in `~/.gemini/projects.json`:

```json
{
  "projects": {
    "/home/ubuntu": "ubuntu",
    "/home/ubuntu/my-project": "my-project"
  }
}
```

## Data Format

### Session JSON (ConversationRecord)

Each session file is a `ConversationRecord` object:

```json
{
  "sessionId": "uuid-string",
  "projectHash": "sha256-of-project-root",
  "startTime": "2026-03-17T09:45:00.000Z",
  "lastUpdated": "2026-03-17T10:20:00.000Z",
  "summary": "Optional auto-generated summary",
  "kind": "main",
  "messages": [
    {
      "id": "uuid-string",
      "timestamp": "2026-03-17T09:45:01.000Z",
      "type": "user",
      "content": [{"text": "User prompt text"}]
    },
    {
      "id": "uuid-string",
      "timestamp": "2026-03-17T09:45:05.000Z",
      "type": "gemini",
      "content": [{"text": "Model response text"}],
      "toolCalls": [],
      "thoughts": [],
      "tokens": {
        "input": 100,
        "output": 200,
        "cached": 50,
        "total": 350
      },
      "model": "gemini-2.5-pro"
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `sessionId` | string | Session UUID |
| `projectHash` | string | SHA256 hash of project root path |
| `startTime` | string | ISO 8601 session start time |
| `lastUpdated` | string | ISO 8601 last update time |
| `messages` | array | Array of `MessageRecord` objects |
| `summary` | string? | Auto-generated session summary |
| `kind` | string? | `"main"` or `"subagent"` |
| `directories` | string[]? | Workspace dirs added via `/dir add` |

### Message Types

| `type` | Description |
|--------|-------------|
| `user` | User input |
| `gemini` | Model response (includes `toolCalls`, `thoughts`, `tokens`, `model`) |
| `info` | Informational message |
| `error` | Error message |
| `warning` | Warning message |

### Session File Naming

Files follow the pattern: `session-YYYY-MM-DDTHH-MM-<sessionId:8>.json`

Example: `session-2026-03-17T09-45-abc12def.json`

### Headless Mode JSON Output (`--output-format json`)

```json
{
  "response": "Plain-text response from the model",
  "stats": {
    "session": {},
    "model": {},
    "tools": {},
    "user": {}
  },
  "error": null
}
```

## Session Management Commands

| Command | Description |
|---------|-------------|
| `/chat save <tag>` | Save current conversation under a tag |
| `/chat list` | List all saved sessions |
| `/chat resume <tag>` | Resume a saved session (accepts tag, index, or `latest`) |
| `/chat share` | Export session to a shareable file |

## Retrieval Method

```python
import json
from pathlib import Path

gemini_dir = Path.home() / ".gemini"

# Read project-specific auto-saved sessions from tmp directory
tmp_dir = gemini_dir / "tmp"
if tmp_dir.exists():
    for project_dir in tmp_dir.iterdir():
        chats_subdir = project_dir / "chats"
        if chats_subdir.exists():
            for session_file in chats_subdir.glob("session-*.json"):
                with open(session_file, "r", encoding="utf-8") as f:
                    record = json.load(f)
                    for msg in record.get("messages", []):
                        if msg.get("type") == "user":
                            content = msg.get("content", [])
                            for part in content:
                                if isinstance(part, dict):
                                    print(part.get("text", ""))
```

## Session Retention Configuration

Configure automatic cleanup in `settings.json`:

```json
{
  "general": {
    "sessionRetention": {
      "enabled": true,
      "maxAge": "30d",
      "maxCount": 50
    }
  }
}
```

| Setting | Default | Description |
|---------|---------|-------------|
| `enabled` | `true` | Enable automatic cleanup |
| `maxAge` | `"30d"` | Maximum session age (e.g., `"24h"`, `"7d"`, `"4w"`) |
| `maxCount` | `50` | Maximum number of sessions to retain |
| `minRetention` | `"1d"` | Safety limit preventing deletion of recent sessions |

Cleanup also removes associated activity logs (`logs/session-*.jsonl`) and tool output directories.

## Notes

- Sessions are project-specific; switching directories changes accessible session history
- Project directories use slugified basenames, not hashes (mapping stored in `projects.json`)
- Using `/chat save` with an existing tag overwrites without warning
- No built-in versioning; each tag stores only one session
- Shell command history is stored separately in `shell_history`
- Subagent sessions (`kind: "subagent"`) are stored alongside main sessions but hidden from the resume list
- Telemetry can be configured to log prompts locally via the `telemetry.outfile` setting
