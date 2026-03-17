# Google Antigravity

## Basic Information

| Item | Details |
|------|---------|
| Type | AI IDE (VS Code fork) + CLI chat |
| Log Format | Protobuf (binary) + SQLite + Text |
| Local Logs | Yes |

## Storage Location

### Agent Data (Gemini)

| OS | Path |
|----|------|
| macOS | `~/.gemini/antigravity/` |
| Linux | `~/.gemini/antigravity/` |
| Windows | `%USERPROFILE%\.gemini\antigravity\` |

### IDE State (VS Code-based)

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/Antigravity/User/globalStorage/state.vscdb` |
| Linux | `~/.config/Antigravity/User/globalStorage/state.vscdb` |
| Windows | `%APPDATA%\Antigravity\User\globalStorage\state.vscdb` |

### Extensions & CLI Config

| OS | Path |
|----|------|
| macOS | `~/.antigravity/` |
| Linux | `~/.antigravity/` |
| Windows | `%USERPROFILE%\.antigravity\` |

## File Structure (Verified)

```
~/.gemini/
├── GEMINI.md                                     # Global memory (shared across projects)
├── projects.json                                 # Project registry
└── antigravity/
    ├── installation_id                           # UUID
    ├── user_settings.pb                          # User settings (protobuf binary)
    ├── mcp_config.json                           # MCP server configuration
    ├── browserAllowlist.txt                      # Browser tool allowlist
    ├── brain/                                    # Agent artifacts per conversation
    │   └── <cascade-id>/
    │       ├── <artifact-files>                  # User-visible artifacts (code, plans)
    │       └── .system_generated/
    │           └── logs/
    │               ├── overview.txt              # Conversation overview
    │               └── <section-name>.txt        # Log sections
    ├── knowledge/                                # Knowledge items (persistent memory)
    ├── context_state/                            # Cross-conversation context
    └── global_workflows/                         # Global workflow definitions

~/.config/Antigravity/                            # IDE state (Linux)
├── User/
│   ├── settings.json                             # Editor settings
│   ├── globalStorage/
│   │   ├── state.vscdb                           # SQLite: app state + trajectory summaries
│   │   └── storage.json                          # Migration state + preferences
│   └── workspaceStorage/
│       └── <workspace-id>/
│           └── state.vscdb                       # SQLite: per-workspace state
└── logs/                                         # Application logs

~/.antigravity/                                   # CLI & extensions
├── argv.json                                     # CLI arguments
└── extensions/
    └── extensions.json                           # Installed extensions

<project-root>/
├── .agent/
│   ├── rules/                                    # Project-scoped agent rules
│   └── workflows/                                # Project-scoped workflow definitions
└── .gemini/
    └── GEMINI.md                                 # Project-level agent instructions
```

## Data Format

### Conversation Storage Architecture

Conversation (trajectory) data uses a multi-layered architecture:

1. **Language Server (gRPC)** — The "Cascade" engine manages trajectory state in-memory via gRPC RPCs
2. **SQLite (state.vscdb)** — Trajectory summaries and app state persisted as base64-encoded protobuf
3. **File System (brain/)** — Agent artifacts and conversation logs as plain text/images

### state.vscdb (SQLite — Primary Persistent State)

Schema: VS Code standard `ItemTable` (key TEXT, value TEXT).

Key entries for conversation data:

| Key | Format | Description |
|-----|--------|-------------|
| `jetskiStateSync.agentManagerInitState` | Base64 protobuf | Master state: trajectory summaries, window position, OAuth token |
| `antigravityUnifiedStateSync.trajectorySummaries` | Base64 protobuf | Conversation list and metadata |
| `antigravityUnifiedStateSync.agentPreferences` | Base64 protobuf | Planning mode, file access policy, terminal execution |
| `antigravityUnifiedStateSync.editorPreferences` | Base64 protobuf | Editor integration settings |
| `chat.ChatSessionStore.index` | JSON | VS Code chat session index |

### Protobuf Schemas (Reconstructed from Source)

#### CascadeTrajectorySummary (Conversation Summary)

```
CascadeTrajectorySummary {
  string summary = 1;
  uint32 step_count = 2;
  Timestamp last_modified_time = 3;
  string trajectory_id = 4;
  CascadeRunStatus status = 5;       // IDLE, RUNNING, CANCELING, BUSY
  Timestamp created_time = 7;
  repeated WaitingStep waiting_steps = 8;
  repeated Workspace workspaces = 9;
  Timestamp last_user_input_time = 10;
  uint32 last_user_input_step_index = 11;
  ConversationAnnotations annotations = 14;
}
```

#### ChatMessage (Individual Message)

```
ChatMessage {
  string message_id = 1;
  MessageSource source = 2;          // USER, SYSTEM, etc.
  Timestamp timestamp = 3;
  string conversation_id = 4;
  oneof content {
    ChatMessageIntent intent = 5;    // User intent
    ChatMessageAction action = 6;    // AI action (edit, search, generic)
    ChatMessageError error = 7;
    ChatMessageStatus status = 8;
  }
  bool in_progress = 9;
  ChatMessageRequest request = 10;
  bool redact = 11;
}
```

#### BrainEntry (Brain File Entry)

```
BrainEntry {
  string id = 1;
  BrainEntryType type = 2;           // UNSPECIFIED, PLAN, TASK
  string content = 3;
}
```

### Brain Directory (Conversation Artifacts)

Located at `~/.gemini/antigravity/brain/<cascade-id>/`.

Each conversation ("cascade") gets its own directory containing:
- User-visible artifacts (generated code, plans)
- `.system_generated/logs/overview.txt` — Conversation overview
- `.system_generated/logs/<section-name>.txt` — Detailed log sections
- Screenshots/images (`.jpeg`, `.png`, `.gif`, `.webp`)

### projects.json (Project Registry)

```json
{
  "projects": {
    "/home/ubuntu": "ubuntu",
    "/home/ubuntu/ai-agent-log-formats": "ai-agent-log-formats"
  }
}
```

### gRPC API (Language Server)

Key RPCs for conversation data:

| RPC | Description |
|-----|-------------|
| `GetCascadeTrajectory` | Retrieve full conversation trajectory |
| `GetCascadeTrajectorySteps` | Retrieve steps with pagination |
| `GetAllCascadeTrajectories` | List all trajectory summaries |
| `ConvertTrajectoryToMarkdown` | Export conversation as Markdown |
| `DeleteCascadeTrajectory` | Delete a conversation |

## CLI Usage

```bash
# Open chat session (requires GUI display)
antigravity chat "your prompt"
antigravity chat -m ask "question"      # Ask mode
antigravity chat -m edit "instruction"  # Edit mode
antigravity chat -m agent "task"        # Agent mode (default)

# Add file context
antigravity chat -a file.py "explain this"
```

## Retrieval Method

```python
import json
import sqlite3
from pathlib import Path
import platform
import base64

# Brain directory logs (text, human-readable)
brain_dir = Path.home() / ".gemini" / "antigravity" / "brain"
if brain_dir.exists():
    for cascade_dir in brain_dir.iterdir():
        if not cascade_dir.is_dir():
            continue
        logs_dir = cascade_dir / ".system_generated" / "logs"
        if not logs_dir.exists():
            continue
        # Overview
        overview = logs_dir / "overview.txt"
        if overview.exists():
            print(f"[{cascade_dir.name[:12]}] {overview.read_text()[:200]}")
        # Section logs
        for log_file in sorted(logs_dir.glob("*.txt")):
            if log_file.name == "overview.txt":
                continue
            text = log_file.read_text(encoding="utf-8").strip()
            if text:
                print(f"  [{log_file.stem}] {text[:200]}")

# state.vscdb trajectory summaries (protobuf, requires decoding)
system = platform.system()
if system == "Darwin":
    vscdb = Path.home() / "Library" / "Application Support" / "Antigravity" / "User" / "globalStorage" / "state.vscdb"
elif system == "Windows":
    import os
    vscdb = Path(os.environ["APPDATA"]) / "Antigravity" / "User" / "globalStorage" / "state.vscdb"
else:
    vscdb = Path.home() / ".config" / "Antigravity" / "User" / "globalStorage" / "state.vscdb"

if vscdb.exists():
    conn = sqlite3.connect(str(vscdb))
    cursor = conn.cursor()
    # Chat session index (JSON)
    cursor.execute("SELECT value FROM ItemTable WHERE key = 'chat.ChatSessionStore.index'")
    row = cursor.fetchone()
    if row:
        data = json.loads(row[0])
        print(f"Chat sessions: {len(data.get('entries', {}))}")
    # Trajectory state (base64 protobuf — requires protobuf library to decode)
    cursor.execute("SELECT value FROM ItemTable WHERE key = 'jetskiStateSync.agentManagerInitState'")
    row = cursor.fetchone()
    if row:
        raw = base64.b64decode(row[0])
        print(f"Agent manager state: {len(raw)} bytes (protobuf)")
    conn.close()
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `ANTIGRAVITY_USER_DATA_DIR` | Override user data directory |
| `GOOGLE_API_KEY` | API key for Gemini models |

## Notes

- Internal codename is "Jetski" (visible in source code and state keys like `jetskiStateSync`)
- Built on Codeium/Windsurf technology (protobuf schemas use `exa.*` namespace)
- Conversation trajectories are primarily managed **in-memory by the language server** and retrieved via gRPC
- Persistent storage in `state.vscdb` uses **base64-encoded protobuf binary** — not human-readable JSON
- The `brain/` directory contains the only human-readable conversation data (text logs and artifacts)
- Knowledge items are extracted by a "Knowledge Subagent" at conversation end
- Known bug: `brain/` directories may remain empty and knowledge items may not generate properly
- `.pb` files are Protocol Buffers binary format — cannot be read as plain text
- Chat session index uses standard VS Code `chat.ChatSessionStore.index` key pattern
- Version 1.104.0 confirmed (1.13.3 user-facing version)
- Requires Google Account for authentication (OAuth flow)
