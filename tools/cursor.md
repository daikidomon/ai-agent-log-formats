# Cursor

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI特化エディタ（VS Code フォーク） |
| ログ形式 | SQLite (`state.vscdb`) + 独自ストレージ |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Cursor\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Cursor/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Cursor/User/workspaceStorage/*/state.vscdb` |

### グローバルストレージ

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Cursor\User\globalStorage\` |
| macOS | `~/Library/Application Support/Cursor/User/globalStorage/` |
| Linux | `~/.config/Cursor/User/globalStorage/` |

## ファイル構造

```
Cursor/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb              # ワークスペース別の状態DB
├── globalStorage/
│   └── storage.json                 # グローバル設定
└── History/                         # ファイル編集履歴
```

## データ形式

VS Code フォークのため基本的に `state.vscdb`（SQLite）を使用。

### state.vscdb

```sql
-- チャット履歴に関連するキーを探索
SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%' OR key LIKE '%cursor%';
```

Cursor 固有の機能:
- **Composer**: マルチファイル編集セッション
- **Chat**: サイドパネルでのチャット
- **Cmd+K**: インラインエディット

各機能のログが異なるキーに保存される可能性がある。

### 想定されるキーパターン

| キー（推定） | 内容 |
|-------------|------|
| `composer.*` | Composer セッション |
| `cursorChat.*` | チャット履歴 |
| `aiChat.*` | AI チャットセッション |

## 取得方法

```python
import json
import sqlite3
from pathlib import Path

def get_cursor_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Cursor" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Cursor" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Cursor" / "User" / "workspaceStorage"

storage = get_cursor_storage()
for vscdb_path in storage.glob("*/state.vscdb"):
    conn = sqlite3.connect(str(vscdb_path))
    cursor = conn.cursor()

    # まずキー一覧を確認
    cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%'")
    keys = [row[0] for row in cursor.fetchall()]
    print(f"Keys found: {keys}")

    # 各キーの値を確認
    for key in keys:
        cursor.execute("SELECT value FROM ItemTable WHERE key = ?", (key,))
        row = cursor.fetchone()
        if row and row[0]:
            try:
                data = json.loads(row[0])
                print(f"[{key}] {json.dumps(data, ensure_ascii=False)[:200]}")
            except (json.JSONDecodeError, TypeError):
                print(f"[{key}] (non-JSON data, {len(row[0])} bytes)")

    conn.close()
```

## 注意事項

- VS Code フォークだが、チャット機能は Cursor 独自実装
- Composer / Chat / Cmd+K でデータの保存場所が異なる可能性
- バージョンアップで頻繁にストレージ構造が変わる
- `state.vscdb` 以外に独自のDBファイルが追加される場合がある
- **要実機調査**: 上記のキーパターンは推定。実環境での確認を推奨
