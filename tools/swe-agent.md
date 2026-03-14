# SWE-agent

## 基本情報

| 項目 | 内容 |
|------|------|
| 種別 | 自律エージェント（OSS、Princeton大学） |
| ログ形式 | JSON（trajectory ファイル） |
| ローカルログ | あり |

## 保存場所

SWE-agent を実行したディレクトリ配下:

```
trajectories/
└── {run-id}/
    └── {instance-id}.traj
```

デフォルトでは実行時のカレントディレクトリに `trajectories/` が作成される。

## データ形式

### .traj ファイル（JSON）

```json
{
  "environment": "swe_main",
  "trajectory": [
    {
      "action": "ユーザーまたはエージェントのアクション",
      "observation": "実行結果の出力",
      "response": "エージェントの思考・応答",
      "state": "...",
      "thought": "エージェントの内部思考"
    }
  ],
  "info": {
    "model": "gpt-4",
    "instance_id": "...",
    "exit_status": "submitted"
  },
  "history": [
    {
      "role": "system",
      "content": "システムプロンプト"
    },
    {
      "role": "user",
      "content": "タスク説明・プロンプト"
    },
    {
      "role": "assistant",
      "content": "エージェントの応答"
    }
  ]
}
```

## 取得方法

```python
import json
from pathlib import Path

trajectories_dir = Path("trajectories")  # 実行ディレクトリ配下
for traj_file in trajectories_dir.rglob("*.traj"):
    with open(traj_file, "r", encoding="utf-8") as f:
        data = json.load(f)

    # history から user メッセージを抽出
    for msg in data.get("history", []):
        if msg.get("role") == "user":
            print(f"[{traj_file.stem}] {msg['content'][:200]}")

    # trajectory からアクションを抽出
    for step in data.get("trajectory", []):
        action = step.get("action", "")
        if action:
            print(f"  Action: {action[:200]}")
```

## 注意事項

- trajectory ファイルは実行ディレクトリ配下に保存（中央集約ではない）
- ファイルサイズが大きくなる場合がある（全ステップの入出力を含む）
- `history` には完全な会話コンテキストが含まれる
- SWE-bench の評価用途で使われることが多い
