# Continue

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code / JetBrains 拡張（OSS） |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.continue\` |
| macOS | `~/.continue/` |
| Linux | `~/.continue/` |

## ファイル構造

```
~/.continue/
├── config.json                    # 設定ファイル
├── config.yaml                    # 設定ファイル（YAML形式）
├── sessions/
│   └── {session-id}.json          # セッション別の会話ログ
├── dev_data/
│   └── *.jsonl                    # 開発用データログ
└── index/                         # コードインデックス
```

## データ形式

### sessions/{session-id}.json

```json
{
  "sessionId": "uuid-string",
  "title": "セッションタイトル",
  "dateCreated": "2025-06-15T10:30:00.000Z",
  "history": [
    {
      "message": {
        "role": "user",
        "content": "ユーザーのプロンプト"
      },
      "editorState": {},
      "contextItems": []
    },
    {
      "message": {
        "role": "assistant",
        "content": "アシスタントの応答"
      }
    }
  ]
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `sessionId` | string | セッションUUID |
| `title` | string | セッションタイトル（自動生成） |
| `dateCreated` | string | ISO 8601 |
| `history[].message.role` | string | `"user"` / `"assistant"` |
| `history[].message.content` | string | メッセージ本文 |
| `history[].contextItems` | array | 添付されたコンテキスト情報 |

### dev_data/*.jsonl

開発用のデータ記録（1行1JSON）。モデルの入出力やメタデータが記録される:

```json
{"timestamp": 1718441400000, "type": "chat", "prompt": "...", "completion": "..."}
```

## 取得方法

```python
import json
from pathlib import Path

continue_dir = Path.home() / ".continue"
sessions_dir = continue_dir / "sessions"

if sessions_dir.exists():
    for session_file in sorted(sessions_dir.glob("*.json"),
                                key=lambda p: p.stat().st_mtime, reverse=True):
        with open(session_file, "r", encoding="utf-8") as f:
            session = json.load(f)

        title = session.get("title", "untitled")
        for entry in session.get("history", []):
            message = entry.get("message", {})
            if message.get("role") == "user":
                text = message.get("content", "").strip()
                if text:
                    print(f"[{title}] {text}")
```

## 注意事項

- `~/.continue/` は中央集約型（全プロジェクトのログが1か所）
- `contextItems` にはファイル内容やコードスニペットが含まれる場合がある
- VS Code と JetBrains で同じディレクトリを使用
- `dev_data/` は Continue が匿名利用データ収集を有効にしている場合のみ生成
