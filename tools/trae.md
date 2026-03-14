# Trae

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI IDE（ByteDance製、VS Code ベース） |
| ログ形式 | SQLite / JSON |
| ローカルログ | あり（推定） |

## 保存場所

VS Code ベースのため、同様のディレクトリ構造を使用（推定）:

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Trae\User\` |
| macOS | `~/Library/Application Support/Trae/User/` |
| Linux | `~/.config/Trae/User/` |

## ファイル構造（推定）

```
Trae/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## データ形式

VS Code フォークのため `state.vscdb`（SQLite）を基本とするが、Trae 独自のAI機能用に追加のストレージがある可能性。

## 取得方法

```python
import sqlite3
from pathlib import Path

def get_trae_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Trae" / "User" / "workspaceStorage"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Trae" / "User" / "workspaceStorage"
    else:
        return Path.home() / ".config" / "Trae" / "User" / "workspaceStorage"

storage = get_trae_storage()
if storage.exists():
    for vscdb_path in storage.glob("*/state.vscdb"):
        conn = sqlite3.connect(str(vscdb_path))
        cursor = conn.cursor()
        cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%ai%'")
        for row in cursor.fetchall():
            print(f"Key: {row[0]}")
        conn.close()
```

## 注意事項

- **要実機調査**: Trae のログ形式・保存場所は未確認
- VS Code ベースだがアプリ名が `Trae` のため、ディレクトリ名も異なる
- ByteDance アカウントと連携している場合、クラウド側にも保存される可能性
