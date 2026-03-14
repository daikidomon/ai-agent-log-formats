# Void

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI特化エディタ（VS Code フォーク、OSS） |
| ログ形式 | SQLite / JSON |
| ローカルログ | あり（推定） |

## 保存場所

VS Code フォークのため、同様のディレクトリ構造を使用（推定）:

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Void\User\` |
| macOS | `~/Library/Application Support/Void/User/` |
| Linux | `~/.config/Void/User/` |

## ファイル構造（推定）

```
Void/User/
├── workspaceStorage/
│   └── {workspace-hash}/
│       └── state.vscdb
├── globalStorage/
│   └── ...
└── ...
```

## データ形式

VS Code ベースのため `state.vscdb` 構造。Void 独自のAIチャットデータは別途保存される可能性がある。

## 注意事項

- **要実機調査**: 詳細なログ構造は未確認
- OSSプロジェクトのためソースコードからストレージ構造を確認可能
- 開発初期段階のため、ストレージ構造が頻繁に変更される可能性
