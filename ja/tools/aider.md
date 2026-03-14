# Aider

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | CLI（AIペアプログラミング） |
| ログ形式 | Markdown / SQLite |
| ローカルログ | あり |

## 保存場所

### チャット履歴（Markdownログ）

プロジェクトディレクトリ直下に保存:

| ファイル | 内容 |
|---------|------|
| `.aider.chat.history.md` | 会話履歴（Markdown形式） |
| `.aider.input.history` | 入力履歴（readline形式） |
| `.aider.tags.cache.v3/` | タグキャッシュ |

### アナリティクスDB

| OS | パス |
|----|------|
| 共通 | `~/.aider/analytics.db` |

### 設定ファイル

| OS | パス |
|----|------|
| 共通 | `~/.aider.conf.yml` または `~/.config/aider/.aider.conf.yml` |

## データ形式

### .aider.chat.history.md

Markdown 形式。ユーザーとアシスタントの会話が交互に記録される:

```markdown
# aider chat started at 2025-06-15 10:30:00

#### ユーザーのプロンプト（1つ目）

アシスタントの応答...

#### ユーザーのプロンプト（2つ目）

アシスタントの応答...
```

- ユーザーのプロンプトは `#### ` プレフィックス（h4）
- アシスタントの応答はプレフィックスなし
- セッションの区切りは `# aider chat started at ...`

### .aider.input.history

readline 形式の入力履歴。1行1エントリ:

```
ファイルを修正して
テストを追加して
```

## 取得方法

```python
import re
from pathlib import Path

def collect_aider_history(project_dir: Path) -> list[dict]:
    """プロジェクトディレクトリ内の .aider.chat.history.md からプロンプトを抽出"""
    history_file = project_dir / ".aider.chat.history.md"
    if not history_file.exists():
        return []

    messages = []
    current_session_time = None
    content = history_file.read_text(encoding="utf-8")

    for line in content.split("\n"):
        # セッション開始
        session_match = re.match(r'^# aider chat started at (.+)', line)
        if session_match:
            current_session_time = session_match.group(1).strip()
            continue

        # ユーザープロンプト（h4）
        if line.startswith("#### "):
            prompt = line[5:].strip()
            if prompt:
                messages.append({
                    "text": prompt,
                    "session_started": current_session_time or "unknown",
                })

    return messages

# 全プロジェクトを走査する場合
for history_file in Path.home().rglob(".aider.chat.history.md"):
    project_dir = history_file.parent
    messages = collect_aider_history(project_dir)
    for msg in messages:
        print(f"[{project_dir.name}] {msg['text']}")
```

### 複数プロジェクトの発見

Aider はプロジェクトディレクトリ直下にログを保存するため、全プロジェクトのログを収集するには再帰検索が必要:

```bash
find ~ -name ".aider.chat.history.md" -type f 2>/dev/null
```

## 注意事項

- ログファイルはプロジェクトディレクトリ直下に保存される（中央集約ではない）
- `.gitignore` に `.aider*` を追加しているプロジェクトが多い
- Markdown 形式のため、プロンプト中に `####` が含まれるとパースが崩れる可能性
- セッション単位のタイムスタンプはあるが、個々のメッセージのタイムスタンプはない
- `analytics.db` にはメタデータ（モデル、トークン数等）が記録されるが会話内容は含まない
