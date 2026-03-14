# ai-agent-log-formats

A reference guide for conversation history storage formats and retrieval methods of AI coding agents/assistants.

**[日本語版 / Japanese](README.ja.md)**

## Tool List

| Category | Tool | Log Format | Local Log | Details |
|----------|------|-----------|:---------:|---------|
| **CLI** | [Claude Code](en/tools/claude-code.md) | JSONL | Yes | history.jsonl + per-project sessions |
| **CLI** | [OpenAI Codex CLI](en/tools/openai-codex.md) | JSONL | Yes | Rollout session files |
| **CLI** | [Aider](en/tools/aider.md) | Markdown / SQLite | Yes | Chat history + DB |
| **CLI** | [amp](en/tools/amp.md) | JSONL | Yes | Session files |
| **Editor** | [Cursor](en/tools/cursor.md) | SQLite | Yes | state.vscdb + custom storage |
| **Editor** | [Trae](en/tools/trae.md) | SQLite / JSON | Yes | VS Code-based structure |
| **Editor** | [Zed AI](en/tools/zed-ai.md) | JSON | Yes | Conversations directory |
| **Editor** | [PearAI](en/tools/pearai.md) | SQLite / JSON | Yes | VS Code-based structure |
| **Editor** | [Void](en/tools/void.md) | SQLite / JSON | Yes | VS Code-based structure |
| **VS Code Ext** | [GitHub Copilot Chat](en/tools/github-copilot-chat.md) | SQLite | Yes | state.vscdb |
| **VS Code Ext** | [Cline](en/tools/cline.md) | JSON | Yes | api_conversation_history.json |
| **VS Code Ext** | [Roo Code](en/tools/roo-code.md) | JSON | Yes | Same structure as Cline |
| **VS Code Ext** | [Kilo Code](en/tools/kilo-code.md) | JSON | Yes | Cline fork |
| **VS Code Ext** | [Continue](en/tools/continue.md) | JSON | Yes | Session JSON |
| **VS Code Ext** | [Sourcegraph Cody](en/tools/sourcegraph-cody.md) | JSON | Yes | In globalStorage |
| **VS Code Ext** | [Amazon Q Developer](en/tools/amazon-q-developer.md) | JSON | Yes | In globalStorage |
| **VS Code Ext** | [Augment Code](en/tools/augment-code.md) | Unknown | TBD | — |
| **IDE Built-in** | [JetBrains AI Assistant](en/tools/jetbrains-ai.md) | XML / JSON | Yes | IDE config directory |
| **IDE Built-in** | [Gemini Code Assist](en/tools/gemini-code-assist.md) | Unknown | TBD | — |
| **IDE/Editor** | [Windsurf (Cascade)](en/tools/windsurf.md) | Text | Yes | Auto-summary memory |
| **IDE/Editor** | [OpenCode](en/tools/opencode.md) | SQLite | Yes | opencode.db |
| **IDE/Editor** | [Google Antigravity](en/tools/google-antigravity.md) | Text | Yes | Log files |
| **Autonomous** | [SWE-agent](en/tools/swe-agent.md) | JSON | Yes | Trajectory files |
| **Autonomous** | [OpenHands](en/tools/openhands.md) | JSON | Yes | Session logs |
| **Cloud** | [Devin](en/tools/devin.md) | — | No | Cloud only |
| **Cloud** | [Replit Agent](en/tools/replit-agent.md) | — | No | Cloud only |
| **Cloud** | [Bolt](en/tools/bolt.md) | — | No | Browser-based |
| **Cloud** | [Lovable](en/tools/lovable.md) | — | No | Browser-based |
| **Cloud** | [v0](en/tools/v0.md) | — | No | Browser-based |
| **Completion** | [Tabnine](en/tools/tabnine.md) | — | Limited | No chat logs (completion-focused) |
| **Completion** | [Supermaven](en/tools/supermaven.md) | — | Limited | No chat logs (completion-focused) |

### Legend

- **Local Log**: Whether prompt history is stored on the user's machine
- **TBD**: Log format unconfirmed; contributions welcome

## Directory Structure

```
ai-agent-log-formats/
├── README.md          # English
├── README.ja.md       # Japanese
├── en/tools/          # English tool docs
└── ja/tools/          # Japanese tool docs
```

## Notes

- Log formats may change with tool version updates
- Paths vary by OS (Windows / macOS / Linux)
- Some tools are marked as unconfirmed (TBD) and based on estimates
- Cloud-based tools do not store logs locally; API access may be required
