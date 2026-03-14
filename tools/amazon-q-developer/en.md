# Amazon Q Developer

## Basic Information

| Item | Details |
|------|---------|
| Type | VS Code / JetBrains extension |
| Log Format | JSON |
| Local Logs | Yes (partial) |
| Former Name | Amazon CodeWhisperer -> Amazon Q Developer |

## Storage Location

### VS Code Extension

| OS | Path |
|----|------|
| Windows | `%APPDATA%\Code\User\globalStorage\amazonwebservices.aws-toolkit-vscode\` |
| macOS | `~/Library/Application Support/Code/User/globalStorage/amazonwebservices.aws-toolkit-vscode/` |
| Linux | `~/.config/Code/User/globalStorage/amazonwebservices.aws-toolkit-vscode/` |

### AWS Configuration Directory

| OS | Path |
|----|------|
| All | `~/.aws/amazonq/` |

## File Structure (Estimated)

```
amazonwebservices.aws-toolkit-vscode/
├── cache/
│   └── ...
└── state/
    └── ...
```

## Data Format

Chat history may be stored in JSON format within VS Code's `globalStorage`.

Amazon Q Developer is deeply integrated with cloud services, so a large portion of conversation history may be stored on the AWS side.

## Retrieval Method

```python
import json
from pathlib import Path

def get_q_storage():
    import platform
    system = platform.system()
    if system == "Windows":
        import os
        return Path(os.environ["APPDATA"]) / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"
    elif system == "Darwin":
        return Path.home() / "Library" / "Application Support" / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"
    else:
        return Path.home() / ".config" / "Code" / "User" / "globalStorage" / "amazonwebservices.aws-toolkit-vscode"

storage = get_q_storage()
# Search for JSON files
for json_file in storage.rglob("*.json"):
    print(f"Found: {json_file}")
    try:
        with open(json_file, "r", encoding="utf-8") as f:
            data = json.load(f)
        print(json.dumps(data, ensure_ascii=False, indent=2)[:500])
    except (json.JSONDecodeError, OSError):
        pass
```

## Notes

- **Requires hands-on investigation**: The detailed log structure has not been verified
- The extension ID is `amazonwebservices.aws-toolkit-vscode` (integrated into AWS Toolkit)
- Much of the conversation history is likely stored on the cloud side (AWS)
- The JetBrains version uses a different plugin directory
- Code completion logs and chat logs may be stored in separate locations
