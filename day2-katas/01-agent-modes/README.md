# Kata 01: Agent Modes (~15 min)

## Theory

Claude Code has different **permission modes** that control how much autonomy the agent has. Think of them as a dial from "ask me everything" to "do whatever you want."

### Available Modes

| Mode | What it does | When to use |
|------|-------------|-------------|
| **Default** | Asks permission for edits and commands | Learning, safety-critical work |
| **Auto-accept edits** | Edits files freely, still asks for bash commands | Refactoring sessions |
| **Plan mode** | Read-only, no code changes allowed | Architecture review, exploration |
| **Bypass permissions** | Skips all checks | Only in isolated containers/CI |

### How to Switch Modes

- **During session**: Press `Shift+Tab` to cycle through modes
- **At startup**: `claude --permission-mode plan`
- **In settings**: Set `defaultMode` in `.claude/settings.json`

### Plan Mode Details

Plan mode is special — Claude can read files, search code, and analyze architecture, but **cannot write or execute anything**. It uses `EnterPlanMode` / `ExitPlanMode` tools to produce an implementation plan for your approval before any code changes happen.

---

## Exercise

### Setup

**macOS / Linux:**

```bash
mkdir -p /tmp/kata-01 && cd /tmp/kata-01
git init
echo "console.log('hello');" > app.js
echo '{ "name": "kata-01", "scripts": { "start": "node app.js" } }' > package.json
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path $env:TEMP\kata-01 | Set-Location
git init
"console.log('hello');" | Out-File -Encoding utf8 app.js
'{ "name": "kata-01", "scripts": { "start": "node app.js" } }' | Out-File -Encoding utf8 package.json
```

### Tasks

#### 1. Start Claude Code in Plan Mode

```bash
cd /tmp/kata-01
claude --permission-mode plan
```

Ask Claude: `"Analyze app.js and suggest how to add Express server functionality. Don't change anything, just plan."`

- Observe that Claude reads files but never writes
- Notice the plan output with steps

Press `Ctrl+C` to exit.

#### 2. Switch Modes with Shift+Tab

```bash
cd /tmp/kata-01
claude
```

- Start in default mode
- Press `Shift+Tab` — observe the mode indicator change
- Press `Shift+Tab` again — cycle to the next mode
- Note which mode you're in (shown in the status bar)

#### 3. Experience Auto-Accept Edits

```bash
cd /tmp/kata-01
claude --permission-mode acceptEdits
```

Ask Claude: `"Add a comment at the top of app.js explaining what it does."`

- Observe: Claude edits the file without asking for permission
- Check the file: `cat app.js`

#### 4. Compare Default Mode

```bash
cd /tmp/kata-01
claude
```

Ask the same thing: `"Add another comment to app.js."`

- Observe: Claude now asks for your approval before editing
- Try approving and denying

### Discussion Points

- When would you use Plan mode in your daily workflow?
- What are the risks of `acceptEdits` mode?
- How would you configure a default mode for your team?
