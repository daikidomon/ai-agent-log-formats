# ai-agent-log-formats

AIコーディングエージェント／アシスタントの対話履歴（プロンプトログ）の保存形式・取得方法をまとめたリファレンス。

**[English](README.md)**

## 対応ツール一覧

| カテゴリ | ツール | ログ形式 | ローカルログ | 詳細 |
|---------|--------|---------|:----------:|------|
| **CLI** | [Claude Code](ja/tools/claude-code.md) | JSONL | Yes | history.jsonl + プロジェクト別セッション |
| **CLI** | [OpenAI Codex CLI](ja/tools/openai-codex.md) | JSONL | Yes | rollout セッションファイル |
| **CLI** | [Aider](ja/tools/aider.md) | Markdown / SQLite | Yes | チャット履歴 + DB |
| **CLI** | [amp](ja/tools/amp.md) | JSONL | Yes | セッションファイル |
| **エディタ** | [Cursor](ja/tools/cursor.md) | SQLite | Yes | state.vscdb + 独自ストレージ |
| **エディタ** | [Trae](ja/tools/trae.md) | SQLite / JSON | Yes | VS Code系構造 |
| **エディタ** | [Zed AI](ja/tools/zed-ai.md) | JSON | Yes | conversations ディレクトリ |
| **エディタ** | [PearAI](ja/tools/pearai.md) | SQLite / JSON | Yes | VS Code系構造 |
| **エディタ** | [Void](ja/tools/void.md) | SQLite / JSON | Yes | VS Code系構造 |
| **VS Code拡張** | [GitHub Copilot Chat](ja/tools/github-copilot-chat.md) | SQLite | Yes | state.vscdb |
| **VS Code拡張** | [Cline](ja/tools/cline.md) | JSON | Yes | api_conversation_history.json |
| **VS Code拡張** | [Roo Code](ja/tools/roo-code.md) | JSON | Yes | Cline同一構造 |
| **VS Code拡張** | [Kilo Code](ja/tools/kilo-code.md) | JSON | Yes | Cline系フォーク |
| **VS Code拡張** | [Continue](ja/tools/continue.md) | JSON | Yes | セッションJSON |
| **VS Code拡張** | [Sourcegraph Cody](ja/tools/sourcegraph-cody.md) | JSON | Yes | globalStorage内 |
| **VS Code拡張** | [Amazon Q Developer](ja/tools/amazon-q-developer.md) | JSON | Yes | globalStorage内 |
| **VS Code拡張** | [Augment Code](ja/tools/augment-code.md) | 不明 | 要調査 | — |
| **IDE内蔵** | [JetBrains AI Assistant](ja/tools/jetbrains-ai.md) | XML / JSON | Yes | IDE設定ディレクトリ |
| **IDE内蔵** | [Gemini Code Assist](ja/tools/gemini-code-assist.md) | 不明 | 要調査 | — |
| **IDE/エディタ** | [Windsurf (Cascade)](ja/tools/windsurf.md) | テキスト | Yes | 自動要約メモリ |
| **IDE/エディタ** | [OpenCode](ja/tools/opencode.md) | SQLite | Yes | opencode.db |
| **IDE/エディタ** | [Google Antigravity](ja/tools/google-antigravity.md) | テキスト | Yes | ログファイル |
| **自律エージェント** | [SWE-agent](ja/tools/swe-agent.md) | JSON | Yes | trajectory ファイル |
| **自律エージェント** | [OpenHands](ja/tools/openhands.md) | JSON | Yes | セッションログ |
| **クラウド** | [Devin](ja/tools/devin.md) | — | No | クラウド側のみ |
| **クラウド** | [Replit Agent](ja/tools/replit-agent.md) | — | No | クラウド側のみ |
| **クラウド** | [Bolt](ja/tools/bolt.md) | — | No | ブラウザ完結 |
| **クラウド** | [Lovable](ja/tools/lovable.md) | — | No | ブラウザ完結 |
| **クラウド** | [v0](ja/tools/v0.md) | — | No | ブラウザ完結 |
| **補完特化** | [Tabnine](ja/tools/tabnine.md) | — | 限定的 | 対話ログなし（補完主体） |
| **補完特化** | [Supermaven](ja/tools/supermaven.md) | — | 限定的 | 対話ログなし（補完主体） |

### 凡例

- **ローカルログ**: ユーザーのマシン上にプロンプト履歴が保存されるか
- **要調査**: ログ形式が未確認のもの。情報提供歓迎

## ディレクトリ構成

```
ai-agent-log-formats/
├── README.md          # 英語版
├── README.ja.md       # 日本語版
├── en/tools/          # 英語版ツールドキュメント
└── ja/tools/          # 日本語版ツールドキュメント
```

## 注意事項

- 各ツールのログ形式はバージョンアップで変更される可能性がある
- パスはOS（Windows / macOS / Linux）によって異なる
- 一部ツールはログ形式が未確認（要調査）のものを含む
- クラウド型ツールはローカルにログが残らないため、API経由の取得が必要
