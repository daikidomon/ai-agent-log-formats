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
└── projects/
    └── {encoded-path}/                    # Path separators (\, /, :) converted to -
        ├── {session-uuid}.jsonl           # Full conversation log per session
        └── {session-uuid}/
            └── subagents/                 # Sub-agent logs
```

## Data Format

### history.jsonl (Index for CLI Usage)

One JSON object per line:

```json
{
  "display": "User input text",
  "pastedContents": {},
  "timestamp": 1759115795246,
  "project": "D:\\path\\to\\project",
  "sessionId": "uuid-string"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `display` | string | The prompt entered by the user |
| `timestamp` | number | Unix epoch in milliseconds |
| `project` | string | Absolute path of the project |
| `sessionId` | string | Session UUID |
| `pastedContents` | object | Pasted content |

### Per-Project Session JSONL (Including VS Code Extension)

One JSON object per line. User messages have `type: "user"`:

```json
{
  "type": "user",
  "message": {
    "role": "user",
    "content": [
      {"type": "text", "text": "Prompt body"}
    ]
  },
  "timestamp": "2026-03-12T05:31:13.875Z",
  "cwd": "/path/to/project"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `"user"` / `"assistant"` etc. |
| `message.content` | array | Array of content blocks |
| `timestamp` | string | ISO 8601 |
| `isMeta` | boolean | If `true`, this is a system message (should be skipped) |

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
                content = entry.get("message", {}).get("content", [])
                # Extract text portions from content
```

### Content to Exclude

- System messages with `isMeta: true`
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
- Session JSONL files also contain assistant responses (`type: "assistant"`)
- Sub-agent logs are saved as separate files under `{session-uuid}/subagents/`
