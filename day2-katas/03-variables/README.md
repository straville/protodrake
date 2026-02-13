# Kata 03: Variables & Configuration (~15 min)

## Theory

Claude Code uses a layered configuration system. Settings can come from multiple sources, with a clear precedence order.

### Settings Precedence (highest to lowest)

1. **CLI arguments** — `claude --permission-mode plan`
2. **Local project settings** — `.claude/settings.local.json` (gitignored, personal)
3. **Project settings** — `.claude/settings.json` (committed, shared with team)
4. **User global settings** — `~/.claude/settings.json` (all your projects)

### CLAUDE.md — Project Memory

CLAUDE.md files provide persistent context that loads automatically every session.

| File | Scope | Shared? |
|------|-------|---------|
| `~/.claude/CLAUDE.md` | All your projects | No |
| `CLAUDE.md` (project root) | This project | Yes (git) |
| `.claude/CLAUDE.md` | This project | Yes (git) |
| `CLAUDE.local.md` | This project | No (gitignored) |

All matching files are loaded and concatenated.

### Environment Variables

Set in `settings.json`:

```json
{
  "env": {
    "NODE_ENV": "development",
    "DEBUG": "app:*"
  }
}
```

Key Claude Code env vars:
- `ANTHROPIC_API_KEY` — API authentication
- `CLAUDE_CODE_MAX_OUTPUT_TOKENS` — Output token limit
- `DISABLE_TELEMETRY` — Opt out of telemetry

---

## Exercise

### Setup

```bash
mkdir -p /tmp/kata-03/.claude && cd /tmp/kata-03
git init
echo '{ "name": "kata-03" }' > package.json
echo "console.log('hello');" > index.js
```

### Tasks

#### 1. Create a CLAUDE.md

Create `/tmp/kata-03/CLAUDE.md`:

```markdown
# Kata 03 Project

## Tech Stack
- Node.js with vanilla JavaScript
- No frameworks

## Conventions
- Use camelCase for variables
- Always add JSDoc comments to functions
- Prefer const over let

## Commands
- Run: node index.js
```

Then start Claude and ask: `"What are this project's coding conventions?"`

Claude should answer using the CLAUDE.md content without reading any other files.

#### 2. Create Personal Overrides

Create `/tmp/kata-03/CLAUDE.local.md`:

```markdown
# My Personal Notes
- I prefer verbose error messages
- Always explain your reasoning step by step
```

Start Claude again and ask: `"How should you communicate with me?"`

Observe that Claude picks up both CLAUDE.md AND CLAUDE.local.md.

#### 3. Layer Settings Files

Create shared project settings:

```json
// /tmp/kata-03/.claude/settings.json
{
  "permissions": {
    "allow": ["Bash(node *)"]
  }
}
```

Create personal local overrides:

```json
// /tmp/kata-03/.claude/settings.local.json
{
  "permissions": {
    "allow": ["Bash(node *)", "Bash(git *)"]
  }
}
```

Start Claude and test:
1. `"Run node index.js"` — allowed by both
2. `"Run git status"` — allowed only by local settings

#### 4. Use /init to Bootstrap

```bash
cd /tmp/kata-03-init && mkdir -p /tmp/kata-03-init && cd /tmp/kata-03-init
git init
claude
```

Type `/init` and let Claude generate a CLAUDE.md for the project. Review what it produces.

### Discussion Points

- What goes in shared CLAUDE.md vs personal CLAUDE.local.md?
- How would you use CLAUDE.md to onboard new team members?
- When would you use settings.local.json over settings.json?
