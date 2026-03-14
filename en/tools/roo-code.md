# Roo Code

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code Extension |
| Log Format | JSON (same structure as Cline) |
| Local Logs | Yes |
| Note | Fork of Cline |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\RooVeterinaryInc.roo-cline\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/RooVeterinaryInc.roo-cline/` |
| Linux | `~/.config/Code/User/globalStorage/RooVeterinaryInc.roo-cline/` |

## File Structure

Same as Cline:

```
RooVeterinaryInc.roo-cline/
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

Same procedure as Cline. Simply replace `saoudrizwan.claude-dev` with `RooVeterinaryInc.roo-cline` in the path.

```python
# The only difference from Cline is the base path
base / "Code" / "User" / "globalStorage" / "RooVeterinaryInc.roo-cline" / "tasks"
```

## Notes

- Additional metadata may be present due to Roo Code-specific features (e.g., mode switching)
- The basic conversation log structure is compatible with Cline
