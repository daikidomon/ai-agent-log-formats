# OpenAI Codex (CLI)

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI |
| Log Format | JSONL (rollout session files) |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%USERPROFILE%\.codex\sessions\` |
| macOS | `~/.codex/sessions/` |
| Linux | `~/.codex/sessions/` |

If the environment variable `CODEX_HOME` is set, `$CODEX_HOME/sessions/` is used.

## File Structure

```
~/.codex/
├── sessions/
│   └── rollout-YYYY-MM-DDThh-mm-ss-<uuid>.jsonl  # Conversation log per session
└── tmp/                                            # Temporary files
```

## Data Format

Each line is a JSON object with `timestamp`, `type`, and `payload` fields. The `type` field determines the entry kind.

### session_meta (Session Start Metadata)

```json
{
  "timestamp": "2026-03-17T08:30:00.000Z",
  "type": "session_meta",
  "payload": {
    "id": "abcd1234-5678-9012-3456-789012345678",
    "cwd": "/home/user/project",
    "cli_version": "0.114.0",
    "source": "cli",
    "model_provider": "openai",
    "git": {
      "commit_hash": "abc123def",
      "branch": "main",
      "repository_url": "https://github.com/user/project"
    }
  }
}
```

### event_msg (User Message / Task Events)

```json
{
  "timestamp": "2026-03-17T08:30:05.000Z",
  "type": "event_msg",
  "payload": {
    "type": "user_message",
    "message": "User's input text"
  }
}
```

### response_item (Model Response)

```json
{
  "timestamp": "2026-03-17T08:30:10.000Z",
  "type": "response_item",
  "payload": {
    "type": "message",
    "role": "assistant",
    "content": [
      {"type": "output_text", "text": "Assistant's response"}
    ]
  }
}
```

### response_item (Shell Command Execution)

```json
{
  "timestamp": "2026-03-17T08:30:18.000Z",
  "type": "response_item",
  "payload": {
    "type": "local_shell_call",
    "action": {
      "command": ["ls", "-la"],
      "cwd": "/home/user/project"
    }
  }
}
```

### response_item (Command Output)

```json
{
  "timestamp": "2026-03-17T08:30:20.000Z",
  "type": "response_item",
  "payload": {
    "type": "function_call_output",
    "output": "total 32\ndrwxr-xr-x  5 user user 4096 ..."
  }
}
```

### Entry Types

| `type` | `payload.type` | Description |
|--------|---------------|-------------|
| `session_meta` | — | Session metadata (cwd, model, git info) |
| `event_msg` | `user_message` | User input message |
| `event_msg` | `task_complete` | Task completion event |
| `response_item` | `message` | Model text response |
| `response_item` | `local_shell_call` | Shell command execution |
| `response_item` | `function_call_output` | Command/function output |

### session_meta Payload Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Session UUID |
| `cwd` | string | Project directory |
| `cli_version` | string | Codex CLI version |
| `source` | string | Source type (e.g., `"cli"`) |
| `model_provider` | string | Model provider (e.g., `"openai"`) |
| `git.commit_hash` | string | Current git commit hash |
| `git.branch` | string | Current git branch |
| `git.repository_url` | string | Git repository URL |

## Retrieval Method

```python
import json
from pathlib import Path

codex_home = Path.home() / ".codex"
sessions_dir = codex_home / "sessions"

for rollout_path in sorted(sessions_dir.glob("rollout-*.jsonl"),
                           key=lambda p: p.stat().st_mtime, reverse=True)[:50]:
    cwd = ""
    with open(rollout_path, "r", encoding="utf-8") as f:
        for line in f:
            entry = json.loads(line.strip())
            entry_type = entry.get("type")
            payload = entry.get("payload", {})

            # Get project info from session_meta
            if entry_type == "session_meta":
                cwd = payload.get("cwd", "")
                continue

            # Extract user messages from event_msg
            if entry_type == "event_msg" and payload.get("type") == "user_message":
                text = payload.get("message", "").strip()
                if text:
                    print(f"[{cwd}] {text}")

            # Extract assistant responses from response_item
            if entry_type == "response_item" and payload.get("type") == "message":
                for part in payload.get("content", []):
                    if isinstance(part, dict) and part.get("type") == "output_text":
                        print(f"  → {part.get('text', '')[:100]}")
```

## Notes

- Session files are stored flat in `sessions/` (no date-based subdirectories)
- Project information is obtained from the `cwd` field in `session_meta` payload
- JSON uses `type`/`payload` structure (not CamelCase top-level keys)
- Content types include `output_text` (responses), `local_shell_call` (commands), `function_call_output` (results)
- Session metadata includes git information (commit, branch, repository URL)
