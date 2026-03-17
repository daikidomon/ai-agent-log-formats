# Ollama CLI

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI (local LLM runner) |
| Log Format | Plain text (command history only) |
| Local Logs | Limited |
| Repository | [ollama/ollama](https://github.com/ollama/ollama) |

## Overview

Ollama is an open-source tool for running LLMs locally. The CLI provides an interactive chat mode, but **does not automatically persist conversation history** to disk. Only command input history (for arrow-key recall) is saved. Full conversation logs require third-party clients or manual shell redirection.

## Storage Location

| OS | Path |
|----|------|
| macOS | `~/.ollama/` |
| Linux | `~/.ollama/` (user) or `/usr/share/ollama/.ollama/` (system service) |
| Windows | `C:\Users\<username>\.ollama\` |

## File Structure

### Client Side (current user)

```
~/.ollama/
├── history                    # Command input history (readline, not conversation logs)
```

### Server Side (ollama user or current user)

```
~/.ollama/                     # or /usr/share/ollama/.ollama/ (Linux systemd service)
├── id_ed25519                 # Private key
├── id_ed25519.pub             # Public key
├── server.json                # Server configuration
└── models/
    ├── manifests/             # Model manifests
    └── blobs/                 # Model binary data (sha256-*)
```

On Linux with systemd, the server runs as the `ollama` user with home at `/usr/share/ollama/`. The client-side `~/.ollama/history` is stored under the invoking user's home directory.

## What Is and Is NOT Saved

### Saved: Command History (`~/.ollama/history`)

A readline history file for CLI input recall (up/down arrow navigation). This stores only **user input text**, not model responses.

- Default limit: 100 entries (configurable)
- Autosave enabled by default
- Can be disabled with `/set nohistory`

### NOT Saved: Conversation History

Full conversation content (user prompts + model responses) is **not automatically saved** to disk. Conversation context exists only in memory during the active session.

## Session Management Commands

| Command | Description |
|---------|-------------|
| `/save <name>` | Save current session as a new model (conversation + system message) |
| `/load <model>` | Load a different model |
| `/clear` | Clear conversation context |
| `/set history` | Enable command history |
| `/set nohistory` | Disable command history |
| `/show info` | Display model details |
| `/show system` | Display system message |

### /save Command Behavior

`/save <name>` creates a **new model** that inherits from the current model and embeds the conversation history and system message as a session snapshot. This is not a log file — it's a model artifact stored in Ollama's model storage.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OLLAMA_MODELS` | Custom model storage directory |
| `OLLAMA_HOST` | Server bind address (default: `127.0.0.1:11434`) |
| `OLLAMA_DEBUG` | Enable debug logging (prompts may appear in server logs) |

## Workarounds for Conversation Logging

Since Ollama does not natively save conversations, common workarounds include:

1. **Shell redirection**: `ollama run model 2>&1 | tee chat.log`
2. **Third-party clients**: Open WebUI, Ollama GUI, etc. (each with their own storage)
3. **API-based logging**: Capture requests/responses via the REST API (`/api/chat`)
4. **Debug mode**: Enable `OLLAMA_DEBUG=1` to include prompts in server logs

## Notes

- Ollama is primarily a local LLM runner, not an AI coding agent
- The `/save` command saves conversation state as a model, not as a log file
- Server logs (when debug mode is enabled) may contain prompts but are not structured for retrieval
- Third-party UIs built on Ollama implement their own conversation persistence
