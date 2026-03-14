# OpenAI Codex（CLI）

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI |
| ログ形式 | JSONL（rollout セッションファイル） |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.codex\sessions\` |
| macOS | `~/.codex/sessions/` |
| Linux | `~/.codex/sessions/` |

環境変数 `CODEX_HOME` が設定されている場合は `$CODEX_HOME/sessions/` を使用。

## ファイル構造

```
~/.codex/
├── config.toml                                    # 設定ファイル
├── state-v5.db                                    # スレッドメタデータ（SQLite）
├── session_index.jsonl                            # セッションインデックス
└── sessions/
    └── YYYY/MM/DD/
        └── rollout-YYYY-MM-DDThh-mm-ss-<id>.jsonl # セッション別会話ログ
```

## データ形式

各行がJSONオブジェクト。行の種別はトップレベルのキーで判別する。

### SessionMeta（セッション開始時のメタ情報）

```json
{
  "timestamp": "2025-06-15T10:30:00.123Z",
  "SessionMeta": {
    "cwd": "/path/to/project",
    "model_provider": "openai"
  }
}
```

### ResponseItem（会話アイテム）

```json
{
  "timestamp": "2025-06-15T10:30:01.000Z",
  "ResponseItem": {
    "type": "message",
    "role": "user",
    "content": [
      {"type": "input_text", "text": "ユーザーの入力"}
    ]
  }
}
```

| キー | 説明 |
|------|------|
| `SessionMeta.cwd` | プロジェクトディレクトリ |
| `ResponseItem.type` | `"message"` = 会話メッセージ |
| `ResponseItem.role` | `"user"` / `"assistant"` |
| `ResponseItem.content[].type` | `"input_text"` or `"text"` |

## 取得方法

```python
import json
from pathlib import Path

codex_home = Path.home() / ".codex"
sessions_dir = codex_home / "sessions"

for rollout_path in sorted(sessions_dir.rglob("rollout-*.jsonl"),
                           key=lambda p: p.stat().st_mtime, reverse=True)[:50]:
    cwd = ""
    with open(rollout_path, "r", encoding="utf-8") as f:
        for line in f:
            entry = json.loads(line.strip())

            # SessionMeta からプロジェクト情報を取得
            session_meta = entry.get("SessionMeta")
            if session_meta:
                cwd = session_meta.get("cwd", "")
                continue

            # ResponseItem からユーザーメッセージを抽出
            response_item = entry.get("ResponseItem")
            if not response_item:
                continue
            if response_item.get("type") != "message" or response_item.get("role") != "user":
                continue

            content = response_item.get("content", [])
            texts = []
            for part in content:
                if isinstance(part, dict) and part.get("type") in ("input_text", "text"):
                    texts.append(part.get("text", ""))

            text = " ".join(texts).strip()
            if text:
                print(f"[{cwd}] {text}")
```

## 注意事項

- セッションはグローバル保存（プロジェクト別ディレクトリではない）
- プロジェクト情報は `SessionMeta` の `cwd` フィールドから取得
- Codex は Rust 製のため、JSONL のキー名が CamelCase（`SessionMeta`, `ResponseItem`）
- `state-v5.db` にもスレッドメタデータがあるが、会話内容は rollout JSONL に保存
