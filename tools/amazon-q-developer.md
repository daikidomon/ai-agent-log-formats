# Amazon Q Developer

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code / JetBrains 拡張 |
| ログ形式 | JSON |
| ローカルログ | あり（部分的） |
| 旧名称 | Amazon CodeWhisperer → Amazon Q Developer |

## 保存場所

### VS Code拡張

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\amazonwebservices.aws-toolkit-vscode\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/amazonwebservices.aws-toolkit-vscode/` |
| Linux | `~/.config/Code/User/globalStorage/amazonwebservices.aws-toolkit-vscode/` |

### AWS設定ディレクトリ

| OS | パス |
|----|------|
| 共通 | `~/.aws/amazonq/` |

## ファイル構造（推定）

```
amazonwebservices.aws-toolkit-vscode/
├── cache/
│   └── ...
└── state/
    └── ...
```

## データ形式

チャット履歴は VS Code の `globalStorage` 内にJSON形式で保存される可能性がある。

Amazon Q Developer はクラウドサービスとの連携が深いため、会話履歴の大部分はAWS側に保存される場合がある。

## 取得方法

```python
import json
from pathlib import Path

def get_q_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"
    else:
        return Path.home() / ".config" / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"

storage = get_q_storage()
# JSON ファイルを探索
for json_file in storage.rglob("*.json"):
    print(f"Found: {json_file}")
    try:
        with open(json_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        print(json.dumps(data, ensure_ascii=False, indent=2)[:500])
    except (json.JSONDecodeError, OSError):
        pass
```

## 注意事項

- **要実機調査**: ログの詳細構造は未確認
- 拡張機能IDは `amazonwebservices.aws-toolkit-vscode`（AWS Toolkit に統合）
- 会話履歴の多くはクラウド側（AWS）に保存される可能性が高い
- JetBrains 版はプラグインディレクトリが異なる
- コード補完ログと チャットログは別の場所に保存される場合がある
