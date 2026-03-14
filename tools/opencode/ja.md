# OpenCode

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | TUI エディタ |
| ログ形式 | SQLite |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.local\share\opencode\` |
| macOS | `~/.local/share/opencode/` |
| Linux | `~/.local/share/opencode/` |

環境変数 `XDG_DATA_HOME` が設定されている場合は `$XDG_DATA_HOME/opencode/` を使用。

## ファイル構造

```
~/.local/share/opencode/
├── opencode.db                # メインDB
└── opencode-<channel>.db     # チャンネル別DB（存在する場合）
```

## データ形式

SQLite データベース。主要テーブル:

| テーブル | 説明 |
|---------|------|
| `project` | プロジェクト情報（`worktree`, `name` 等） |
| `session` | セッション単位のメタデータ |
| `message` | メッセージ単位。`data` カラムにJSON |
| `part` | メッセージの各パート。`data` カラムにJSON |

### message.data の構造

```json
{
  "role": "user",
  "metadata": {}
}
```

### part.data の構造

```json
{
  "type": "text",
  "text": "ユーザーのプロンプト",
  "synthetic": false,
  "ignored": false
}
```

## 取得方法

```python
import json
import sqlite3
from pathlib import Path

db_path = Path.home() / ".local" / "share" / "opencode" / "opencode.db"
conn = sqlite3.connect(str(db_path))
conn.row_factory = sqlite3.Row
cursor = conn.cursor()

cursor.execute("""
    SELECT
        m.id AS message_id,
        m.time_created,
        m.data AS message_data,
        p.worktree AS project_worktree,
        s.parent_id AS session_parent_id,
        pt.data AS part_data
    FROM message m
    JOIN session s ON s.id = m.session_id
    JOIN project p ON p.id = s.project_id
    JOIN part pt ON pt.message_id = m.id
    WHERE s.parent_id IS NULL
    ORDER BY m.time_created ASC, pt.time_created ASC
""")

for row in cursor.fetchall():
    message_data = json.loads(row["message_data"])
    if message_data.get("role") != "user":
        continue

    part_data = json.loads(row["part_data"])
    if part_data.get("type") != "text":
        continue
    if part_data.get("synthetic") is True or part_data.get("ignored") is True:
        continue

    text = part_data.get("text", "").strip()
    project = Path(row["project_worktree"]).name if row["project_worktree"] else "unknown"
    if text:
        print(f"[{project}] {text}")

conn.close()
```

## 注意事項

- `session.parent_id IS NULL` で親セッションのみ対象（サブエージェントの子セッションを除外）
- `part.data.synthetic = true` はシステム生成パート（スキップ対象）
- `part.data.ignored = true` はユーザーが無視したパート（スキップ対象）
- タイムスタンプは `message.time_created`（Unix epoch ミリ秒）
- プロジェクト名は `project.worktree` の basename
- `opencode.db` が優先、なければ `opencode-*.db` を探索
