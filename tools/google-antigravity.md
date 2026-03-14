# Google Antigravity

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI IDE / エージェント |
| ログ形式 | テキスト（ログファイル） |
| ローカルログ | あり（部分的） |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.gemini\antigravity\brain\` |
| macOS | `~/.gemini/antigravity/brain/` + `~/.gemini/antigravity/conversations/` |
| Linux | `~/.gemini/antigravity/brain/` |

## ファイル構造

```
~/.gemini/antigravity/
├── brain/
│   └── {conversation-id}/
│       └── .system_generated/
│           └── logs/                  # 会話ログ（テキスト形式）
└── conversations/
    └── *.pb                           # Protocol Buffers 形式（読み取り不可）
```

## データ形式

- `logs/` 配下: テキスト形式のログファイル
- `conversations/*.pb`: Protocol Buffers 形式（バイナリ、テキストとして読み取り不可）

## 取得方法

```python
from pathlib import Path

brain_dir = Path.home() / ".gemini" / "antigravity" / "brain"
for log_dir in brain_dir.glob("*/.system_generated/logs"):
    for log_file in sorted(log_dir.rglob("*"), key=lambda p: p.stat().st_mtime, reverse=True):
        if not log_file.is_file() or log_file.suffix == ".pb":
            continue
        text = log_file.read_text(encoding="utf-8").strip()
        if text:
            print(f"[{log_file.parent.parent.parent.name[:12]}] {text[:200]}")
```

## 注意事項

- 比較的新しいツールのため、ログ形式が変更される可能性がある
- `.gemini/` フォルダが削除されると会話リストは残るが内容は読めなくなる（既知のバグ）
- `.pb` ファイルは Protocol Buffers 形式でテキストとして読めないためスキップ
- タイムスタンプはファイル更新日時で代用
