# Kata 04: Hotkeys & Keyboard Shortcuts (~15 min)

## Theory

Claude Code has built-in keyboard shortcuts and supports full customization via `~/.claude/keybindings.json`.

### Essential Default Shortcuts

| Shortcut | Action |
|----------|--------|
| `Enter` | Submit message |
| `Ctrl+C` | Cancel current operation (hardcoded) |
| `Ctrl+D` | Exit Claude Code (hardcoded) |
| `Ctrl+L` | Clear screen |
| `Shift+Tab` | Cycle permission modes |
| `Esc Esc` | Rewind/undo last response |
| `Ctrl+R` | Search command history |
| `Ctrl+G` | Open prompt in external editor ($EDITOR) |

### Multiline Input

| Terminal | Shortcut |
|----------|----------|
| Any | `\` then `Enter` |
| macOS Terminal | `Option+Enter` |
| iTerm2, WezTerm, Ghostty, Kitty | `Shift+Enter` |
| Windows Terminal | `Shift+Enter` |
| PowerShell (legacy) | `Shift+Enter` |

### Vim Mode

Enable via the `editorMode` setting (no longer a slash command). Either:

- Run `/config` and set **Editor mode** to `vim`, or
- Add to `~/.claude/settings.json` (or project/local settings):

  ```json
  { "editorMode": "vim" }
  ```

  Set back to `"normal"` to disable.

Provides vi-style editing with NORMAL and VISUAL modes:
- `Esc` — Normal mode
- `i`, `a`, `o` / `O` — Insert mode (use `o`/`O` or `Ctrl+J` for a newline; `Enter` still submits)
- `h/j/k/l` — Navigation
- `w`, `b`, `e` — Word movement
- `v` / `V` — Visual / visual-line selection
- `d`, `c`, `y`, `p` — Delete / change / yank / paste

### Custom Keybindings

File: `~/.claude/keybindings.json` (macOS/Linux) or `%USERPROFILE%\.claude\keybindings.json` (Windows)

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "meta+return": "chat:submit"
      }
    }
  ]
}
```

Available contexts: `Global`, `Chat`, `Autocomplete`, `Confirmation`, `Transcript`, `Help`, and more.

---

## Exercise

### Tasks

#### 1. Learn the Essentials

Start Claude Code in any directory:

```bash
cd /tmp && claude
```

Practice these shortcuts:

1. Type a message but **don't submit** — press `Escape` to clear it
2. Type a message and submit with `Enter`
3. Press `Ctrl+L` to clear the screen
4. Press `Shift+Tab` to cycle permission modes — watch the indicator
5. Press `Ctrl+R` to search your command history
6. Press `Esc Esc` to rewind the last response

#### 2. Try Multiline Input

In the same Claude session:

1. Type `Tell me about` then press `\` and `Enter` — continue on the next line
2. Try `Option+Enter` (macOS) or `Shift+Enter` (iTerm2) for multiline
3. Write a 3-line prompt and submit it

#### 3. Try Vim Mode

Enable it via `/config` → **Editor mode** → `vim` (or add `"editorMode": "vim"` to `~/.claude/settings.json`).

1. Type something in insert mode
2. Press `Esc` to go to normal mode
3. Navigate with `h/j/k/l`
4. Press `i` to go back to insert mode
5. Type your prompt and submit (use `o` or `Ctrl+J` for newlines — `Enter` submits)

Toggle vim off when done: set **Editor mode** back to `normal` in `/config` (or `"editorMode": "normal"`).

#### 4. Create Custom Keybindings

Create `~/.claude/keybindings.json`:

```json
{
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor"
      }
    }
  ]
}
```

Then in Claude, press `Ctrl+E` — it should open your `$EDITOR` for composing a longer prompt.

#### 5. Explore Available Bindings

Run `/keybindings` in Claude to see the interactive keybindings editor. Browse available actions and contexts.

### Discussion Points

- Which shortcuts do you see yourself using daily?
- Would you change the submit key from `Enter` to `Ctrl+Enter`? Why?
- How does Vim mode compare to your regular editor keybindings?
