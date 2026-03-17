# GitHub Copilot CLI

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI coding agent |
| Log Format | JSONL + YAML |
| Local Logs | Yes |

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.copilot/` |
| Linux | `~/.copilot/` |
| Windows | `%USERPROFILE%\.copilot\` |

## File Structure (Verified)

```
~/.copilot/
├── config.json                              # Core settings
├── logs/                                    # Debug logs
│   ├── copilot.log                         # Main log (auth, startup, etc.)
│   └── process-<timestamp>-<pid>.log       # Per-process debug logs
└── session-state/
    └── <session-uuid>/                     # One directory per session
        ├── events.jsonl                    # Full session event log
        ├── workspace.yaml                  # Workspace metadata
        ├── checkpoints/
        │   └── index.md                   # Checkpoint history (markdown table)
        ├── files/                          # (created on tool use)
        └── research/                       # (created on tool use)
```

## Data Format (Verified)

### config.json

Global configuration file. Contains first-launch timestamp:

```json
{
  "firstLaunchAt": "2026-03-17T09:01:20.162Z"
}
```

### workspace.yaml

Per-session workspace metadata in YAML format:

```yaml
id: 45a7d084-a838-4dda-9aca-9c268740bb35
cwd: /home/ubuntu/project
git_root: /home/ubuntu/project
repository: user/project
host_type: github
branch: main
summary_count: 0
created_at: 2026-03-17T09:01:20.579Z
updated_at: 2026-03-17T09:01:25.754Z
summary: say hello
```

| Field | Description |
|-------|-------------|
| `id` | Session UUID (matches directory name) |
| `cwd` | Working directory at session start |
| `git_root` | Git repository root |
| `repository` | `owner/repo` format |
| `host_type` | `github` |
| `branch` | Active branch |
| `summary_count` | Number of conversation summaries |
| `created_at` / `updated_at` | ISO 8601 timestamps |
| `summary` | First user message (used as session title) |

### events.jsonl

The main session log. Each line is a JSON object with common fields:

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | Event type (see below) |
| `data` | object | Event-specific payload |
| `id` | UUID | Unique event identifier |
| `timestamp` | string | ISO 8601 timestamp |
| `parentId` | UUID / null | Previous event in the chain |

### Event Types

#### `session.start`

First event in every session.

```json
{
  "type": "session.start",
  "data": {
    "sessionId": "45a7d084-a838-4dda-9aca-9c268740bb35",
    "version": 1,
    "producer": "copilot-agent",
    "copilotVersion": "1.0.6",
    "startTime": "2026-03-17T09:01:20.576Z",
    "context": {
      "cwd": "/home/ubuntu/project",
      "gitRoot": "/home/ubuntu/project",
      "branch": "main",
      "headCommit": "e77111f...",
      "repository": "user/project",
      "hostType": "github"
    },
    "alreadyInUse": false
  }
}
```

#### `session.model_change`

Emitted when the model is selected or changed.

```json
{
  "type": "session.model_change",
  "data": {
    "newModel": "gpt-5-mini"
  }
}
```

#### `user.message`

User input. Includes original content and system-augmented `transformedContent`.

```json
{
  "type": "user.message",
  "data": {
    "content": "say hello",
    "transformedContent": "<current_datetime>...</current_datetime>\n\nsay hello\n\n<reminder>...</reminder>",
    "attachments": [],
    "interactionId": "9d39d375-eda4-4709-98d3-10d0fb3574fe"
  }
}
```

#### `assistant.turn_start` / `assistant.turn_end`

Marks the beginning and end of an assistant turn. Multiple turns may occur in a single interaction (e.g., tool call followed by response).

```json
{
  "type": "assistant.turn_start",
  "data": {
    "turnId": "0",
    "interactionId": "9d39d375-..."
  }
}
```

#### `assistant.message`

Assistant response. May contain tool requests.

```json
{
  "type": "assistant.message",
  "data": {
    "messageId": "d4937b1b-...",
    "content": "Here are the files...",
    "toolRequests": [
      {
        "toolCallId": "call_XaRk2O8Qf4...",
        "name": "bash",
        "arguments": {
          "command": "ls -la --color=never",
          "description": "List files in the current directory",
          "initial_wait": 10
        },
        "type": "function",
        "intentionSummary": "List files in the current directory"
      }
    ],
    "interactionId": "db3779dc-...",
    "reasoningOpaque": "...",
    "encryptedContent": "...",
    "outputTokens": 593
  }
}
```

| Field | Description |
|-------|-------------|
| `content` | Plain text response |
| `toolRequests` | Array of tool calls (empty if no tools used) |
| `reasoningOpaque` | Encrypted/opaque reasoning trace (base64) |
| `encryptedContent` | Encrypted full response (base64) |
| `outputTokens` | Token count for this message |

#### `tool.execution_start`

Emitted when a tool begins execution.

```json
{
  "type": "tool.execution_start",
  "data": {
    "toolCallId": "call_XaRk2O8Qf4...",
    "toolName": "bash",
    "arguments": {
      "command": "ls -la --color=never",
      "description": "List files in the current directory",
      "initial_wait": 10
    }
  }
}
```

#### `tool.execution_complete`

Emitted when a tool finishes. Contains both `content` (short) and `detailedContent` (full output).

```json
{
  "type": "tool.execution_complete",
  "data": {
    "toolCallId": "call_XaRk2O8Qf4...",
    "model": "gpt-5-mini",
    "interactionId": "db3779dc-...",
    "success": true,
    "result": {
      "content": "total 36\ndrwxrwxr-x 5 ubuntu ...",
      "detailedContent": "total 36\ndrwxrwxr-x 5 ubuntu ..."
    },
    "toolTelemetry": {
      "properties": {
        "customTimeout": "true",
        "executionMode": "sync",
        "detached": "false"
      },
      "metrics": {
        "commandTimeout": 30000
      }
    }
  }
}
```

#### `session.shutdown`

Final event. Contains usage metrics and session statistics.

```json
{
  "type": "session.shutdown",
  "data": {
    "shutdownType": "routine",
    "totalPremiumRequests": 1,
    "totalApiDurationMs": 21883,
    "sessionStartTime": 1773738158854,
    "codeChanges": {
      "linesAdded": 0,
      "linesRemoved": 0,
      "filesModified": []
    },
    "modelMetrics": {
      "gpt-5-mini": {
        "requests": { "count": 2, "cost": 1 },
        "usage": {
          "inputTokens": 27911,
          "outputTokens": 629,
          "cacheReadTokens": 26880,
          "cacheWriteTokens": 0
        }
      }
    },
    "currentModel": "gpt-5-mini"
  }
}
```

### Other Event Types

| Type | Description |
|------|-------------|
| `session.tools_updated` | Tool list changed during session |
| `session.background_tasks_changed` | Background task state changed |
| `session.info` | Informational session messages |
| `result` | Final result with `sessionId`, `exitCode`, `usage` summary |

### checkpoints/index.md

Markdown table tracking checkpoints (git snapshots) created during the session:

```markdown
# Checkpoint History

| # | Title | File |
|---|-------|------|
| 1 | Added feature X | checkpoint-1.patch |
```

## Event Chain

Events are linked via `parentId`, forming a chain:

```
session.start (parentId: null)
  └─ session.model_change
       └─ user.message
            └─ assistant.turn_start
                 └─ assistant.message (with toolRequests)
                      └─ tool.execution_start
                           └─ tool.execution_complete
                                └─ assistant.turn_end
                                     └─ assistant.turn_start (turn 2)
                                          └─ assistant.message (final response)
                                               └─ assistant.turn_end
                                                    └─ session.shutdown
```

## Authentication

Copilot CLI authenticates via GitHub OAuth device flow. Token storage requires a system keychain; alternatively, set `GH_TOKEN` environment variable from an existing `gh auth token`.

## CLI Usage

```bash
# Interactive session
copilot

# Non-interactive with prompt
copilot -p "your prompt"

# With JSON output
copilot -p "your prompt" --output-format json -s

# With auto-approve tools
copilot -p "your prompt" --allow-all
```

## Notes

- Copilot CLI is a standalone binary (`~/.local/bin/copilot`), separate from the VS Code GitHub Copilot extension
- Session IDs are UUIDs (v4)
- `assistant.message_delta` events may appear during streaming (ephemeral, not always persisted)
- The `reasoningOpaque` and `encryptedContent` fields contain encrypted data not decodable without the session key
- `report_intent` is a special internal tool used to log the assistant's intent before executing actions
- Verified with Copilot CLI v1.0.6 on Linux
