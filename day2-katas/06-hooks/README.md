# Kata 06: Hooks (Lifecycle Automation) (~15 min)

## Theory

Hooks are shell commands that run automatically at specific points in Claude Code's lifecycle. Unlike skills (which Claude chooses to invoke), hooks **always run** — they're deterministic automation.

### Hook Events

| Event | When it fires | Can block? |
|-------|--------------|------------|
| `PreToolUse` | Before a tool executes | Yes |
| `PostToolUse` | After a tool succeeds | No |
| `Stop` | Claude finishes responding | Yes |
| `Notification` | Notification is sent | No |
| `SessionStart` | Session begins | No |

### Hook Types

| Type | How it works |
|------|-------------|
| `command` | Runs a shell script, receives JSON on stdin |
| `prompt` | Sends context to a fast model for judgment |
| `agent` | Spawns a subagent with tool access |

### How Command Hooks Work

1. Claude triggers an event (e.g., about to run a Bash command)
2. Your hook script receives event JSON on stdin
3. Your script decides: **exit 0** (allow) or **exit 2** (block with error message)

### Configuration

In `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./my-hook.sh"
          }
        ]
      }
    ]
  }
}
```

`matcher` is a regex that filters which tools trigger the hook (e.g., `"Bash"`, `"Edit|Write"`, `".*"` for all).

---

## Exercise

### Setup

```bash
mkdir -p /tmp/kata-06/.claude/hooks && cd /tmp/kata-06
git init
echo "console.log('hello');" > app.js
echo '{ "name": "kata-06" }' > package.json
```

### Tasks

#### 1. Create a Logging Hook

Create a hook that logs every tool Claude uses.

`.claude/hooks/log-tools.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
TOOL=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
TIMESTAMP=$(date '+%H:%M:%S')

echo "[$TIMESTAMP] Tool used: $TOOL" >> /tmp/kata-06/claude-tool-log.txt

exit 0
```

Make it executable:

```bash
chmod +x /tmp/kata-06/.claude/hooks/log-tools.sh
```

Add to `/tmp/kata-06/.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/log-tools.sh"
          }
        ]
      }
    ]
  }
}
```

Start Claude and give it a task: `"Read app.js and add a comment to it."`

Then check the log:

```bash
cat /tmp/kata-06/claude-tool-log.txt
```

#### 2. Create a Blocking Hook

Create a hook that prevents deletion commands.

`.claude/hooks/block-delete.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""')

if echo "$COMMAND" | grep -qiE '(rm -rf|rm -r|rmdir)'; then
  echo "BLOCKED: Destructive delete commands are not allowed." >&2
  exit 2
fi

exit 0
```

```bash
chmod +x /tmp/kata-06/.claude/hooks/block-delete.sh
```

Update settings to add a PreToolUse hook:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-delete.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/log-tools.sh"
          }
        ]
      }
    ]
  }
}
```

Test it:

```bash
cd /tmp/kata-06
claude
```

Ask: `"Delete the app.js file using rm"`

Observe: The hook blocks the command and Claude sees the error message.

Then ask: `"Read app.js"` — this should work fine (only Bash is matched).

#### 3. Try the Interactive Hook Editor

In a Claude session, type:

```
/hooks
```

Browse the available events and see how hooks are configured interactively.

#### 4. Desktop Notification Hook (Bonus)

If on macOS, add a notification when Claude needs attention:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Discussion Points

- What team rules could you enforce with PreToolUse hooks?
- How would you use PostToolUse hooks for code quality (e.g., auto-formatting)?
- What's the difference between using hooks vs. permission rules for security?
