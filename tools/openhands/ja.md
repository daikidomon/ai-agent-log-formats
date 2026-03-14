# OpenHands（旧 OpenDevin）

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | 自律エージェント（OSS） |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

```
workspace/
└── .openhands/
    └── sessions/
        └── {session-id}/
            ├── events.json        # イベントログ
            └── state.json         # セッション状態
```

または環境変数 `WORKSPACE_BASE` で指定したディレクトリ配下。

## データ形式

### events.json

イベント駆動型のログ。各イベントが1オブジェクト:

```json
[
  {
    "id": 1,
    "timestamp": "2025-06-15T10:30:00Z",
    "source": "user",
    "action": "message",
    "args": {
      "content": "ユーザーのプロンプト"
    }
  },
  {
    "id": 2,
    "timestamp": "2025-06-15T10:30:05Z",
    "source": "agent",
    "action": "run",
    "args": {
      "command": "ls -la"
    }
  },
  {
    "id": 3,
    "timestamp": "2025-06-15T10:30:06Z",
    "source": "agent",
    "observation": "CmdOutputObservation",
    "content": "..."
  }
]
```

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `source` | string | `"user"` / `"agent"` |
| `action` | string | アクション種別（`"message"`, `"run"`, `"edit"` 等） |
| `args.content` | string | ユーザーのメッセージ |
| `observation` | string | 実行結果の種別 |

## 取得方法

```python
import json
from pathlib import Path

sessions_dir = Path("workspace/.openhands/sessions")
if sessions_dir.exists():
    for session_dir in sessions_dir.iterdir():
        events_file = session_dir / "events.json"
        if not events_file.exists():
            continue

        with open(events_file, "r", encoding="utf-8") as f:
            events = json.load(f)

        for event in events:
            if event.get("source") == "user" and event.get("action") == "message":
                text = event.get("args", {}).get("content", "").strip()
                if text:
                    print(f"[{session_dir.name}] {text}")
```

## 注意事項

- Docker コンテナ内で実行される場合、ログはコンテナ内のパスに保存される
- `WORKSPACE_BASE` 環境変数でワークスペースの場所を変更可能
- イベントログにはエージェントのすべてのアクション（コマンド実行、ファイル編集等）が含まれる
- Web UI 経由で使用する場合は、サーバー側にもセッションデータが保存される
