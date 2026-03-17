# Google Antigravity

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI IDE（VS Codeフォーク）+ CLIチャット |
| ログ形式 | Protobuf（バイナリ）+ SQLite + テキスト |
| ローカルログ | あり |

## 保存場所

### エージェントデータ（Gemini）

| OS | パス |
|----|------|
| macOS | `~/.gemini/antigravity/` |
| Linux | `~/.gemini/antigravity/` |
| Windows | `%USERPROFILE%\.gemini\antigravity\` |

### IDE状態（VS Codeベース）

| OS | パス |
|----|------|
| macOS | `~/Library/Application Support/Antigravity/User/globalStorage/state.vscdb` |
| Linux | `~/.config/Antigravity/User/globalStorage/state.vscdb` |
| Windows | `%APPDATA%\Antigravity\User\globalStorage\state.vscdb` |

### 拡張機能 & CLI設定

| OS | パス |
|----|------|
| macOS | `~/.antigravity/` |
| Linux | `~/.antigravity/` |
| Windows | `%USERPROFILE%\.antigravity\` |

## ファイル構造（実機検証済み）

```
~/.gemini/
├── GEMINI.md                                     # グローバルメモリ（プロジェクト横断）
├── projects.json                                 # プロジェクトレジストリ
└── antigravity/
    ├── installation_id                           # UUID
    ├── user_settings.pb                          # ユーザー設定（protobufバイナリ）
    ├── mcp_config.json                           # MCPサーバー設定
    ├── browserAllowlist.txt                      # ブラウザツール許可リスト
    ├── brain/                                    # 会話毎のエージェント成果物
    │   └── <cascade-id>/
    │       ├── <artifact-files>                  # ユーザー向け成果物（コード、計画）
    │       └── .system_generated/
    │           └── logs/
    │               ├── overview.txt              # 会話概要
    │               └── <section-name>.txt        # ログセクション
    ├── knowledge/                                # ナレッジアイテム（永続記憶）
    ├── context_state/                            # 会話横断コンテキスト
    └── global_workflows/                         # グローバルワークフロー定義

~/.config/Antigravity/                            # IDE状態（Linux）
├── User/
│   ├── settings.json                             # エディタ設定
│   ├── globalStorage/
│   │   ├── state.vscdb                           # SQLite: アプリ状態 + 軌跡サマリ
│   │   └── storage.json                          # マイグレーション状態 + 設定
│   └── workspaceStorage/
│       └── <workspace-id>/
│           └── state.vscdb                       # SQLite: ワークスペース毎の状態
└── logs/                                         # アプリケーションログ

~/.antigravity/                                   # CLI & 拡張機能
├── argv.json                                     # CLI引数
└── extensions/
    └── extensions.json                           # インストール済み拡張機能

<project-root>/
├── .agent/
│   ├── rules/                                    # プロジェクトスコープのエージェントルール
│   └── workflows/                                # プロジェクトスコープのワークフロー定義
└── .gemini/
    └── GEMINI.md                                 # プロジェクトレベルのエージェント指示
```

## データ形式

### 会話ストレージアーキテクチャ

会話（軌跡）データは多層アーキテクチャを使用:

1. **言語サーバー（gRPC）** — 「Cascade」エンジンが軌跡状態をメモリ内でgRPC RPCを通じて管理
2. **SQLite（state.vscdb）** — 軌跡サマリとアプリ状態をbase64エンコードprotobufとして永続化
3. **ファイルシステム（brain/）** — エージェント成果物と会話ログをプレーンテキスト/画像として保存

### state.vscdb（SQLite — 主要永続化ストア）

スキーマ: VS Code標準の `ItemTable`（key TEXT, value TEXT）。

会話データの主要エントリ:

| キー | 形式 | 説明 |
|------|------|------|
| `jetskiStateSync.agentManagerInitState` | Base64 protobuf | マスター状態: 軌跡サマリ、ウィンドウ位置、OAuthトークン |
| `antigravityUnifiedStateSync.trajectorySummaries` | Base64 protobuf | 会話一覧とメタデータ |
| `antigravityUnifiedStateSync.agentPreferences` | Base64 protobuf | プランニングモード、ファイルアクセスポリシー、ターミナル実行 |
| `antigravityUnifiedStateSync.editorPreferences` | Base64 protobuf | エディタ連携設定 |
| `chat.ChatSessionStore.index` | JSON | VS Codeチャットセッションインデックス |

### Protobufスキーマ（ソースコードから再構成）

#### CascadeTrajectorySummary（会話サマリ）

```
CascadeTrajectorySummary {
  string summary = 1;
  uint32 step_count = 2;
  Timestamp last_modified_time = 3;
  string trajectory_id = 4;
  CascadeRunStatus status = 5;       // IDLE, RUNNING, CANCELING, BUSY
  Timestamp created_time = 7;
  repeated WaitingStep waiting_steps = 8;
  repeated Workspace workspaces = 9;
  Timestamp last_user_input_time = 10;
  uint32 last_user_input_step_index = 11;
  ConversationAnnotations annotations = 14;
}
```

#### ChatMessage（個別メッセージ）

```
ChatMessage {
  string message_id = 1;
  MessageSource source = 2;          // USER, SYSTEM 等
  Timestamp timestamp = 3;
  string conversation_id = 4;
  oneof content {
    ChatMessageIntent intent = 5;    // ユーザーの意図
    ChatMessageAction action = 6;    // AIのアクション（編集、検索、汎用）
    ChatMessageError error = 7;
    ChatMessageStatus status = 8;
  }
  bool in_progress = 9;
  ChatMessageRequest request = 10;
  bool redact = 11;
}
```

#### BrainEntry（Brain ファイルエントリ）

```
BrainEntry {
  string id = 1;
  BrainEntryType type = 2;           // UNSPECIFIED, PLAN, TASK
  string content = 3;
}
```

### Brain ディレクトリ（会話成果物）

保存先: `~/.gemini/antigravity/brain/<cascade-id>/`

各会話（「cascade」）は独自のディレクトリを持ち、以下を格納:
- ユーザー向け成果物（生成コード、計画）
- `.system_generated/logs/overview.txt` — 会話概要
- `.system_generated/logs/<section-name>.txt` — 詳細ログセクション
- スクリーンショット/画像（`.jpeg`, `.png`, `.gif`, `.webp`）

### projects.json（プロジェクトレジストリ）

```json
{
  "projects": {
    "/home/ubuntu": "ubuntu",
    "/home/ubuntu/ai-agent-log-formats": "ai-agent-log-formats"
  }
}
```

### gRPC API（言語サーバー）

会話データの主要RPC:

| RPC | 説明 |
|-----|------|
| `GetCascadeTrajectory` | 完全な会話軌跡の取得 |
| `GetCascadeTrajectorySteps` | ステップのページネーション付き取得 |
| `GetAllCascadeTrajectories` | 全軌跡サマリの一覧 |
| `ConvertTrajectoryToMarkdown` | 会話のMarkdownエクスポート |
| `DeleteCascadeTrajectory` | 会話の削除 |

## CLI使用方法

```bash
# チャットセッションを開始（GUIディスプレイが必要）
antigravity chat "your prompt"
antigravity chat -m ask "question"      # Askモード
antigravity chat -m edit "instruction"  # Editモード
antigravity chat -m agent "task"        # Agentモード（デフォルト）

# ファイルコンテキストの追加
antigravity chat -a file.py "explain this"
```

## 取得方法

```python
import json
import sqlite3
from pathlib import Path
import platform
import base64

# Brain ディレクトリのログ（テキスト、人間可読）
brain_dir = Path.home() / ".gemini" / "antigravity" / "brain"
if brain_dir.exists():
    for cascade_dir in brain_dir.iterdir():
        if not cascade_dir.is_dir():
            continue
        logs_dir = cascade_dir / ".system_generated" / "logs"
        if not logs_dir.exists():
            continue
        # 概要
        overview = logs_dir / "overview.txt"
        if overview.exists():
            print(f"[{cascade_dir.name[:12]}] {overview.read_text()[:200]}")
        # セクションログ
        for log_file in sorted(logs_dir.glob("*.txt")):
            if log_file.name == "overview.txt":
                continue
            text = log_file.read_text(encoding="utf-8").strip()
            if text:
                print(f"  [{log_file.stem}] {text[:200]}")

# state.vscdb の軌跡サマリ（protobuf、デコード必要）
system = platform.system()
if system == "Darwin":
    vscdb = Path.home() / "Library" / "Application Support" / "Antigravity" / "User" / "globalStorage" / "state.vscdb"
elif system == "Windows":
    import os
    vscdb = Path(os.environ["APPDATA"]) / "Antigravity" / "User" / "globalStorage" / "state.vscdb"
else:
    vscdb = Path.home() / ".config" / "Antigravity" / "User" / "globalStorage" / "state.vscdb"

if vscdb.exists():
    conn = sqlite3.connect(str(vscdb))
    cursor = conn.cursor()
    # チャットセッションインデックス（JSON）
    cursor.execute("SELECT value FROM ItemTable WHERE key = 'chat.ChatSessionStore.index'")
    row = cursor.fetchone()
    if row:
        data = json.loads(row[0])
        print(f"チャットセッション数: {len(data.get('entries', {}))}")
    # 軌跡状態（base64 protobuf — デコードにはprotobufライブラリが必要）
    cursor.execute("SELECT value FROM ItemTable WHERE key = 'jetskiStateSync.agentManagerInitState'")
    row = cursor.fetchone()
    if row:
        raw = base64.b64decode(row[0])
        print(f"エージェントマネージャー状態: {len(raw)} bytes (protobuf)")
    conn.close()
```

## 環境変数

| 変数 | 説明 |
|------|------|
| `ANTIGRAVITY_USER_DATA_DIR` | ユーザーデータディレクトリのオーバーライド |
| `GOOGLE_API_KEY` | Geminiモデル用APIキー |

## 注意事項

- 内部コードネームは「Jetski」（ソースコードや状態キー `jetskiStateSync` に表れる）
- Codeium/Windsurf技術基盤上に構築（protobufスキーマは `exa.*` 名前空間を使用）
- 会話軌跡は主に**言語サーバーのメモリ内**で管理され、gRPCで取得
- `state.vscdb` の永続化ストレージは **base64エンコードprotobufバイナリ** — 人間可読JSONではない
- `brain/` ディレクトリが唯一の人間可読な会話データ（テキストログと成果物）を格納
- ナレッジアイテムは会話終了時に「ナレッジサブエージェント」が抽出
- 既知のバグ: `brain/` ディレクトリが空のまま、ナレッジアイテムが生成されないことがある
- `.pb` ファイルは Protocol Buffers バイナリ形式 — プレーンテキストとして読み取り不可
- チャットセッションインデックスはVS Code標準の `chat.ChatSessionStore.index` キーパターンを使用
- バージョン 1.104.0 確認（ユーザー向けバージョン 1.13.3）
- 認証にGoogleアカウントが必要（OAuthフロー）
