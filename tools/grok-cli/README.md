# Grok CLI

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI (Claude Code proxy wrapper) |
| Log Format | — (delegates to Claude Code) |
| Local Logs | No (own logs); Yes (via Claude Code) |
| npm Package | [grok-cli](https://www.npmjs.com/package/grok-cli) |

## Overview

The npm `grok-cli` package is a **proxy wrapper that runs Claude Code with xAI Grok models**. It starts a local proxy server that translates Anthropic API calls to xAI's Grok API, then spawns Claude Code pointing to the proxy. It is not an official xAI product.

**This tool does not have its own conversation storage.** All conversation history is saved by Claude Code in `~/.claude/` (see [Claude Code](../claude-code/)).

## How It Works

```
┌─────────┐     ┌──────────────────┐     ┌──────────────┐
│ grok-cli │────▶│ Local Proxy      │────▶│ xAI Grok API │
│          │     │ (localhost:3000)  │     │ (api.x.ai)   │
└─────────┘     └──────────────────┘     └──────────────┘
     │
     └──▶ Spawns Claude Code with:
          ANTHROPIC_BASE_URL=http://localhost:3000
          ANTHROPIC_MODEL=grok-4
```

## API Key Storage

API keys are stored in the **OS keychain** (not on the filesystem):

- Service name: `grok-cli`
- Account name: `grok-api-key`
- Uses the `keytar` npm package for secure credential storage

| Command | Description |
|---------|-------------|
| `grok --api-key <key>` | Set and store API key in keychain |
| `grok --reset-key` | Delete stored API key from keychain |

## Conversation Storage

Since Grok CLI spawns Claude Code, all conversation data is stored in Claude Code's directories:

| Data | Location |
|------|----------|
| Session logs | `~/.claude/projects/<encoded-path>/<session-uuid>.jsonl` |
| CLI history | `~/.claude/history.jsonl` |

See [Claude Code log format](../claude-code/) for full details.

## Command-Line Options

| Option | Default | Description |
|--------|---------|-------------|
| `--api-key <key>` | — | Grok API key (stored in keychain) |
| `--port <port>` | `3000` | Port for the local proxy server |
| `--base-url <url>` | `https://api.x.ai` | xAI API base URL |
| `--reasoning-model <model>` | `grok-4` | Model for reasoning tasks |
| `--completion-model <model>` | `grok-3` | Model for fast completions |
| `--debug` | — | Enable debug logging |
| `--reset-key` | — | Clear stored API key |

## Environment Variables Set by Proxy

When spawning Claude Code, Grok CLI sets:

| Variable | Value | Description |
|----------|-------|-------------|
| `ANTHROPIC_BASE_URL` | `http://localhost:<port>` | Points to local proxy |
| `ANTHROPIC_API_KEY` | `sk-ant-api03-demo` | Dummy key (proxy handles auth) |
| `ANTHROPIC_MODEL` | `grok-4` | Reasoning model |
| `ANTHROPIC_SMALL_FAST_MODEL` | `grok-3` | Completion model |
| `DISABLE_TELEMETRY` | `1` | Disables Claude Code telemetry |

## Notes

- Not an official xAI product (community-built by [whitesmith](https://github.com/whitesmith/grok-cli))
- Requires Claude Code to be installed and available in `PATH`
- No independent conversation persistence — fully delegates to Claude Code
- Different from [superagent-ai/grok-cli](https://github.com/superagent-ai/grok-cli), which is a standalone Grok agent (stores config in `~/.grok/` but does not persist conversations)
