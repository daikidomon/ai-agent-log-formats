# SWE-agent

## Basic Information

| Item | Details |
|------|---------|
| Type | Autonomous agent (OSS, Princeton University) |
| Log Format | JSON (trajectory files) |
| Local Logs | Yes |

## Storage Location

Under the directory where SWE-agent was executed:

```
trajectories/
└── {run-id}/
    └── {instance-id}.traj
```

By default, a `trajectories/` directory is created in the current working directory at execution time.

## Data Format

### .traj File (JSON)

```json
{
  "environment": "swe_main",
  "trajectory": [
    {
      "action": "User or agent action",
      "observation": "Execution result output",
      "response": "Agent's thought/response",
      "state": "...",
      "thought": "Agent's internal reasoning"
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
      "content": "System prompt"
    },
    {
      "role": "user",
      "content": "Task description/prompt"
    },
    {
      "role": "assistant",
      "content": "Agent's response"
    }
  ]
}
```

## Retrieval Method

```python
import json
from pathlib import Path

trajectories_dir = Path("trajectories")  # Under the execution directory
for traj_file in trajectories_dir.rglob("*.traj"):
    with open(traj_file, "r", encoding="utf-8") as f:
        data = json.load(f)

    # Extract user messages from history
    for msg in data.get("history", []):
        if msg.get("role") == "user":
            print(f"[{traj_file.stem}] {msg['content'][:200]}")

    # Extract actions from trajectory
    for step in data.get("trajectory", []):
        action = step.get("action", "")
        if action:
            print(f"  Action: {action[:200]}")
```

## Notes

- Trajectory files are saved under the execution directory (not centrally aggregated)
- File sizes can become large (as they include input/output for all steps)
- `history` contains the complete conversation context
- Often used for SWE-bench evaluation purposes
