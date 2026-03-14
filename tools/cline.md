# Cline

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code拡張 |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\saoudrizwan.claude-dev\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/saoudrizwan.claude-dev/` |
| Linux | `~/.config/Code/User/globalStorage/saoudrizwan.claude-dev/` |

## ファイル構造

```
saoudrizwan.claude-dev/
├── state/
│   └── taskHistory.json              # タスク履歴インデックス
└── tasks/
    └── {task-id}/
        ├── api_conversation_history.json  # API会話ログ（主要）
        ├── ui_messages.json               # UI表示メッセージ
        └── task_metadata.json             # タスクメタデータ
```

## データ形式

### api_conversation_history.json

```json
[
  {
    "role": "user",
    "content": [
      {
        "type": "text",
        "text": "ユーザーのプロンプト"
      }
    ]
  },
  {
    "role": "assistant",
    "content": [
      {
        "type": "text",
        "text": "Clineの応答..."
      }
    ]
  }
]
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `role` | string | `"user"` or `"assistant"` |
| `content` | array/string | コンテンツブロック配列またはテキスト |
| `content[].type` | string | `"text"`, `"image"` 等 |
| `content[].text` | string | テキスト本文 |

### ui_messages.json

UI側で表示されたメッセージ。`api_conversation_history.json` より人間に読みやすい形式だが、API送信内容とは異なる。

## 取得方法

```python
import json
from pathlib import Path

def get_cline_tasks_dir():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        base = Path(os.environ["APPDATA"])
    elif system == "Darwin":
        base = Path.home() / "Library" / "Application Support"
    else:
        base = Path.home() / ".config"
    return base / "Code" / "User" / "globalStorage" / "saoudrizwan.claude-dev" / "tasks"

tasks_dir = get_cline_tasks_dir()
for task_dir in sorted(tasks_dir.iterdir(), key=lambda p: p.stat().st_mtime, reverse=True):
    history_file = task_dir / "api_conversation_history.json"
    if not history_file.exists():
        continue

    with open(history_file, "r", encoding="utf-8") as f:
        data = json.load(f)

    for msg in data:
        if msg.get("role") == "user":
            content = msg.get("content", "")
            if isinstance(content, list):
                text = " ".join(c.get("text", "") for c in content if c.get("type") == "text")
            else:
                text = str(content)
            if text.strip():
                print(text.strip())
```

## 注意事項

- タスクIDは UUID 形式
- `api_conversation_history.json` にはツール呼び出しの結果等も含まれる（`content` にXMLタグ付きの長いテキスト）
- 個別のタイムスタンプはメッセージに含まれない（`task_metadata.json` またはファイル更新日時で代用）
- Roo Code, Kilo Code は同一構造（Clineフォーク）
