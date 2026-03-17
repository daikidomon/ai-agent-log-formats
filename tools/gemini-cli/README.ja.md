# Gemini CLI

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| macOS | `~/.gemini/` |
| Linux | `~/.gemini/` |
| Windows | `%USERPROFILE%\.gemini\` |

環境変数 `GEMINI_CLI_HOME` が設定されている場合はそのパスを優先。

## ファイル構造

```
~/.gemini/
├── settings.json                              # ユーザー設定
├── GEMINI.md                                  # グローバルカスタム指示
├── projects.json                              # プロジェクトパス → slug のマッピング
├── history/
│   └── <project-slug>/                        # プロジェクト別 history ディレクトリ
│       └── .project_root                      # プロジェクトフルパスのマーカーファイル
└── tmp/
    └── <project-slug>/                        # プロジェクトの basename をスラッグ化（ハッシュではない）
        ├── .project_root                      # プロジェクトフルパスのマーカーファイル
        ├── chats/
        │   └── session-<timestamp>-<id>.json  # 自動保存されたセッションファイル
        ├── logs/
        │   └── session-<sessionId>.jsonl      # アクティビティログ
        ├── checkpoints/                       # セッションチェックポイント
        ├── shell_history                      # シェルコマンド履歴
        └── <sessionId>/
            ├── plans/                         # エージェントプラン
            └── tasks/                         # エージェントタスク
```

### プロジェクト slug の命名規則

プロジェクトディレクトリは、プロジェクトディレクトリの **basename を元にした人間が読める slug** で命名される（ハッシュではない）:

- `/home/ubuntu/my-project` → `my-project`
- `/home/ubuntu` → `ubuntu`
- 衝突時はカウンタサフィックスで対応: `my-project`, `my-project-1`, `my-project-2`

マッピングは `~/.gemini/projects.json` に保存:

```json
{
  "projects": {
    "/home/ubuntu": "ubuntu",
    "/home/ubuntu/my-project": "my-project"
  }
}
```

## データ形式

### セッション JSON (ConversationRecord)

各セッションファイルは `ConversationRecord` オブジェクト:

```json
{
  "sessionId": "uuid-string",
  "projectHash": "sha256-of-project-root",
  "startTime": "2026-03-17T09:45:00.000Z",
  "lastUpdated": "2026-03-17T10:20:00.000Z",
  "summary": "オプションの自動生成サマリー",
  "kind": "main",
  "messages": [
    {
      "id": "uuid-string",
      "timestamp": "2026-03-17T09:45:01.000Z",
      "type": "user",
      "content": [{"text": "ユーザーのプロンプト"}]
    },
    {
      "id": "uuid-string",
      "timestamp": "2026-03-17T09:45:05.000Z",
      "type": "gemini",
      "content": [{"text": "モデルの応答"}],
      "toolCalls": [],
      "thoughts": [],
      "tokens": {
        "input": 100,
        "output": 200,
        "cached": 50,
        "total": 350
      },
      "model": "gemini-2.5-pro"
    }
  ]
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `sessionId` | string | セッション UUID |
| `projectHash` | string | プロジェクトルートパスの SHA256 ハッシュ |
| `startTime` | string | ISO 8601 セッション開始時刻 |
| `lastUpdated` | string | ISO 8601 最終更新時刻 |
| `messages` | array | `MessageRecord` オブジェクトの配列 |
| `summary` | string? | 自動生成されたセッションサマリー |
| `kind` | string? | `"main"` または `"subagent"` |
| `directories` | string[]? | `/dir add` で追加されたワークスペースディレクトリ |

### メッセージタイプ

| `type` | 説明 |
|--------|------|
| `user` | ユーザー入力 |
| `gemini` | モデル応答（`toolCalls`, `thoughts`, `tokens`, `model` を含む） |
| `info` | 情報メッセージ |
| `error` | エラーメッセージ |
| `warning` | 警告メッセージ |

### セッションファイルの命名

ファイル名パターン: `session-YYYY-MM-DDTHH-MM-<sessionId:8>.json`

例: `session-2026-03-17T09-45-abc12def.json`

### ヘッドレスモード JSON出力 (`--output-format json`)

```json
{
  "response": "モデルからのプレーンテキスト応答",
  "stats": {
    "session": {},
    "model": {},
    "tools": {},
    "user": {}
  },
  "error": null
}
```

## セッション管理コマンド

| コマンド | 説明 |
|---------|------|
| `/chat save <tag>` | 現在の会話をタグ付きで保存 |
| `/chat list` | 保存済みセッション一覧を表示 |
| `/chat resume <tag>` | 保存済みセッションを再開（タグ、インデックス、`latest` を指定可能） |
| `/chat share` | セッションを共有可能なファイルにエクスポート |

## 取得方法

```python
import json
from pathlib import Path

gemini_dir = Path.home() / ".gemini"

# tmp ディレクトリからプロジェクト別の自動保存セッションを読み取り
tmp_dir = gemini_dir / "tmp"
if tmp_dir.exists():
    for project_dir in tmp_dir.iterdir():
        chats_subdir = project_dir / "chats"
        if chats_subdir.exists():
            for session_file in chats_subdir.glob("session-*.json"):
                with open(session_file, "r", encoding="utf-8") as f:
                    record = json.load(f)
                    for msg in record.get("messages", []):
                        if msg.get("type") == "user":
                            content = msg.get("content", [])
                            for part in content:
                                if isinstance(part, dict):
                                    print(part.get("text", ""))
```

## セッション保持設定

`settings.json` で自動クリーンアップを設定:

```json
{
  "general": {
    "sessionRetention": {
      "enabled": true,
      "maxAge": "30d",
      "maxCount": 50
    }
  }
}
```

| 設定 | デフォルト | 説明 |
|------|-----------|------|
| `enabled` | `true` | 自動クリーンアップの有効化 |
| `maxAge` | `"30d"` | セッションの最大保持期間（例: `"24h"`, `"7d"`, `"4w"`） |
| `maxCount` | `50` | 保持するセッションの最大数 |
| `minRetention` | `"1d"` | 最近のセッション削除を防ぐ安全制限 |

クリーンアップ時は関連するアクティビティログ (`logs/session-*.jsonl`) やツール出力ディレクトリも削除される。

## 注意事項

- セッションはプロジェクト固有。ディレクトリを切り替えるとアクセスできるセッション履歴が変わる
- プロジェクトディレクトリはハッシュではなくスラッグ化された basename を使用（マッピングは `projects.json` に保存）
- `/chat save` で既存タグを指定すると警告なしで上書きされる
- バージョン管理機能なし。各タグに保存されるセッションは1つのみ
- シェルコマンド履歴は `shell_history` に別途保存
- サブエージェントセッション (`kind: "subagent"`) はメインセッションと並行して保存されるが、再開リストには表示されない
- テレメトリ設定の `telemetry.outfile` でプロンプトをローカルにログ出力可能
