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
├── settings.json                          # グローバル設定
├── settings.local.json                    # ローカル設定オーバーライド
├── projects/
│   └── {encoded-path}/                    # パス区切り(\, /, :) → - に変換
│       ├── {session-uuid}.jsonl           # セッション別の完全な会話ログ
│       └── {session-uuid}/
│           └── subagents/                 # サブエージェントのログ
├── backups/                               # セッションバックアップ
├── cache/                                 # キャッシュデータ
├── debug/                                 # デバッグログ
├── file-history/                          # セッション別ファイル編集履歴
├── plans/                                 # プランドキュメント (*.md)
├── session-env/                           # セッション環境スナップショット
├── shell-snapshots/                       # シェル状態スナップショット
├── tasks/                                 # タスクトラッキングデータ
├── telemetry/                             # テレメトリデータ
└── todos/                                 # Todoアイテム
```

## データ形式

### history.jsonl（CLI使用時のインデックス）

1行1JSONオブジェクト:

```json
{
  "display": "ユーザーの入力テキスト",
  "pastedContents": {},
  "timestamp": 1759115795246,
  "project": "/home/user/project",
  "sessionId": "uuid-string"
}
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `display` | string | ユーザーが入力したプロンプト |
| `timestamp` | number | Unix epoch ミリ秒 |
| `project` | string | プロジェクトの絶対パス |
| `sessionId` | string | セッションUUID |
| `pastedContents` | object | ペーストされたコンテンツ（数値IDでキー付き） |

### プロジェクト別セッション JSONL（VS Code拡張含む）

1行1JSONオブジェクト。各行は `type` フィールドでエントリ種別を示す。

#### ユーザーメッセージ (`type: "user"`)

```json
{
  "type": "user",
  "uuid": "a08c057d-cf4e-44d3-afad-5795bbc7539f",
  "parentUuid": null,
  "sessionId": "20223f6f-23fc-4691-a054-6146beec7770",
  "version": "2.1.58",
  "cwd": "/path/to/project",
  "gitBranch": "main",
  "userType": "external",
  "permissionMode": "default",
  "isSidechain": false,
  "timestamp": "2026-03-17T00:25:44.657Z",
  "message": {
    "role": "user",
    "content": "ユーザーのプロンプト"
  },
  "todos": []
}
```

#### アシスタントメッセージ (`type: "assistant"`)

```json
{
  "type": "assistant",
  "uuid": "d09ed28e-1f0d-4849-8837-6bbba1bde931",
  "parentUuid": "a08c057d-cf4e-44d3-afad-5795bbc7539f",
  "sessionId": "20223f6f-23fc-4691-a054-6146beec7770",
  "version": "2.1.58",
  "cwd": "/path/to/project",
  "gitBranch": "main",
  "isSidechain": false,
  "timestamp": "2026-03-17T00:25:50.123Z",
  "message": {
    "model": "claude-opus-4-6",
    "id": "msg_01VyfRCVSGXTj8HSxTXTSXVt",
    "type": "message",
    "role": "assistant",
    "content": [
      {"type": "thinking", "thinking": "..."},
      {"type": "text", "text": "応答テキスト"},
      {"type": "tool_use", "id": "toolu_...", "name": "Bash", "input": {}}
    ],
    "usage": {
      "input_tokens": 1000,
      "output_tokens": 500,
      "cache_creation_input_tokens": 200,
      "cache_read_input_tokens": 800
    },
    "stop_reason": "end_turn"
  },
  "requestId": "req_..."
}
```

### セッション JSONL の共通フィールド

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `type` | string | エントリ種別（下記参照） |
| `uuid` | string | メッセージの一意ID |
| `parentUuid` | string/null | 親メッセージのUUID（会話ツリー） |
| `sessionId` | string | セッションUUID |
| `version` | string | Claude Code バージョン |
| `cwd` | string | 作業ディレクトリ |
| `gitBranch` | string | 現在の git ブランチ |
| `timestamp` | string | ISO 8601 |
| `isSidechain` | boolean | サブエージェントメッセージの場合 `true` |
| `userType` | string | 例: `"external"` |
| `permissionMode` | string | 例: `"default"` |
| `isMeta` | boolean | `true` の場合はシステムメッセージ（スキップ対象） |

### エントリ種別

| Type | 説明 |
|------|------|
| `user` | ユーザー入力メッセージ |
| `assistant` | モデル応答（ツール呼び出し、思考、使用量を含む） |
| `progress` | フック/ツール実行の進捗 |
| `system` | システムメッセージ（例: `turn_duration`） |
| `file-history-snapshot` | ファイルバージョン追跡スナップショット |
| `queue-operation` | キュー管理操作 |

### content フィールドの形式

`message.content` フィールドは以下のいずれか:
- **文字列**: シンプルなテキストメッセージ（例: `"content": "こんにちは"`）
- **配列**: 複数ブロックを含む構造化コンテンツ:
  - `{"type": "text", "text": "..."}` — テキスト応答
  - `{"type": "thinking", "thinking": "..."}` — 推論ブロック
  - `{"type": "tool_use", "id": "...", "name": "...", "input": {...}}` — ツール呼び出し
  - `{"type": "tool_result", "tool_use_id": "...", "content": "..."}` — ツール実行結果

### サブエージェントメッセージ

サブエージェントメッセージには追加フィールドが含まれる:

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `agentId` | string | エージェント識別子 |
| `slug` | string | 人間が読めるエージェント名 |
| `isSidechain` | boolean | サブエージェントでは常に `true` |

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
                content = entry.get("message", {}).get("content", "")
                # content は文字列またはコンテンツブロックの配列
                if isinstance(content, str):
                    text = content
                elif isinstance(content, list):
                    text = " ".join(
                        p.get("text", "") for p in content
                        if isinstance(p, dict) and p.get("type") == "text"
                    )
```

### 除外すべきコンテンツ

- `isMeta: true` のシステムメッセージ
- `type` が `"user"` / `"assistant"` 以外のエントリ（`progress`, `system`, `file-history-snapshot` 等）
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
- セッション JSONL にはユーザー/アシスタント以外にも複数のエントリ種別が含まれる（progress, system, file-history-snapshot 等）
- サブエージェントのログは `{session-uuid}/subagents/` 配下に別ファイルとして保存
- `message.content` フィールドはプレーン文字列またはコンテンツブロックの配列のいずれか
- アシスタントメッセージにはトークン使用量やモデル情報が含まれる
