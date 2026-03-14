# JetBrains AI Assistant

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | IDE内蔵（JetBrains全製品） |
| ログ形式 | XML / JSON |
| ローカルログ | あり |

## 保存場所

JetBrains IDE の設定ディレクトリに保存:

| OS | パス |
|----|------|
| Windows | `%APPDATA%\JetBrains\<IDE><version>\` |
| macOS | `~/Library/Application Support/JetBrains/<IDE><version>/` |
| Linux | `~/.config/JetBrains/<IDE><version>/` |

`<IDE>` は製品名:
- `IntelliJIdea` (IntelliJ IDEA Ultimate)
- `IdeaIC` (IntelliJ IDEA Community)
- `PyCharm` / `PyCharmCE`
- `WebStorm`
- `GoLand`
- `RubyMine`
- `CLion`
- `Rider`
- `DataGrip`

## ファイル構造（推定）

```
JetBrains/<IDE><version>/
├── options/
│   └── ai-assistant.xml           # AI Assistant設定
├── ai-assistant/
│   └── chats/
│       └── {session-id}.json      # チャットセッション
└── log/
    └── idea.log                   # IDEログ（AI関連のログを含む場合あり）
```

## データ形式

チャット履歴はJSON形式:

```json
{
  "id": "session-uuid",
  "messages": [
    {
      "role": "user",
      "content": "ユーザーのプロンプト",
      "timestamp": "2025-06-15T10:30:00Z"
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

def get_jetbrains_dirs():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        base = Path(os.environ["APPDATA"]) / "JetBrains"
    elif system == "Darwin":
        base = Path.home() / "Library" / "Application Support" / "JetBrains"
    else:
        base = Path.home() / ".config" / "JetBrains"
    return list(base.iterdir()) if base.exists() else []

for ide_dir in get_jetbrains_dirs():
    chats_dir = ide_dir / "ai-assistant" / "chats"
    if not chats_dir.exists():
        continue
    for chat_file in chats_dir.glob("*.json"):
        with open(chat_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        for msg in data.get("messages", []):
            if msg.get("role") == "user":
                print(f"[{ide_dir.name}] {msg.get('content', '')}")
```

## 注意事項

- **要実機調査**: チャット履歴の保存パスとデータ構造は推定
- IDE のバージョンごとに別ディレクトリが作られる
- AI Assistant はプラグインとして提供されるため、プラグインの設定ディレクトリにデータがある可能性もある
- JetBrains アカウントと連携している場合、クラウド側にも履歴が保存される
