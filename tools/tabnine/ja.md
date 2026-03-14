# Tabnine

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | コード補完拡張（VS Code / JetBrains 等） |
| ログ形式 | — |
| ローカルログ | 限定的 |
| 備考 | 補完主体のため対話ログはほぼなし |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%APPDATA%\TabNine\` |
| macOS | `~/Library/Application Support/TabNine/` |
| Linux | `~/.config/TabNine/` |

## データ形式

Tabnine はコード補完が主機能であり、チャット形式の対話ログは基本的に生成されない。

チャット機能（Tabnine Chat）が追加されている場合は、VS Code の `globalStorage` に保存される可能性がある。

## 注意事項

- 補完主体のツールのため、プロンプト分析には不向き
- Tabnine Chat がある場合のみ対話ログが存在する可能性
- ローカルモデル（Tabnine Enterprise）使用時は異なるログパスの可能性
