# PearAI

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI特化エディタ（VS Code フォーク） |
| ログ形式 | SQLite / JSON |
| ローカルログ | あり（推定） |

## 保存場所

VS Code フォークのため、同様のディレクトリ構造を使用（推定）:

| OS | パス |
|----|------|
| Windows | `%APPDATA%\PearAI\User\` |
| macOS | `~/Library/Application Support/PearAI/User/` |
| Linux | `~/.config/PearAI/User/` |

## ファイル構造（推定）

```
PearAI/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## データ形式

VS Code ベースのため `state.vscdb` 構造。PearAI 独自のAIチャットデータは `globalStorage` または専用ディレクトリに保存される可能性がある。

## 注意事項

- **要実機調査**: 詳細なログ構造は未確認
- PearAI は Continue を内蔵しているため、`~/.continue/` にもデータがある可能性
