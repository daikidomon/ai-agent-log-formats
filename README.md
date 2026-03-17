# ai-agent-log-formats

A reference guide for conversation history storage formats and retrieval methods of AI coding agents/assistants.

**[日本語版 / Japanese](README.ja.md)**

## Tool List

| Category | Tool | Log Format | Local Log | Verified | Details |
|----------|------|-----------|:---------:|:--------:|---------|
| **CLI** | [Claude Code](tools/claude-code/) | JSONL | Yes | :white_check_mark: | history.jsonl + per-project sessions |
| **CLI** | [OpenAI Codex CLI](tools/openai-codex/) | JSONL | Yes | :white_check_mark: | Rollout session files |
| **CLI** | [Aider](tools/aider/) | Markdown / SQLite | Yes | | Chat history + DB |
| **CLI** | [amp](tools/amp/) | JSONL | Yes | | Session files |
| **CLI** | [Gemini CLI](tools/gemini-cli/) | JSON | Yes | :white_check_mark: | Project-specific sessions + named checkpoints |
| **CLI** | [Grok CLI](tools/grok-cli/) | — | No | :white_check_mark: | Claude Code proxy wrapper; logs stored in `~/.claude/` |
| **CLI** | [Ollama](tools/ollama/) | — | Limited | :white_check_mark: | Command history only; no conversation persistence |
| **CLI** | [GitHub Copilot CLI](tools/github-copilot-cli/) | JSONL + YAML | Yes | :white_check_mark: | events.jsonl + workspace.yaml per session |
| **Editor** | [Cursor](tools/cursor/) | JSONL + SQLite | Yes | :white_check_mark: | CLI: `~/.cursor/` transcripts + store.db; GUI: state.vscdb |
| **Editor** | [Trae](tools/trae/) | SQLite / JSON | Yes | | VS Code-based structure |
| **Editor** | [Zed AI](tools/zed-ai/) | JSON | Yes | | Conversations directory |
| **Editor** | [PearAI](tools/pearai/) | SQLite / JSON | Yes | | VS Code-based structure |
| **Editor** | [Void](tools/void/) | SQLite / JSON | Yes | | VS Code-based structure |
| **VS Code Ext** | [GitHub Copilot Chat](tools/github-copilot-chat/) | SQLite | Yes | | state.vscdb |
| **VS Code Ext** | [Cline](tools/cline/) | JSON | Yes | | api_conversation_history.json |
| **VS Code Ext** | [Roo Code](tools/roo-code/) | JSON | Yes | | Same structure as Cline |
| **VS Code Ext** | [Kilo Code](tools/kilo-code/) | JSON | Yes | | Cline fork |
| **VS Code Ext** | [Continue](tools/continue/) | JSON | Yes | | Session JSON |
| **VS Code Ext** | [Sourcegraph Cody](tools/sourcegraph-cody/) | JSON | Yes | | In globalStorage |
| **VS Code Ext** | [Amazon Q Developer](tools/amazon-q-developer/) | JSON | Yes | | In globalStorage |
| **VS Code Ext** | [Augment Code](tools/augment-code/) | Unknown | TBD | | — |
| **IDE Built-in** | [JetBrains AI Assistant](tools/jetbrains-ai/) | XML / JSON | Yes | | IDE config directory |
| **IDE Built-in** | [Gemini Code Assist](tools/gemini-code-assist/) | Unknown | TBD | | — |
| **IDE/Editor** | [Windsurf (Cascade)](tools/windsurf/) | Text | Yes | | Auto-summary memory |
| **IDE/Editor** | [OpenCode](tools/opencode/) | SQLite | Yes | :white_check_mark: | opencode.db |
| **IDE/Editor** | [Google Antigravity](tools/google-antigravity/) | Protobuf + SQLite + Text | Yes | :white_check_mark: | brain/ logs + state.vscdb |
| **Autonomous** | [SWE-agent](tools/swe-agent/) | JSON | Yes | | Trajectory files |
| **Autonomous** | [OpenHands](tools/openhands/) | JSON | Yes | | Session logs |
| **Cloud** | [Devin](tools/devin/) | — | No | | Cloud only |
| **Cloud** | [Replit Agent](tools/replit-agent/) | — | No | | Cloud only |
| **Cloud** | [Bolt](tools/bolt/) | — | No | | Browser-based |
| **Cloud** | [Lovable](tools/lovable/) | — | No | | Browser-based |
| **Cloud** | [v0](tools/v0/) | — | No | | Browser-based |
| **Completion** | [Tabnine](tools/tabnine/) | — | Limited | | No chat logs (completion-focused) |
| **Completion** | [Supermaven](tools/supermaven/) | — | Limited | | No chat logs (completion-focused) |

### Legend

- **Local Log**: Whether prompt history is stored on the user's machine
- **Verified**: :white_check_mark: = Installed and verified against actual data on a real machine
- **TBD**: Log format unconfirmed; contributions welcome

## Directory Structure

```
ai-agent-log-formats/
├── README.md              # English
├── README.ja.md           # Japanese
└── tools/
    ├── claude-code/
    │   ├── README.md      # English (auto-displayed on GitHub)
    │   └── README.ja.md   # Japanese
    ├── cursor/
    │   ├── README.md
    │   └── README.ja.md
    └── ...
```

## Notes

- Log formats may change with tool version updates
- Paths vary by OS (Windows / macOS / Linux)
- Some tools are marked as unconfirmed (TBD) and based on estimates
- Cloud-based tools do not store logs locally; API access may be required
