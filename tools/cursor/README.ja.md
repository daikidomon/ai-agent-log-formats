# Cursor

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI特化エディタ（VS Codeフォーク）+ CLIエージェント |
| ログ形式 | JSONL（トランスクリプト）+ SQLite（チャット状態） |
| ローカルログ | あり |

## 保存場所

### Cursor Agent CLI

| OS | パス |
|----|------|
| macOS | `~/.cursor/` |
| Linux | `~/.cursor/` |
| Windows | `%USERPROFILE%\.cursor\` |

環境変数 `CURSOR_CONFIG_DIR` または `CURSOR_DATA_DIR` でオーバーライド可能。

### Cursor GUI（エディタ）

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Cursor\User\workspaceStorage\*\state.vscdb` |
| macOS | `~/Library/Application Support/Cursor/User/workspaceStorage/*/state.vscdb` |
| Linux | `~/.config/Cursor/User/workspaceStorage/*/state.vscdb` |

## ファイル構造（CLI Agent — 実機検証済み）

```
~/.cursor/
├── cli-config.json                                        # CLI設定 + 認証情報
├── statsig-cache.json                                     # 機能フラグキャッシュ
├── chats/
│   └── <project-md5>/                                     # プロジェクト絶対パスのMD5ハッシュ
│       └── <session-uuid>/
│           └── store.db                                   # SQLite: チャット状態（Merkle treeブロブ）
├── projects/
│   └── <encoded-path>/                                    # パス区切り(/) → -（先頭の - なし）
│       ├── repo.json                                      # プロジェクトメタデータ (id)
│       ├── worker.log                                     # ワーカー/インデックスログ
│       ├── worker.sock                                    # ワーカーUnixソケット
│       ├── .workspace-trusted                             # 信頼マーカー
│       └── agent-transcripts/
│           └── <session-uuid>/
│               └── <session-uuid>.jsonl                   # 会話トランスクリプト (JSONL)
└── skills-cursor/                                         # 組み込みスキル
    ├── .cursor-managed-skills-manifest.json
    ├── create-rule/SKILL.md
    ├── create-skill/SKILL.md
    ├── create-subagent/SKILL.md
    ├── shell/SKILL.md
    └── update-cursor-settings/SKILL.md
```

### パスエンコード規則

#### プロジェクトパス (`projects/`)

パス区切り `/` を `-` に変換。**先頭の `-` なし**:

```
/home/ubuntu/my-project → home-ubuntu-my-project
```

#### チャットディレクトリ (`chats/`)

ディレクトリ名はプロジェクト絶対パスの **MD5 ハッシュ**:

```
/home/ubuntu/ai-agent-log-formats → cda24b2b5f52729dbc12e2aaada6b933
```

## データ形式（CLI Agent）

### エージェントトランスクリプト JSONL（主要な会話ログ）

保存先: `~/.cursor/projects/<encoded-path>/agent-transcripts/<session-uuid>/<session-uuid>.jsonl`

1行1JSONオブジェクト:

```json
{
  "role": "user",
  "message": {
    "content": [
      {
        "type": "text",
        "text": "<user_query>\nユーザーの入力テキスト\n</user_query>"
      }
    ]
  }
}
```

```json
{
  "role": "assistant",
  "message": {
    "content": [
      {
        "type": "text",
        "text": "アシスタントの応答"
      }
    ]
  }
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `role` | string | `"user"` または `"assistant"` |
| `message.content` | array | コンテンツブロックの配列 |
| `message.content[].type` | string | `"text"` |
| `message.content[].text` | string | メッセージ本文（ユーザー入力は `<user_query>` タグで囲まれる） |

### store.db（SQLite チャット状態）

保存先: `~/.cursor/chats/<project-md5>/<session-uuid>/store.db`

#### スキーマ

```sql
CREATE TABLE blobs (id TEXT PRIMARY KEY, data BLOB);
CREATE TABLE meta (key TEXT PRIMARY KEY, value TEXT);
```

#### meta テーブル

セッションメタデータを16進エンコードされた JSON として保存:

```json
{
  "agentId": "b172f4ff-d714-4b6d-9caf-a3f2a12a0188",
  "latestRootBlobId": "5c5994a555062ffea...",
  "name": "New Agent",
  "mode": "default",
  "createdAt": 1773719337438
}
```

#### blobs テーブル

会話データを Merkle tree 構造で保存。Blob ID は SHA-256 ハッシュ。ブロブに含まれるもの:
- `providerOptions` と `requestId` 付きのユーザーメッセージ
- コンテンツ配列付きのアシスタント応答
- メッセージシーケンスをリンクするツリー構造ノード

### cli-config.json

```json
{
  "version": 1,
  "authInfo": {
    "email": "user@example.com",
    "displayName": "User Name",
    "userId": 123456789,
    "authId": "github|user_..."
  },
  "permissions": {
    "allow": ["Shell(ls)"],
    "deny": []
  },
  "approvalMode": "allowlist",
  "sandbox": {
    "mode": "disabled",
    "networkAccess": "user_config_with_defaults"
  },
  "editor": {
    "vimMode": false
  },
  "attribution": {
    "attributeCommitsToAgent": true,
    "attributePRsToAgent": true
  }
}
```

## 取得方法（CLI Agent）

```python
import json
from pathlib import Path

cursor_dir = Path.home() / ".cursor"

# エージェントトランスクリプト (JSONL) の読み取り
projects_dir = cursor_dir / "projects"
for project_dir in projects_dir.iterdir():
    transcripts_dir = project_dir / "agent-transcripts"
    if not transcripts_dir.exists():
        continue
    for session_dir in transcripts_dir.iterdir():
        for jsonl_file in session_dir.glob("*.jsonl"):
            with open(jsonl_file, "r", encoding="utf-8") as f:
                for line in f:
                    entry = json.loads(line.strip())
                    if entry.get("role") == "user":
                        for part in entry.get("message", {}).get("content", []):
                            text = part.get("text", "")
                            # <user_query> ラッパーを除去
                            text = text.replace("<user_query>\n", "").replace("\n</user_query>", "")
                            print(text)
```

## データ形式（GUI エディタ）

Cursor GUI エディタ（VS Codeフォーク）はワークスペースレベルのストレージに `state.vscdb`（SQLite）を使用。

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
    cursor.execute("SELECT key FROM ItemTable WHERE key LIKE '%chat%' OR key LIKE '%composer%'")
    keys = [row[0] for row in cursor.fetchall()]
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

## 環境変数

| 変数 | 説明 |
|------|------|
| `CURSOR_API_KEY` | API 認証キー |
| `CURSOR_CONFIG_DIR` | 設定ディレクトリのオーバーライド |
| `CURSOR_DATA_DIR` | データディレクトリのオーバーライド |
| `CURSOR_WORKTREES_ROOT` | ワークツリーのルートディレクトリ |
| `AGENT_CLI_LOG_PATH` | デバッグログの出力パス |

## 注意事項

- Cursor Agent CLI は GUI エディタとは別の場所 (`~/.cursor/`) にデータを保存
- CLI トランスクリプトはプレーン JSONL（パースが容易）; GUI は `state.vscdb`（SQLite）
- チャットディレクトリはプロジェクトパスの **MD5 ハッシュ**; プロジェクトディレクトリは **ダッシュエンコード**されたパスを使用
- `store.db` は会話状態に Merkle tree ブロブ構造を使用
- トランスクリプト内のユーザーメッセージは `<user_query>` タグで囲まれる
- GUI エディタの `state.vscdb` のキーパターンはバージョンにより変動する可能性がある
- スキルは `skills-cursor/` 配下のマネージド SKILL.md ファイルとして保存
