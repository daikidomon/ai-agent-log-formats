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
├── config.toml                                    # Configuration file
├── state-v5.db                                    # Thread metadata (SQLite)
├── session_index.jsonl                            # Session index
└── sessions/
    └── YYYY/MM/DD/
        └── rollout-YYYY-MM-DDThh-mm-ss-<id>.jsonl # Conversation log per session
```

## Data Format

Each line is a JSON object. The type of each line is determined by the top-level key.

### SessionMeta (Session Start Metadata)

```json
{
  "timestamp": "2025-06-15T10:30:00.123Z",
  "SessionMeta": {
    "cwd": "/path/to/project",
    "model_provider": "openai"
  }
}
```

### ResponseItem (Conversation Item)

```json
{
  "timestamp": "2025-06-15T10:30:01.000Z",
  "ResponseItem": {
    "type": "message",
    "role": "user",
    "content": [
      {"type": "input_text", "text": "User's input"}
    ]
  }
}
```

| Key | Description |
|-----|-------------|
| `SessionMeta.cwd` | Project directory |
| `ResponseItem.type` | `"message"` = conversation message |
| `ResponseItem.role` | `"user"` / `"assistant"` |
| `ResponseItem.content[].type` | `"input_text"` or `"text"` |

## Retrieval Method

```python
import json
from pathlib import Path

codex_home = Path.home() / ".codex"
sessions_dir = codex_home / "sessions"

for rollout_path in sorted(sessions_dir.rglob("rollout-*.jsonl"),
                           key=lambda p: p.stat().st_mtime, reverse=True)[:50]:
    cwd = ""
    with open(rollout_path, "r", encoding="utf-8") as f:
        for line in f:
            entry = json.loads(line.strip())

            # Get project information from SessionMeta
            session_meta = entry.get("SessionMeta")
            if session_meta:
                cwd = session_meta.get("cwd", "")
                continue

            # Extract user messages from ResponseItem
            response_item = entry.get("ResponseItem")
            if not response_item:
                continue
            if response_item.get("type") != "message" or response_item.get("role") != "user":
                continue

            content = response_item.get("content", [])
            texts = []
            for part in content:
                if isinstance(part, dict) and part.get("type") in ("input_text", "text"):
                    texts.append(part.get("text", ""))

            text = " ".join(texts).strip()
            if text:
                print(f"[{cwd}] {text}")
```

## Notes

- Sessions are stored globally (not in per-project directories)
- Project information is obtained from the `cwd` field in `SessionMeta`
- Codex is built in Rust, so JSONL key names use CamelCase (`SessionMeta`, `ResponseItem`)
- `state-v5.db` also contains thread metadata, but conversation content is stored in rollout JSONL files
