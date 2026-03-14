# Claude Code（CLI / VS Code拡張）

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI / VS Code拡張 |
| ログ形式 | JSONL |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `C:\Users\<user>\.claude\` |
| macOS | `~/.claude/` |
| Linux | `~/.claude/` |

環境変数 `CLAUDE_CONFIG_DIR` が設定されている場合はそのパスを優先。

## ファイル構造

```
~/.claude/
├── history.jsonl                          # 全プロジェクトの軽量インデックス（CLI使用時）
└── projects/
    └── {encoded-path}/                    # パス区切り(\, /, :) → - に変換
        ├── {session-uuid}.jsonl           # セッション別の完全な会話ログ
        └── {session-uuid}/
            └── subagents/                 # サブエージェントのログ
```

## データ形式

### history.jsonl（CLI使用時のインデックス）

1行1JSONオブジェクト:

```json
{
  "display": "ユーザーの入力テキスト",
  "pastedContents": {},
  "timestamp": 1759115795246,
  "project": "D:\\path\\to\\project",
  "sessionId": "uuid-string"
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `display` | string | ユーザーが入力したプロンプト |
| `timestamp` | number | Unix epoch ミリ秒 |
| `project` | string | プロジェクトの絶対パス |
| `sessionId` | string | セッションUUID |
| `pastedContents` | object | ペーストされたコンテンツ |

### プロジェクト別セッション JSONL（VS Code拡張含む）

1行1JSONオブジェクト。ユーザーメッセージは `type: "user"`:

```json
{
  "type": "user",
  "message": {
    "role": "user",
    "content": [
      {"type": "text", "text": "プロンプト本文"}
    ]
  },
  "timestamp": "2026-03-12T05:31:13.875Z",
  "cwd": "/path/to/project"
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `type` | string | `"user"` / `"assistant"` 等 |
| `message.content` | array | コンテンツブロックの配列 |
| `timestamp` | string | ISO 8601 |
| `isMeta` | boolean | `true` の場合はシステムメッセージ（スキップ対象） |

## 取得方法

2ソース方式で取得する。VS Code拡張使用時は `history.jsonl` に記録されないため、両方の読み取りが必要。

### ソース1: history.jsonl

```python
import json
from pathlib import Path

claude_dir = Path.home() / ".claude"
history_path = claude_dir / "history.jsonl"

messages = []
with open(history_path, "r", encoding="utf-8") as f:
    for line in f:
        entry = json.loads(line.strip())
        display = entry.get("display", "").strip()
        if display and not display.startswith("/clear"):
            messages.append({
                "text": display,
                "timestamp": entry.get("timestamp"),
                "project": entry.get("project", ""),
            })
```

### ソース2: プロジェクト別セッション JSONL

```python
projects_dir = claude_dir / "projects"
for project_dir in projects_dir.iterdir():
    for session_file in project_dir.glob("*.jsonl"):
        with open(session_file, "r", encoding="utf-8") as f:
            for line in f:
                entry = json.loads(line.strip())
                if entry.get("type") != "user":
                    continue
                if entry.get("isMeta"):
                    continue
                content = entry.get("message", {}).get("content", [])
                # content からテキスト部分を抽出
```

### 除外すべきコンテンツ

- `isMeta: true` のシステムメッセージ
- `<ide_opened_file>`, `<local-command-stdout>` 等のシステムタグで始まるテキスト
- `/clear`, `/help` 等のスラッシュコマンド

## パスエンコード規則

プロジェクトディレクトリ名は、元のパスの `\`, `/`, `:` を `-` に変換:

```
C:\Users\user\project → c--Users-user-project
/home/user/project    → -home-user-project
```

## 注意事項

- CLI と VS Code拡張 でログの保存先が異なる
- `history.jsonl` はCLI使用時のみ記録される
- セッションJSONLにはアシスタントの応答も含まれる（`type: "assistant"`）
- サブエージェントのログは `{session-uuid}/subagents/` 配下に別ファイルとして保存
