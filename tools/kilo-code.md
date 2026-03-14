# Kilo Code

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | VS Code拡張 |
| ログ形式 | JSON（Cline系構造） |
| ローカルログ | あり |
| 備考 | Cline のフォーク |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\kilocode.kilo-code\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/kilocode.kilo-code/` |
| Linux | `~/.config/Code/User/globalStorage/kilocode.kilo-code/` |

## ファイル構造

Cline と同一構造:

```
kilocode.kilo-code/
├── state/
│   └── taskHistory.json
└── tasks/
    └── {task-id}/
        ├── api_conversation_history.json
        ├── ui_messages.json
        └── task_metadata.json
```

## データ形式

[Cline](cline.md) と同一。`api_conversation_history.json` から `role: "user"` のメッセージを抽出。

## 取得方法

Cline と同じ手順。パスの `saoudrizwan.claude-dev` を `kilocode.kilo-code` に置き換える。

```python
base / "Code" / "User" / "globalStorage" / "kilocode.kilo-code" / "tasks"
```

## 注意事項

- 拡張機能IDが変更される可能性がある（要確認）
- Kilo Code 独自のカスタマイズがメタデータに反映される場合がある
