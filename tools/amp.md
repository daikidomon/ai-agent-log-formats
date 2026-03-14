# amp (Sourcegraph)

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI エージェント |
| ログ形式 | JSONL |
| ローカルログ | あり |

## 保存場所

| OS | パス |
|----|------|
| macOS | `~/.amp/` |
| Linux | `~/.amp/` |

環境変数 `AMP_HOME` が設定されている場合はそのパスを使用（推定）。

## ファイル構造（推定）

```
~/.amp/
├── config.yaml                    # 設定
├── conversations/
│   └── {session-id}.jsonl         # 会話ログ
└── threads/
    └── ...
```

## データ形式

JSONL 形式で会話がログされる（推定）。Sourcegraph のエージェントフレームワークに基づく構造。

## 取得方法

```python
from pathlib import Path

amp_dir = Path.home() / ".amp"
if amp_dir.exists():
    # 会話ファイルを探索
    for f in amp_dir.rglob("*.jsonl"):
        print(f"Found: {f}")
    for f in amp_dir.rglob("*.json"):
        print(f"Found: {f}")
```

## 注意事項

- **要実機調査**: amp は比較的新しいツールであり、ログの詳細構造は未確認
- Sourcegraph サーバーと連携している場合、会話履歴はサーバー側に保存される可能性
