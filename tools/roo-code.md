# Roo Code

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code拡張 |
| ログ形式 | JSON（Cline同一構造） |
| ローカルログ | あり |
| 備考 | Cline のフォーク |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\RooVeterinaryInc.roo-cline\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/RooVeterinaryInc.roo-cline/` |
| Linux | `~/.config/Code/User/globalStorage/RooVeterinaryInc.roo-cline/` |

## ファイル構造

Cline と同一:

```
RooVeterinaryInc.roo-cline/
├── state/
│   └── taskHistory.json
└── tasks/
    └── {task-id}/
        ├── api_conversation_history.json
        ├── ui_messages.json
        └── task_metadata.json
```

## データ形式

[Cline](cline.md) と同一。`api_conversation_history.json` から `role: "user"` のメッセージを抽出する。

## 取得方法

Cline と同じ手順。パスの `saoudrizwan.claude-dev` を `RooVeterinaryInc.roo-cline` に置き換えるだけ。

```python
# Clineとの差分はベースパスのみ
base / "Code" / "User" / "globalStorage" / "RooVeterinaryInc.roo-cline" / "tasks"
```

## 注意事項

- Roo Code 独自の機能（モード切替等）によるメタデータが追加されている場合がある
- 基本的な会話ログ構造は Cline と互換
