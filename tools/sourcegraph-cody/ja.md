# Sourcegraph Cody

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code / JetBrains / CLI 拡張 |
| ログ形式 | JSON |
| ローカルログ | あり |

## 保存場所

### VS Code拡張

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\sourcegraph.cody-ai\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/sourcegraph.cody-ai/` |
| Linux | `~/.config/Code/User/globalStorage/sourcegraph.cody-ai/` |

### JetBrains

JetBrains のプラグインデータディレクトリに保存（IDE設定ディレクトリ配下）。

## ファイル構造（推定）

```
sourcegraph.cody-ai/
├── chat/
│   └── {session-id}.json          # チャットセッション
└── state/
    └── ...
```

## データ形式

チャット履歴はJSON形式で保存される。

```json
{
  "id": "session-uuid",
  "interactions": [
    {
      "humanMessage": {
        "text": "ユーザーのプロンプト"
      },
      "assistantMessage": {
        "text": "Codyの応答"
      }
    }
  ]
}
```

## 取得方法

```python
import json
from pathlib import Path

def get_cody_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"
    else:
        return Path.home() / ".config" / "Code" / "User" / "globalStorage" / "sourcegraph.cody-ai"

storage = get_cody_storage()
# チャット履歴ファイルを探索
for json_file in storage.rglob("*.json"):
    try:
        with open(json_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        # 構造に応じてパース
        if "interactions" in data:
            for interaction in data["interactions"]:
                human_msg = interaction.get("humanMessage", {})
                text = human_msg.get("text", "").strip()
                if text:
                    print(f"[cody] {text}")
    except (json.JSONDecodeError, OSError):
        continue
```

## 注意事項

- **要実機調査**: 上記の構造は推定。Cody のバージョンにより異なる可能性
- VS Code の `state.vscdb` にもデータが格納されている場合がある
- Cody CLI (`cody`) を使用している場合は別のログパスの可能性
- Sourcegraph サーバー連携時は、一部のログがサーバー側に保存される
