# Windsurf (Cascade)

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | AI IDE |
| ログ形式 | テキスト（自動要約メモリ） |
| ローカルログ | あり（ただしメモリ形式） |

## 保存場所

| OS | パス |
|----|------|
| Windows | `%USERPROFILE%\.codeium\windsurf\memories\` |
| macOS | `~/.codeium/windsurf/memories/` |
| Linux | `~/.codeium/windsurf/memories/` |

バックアップ（cascade-backup-utils 使用時）: `~/.cascade_backups/`

## ファイル構造

```
~/.codeium/windsurf/
└── memories/
    └── {workspace-id}/
        └── (メモリファイル群)
```

## データ形式

テキストファイル。Cascade が会話から自動生成した要約・メモリ。

元のプロンプトそのものではなく、Cascade が重要と判断した情報の要約。

## 取得方法

```python
from pathlib import Path

memories_dir = Path.home() / ".codeium" / "windsurf" / "memories"
for mem_file in sorted(memories_dir.rglob("*"), key=lambda p: p.stat().st_mtime, reverse=True):
    if not mem_file.is_file():
        continue
    text = mem_file.read_text(encoding="utf-8").strip()
    if text:
        print(f"[{mem_file.parent.name}] {text[:200]}")
```

## 注意事項

- 会話ログ自体はローカルに直接保存されない場合がある
- `memories/` はワークスペース単位で分離
- メモリの内容は要約であり、元のプロンプトとは異なる
- タイムスタンプはファイル更新日時で代用
- `~/.cascade_backups/` が存在する場合はそちらも参照可能
