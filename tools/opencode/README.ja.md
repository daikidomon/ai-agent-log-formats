# OpenCode

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | TUI コーディングエージェント |
| ログ形式 | SQLite |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| macOS | `~/.local/share/opencode/` |
| Linux | `~/.local/share/opencode/` |
| Windows | `%USERPROFILE%\.local\share\opencode\` |

環境変数 `XDG_DATA_HOME` が設定されている場合は `$XDG_DATA_HOME/opencode/` を使用。

### 設定ファイルの検索順

```
~/.config/opencode/config.json
~/.config/opencode/opencode.json
~/.config/opencode/opencode.jsonc
~/.opencode/opencode.jsonc
~/.opencode/opencode.json
```

## ファイル構造（実機検証済み）

```
~/.local/share/opencode/
├── bin/                                  # インストール済みバイナリ
│   └── opencode
├── log/                                  # デバッグログ（タイムスタンプ付き）
│   └── YYYY-MM-DDTHHMMSS.log
├── opencode.db                           # メインSQLiteデータベース
├── snapshot/                             # 差分スナップショット用のbare gitリポ
│   └── <project-id>/                    # プロジェクト毎のbare gitリポジトリ
└── storage/
    ├── migration                         # マイグレーションマーカー
    └── session_diff/                     # セッション差分JSONファイル
        └── <session-id>.json            # セッション毎の差分データ
```

## データ形式（実機検証済み）

SQLiteデータベース（`opencode.db`）。Drizzle ORMで管理。

### テーブル一覧

| テーブル | 説明 |
|---------|------|
| `project` | プロジェクト情報（ワークツリーパス、VCS種別等） |
| `session` | セッションメタデータ（タイトル、スラッグ、バージョン、要約、権限） |
| `message` | メッセージ（ロールとモデル情報）。`data` カラムにJSON |
| `part` | メッセージの各パート（テキスト、ツール、ステップ）。`data` カラムにJSON |
| `todo` | セッションスコープのTODO項目 |
| `workspace` | ワークスペースブランチと種別 |
| `session_share` | セッション共有メタデータ（URL、シークレット） |
| `account` | OAuthアカウント認証情報 |
| `account_state` | アクティブアカウント選択 |
| `control_account` | コントロールプレーンアカウント認証情報 |
| `permission` | プロジェクト毎の権限ルール |

### project テーブル

```sql
CREATE TABLE `project` (
  `id` text PRIMARY KEY,           -- SHA-1ハッシュ
  `worktree` text NOT NULL,        -- プロジェクト絶対パス
  `vcs` text,                      -- "git" または null
  `name` text,
  `icon_url` text,
  `icon_color` text,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `time_initialized` integer,
  `sandboxes` text NOT NULL,       -- JSON配列
  `commands` text
);
```

例:

```json
{
  "id": "5d04e1f82490ad90bbb09a6241051203cd6cbb47",
  "worktree": "/home/ubuntu/ai-agent-log-formats",
  "vcs": "git",
  "sandboxes": "[]"
}
```

### session テーブル

```sql
CREATE TABLE `session` (
  `id` text PRIMARY KEY,              -- プレフィックス: ses_...
  `project_id` text NOT NULL,
  `parent_id` text,                   -- サブエージェントセッション用
  `slug` text NOT NULL,               -- 人間可読名（例: "brave-falcon"）
  `directory` text NOT NULL,          -- 作業ディレクトリパス
  `title` text NOT NULL,              -- 自動生成タイトル
  `version` text NOT NULL,            -- OpenCodeバージョン（例: "1.2.27"）
  `share_url` text,
  `summary_additions` integer,        -- コード変更サマリ
  `summary_deletions` integer,
  `summary_files` integer,
  `summary_diffs` text,
  `revert` text,
  `permission` text,                  -- JSON: セッション権限ルール
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `time_compacting` integer,
  `time_archived` integer,
  `workspace_id` text,
  FOREIGN KEY (`project_id`) REFERENCES `project`(`id`) ON DELETE CASCADE
);
```

例:

```json
{
  "id": "ses_305eeae76ffeZFzOzz903nD0J0",
  "project_id": "5d04e1f82490ad90bbb09a6241051203cd6cbb47",
  "slug": "shiny-wolf",
  "directory": "/home/ubuntu/ai-agent-log-formats",
  "title": "Greeting in conversation setup",
  "version": "1.2.27",
  "permission": "[{\"permission\":\"question\",\"pattern\":\"*\",\"action\":\"deny\"}]"
}
```

### message テーブル

```sql
CREATE TABLE `message` (
  `id` text PRIMARY KEY,              -- プレフィックス: msg_...
  `session_id` text NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `data` text NOT NULL,               -- JSON文字列
  FOREIGN KEY (`session_id`) REFERENCES `session`(`id`) ON DELETE CASCADE
);
```

#### message.data（ユーザー）

```json
{
  "role": "user",
  "time": { "created": 1773721964971 },
  "summary": { "diffs": [] },
  "agent": "build",
  "model": {
    "providerID": "openai",
    "modelID": "gpt-4o-mini"
  }
}
```

#### message.data（アシスタント）

```json
{
  "role": "assistant",
  "time": {
    "created": 1773721965043,
    "completed": 1773721968543
  },
  "parentID": "msg_cfa1151ab001QbI6kqQG7fwKae",
  "modelID": "gpt-4o-mini",
  "providerID": "openai",
  "mode": "build",
  "agent": "build",
  "path": {
    "cwd": "/home/ubuntu/ai-agent-log-formats",
    "root": "/home/ubuntu/ai-agent-log-formats"
  },
  "cost": 0.0014427,
  "tokens": {
    "total": 9585,
    "input": 9574,
    "output": 11,
    "reasoning": 0,
    "cache": { "read": 0, "write": 0 }
  },
  "finish": "stop"
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `role` | string | `"user"` または `"assistant"` |
| `time.created` | integer | Unix epoch ミリ秒 |
| `time.completed` | integer | 完了時刻（アシスタントのみ） |
| `parentID` | string | 親メッセージID（アシスタントのみ） |
| `model.providerID` / `providerID` | string | プロバイダー名（例: `"openai"`, `"anthropic"`） |
| `model.modelID` / `modelID` | string | モデル名 |
| `agent` | string | エージェントモード（例: `"build"`） |
| `cost` | number | リクエストコスト（USD）（アシスタントのみ） |
| `tokens` | object | トークン使用量の内訳（アシスタントのみ） |
| `finish` | string | `"stop"` または `"tool-calls"`（アシスタントのみ） |

### part テーブル

```sql
CREATE TABLE `part` (
  `id` text PRIMARY KEY,              -- プレフィックス: prt_...
  `message_id` text NOT NULL,
  `session_id` text NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  `data` text NOT NULL,               -- JSON文字列
  FOREIGN KEY (`message_id`) REFERENCES `message`(`id`) ON DELETE CASCADE
);
```

#### パートタイプ

| タイプ | 説明 |
|--------|------|
| `text` | テキストコンテンツ（ユーザー入力またはアシスタント応答） |
| `step-start` | 推論ステップの開始（gitスナップショットハッシュ付き） |
| `step-finish` | 推論ステップの終了（コスト、トークン、理由付き） |
| `tool` | ツール呼び出し（入力、出力、実行メタデータ付き） |

#### part.data — text

```json
{
  "type": "text",
  "text": "Hello! How can I assist you today?",
  "time": { "start": 1773721968509, "end": 1773721968509 },
  "metadata": {
    "openai": { "itemId": "msg_008bb2297fce..." }
  }
}
```

#### part.data — step-start

```json
{
  "type": "step-start",
  "snapshot": "c432666404def571f8762ae88063a9e6ffb1de81"
}
```

#### part.data — step-finish

```json
{
  "type": "step-finish",
  "reason": "stop",
  "snapshot": "c432666404def571f8762ae88063a9e6ffb1de81",
  "cost": 0.0014427,
  "tokens": {
    "total": 9585,
    "input": 9574,
    "output": 11,
    "reasoning": 0,
    "cache": { "read": 0, "write": 0 }
  }
}
```

#### part.data — tool

```json
{
  "type": "tool",
  "callID": "call_01nYsvZD3mzrD1MnqlCWp2Jz",
  "tool": "bash",
  "state": {
    "status": "completed",
    "input": {
      "command": "ls",
      "description": "Lists files in current directory"
    },
    "output": "README.ja.md\nREADME.md\ntools\n",
    "title": "Lists files in current directory",
    "metadata": {
      "output": "README.ja.md\nREADME.md\ntools\n",
      "exit": 0,
      "description": "Lists files in current directory",
      "truncated": false
    },
    "time": { "start": 1773722148816, "end": 1773722148855 }
  },
  "metadata": {
    "openai": { "itemId": "fc_0447f6e49c79..." }
  }
}
```

### todo テーブル

```sql
CREATE TABLE `todo` (
  `session_id` text NOT NULL,
  `content` text NOT NULL,
  `status` text NOT NULL,
  `priority` text NOT NULL,
  `position` integer NOT NULL,
  `time_created` integer NOT NULL,
  `time_updated` integer NOT NULL,
  PRIMARY KEY(`session_id`, `position`),
  FOREIGN KEY (`session_id`) REFERENCES `session`(`id`) ON DELETE CASCADE
);
```

### workspace テーブル

```sql
CREATE TABLE `workspace` (
  `id` text PRIMARY KEY,
  `branch` text,
  `project_id` text NOT NULL,
  `type` text NOT NULL,
  `name` text,
  `directory` text,
  `extra` text,
  FOREIGN KEY (`project_id`) REFERENCES `project`(`id`) ON DELETE CASCADE
);
```

## IDフォーマット

| エンティティ | プレフィックス | 例 |
|-------------|--------------|-----|
| セッション | `ses_` | `ses_305eeae76ffeZFzOzz903nD0J0` |
| メッセージ | `msg_` | `msg_cfa1151ab001QbI6kqQG7fwKae` |
| パート | `prt_` | `prt_cfa1151ac001ncI58skuBtS8I8` |
| プロジェクト | （なし） | SHA-1ハッシュ |

## CLIツール

```bash
# 非対話セッションの実行
opencode run "your message"
opencode run -m openai/gpt-4o-mini "say hello"

# セッションをJSONでエクスポート
opencode export <session-id>

# セッションの一覧・削除
opencode session list
opencode session delete <session-id>

# 直接SQLクエリ
opencode db "SELECT * FROM session" --format json
opencode db path   # データベースパスを表示

# トークン使用量の統計
opencode stats

# 利用可能モデルの一覧
opencode models [provider]
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
        s.slug AS session_slug,
        s.title AS session_title,
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

    text = part_data.get("text", "").strip()
    project = Path(row["project_worktree"]).name if row["project_worktree"] else "unknown"
    if text:
        print(f"[{project}/{row['session_slug']}] {text}")

conn.close()
```

## 注意事項

- `session.parent_id IS NULL` で親セッションのみ対象（サブエージェントの子セッションを除外）
- タイムスタンプはUnix epochミリ秒
- プロジェクトIDはワークツリーパスのSHA-1ハッシュ
- セッションスラッグは「brave-falcon」のような2語の人間可読名
- `snapshot/` ディレクトリはステップ間の差分計算に使用するbare gitリポジトリを格納
- `storage/session_diff/` にはセッション毎の差分データがJSON配列として保存
- `log/` 内のデバッグログはタイムスタンプ付きテキストファイル
- スキーマはDrizzle ORMでバンドルマイグレーションとして管理
- `opencode export` コマンドでセッションデータの構造化JSON出力が可能
- `opencode db` コマンドで直接SQLクエリが実行可能
