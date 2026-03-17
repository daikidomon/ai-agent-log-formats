# GitHub Copilot CLI

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLIコーディングエージェント |
| ログ形式 | JSONL + YAML |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| macOS | `~/.copilot/` |
| Linux | `~/.copilot/` |
| Windows | `%USERPROFILE%\.copilot\` |

## ファイル構成（実機確認済み）

```
~/.copilot/
├── config.json                              # 基本設定
├── logs/                                    # デバッグログ
│   ├── copilot.log                         # メインログ（認証、起動等）
│   └── process-<timestamp>-<pid>.log       # プロセス別デバッグログ
└── session-state/
    └── <session-uuid>/                     # セッションごとのディレクトリ
        ├── events.jsonl                    # セッション全イベントログ
        ├── workspace.yaml                  # ワークスペースメタデータ
        ├── checkpoints/
        │   └── index.md                   # チェックポイント履歴（Markdownテーブル）
        ├── files/                          # （ツール使用時に作成）
        └── research/                       # （ツール使用時に作成）
```

## データ形式（実機確認済み）

### config.json

グローバル設定ファイル。初回起動時刻を記録：

```json
{
  "firstLaunchAt": "2026-03-17T09:01:20.162Z"
}
```

### workspace.yaml

セッション単位のワークスペースメタデータ（YAML形式）：

```yaml
id: 45a7d084-a838-4dda-9aca-9c268740bb35
cwd: /home/ubuntu/project
git_root: /home/ubuntu/project
repository: user/project
host_type: github
branch: main
summary_count: 0
created_at: 2026-03-17T09:01:20.579Z
updated_at: 2026-03-17T09:01:25.754Z
summary: say hello
```

| フィールド | 説明 |
|-----------|------|
| `id` | セッションUUID（ディレクトリ名と一致） |
| `cwd` | セッション開始時の作業ディレクトリ |
| `git_root` | Gitリポジトリルート |
| `repository` | `owner/repo` 形式 |
| `host_type` | `github` |
| `branch` | アクティブブランチ |
| `summary_count` | 会話サマリー回数 |
| `created_at` / `updated_at` | ISO 8601タイムスタンプ |
| `summary` | 最初のユーザーメッセージ（セッションタイトルとして使用） |

### events.jsonl

メインのセッションログ。各行は以下の共通フィールドを持つJSONオブジェクト：

| フィールド | 型 | 説明 |
|-----------|------|------|
| `type` | string | イベント種別（下記参照） |
| `data` | object | イベント固有のペイロード |
| `id` | UUID | イベント一意識別子 |
| `timestamp` | string | ISO 8601タイムスタンプ |
| `parentId` | UUID / null | チェーン内の前イベント |

### イベント種別

#### `session.start`

セッション最初のイベント。

```json
{
  "type": "session.start",
  "data": {
    "sessionId": "45a7d084-...",
    "version": 1,
    "producer": "copilot-agent",
    "copilotVersion": "1.0.6",
    "startTime": "2026-03-17T09:01:20.576Z",
    "context": {
      "cwd": "/home/ubuntu/project",
      "gitRoot": "/home/ubuntu/project",
      "branch": "main",
      "headCommit": "e77111f...",
      "repository": "user/project",
      "hostType": "github"
    },
    "alreadyInUse": false
  }
}
```

#### `session.model_change`

モデル選択・変更時に発行。

```json
{
  "type": "session.model_change",
  "data": { "newModel": "gpt-5-mini" }
}
```

#### `user.message`

ユーザー入力。元のコンテンツとシステム拡張版の `transformedContent` を含む。

```json
{
  "type": "user.message",
  "data": {
    "content": "say hello",
    "transformedContent": "<current_datetime>...</current_datetime>\n\nsay hello\n\n<reminder>...</reminder>",
    "attachments": [],
    "interactionId": "9d39d375-..."
  }
}
```

#### `assistant.turn_start` / `assistant.turn_end`

アシスタントターンの開始と終了。1つのインタラクション内で複数ターンが発生しうる（例：ツール呼び出し後の応答）。

#### `assistant.message`

アシスタント応答。ツールリクエストを含む場合がある。

```json
{
  "type": "assistant.message",
  "data": {
    "messageId": "d4937b1b-...",
    "content": "Here are the files...",
    "toolRequests": [
      {
        "toolCallId": "call_XaRk2O8Qf4...",
        "name": "bash",
        "arguments": {
          "command": "ls -la --color=never",
          "description": "List files in the current directory"
        },
        "type": "function",
        "intentionSummary": "List files in the current directory"
      }
    ],
    "interactionId": "db3779dc-...",
    "reasoningOpaque": "...",
    "encryptedContent": "...",
    "outputTokens": 593
  }
}
```

| フィールド | 説明 |
|-----------|------|
| `content` | プレーンテキスト応答 |
| `toolRequests` | ツール呼び出し配列（ツール不使用時は空） |
| `reasoningOpaque` | 暗号化された推論トレース（base64） |
| `encryptedContent` | 暗号化された完全応答（base64） |
| `outputTokens` | このメッセージのトークン数 |

#### `tool.execution_start` / `tool.execution_complete`

ツール実行の開始と完了。完了時には `content`（短縮版）と `detailedContent`（全出力）を含む。

```json
{
  "type": "tool.execution_complete",
  "data": {
    "toolCallId": "call_XaRk2O8Qf4...",
    "model": "gpt-5-mini",
    "success": true,
    "result": {
      "content": "total 36\ndrwxrwxr-x 5 ubuntu ...",
      "detailedContent": "total 36\ndrwxrwxr-x 5 ubuntu ..."
    },
    "toolTelemetry": {
      "properties": { "executionMode": "sync" },
      "metrics": { "commandTimeout": 30000 }
    }
  }
}
```

#### `session.shutdown`

最終イベント。使用量メトリクスとセッション統計を含む。

```json
{
  "type": "session.shutdown",
  "data": {
    "shutdownType": "routine",
    "totalPremiumRequests": 1,
    "totalApiDurationMs": 21883,
    "codeChanges": {
      "linesAdded": 0,
      "linesRemoved": 0,
      "filesModified": []
    },
    "modelMetrics": {
      "gpt-5-mini": {
        "requests": { "count": 2, "cost": 1 },
        "usage": {
          "inputTokens": 27911,
          "outputTokens": 629,
          "cacheReadTokens": 26880,
          "cacheWriteTokens": 0
        }
      }
    }
  }
}
```

### その他のイベント種別

| 種別 | 説明 |
|------|------|
| `session.tools_updated` | セッション中のツールリスト変更 |
| `session.background_tasks_changed` | バックグラウンドタスク状態の変更 |
| `session.info` | セッション情報メッセージ |
| `result` | 最終結果（`sessionId`, `exitCode`, `usage` サマリー） |

### checkpoints/index.md

セッション中に作成されたチェックポイント（gitスナップショット）のMarkdownテーブル：

```markdown
# Checkpoint History

| # | Title | File |
|---|-------|------|
| 1 | Added feature X | checkpoint-1.patch |
```

## イベントチェーン

イベントは `parentId` でリンクされ、チェーンを形成する：

```
session.start (parentId: null)
  └─ session.model_change
       └─ user.message
            └─ assistant.turn_start
                 └─ assistant.message (toolRequestsあり)
                      └─ tool.execution_start
                           └─ tool.execution_complete
                                └─ assistant.turn_end
                                     └─ assistant.turn_start (ターン2)
                                          └─ assistant.message (最終応答)
                                               └─ assistant.turn_end
                                                    └─ session.shutdown
```

## 認証

GitHub OAuthデバイスフローで認証。トークン保存にはシステムキーチェーンが必要。代替として、既存の `gh auth token` から `GH_TOKEN` 環境変数を設定可能。

## CLI使用方法

```bash
# 対話セッション
copilot

# プロンプト指定（非対話）
copilot -p "your prompt"

# JSON出力
copilot -p "your prompt" --output-format json -s

# ツール自動承認
copilot -p "your prompt" --allow-all
```

## 備考

- Copilot CLIはスタンドアロンバイナリ（`~/.local/bin/copilot`）で、VS CodeのGitHub Copilot拡張とは別製品
- セッションIDはUUID v4
- `assistant.message_delta` イベントはストリーミング中に発生する場合がある（エフェメラル、常に永続化されるわけではない）
- `reasoningOpaque` と `encryptedContent` フィールドはセッションキーなしでは復号不可
- `report_intent` はアクション実行前にアシスタントの意図を記録する内部ツール
- Copilot CLI v1.0.6（Linux）で実機確認済み
