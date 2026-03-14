# Aider

## Basic Information

| Item | Details |
|------|---------|
| Type | CLI (AI pair programming) |
| Log Format | Markdown / SQLite |
| Local Logs | Yes |

## Storage Location

### Chat History (Markdown Logs)

Saved directly under the project directory:

| File | Description |
|------|-------------|
| `.aider.chat.history.md` | Conversation history (Markdown format) |
| `.aider.input.history` | Input history (readline format) |
| `.aider.tags.cache.v3/` | Tag cache |

### Analytics DB

| OS | Path |
|----|------|
| All | `~/.aider/analytics.db` |

### Configuration File

| OS | Path |
|----|------|
| All | `~/.aider.conf.yml` or `~/.config/aider/.aider.conf.yml` |

## Data Format

### .aider.chat.history.md

Markdown format. User and assistant conversations are recorded alternately:

```markdown
# aider chat started at 2025-06-15 10:30:00

#### User's prompt (1st)

Assistant's response...

#### User's prompt (2nd)

Assistant's response...
```

- User prompts are prefixed with `#### ` (h4)
- Assistant responses have no prefix
- Sessions are delimited by `# aider chat started at ...`

### .aider.input.history

Input history in readline format. One entry per line:

```
Fix the file
Add tests
```

## Retrieval Method

```python
import re
from pathlib import Path

def collect_aider_history(project_dir: Path) -> list[dict]:
    """Extract prompts from .aider.chat.history.md in the project directory"""
    history_file = project_dir / ".aider.chat.history.md"
    if not history_file.exists():
        return []

    messages = []
    current_session_time = None
    content = history_file.read_text(encoding="utf-8")

    for line in content.split("\n"):
        # Session start
        session_match = re.match(r'^# aider chat started at (.+)', line)
        if session_match:
            current_session_time = session_match.group(1).strip()
            continue

        # User prompt (h4)
        if line.startswith("#### "):
            prompt = line[5:].strip()
            if prompt:
                messages.append({
                    "text": prompt,
                    "session_started": current_session_time or "unknown",
                })

    return messages

# To scan all projects
for history_file in Path.home().rglob(".aider.chat.history.md"):
    project_dir = history_file.parent
    messages = collect_aider_history(project_dir)
    for msg in messages:
        print(f"[{project_dir.name}] {msg['text']}")
```

### Discovering Multiple Projects

Since Aider saves logs directly under the project directory, a recursive search is required to collect logs from all projects:

```bash
find ~ -name ".aider.chat.history.md" -type f 2>/dev/null
```

## Notes

- Log files are saved directly under the project directory (not centralized)
- Many projects add `.aider*` to `.gitignore`
- Since the format is Markdown, parsing may break if a prompt contains `####`
- There is a timestamp per session, but no timestamps for individual messages
- `analytics.db` records metadata (model, token counts, etc.) but does not contain conversation content
