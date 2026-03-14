# Tabnine

## Basic Information

| Item | Details |
|------|---------|
| Type | Code completion extension (VS Code / JetBrains, etc.) |
| Log Format | — |
| Local Logs | Limited |
| Remarks | Primarily completion-based, so there are almost no conversation logs |

## Storage Location

| OS | Path |
|----|------|
| Windows | `%APPDATA%\TabNine\` |
| macOS | `~/Library/Application Support/TabNine/` |
| Linux | `~/.config/TabNine/` |

## Data Format

Tabnine's primary function is code completion, and it generally does not generate chat-style conversation logs.

If the chat feature (Tabnine Chat) has been added, data may be stored in VS Code's `globalStorage`.

## Notes

- As a completion-focused tool, it is not well suited for prompt analysis
- Conversation logs may exist only if Tabnine Chat is available
- When using a local model (Tabnine Enterprise), the log path may differ
