# GitHub Copilot Chat

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code拡張 |
| ログ形式 | SQLite (`state.vscdb`) |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Code/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Code/User/workspaceStorage/*/state.vscdb` |

## データ形式

SQLite データベース。チャット履歴は `ItemTable` テーブルの特定キーにJSON文字列として格納。

### テーブル構造

```sql
CREATE TABLE ItemTable (key TEXT UNIQUE ON CONFLICT REPLACE, value BLOB);
```

### 主要キー

| キー | 内容 |
|------|------|
| `memento/interactive-session` | プロンプト履歴（主要） |
| `interactive.sessions` | チャットセッションインデックス |
| `chat.ChatSessionStore.index` | 代替キー（バージョンにより異なる） |

### データ構造（`memento/interactive-session`）

```json
{
  "history": {
    "panel": [
      {
        "text": "ユーザーのプロンプト",
        "result": "Copilotの応答..."
      }
    ],
    "inline": [
      {
        "text": "インラインで入力したプロンプト"
      }
    ]
  }
}
```

## 取得方法

```python
import json
import sqlite3
from pathlib import Path

def get_workspace_storage():
    """OSに応じたworkspaceStorageパスを返す"""
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Code" / "User" / "workspaceStorage"

storage = get_workspace_storage()
for vscdb_path in storage.glob("*/state.vscdb"):
    conn = sqlite3.connect(str(vscdb_path))
    cursor = conn.cursor()

    cursor.execute(
        "SELECT value FROM ItemTable WHERE key = 'memento/interactive-session'"
    )
    row = cursor.fetchone()
    if row and row[0]:
        data = json.loads(row[0])
        history = data.get("history", {})
        for mode_key, entries in history.items():
            if isinstance(entries, list):
                for entry in entries:
                    text = entry.get("text", "").strip()
                    if text:
                        print(f"[{mode_key}] {text}")
    conn.close()
```

### sqlite3 コマンドでの確認

```bash
# 利用可能なキーの一覧
sqlite3 "<path>/state.vscdb" "SELECT key FROM ItemTable WHERE key LIKE '%chat%';"

# セッションインデックス
sqlite3 "<path>/state.vscdb" "SELECT value FROM ItemTable WHERE key = 'interactive.sessions';"
```

## 注意事項

- ワークスペースごとに別の `state.vscdb` が存在する
- チャットメッセージ自体にタイムスタンプが含まれない場合がある（ファイル更新日時で代用）
- `sqlite3` コマンドが必要
- キー名はCopilotのバージョンにより変更される場合がある
