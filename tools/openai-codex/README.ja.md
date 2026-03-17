# OpenAI Codex（CLI）

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI |
| ログ形式 | JSONL（rollout セッションファイル） |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.codex\sessions\` |
| macOS | `~/.codex/sessions/` |
| Linux | `~/.codex/sessions/` |

環境変数 `CODEX_HOME` が設定されている場合は `$CODEX_HOME/sessions/` を使用。

## ファイル構造

```
~/.codex/
├── sessions/
│   └── rollout-YYYY-MM-DDThh-mm-ss-<uuid>.jsonl  # セッション別会話ログ
└── tmp/                                            # 一時ファイル
```

## データ形式

各行が `timestamp`, `type`, `payload` フィールドを持つ JSON オブジェクト。`type` フィールドでエントリ種別を判別する。

### session_meta（セッション開始時のメタ情報）

```json
{
  "timestamp": "2026-03-17T08:30:00.000Z",
  "type": "session_meta",
  "payload": {
    "id": "abcd1234-5678-9012-3456-789012345678",
    "cwd": "/home/user/project",
    "cli_version": "0.114.0",
    "source": "cli",
    "model_provider": "openai",
    "git": {
      "commit_hash": "abc123def",
      "branch": "main",
      "repository_url": "https://github.com/user/project"
    }
  }
}
```

### event_msg（ユーザーメッセージ / タスクイベント）

```json
{
  "timestamp": "2026-03-17T08:30:05.000Z",
  "type": "event_msg",
  "payload": {
    "type": "user_message",
    "message": "ユーザーの入力テキスト"
  }
}
```

### response_item（モデル応答）

```json
{
  "timestamp": "2026-03-17T08:30:10.000Z",
  "type": "response_item",
  "payload": {
    "type": "message",
    "role": "assistant",
    "content": [
      {"type": "output_text", "text": "アシスタントの応答"}
    ]
  }
}
```

### response_item（シェルコマンド実行）

```json
{
  "timestamp": "2026-03-17T08:30:18.000Z",
  "type": "response_item",
  "payload": {
    "type": "local_shell_call",
    "action": {
      "command": ["ls", "-la"],
      "cwd": "/home/user/project"
    }
  }
}
```

### response_item（コマンド出力）

```json
{
  "timestamp": "2026-03-17T08:30:20.000Z",
  "type": "response_item",
  "payload": {
    "type": "function_call_output",
    "output": "total 32\ndrwxr-xr-x  5 user user 4096 ..."
  }
}
```

### エントリ種別

| `type` | `payload.type` | 説明 |
|--------|---------------|------|
| `session_meta` | — | セッションメタデータ（cwd、モデル、git情報） |
| `event_msg` | `user_message` | ユーザー入力メッセージ |
| `event_msg` | `task_complete` | タスク完了イベント |
| `response_item` | `message` | モデルのテキスト応答 |
| `response_item` | `local_shell_call` | シェルコマンド実行 |
| `response_item` | `function_call_output` | コマンド/関数の出力 |

### session_meta payload のフィールド

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `id` | string | セッション UUID |
| `cwd` | string | プロジェクトディレクトリ |
| `cli_version` | string | Codex CLI バージョン |
| `source` | string | ソース種別（例: `"cli"`） |
| `model_provider` | string | モデルプロバイダー（例: `"openai"`） |
| `git.commit_hash` | string | 現在の git コミットハッシュ |
| `git.branch` | string | 現在の git ブランチ |
| `git.repository_url` | string | Git リポジトリ URL |

## 取得方法

```python
import json
from pathlib import Path

codex_home = Path.home() / ".codex"
sessions_dir = codex_home / "sessions"

for rollout_path in sorted(sessions_dir.glob("rollout-*.jsonl"),
                           key=lambda p: p.stat().st_mtime, reverse=True)[:50]:
    cwd = ""
    with open(rollout_path, "r", encoding="utf-8") as f:
        for line in f:
            entry = json.loads(line.strip())
            entry_type = entry.get("type")
            payload = entry.get("payload", {})

            # session_meta からプロジェクト情報を取得
            if entry_type == "session_meta":
                cwd = payload.get("cwd", "")
                continue

            # event_msg からユーザーメッセージを抽出
            if entry_type == "event_msg" and payload.get("type") == "user_message":
                text = payload.get("message", "").strip()
                if text:
                    print(f"[{cwd}] {text}")

            # response_item からアシスタント応答を抽出
            if entry_type == "response_item" and payload.get("type") == "message":
                for part in payload.get("content", []):
                    if isinstance(part, dict) and part.get("type") == "output_text":
                        print(f"  → {part.get('text', '')[:100]}")
```

## 注意事項

- セッションファイルは `sessions/` 直下にフラットに保存（日付別サブディレクトリなし）
- プロジェクト情報は `session_meta` の `payload.cwd` フィールドから取得
- JSON は `type`/`payload` 構造を使用（CamelCase トップレベルキーではない）
- コンテンツ種別: `output_text`（応答）、`local_shell_call`（コマンド）、`function_call_output`（結果）
- セッションメタデータに git 情報（コミット、ブランチ、リポジトリ URL）が含まれる
