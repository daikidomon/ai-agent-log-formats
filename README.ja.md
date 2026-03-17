# ai-agent-log-formats

AIコーディングエージェント／アシスタントの対話履歴（プロンプトログ）の保存形式・取得方法をまとめたリファレンス。

**[English](README.md)**

## 対応ツール一覧

| カテゴリ | ツール | ログ形式 | ローカルログ | 実機確認 | 詳細 |
|---------|--------|---------|:----------:|:------:|------|
| **CLI** | [Claude Code](tools/claude-code/README.ja.md) | JSONL | Yes | :white_check_mark: | history.jsonl + プロジェクト別セッション |
| **CLI** | [OpenAI Codex CLI](tools/openai-codex/README.ja.md) | JSONL | Yes | :white_check_mark: | rollout セッションファイル |
| **CLI** | [Aider](tools/aider/README.ja.md) | Markdown / SQLite | Yes | | チャット履歴 + DB |
| **CLI** | [amp](tools/amp/README.ja.md) | JSONL | Yes | | セッションファイル |
| **CLI** | [Gemini CLI](tools/gemini-cli/README.ja.md) | JSON | Yes | :white_check_mark: | プロジェクト別セッション + 名前付きチェックポイント |
| **CLI** | [Grok CLI](tools/grok-cli/README.ja.md) | — | No | :white_check_mark: | Claude Code プロキシラッパー。ログは `~/.claude/` に保存 |
| **CLI** | [Ollama](tools/ollama/README.ja.md) | — | 限定的 | :white_check_mark: | コマンド履歴のみ。会話の永続化なし |
| **エディタ** | [Cursor](tools/cursor/README.ja.md) | JSONL + SQLite | Yes | :white_check_mark: | CLI: `~/.cursor/` トランスクリプト + store.db; GUI: state.vscdb |
| **エディタ** | [Trae](tools/trae/README.ja.md) | SQLite / JSON | Yes | | VS Code系構造 |
| **エディタ** | [Zed AI](tools/zed-ai/README.ja.md) | JSON | Yes | | conversations ディレクトリ |
| **エディタ** | [PearAI](tools/pearai/README.ja.md) | SQLite / JSON | Yes | | VS Code系構造 |
| **エディタ** | [Void](tools/void/README.ja.md) | SQLite / JSON | Yes | | VS Code系構造 |
| **VS Code拡張** | [GitHub Copilot Chat](tools/github-copilot-chat/README.ja.md) | SQLite | Yes | | state.vscdb |
| **VS Code拡張** | [Cline](tools/cline/README.ja.md) | JSON | Yes | | api_conversation_history.json |
| **VS Code拡張** | [Roo Code](tools/roo-code/README.ja.md) | JSON | Yes | | Cline同一構造 |
| **VS Code拡張** | [Kilo Code](tools/kilo-code/README.ja.md) | JSON | Yes | | Cline系フォーク |
| **VS Code拡張** | [Continue](tools/continue/README.ja.md) | JSON | Yes | | セッションJSON |
| **VS Code拡張** | [Sourcegraph Cody](tools/sourcegraph-cody/README.ja.md) | JSON | Yes | | globalStorage内 |
| **VS Code拡張** | [Amazon Q Developer](tools/amazon-q-developer/README.ja.md) | JSON | Yes | | globalStorage内 |
| **VS Code拡張** | [Augment Code](tools/augment-code/README.ja.md) | 不明 | 要調査 | | — |
| **IDE内蔵** | [JetBrains AI Assistant](tools/jetbrains-ai/README.ja.md) | XML / JSON | Yes | | IDE設定ディレクトリ |
| **IDE内蔵** | [Gemini Code Assist](tools/gemini-code-assist/README.ja.md) | 不明 | 要調査 | | — |
| **IDE/エディタ** | [Windsurf (Cascade)](tools/windsurf/README.ja.md) | テキスト | Yes | | 自動要約メモリ |
| **IDE/エディタ** | [OpenCode](tools/opencode/README.ja.md) | SQLite | Yes | :white_check_mark: | opencode.db |
| **IDE/エディタ** | [Google Antigravity](tools/google-antigravity/README.ja.md) | テキスト | Yes | | ログファイル |
| **自律エージェント** | [SWE-agent](tools/swe-agent/README.ja.md) | JSON | Yes | | trajectory ファイル |
| **自律エージェント** | [OpenHands](tools/openhands/README.ja.md) | JSON | Yes | | セッションログ |
| **クラウド** | [Devin](tools/devin/README.ja.md) | — | No | | クラウド側のみ |
| **クラウド** | [Replit Agent](tools/replit-agent/README.ja.md) | — | No | | クラウド側のみ |
| **クラウド** | [Bolt](tools/bolt/README.ja.md) | — | No | | ブラウザ完結 |
| **クラウド** | [Lovable](tools/lovable/README.ja.md) | — | No | | ブラウザ完結 |
| **クラウド** | [v0](tools/v0/README.ja.md) | — | No | | ブラウザ完結 |
| **補完特化** | [Tabnine](tools/tabnine/README.ja.md) | — | 限定的 | | 対話ログなし（補完主体） |
| **補完特化** | [Supermaven](tools/supermaven/README.ja.md) | — | 限定的 | | 対話ログなし（補完主体） |

### 凡例

- **ローカルログ**: ユーザーのマシン上にプロンプト履歴が保存されるか
- **実機確認**: :white_check_mark: = 実際にインストール・実行し、データ形式を検証済み
- **要調査**: ログ形式が未確認のもの。情報提供歓迎

## ディレクトリ構成

```
ai-agent-log-formats/
├── README.md              # 英語版
├── README.ja.md           # 日本語版
└── tools/
    ├── claude-code/
    │   ├── README.md      # English（GitHubで自動表示）
    │   └── README.ja.md   # 日本語
    ├── cursor/
    │   ├── README.md
    │   └── README.ja.md
    └── ...
```

## 注意事項

- 各ツールのログ形式はバージョンアップで変更される可能性がある
- パスはOS（Windows / macOS / Linux）によって異なる
- 一部ツールはログ形式が未確認（要調査）のものを含む
- クラウド型ツールはローカルにログが残らないため、API経由の取得が必要
