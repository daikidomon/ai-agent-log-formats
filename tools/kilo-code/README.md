# Kilo Code

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code extension |
| Log Format | JSON (Cline-based structure) |
| Local Logs | Yes |
| Note | Fork of Cline |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\kilocode.kilo-code\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/kilocode.kilo-code/` |
| Linux | `~/.config/Code/User/globalStorage/kilocode.kilo-code/` |

## File Structure

Same structure as Cline:

```
kilocode.kilo-code/
├── state/
│   └── taskHistory.json
└── tasks/
    └── {task-id}/
        ├── api_conversation_history.json
        ├── ui_messages.json
        └── task_metadata.json
```

## Data Format

Same as [Cline](cline.md). Extract messages with `role: "user"` from `api_conversation_history.json`.

## Retrieval Method

Same procedure as Cline. Replace `saoudrizwan.claude-dev` with `kilocode.kilo-code` in the path.

```python
base / "Code" / "User" / "globalStorage" / "kilocode.kilo-code" / "tasks"
```

## Notes

- The extension ID may change (verify as needed)
- Kilo Code's own customizations may be reflected in the metadata
