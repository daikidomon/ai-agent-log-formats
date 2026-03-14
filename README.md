# ai-agent-log-formats

A reference guide for conversation history storage formats and retrieval methods of AI coding agents/assistants.

**[日本語版 / Japanese](README.ja.md)**

## Tool List

| Category | Tool | Log Format | Local Log | Details |
|----------|------|-----------|:---------:|---------|
| **CLI** | [Claude Code](tools/claude-code/en.md) | JSONL | Yes | history.jsonl + per-project sessions |
| **CLI** | [OpenAI Codex CLI](tools/openai-codex/en.md) | JSONL | Yes | Rollout session files |
| **CLI** | [Aider](tools/aider/en.md) | Markdown / SQLite | Yes | Chat history + DB |
| **CLI** | [amp](tools/amp/en.md) | JSONL | Yes | Session files |
| **Editor** | [Cursor](tools/cursor/en.md) | SQLite | Yes | state.vscdb + custom storage |
| **Editor** | [Trae](tools/trae/en.md) | SQLite / JSON | Yes | VS Code-based structure |
| **Editor** | [Zed AI](tools/zed-ai/en.md) | JSON | Yes | Conversations directory |
| **Editor** | [PearAI](tools/pearai/en.md) | SQLite / JSON | Yes | VS Code-based structure |
| **Editor** | [Void](tools/void/en.md) | SQLite / JSON | Yes | VS Code-based structure |
| **VS Code Ext** | [GitHub Copilot Chat](tools/github-copilot-chat/en.md) | SQLite | Yes | state.vscdb |
| **VS Code Ext** | [Cline](tools/cline/en.md) | JSON | Yes | api_conversation_history.json |
| **VS Code Ext** | [Roo Code](tools/roo-code/en.md) | JSON | Yes | Same structure as Cline |
| **VS Code Ext** | [Kilo Code](tools/kilo-code/en.md) | JSON | Yes | Cline fork |
| **VS Code Ext** | [Continue](tools/continue/en.md) | JSON | Yes | Session JSON |
| **VS Code Ext** | [Sourcegraph Cody](tools/sourcegraph-cody/en.md) | JSON | Yes | In globalStorage |
| **VS Code Ext** | [Amazon Q Developer](tools/amazon-q-developer/en.md) | JSON | Yes | In globalStorage |
| **VS Code Ext** | [Augment Code](tools/augment-code/en.md) | Unknown | TBD | — |
| **IDE Built-in** | [JetBrains AI Assistant](tools/jetbrains-ai/en.md) | XML / JSON | Yes | IDE config directory |
| **IDE Built-in** | [Gemini Code Assist](tools/gemini-code-assist/en.md) | Unknown | TBD | — |
| **IDE/Editor** | [Windsurf (Cascade)](tools/windsurf/en.md) | Text | Yes | Auto-summary memory |
| **IDE/Editor** | [OpenCode](tools/opencode/en.md) | SQLite | Yes | opencode.db |
| **IDE/Editor** | [Google Antigravity](tools/google-antigravity/en.md) | Text | Yes | Log files |
| **Autonomous** | [SWE-agent](tools/swe-agent/en.md) | JSON | Yes | Trajectory files |
| **Autonomous** | [OpenHands](tools/openhands/en.md) | JSON | Yes | Session logs |
| **Cloud** | [Devin](tools/devin/en.md) | — | No | Cloud only |
| **Cloud** | [Replit Agent](tools/replit-agent/en.md) | — | No | Cloud only |
| **Cloud** | [Bolt](tools/bolt/en.md) | — | No | Browser-based |
| **Cloud** | [Lovable](tools/lovable/en.md) | — | No | Browser-based |
| **Cloud** | [v0](tools/v0/en.md) | — | No | Browser-based |
| **Completion** | [Tabnine](tools/tabnine/en.md) | — | Limited | No chat logs (completion-focused) |
| **Completion** | [Supermaven](tools/supermaven/en.md) | — | Limited | No chat logs (completion-focused) |

### Legend

- **Local Log**: Whether prompt history is stored on the user's machine
- **TBD**: Log format unconfirmed; contributions welcome

## Directory Structure

```
ai-agent-log-formats/
├── README.md              # English
├── README.ja.md           # Japanese
└── tools/
    ├── claude-code/
    │   ├── en.md          # English
    │   └── ja.md          # Japanese
    ├── cursor/
    │   ├── en.md
    │   └── ja.md
    └── ...
```

## Notes

- Log formats may change with tool version updates
- Paths vary by OS (Windows / macOS / Linux)
- Some tools are marked as unconfirmed (TBD) and based on estimates
- Cloud-based tools do not store logs locally; API access may be required
