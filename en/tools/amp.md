# amp (Sourcegraph)

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI agent |
| Log Format | JSONL |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.amp/` |
| Linux | `~/.amp/` |

If the environment variable `AMP_HOME` is set, that path is used (estimated).

## File Structure (Estimated)

```
~/.amp/
├── config.yaml                    # Configuration
├── conversations/
│   └── {session-id}.jsonl         # Conversation logs
└── threads/
    └── ...
```

## Data Format

Conversations are logged in JSONL format (estimated). The structure is based on Sourcegraph's agent framework.

## Retrieval Method

```python
from pathlib import Path

amp_dir = Path.home() / ".amp"
if amp_dir.exists():
    # Search for conversation files
    for f in amp_dir.rglob("*.jsonl"):
        print(f"Found: {f}")
    for f in amp_dir.rglob("*.json"):
        print(f"Found: {f}")
```

## Notes

- **Requires hands-on investigation**: amp is a relatively new tool, and the detailed log structure has not been verified
- When connected to a Sourcegraph server, conversation history may be stored on the server side
