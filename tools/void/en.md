# Void

## Basic Information

| Item | Details |
|------|---------|
| Type | AI-focused editor (VS Code fork, OSS) |
| Log Format | SQLite / JSON |
| Local Logs | Yes (estimated) |

## Storage Location

Uses a similar directory structure to VS Code since it is a VS Code fork (estimated):

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Void\User\` |
| macOS | `~/Library/Application Support/Void/User/` |
| Linux | `~/.config/Void/User/` |

## File Structure (Estimated)

```
Void/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## Data Format

As a VS Code based tool, it uses the `state.vscdb` structure. Void's proprietary AI chat data may be stored separately.

## Notes

- **Requires hands-on investigation**: The detailed log structure has not been confirmed
- As an OSS project, the storage structure can be verified from the source code
- The project is in its early development stage, so the storage structure may change frequently
