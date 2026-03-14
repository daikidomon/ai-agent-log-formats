# PearAI

## Basic Information

| Item | Details |
|------|---------|
| Type | AI-focused editor (VS Code fork) |
| Log Format | SQLite / JSON |
| Local Logs | Yes (estimated) |

## Storage Location

Uses a similar directory structure to VS Code since it is a VS Code fork (estimated):

| OS | Path |
|----|------|
| Windows | `%APPDATA%\PearAI\User\` |
| macOS | `~/Library/Application Support/PearAI/User/` |
| Linux | `~/.config/PearAI/User/` |

## File Structure (Estimated)

```
PearAI/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## Data Format

As a VS Code based tool, it uses the `state.vscdb` structure. PearAI's proprietary AI chat data may be stored in `globalStorage` or a dedicated directory.

## Notes

- **Requires hands-on investigation**: The detailed log structure has not been confirmed
- PearAI has Continue built in, so data may also exist in `~/.continue/`
