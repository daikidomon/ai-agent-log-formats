# Claude Code (CLI / VS Code Extension)

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI / VS Code Extension |
| Log Format | JSONL |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| Windows | `C:\Users\<user>\.claude\` |
| macOS | `~/.claude/` |
| Linux | `~/.claude/` |

If the environment variable `CLAUDE_CONFIG_DIR` is set, that path takes precedence.

## File Structure

```
~/.claude/
├── history.jsonl                          # Lightweight index for all projects (CLI usage)
├── settings.json                          # Global settings
├── settings.local.json                    # Local settings overrides
├── projects/
│   └── {encoded-path}/                    # Path separators (\, /, :) converted to -
│       ├── {session-uuid}.jsonl           # Full conversation log per session
│       └── {session-uuid}/
│           └── subagents/                 # Sub-agent logs
├── backups/                               # Session backups
├── cache/                                 # Cached data
├── debug/                                 # Debug logs
├── file-history/                          # File edit history per session
├── plans/                                 # Plan documents (*.md)
├── session-env/                           # Session environment snapshots
├── shell-snapshots/                       # Shell state snapshots
├── tasks/                                 # Task tracking data
├── telemetry/                             # Telemetry data
└── todos/                                 # Todo items
```

## Data Format

### history.jsonl (Index for CLI Usage)

One JSON object per line:

```json
{
  "display": "User input text",
  "pastedContents": {},
  "timestamp": 1759115795246,
  "project": "/home/user/project",
  "sessionId": "uuid-string"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `display` | string | The prompt entered by the user |
| `timestamp` | number | Unix epoch in milliseconds |
| `project` | string | Absolute path of the project |
| `sessionId` | string | Session UUID |
| `pastedContents` | object | Pasted content (keyed by numeric ID) |

### Per-Project Session JSONL (Including VS Code Extension)

One JSON object per line. Each line has a `type` field to indicate the entry type.

#### User Message (`type: "user"`)

```json
{
  "type": "user",
  "uuid": "a08c057d-cf4e-44d3-afad-5795bbc7539f",
  "parentUuid": null,
  "sessionId": "20223f6f-23fc-4691-a054-6146beec7770",
  "version": "2.1.58",
  "cwd": "/path/to/project",
  "gitBranch": "main",
  "userType": "external",
  "permissionMode": "default",
  "isSidechain": false,
  "timestamp": "2026-03-17T00:25:44.657Z",
  "message": {
    "role": "user",
    "content": "User prompt text"
  },
  "todos": []
}
```

#### Assistant Message (`type: "assistant"`)

```json
{
  "type": "assistant",
  "uuid": "d09ed28e-1f0d-4849-8837-6bbba1bde931",
  "parentUuid": "a08c057d-cf4e-44d3-afad-5795bbc7539f",
  "sessionId": "20223f6f-23fc-4691-a054-6146beec7770",
  "version": "2.1.58",
  "cwd": "/path/to/project",
  "gitBranch": "main",
  "isSidechain": false,
  "timestamp": "2026-03-17T00:25:50.123Z",
  "message": {
    "model": "claude-opus-4-6",
    "id": "msg_01VyfRCVSGXTj8HSxTXTSXVt",
    "type": "message",
    "role": "assistant",
    "content": [
      {"type": "thinking", "thinking": "..."},
      {"type": "text", "text": "Response text"},
      {"type": "tool_use", "id": "toolu_...", "name": "Bash", "input": {}}
    ],
    "usage": {
      "input_tokens": 1000,
      "output_tokens": 500,
      "cache_creation_input_tokens": 200,
      "cache_read_input_tokens": 800
    },
    "stop_reason": "end_turn"
  },
  "requestId": "req_..."
}
```

### Common Fields in Session JSONL

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Entry type (see below) |
| `uuid` | string | Unique message ID |
| `parentUuid` | string/null | Parent message UUID (conversation tree) |
| `sessionId` | string | Session UUID |
| `version` | string | Claude Code version |
| `cwd` | string | Working directory |
| `gitBranch` | string | Current git branch |
| `timestamp` | string | ISO 8601 |
| `isSidechain` | boolean | `true` for sub-agent messages |
| `userType` | string | e.g., `"external"` |
| `permissionMode` | string | e.g., `"default"` |
| `isMeta` | boolean | If `true`, this is a system message (should be skipped) |

### Entry Types

| Type | Description |
|------|-------------|
| `user` | User input message |
| `assistant` | Model response (includes tool calls, thinking, usage) |
| `progress` | Hook/tool execution progress |
| `system` | System messages (e.g., `turn_duration`) |
| `file-history-snapshot` | File version tracking snapshot |
| `queue-operation` | Queue management operations |

### Content Field Format

The `message.content` field can be either:
- **String**: For simple text messages (e.g., `"content": "Hello"`)
- **Array**: For structured content with multiple blocks:
  - `{"type": "text", "text": "..."}` — text response
  - `{"type": "thinking", "thinking": "..."}` — reasoning block
  - `{"type": "tool_use", "id": "...", "name": "...", "input": {...}}` — tool call
  - `{"type": "tool_result", "tool_use_id": "...", "content": "..."}` — tool result

### Sub-agent Messages

Sub-agent messages include additional fields:

| Field | Type | Description |
|-------|------|-------------|
| `agentId` | string | Agent identifier |
| `slug` | string | Human-readable agent name |
| `isSidechain` | boolean | Always `true` for sub-agents |

## Retrieval Method

Use a two-source approach for retrieval. When using the VS Code extension, entries are not recorded in `history.jsonl`, so both sources must be read.

### Source 1: history.jsonl

```python
import json
from pathlib import Path

claude_dir = Path.home() / ".claude"
history_path = claude_dir / "history.jsonl"

messages = []
with open(history_path, "r", encoding="utf-8") as f:
    for line in f:
        entry = json.loads(line.strip())
        display = entry.get("display", "").strip()
        if display and not display.startswith("/clear"):
            messages.append({
                "text": display,
                "timestamp": entry.get("timestamp"),
                "project": entry.get("project", ""),
            })
```

### Source 2: Per-Project Session JSONL

```python
projects_dir = claude_dir / "projects"
for project_dir in projects_dir.iterdir():
    for session_file in project_dir.glob("*.jsonl"):
        with open(session_file, "r", encoding="utf-8") as f:
            for line in f:
                entry = json.loads(line.strip())
                if entry.get("type") != "user":
                    continue
                if entry.get("isMeta"):
                    continue
                content = entry.get("message", {}).get("content", "")
                # content can be a string or an array of content blocks
                if isinstance(content, str):
                    text = content
                elif isinstance(content, list):
                    text = " ".join(
                        p.get("text", "") for p in content
                        if isinstance(p, dict) and p.get("type") == "text"
                    )
```

### Content to Exclude

- System messages with `isMeta: true`
- Entries with `type` other than `"user"` / `"assistant"` (e.g., `progress`, `system`, `file-history-snapshot`)
- Text starting with system tags such as `<ide_opened_file>`, `<local-command-stdout>`, etc.
- Slash commands such as `/clear`, `/help`, etc.

## Path Encoding Rules

Project directory names are derived by converting `\`, `/`, and `:` in the original path to `-`:

```
C:\Users\user\project → c--Users-user-project
/home/user/project    → -home-user-project
```

## Notes

- Log storage locations differ between the CLI and the VS Code extension
- `history.jsonl` is only recorded when using the CLI
- Session JSONL files contain multiple entry types beyond user/assistant (progress, system, file-history-snapshot, etc.)
- Sub-agent logs are saved as separate files under `{session-uuid}/subagents/`
- The `message.content` field can be either a plain string or an array of content blocks
- Assistant messages include token usage metrics and model information
