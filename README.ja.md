# ai-agent-log-formats

AIコーディングエージェント／アシスタントの対話履歴（プロンプトログ）の保存形式・取得方法をまとめたリファレンス。

**[English](README.md)**

## 対応ツール一覧

| カテゴリ | ツール | ログ形式 | ローカルログ | 詳細 |
|---------|--------|---------|:----------:|------|
| **CLI** | [Claude Code](tools/claude-code/ja.md) | JSONL | Yes | history.jsonl + プロジェクト別セッション |
| **CLI** | [OpenAI Codex CLI](tools/openai-codex/ja.md) | JSONL | Yes | rollout セッションファイル |
| **CLI** | [Aider](tools/aider/ja.md) | Markdown / SQLite | Yes | チャット履歴 + DB |
| **CLI** | [amp](tools/amp/ja.md) | JSONL | Yes | セッションファイル |
| **エディタ** | [Cursor](tools/cursor/ja.md) | SQLite | Yes | state.vscdb + 独自ストレージ |
| **エディタ** | [Trae](tools/trae/ja.md) | SQLite / JSON | Yes | VS Code系構造 |
| **エディタ** | [Zed AI](tools/zed-ai/ja.md) | JSON | Yes | conversations ディレクトリ |
| **エディタ** | [PearAI](tools/pearai/ja.md) | SQLite / JSON | Yes | VS Code系構造 |
| **エディタ** | [Void](tools/void/ja.md) | SQLite / JSON | Yes | VS Code系構造 |
| **VS Code拡張** | [GitHub Copilot Chat](tools/github-copilot-chat/ja.md) | SQLite | Yes | state.vscdb |
| **VS Code拡張** | [Cline](tools/cline/ja.md) | JSON | Yes | api_conversation_history.json |
| **VS Code拡張** | [Roo Code](tools/roo-code/ja.md) | JSON | Yes | Cline同一構造 |
| **VS Code拡張** | [Kilo Code](tools/kilo-code/ja.md) | JSON | Yes | Cline系フォーク |
| **VS Code拡張** | [Continue](tools/continue/ja.md) | JSON | Yes | セッションJSON |
| **VS Code拡張** | [Sourcegraph Cody](tools/sourcegraph-cody/ja.md) | JSON | Yes | globalStorage内 |
| **VS Code拡張** | [Amazon Q Developer](tools/amazon-q-developer/ja.md) | JSON | Yes | globalStorage内 |
| **VS Code拡張** | [Augment Code](tools/augment-code/ja.md) | 不明 | 要調査 | — |
| **IDE内蔵** | [JetBrains AI Assistant](tools/jetbrains-ai/ja.md) | XML / JSON | Yes | IDE設定ディレクトリ |
| **IDE内蔵** | [Gemini Code Assist](tools/gemini-code-assist/ja.md) | 不明 | 要調査 | — |
| **IDE/エディタ** | [Windsurf (Cascade)](tools/windsurf/ja.md) | テキスト | Yes | 自動要約メモリ |
| **IDE/エディタ** | [OpenCode](tools/opencode/ja.md) | SQLite | Yes | opencode.db |
| **IDE/エディタ** | [Google Antigravity](tools/google-antigravity/ja.md) | テキスト | Yes | ログファイル |
| **自律エージェント** | [SWE-agent](tools/swe-agent/ja.md) | JSON | Yes | trajectory ファイル |
| **自律エージェント** | [OpenHands](tools/openhands/ja.md) | JSON | Yes | セッションログ |
| **クラウド** | [Devin](tools/devin/ja.md) | — | No | クラウド側のみ |
| **クラウド** | [Replit Agent](tools/replit-agent/ja.md) | — | No | クラウド側のみ |
| **クラウド** | [Bolt](tools/bolt/ja.md) | — | No | ブラウザ完結 |
| **クラウド** | [Lovable](tools/lovable/ja.md) | — | No | ブラウザ完結 |
| **クラウド** | [v0](tools/v0/ja.md) | — | No | ブラウザ完結 |
| **補完特化** | [Tabnine](tools/tabnine/ja.md) | — | 限定的 | 対話ログなし（補完主体） |
| **補完特化** | [Supermaven](tools/supermaven/ja.md) | — | 限定的 | 対話ログなし（補完主体） |

### 凡例

- **ローカルログ**: ユーザーのマシン上にプロンプト履歴が保存されるか
- **要調査**: ログ形式が未確認のもの。情報提供歓迎

## ディレクトリ構成

```
ai-agent-log-formats/
├── README.md              # 英語版
├── README.ja.md           # 日本語版
└── tools/
    ├── claude-code/
    │   ├── en.md          # English
    │   └── ja.md          # 日本語
    ├── cursor/
    │   ├── en.md
    │   └── ja.md
    └── ...
```

## 注意事項

- 各ツールのログ形式はバージョンアップで変更される可能性がある
- パスはOS（Windows / macOS / Linux）によって異なる
- 一部ツールはログ形式が未確認（要調査）のものを含む
- クラウド型ツールはローカルにログが残らないため、API経由の取得が必要
