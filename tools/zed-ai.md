# Zed AI

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | エディタ内蔵AI（Rust製エディタ） |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| macOS | `~/.config/zed/conversations/` |
| Linux | `~/.config/zed/conversations/` |

※ Zed は現在 macOS / Linux のみ対応。

## ファイル構造（推定）

```
~/.config/zed/
├── settings.json                  # エディタ設定
├── conversations/
│   └── {conversation-id}.json     # AI会話ログ
└── prompts/
    └── *.md                       # カスタムプロンプト
```

## データ形式

会話ログはJSON形式（推定）:

```json
{
  "id": "conversation-uuid",
  "messages": [
    {
      "role": "user",
      "content": "ユーザーのプロンプト"
    },
    {
      "role": "assistant",
      "content": "AIの応答"
    }
  ]
}
```

## 取得方法

```python
import json
from pathlib import Path

zed_conversations = Path.home() / ".config" / "zed" / "conversations"
if zed_conversations.exists():
    for conv_file in sorted(zed_conversations.glob("*.json"),
                            key=lambda p: p.stat().st_mtime, reverse=True):
        with open(conv_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        for msg in data.get("messages", []):
            if msg.get("role") == "user":
                print(f"[zed] {msg.get('content', '')}")
```

## 注意事項

- **要実機調査**: 会話ログの保存パスとデータ構造は推定
- Zed の AI 機能は急速に開発中で、ストレージ構造が変更される可能性が高い
- Windows 版は未リリース
- `prompts/` にはユーザー定義のカスタムプロンプトが保存される
