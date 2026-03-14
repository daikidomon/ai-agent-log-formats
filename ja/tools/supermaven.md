# Supermaven

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | コード補完拡張（VS Code / JetBrains / Neovim） |
| ログ形式 | — |
| ローカルログ | 限定的 |
| 備考 | 補完主体のため対話ログはほぼなし |

## 保存場所

VS Code 拡張として:

| OS | パス |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\supermaven.supermaven\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/supermaven.supermaven/` |
| Linux | `~/.config/Code/User/globalStorage/supermaven.supermaven/` |

## データ形式

Supermaven はコード補完が主機能であり、チャット形式の対話ログは基本的に生成されない。

## 注意事項

- 高速なコード補完に特化したツール
- チャット機能がないため、プロンプト分析には不向き
- 2024年に Cursor に買収されたため、今後統合される可能性がある
