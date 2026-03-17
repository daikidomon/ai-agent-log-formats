# Grok CLI

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI（Claude Code プロキシラッパー） |
| ログ形式 | —（Claude Code に委譲） |
| ローカルログ | 独自のログなし。Claude Code 経由であり |
| npm パッケージ | [grok-cli](https://www.npmjs.com/package/grok-cli) |

## 概要

npm の `grok-cli` パッケージは、**xAI の Grok モデルで Claude Code を実行するプロキシラッパー**。ローカルプロキシサーバーを起動して Anthropic API のリクエストを xAI の Grok API に変換し、Claude Code をそのプロキシ経由で起動する。xAI の公式製品ではない。

**このツールは独自の会話ストレージを持たない。** すべての会話履歴は Claude Code により `~/.claude/` に保存される（詳細は [Claude Code](../claude-code/README.ja.md) を参照）。

## 動作の仕組み

```
┌─────────┐     ┌──────────────────┐     ┌──────────────┐
│ grok-cli │────▶│ ローカルプロキシ  │────▶│ xAI Grok API │
│          │     │ (localhost:3000)  │     │ (api.x.ai)   │
└─────────┘     └──────────────────┘     └──────────────┘
     │
     └──▶ Claude Code を以下の設定で起動:
          ANTHROPIC_BASE_URL=http://localhost:3000
          ANTHROPIC_MODEL=grok-4
```

## API キーの保存

API キーは **OS のキーチェーン**に保存される（ファイルシステム上ではない）:

- サービス名: `grok-cli`
- アカウント名: `grok-api-key`
- `keytar` npm パッケージによるセキュアな資格情報管理

| コマンド | 説明 |
|---------|------|
| `grok --api-key <key>` | API キーを設定しキーチェーンに保存 |
| `grok --reset-key` | キーチェーンから API キーを削除 |

## 会話の保存先

Grok CLI は Claude Code を起動するため、すべての会話データは Claude Code のディレクトリに保存:

| データ | 保存先 |
|--------|--------|
| セッションログ | `~/.claude/projects/<encoded-path>/<session-uuid>.jsonl` |
| CLI 履歴 | `~/.claude/history.jsonl` |

詳細は [Claude Code のログ形式](../claude-code/README.ja.md) を参照。

## コマンドラインオプション

| オプション | デフォルト | 説明 |
|-----------|-----------|------|
| `--api-key <key>` | — | Grok API キー（キーチェーンに保存） |
| `--port <port>` | `3000` | ローカルプロキシサーバーのポート |
| `--base-url <url>` | `https://api.x.ai` | xAI API ベース URL |
| `--reasoning-model <model>` | `grok-4` | 推論タスク用モデル |
| `--completion-model <model>` | `grok-3` | 高速補完用モデル |
| `--debug` | — | デバッグログの有効化 |
| `--reset-key` | — | 保存済み API キーをクリア |

## プロキシが設定する環境変数

Claude Code 起動時に Grok CLI が設定する環境変数:

| 変数 | 値 | 説明 |
|------|-----|------|
| `ANTHROPIC_BASE_URL` | `http://localhost:<port>` | ローカルプロキシを指定 |
| `ANTHROPIC_API_KEY` | `sk-ant-api03-demo` | ダミーキー（プロキシが認証を処理） |
| `ANTHROPIC_MODEL` | `grok-4` | 推論モデル |
| `ANTHROPIC_SMALL_FAST_MODEL` | `grok-3` | 補完モデル |
| `DISABLE_TELEMETRY` | `1` | Claude Code のテレメトリを無効化 |

## 注意事項

- xAI の公式製品ではない（[whitesmith](https://github.com/whitesmith/grok-cli) によるコミュニティ製）
- Claude Code がインストール済みで `PATH` に含まれている必要がある
- 独自の会話永続化機能なし — すべて Claude Code に委譲
- [superagent-ai/grok-cli](https://github.com/superagent-ai/grok-cli)（スタンドアロンの Grok エージェント、設定は `~/.grok/` に保存するが会話の永続化なし）とは別のツール
