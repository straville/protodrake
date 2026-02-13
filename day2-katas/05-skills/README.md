# Kata 05: Skills (Custom Slash Commands) (~15 min)

## Theory

Skills are reusable prompt templates that extend Claude Code with custom slash commands. They let you package workflows, coding standards, and automation into shareable commands.

### Where Skills Live

| Location | Scope | Shared with team? |
|----------|-------|-------------------|
| `~/.claude/skills/<name>/SKILL.md` | All your projects | No |
| `.claude/skills/<name>/SKILL.md` | This project only | Yes (via git) |

### SKILL.md Structure

```yaml
---
name: my-skill
description: What this skill does
allowed-tools: Read, Grep, Bash(npm *)
argument-hint: [filename]
---

Your instructions to Claude here.

Use $ARGUMENTS to reference what the user passes.
Use $0 for the first argument, $1 for second, etc.
```

### Key Frontmatter Options

| Option | Purpose |
|--------|---------|
| `name` | Becomes the `/command-name` |
| `description` | Helps Claude decide when to auto-invoke |
| `disable-model-invocation: true` | Only manual `/name` trigger, Claude won't auto-use |
| `allowed-tools` | Restrict which tools the skill can use |
| `argument-hint` | Shown in autocomplete |
| `context: fork` | Run in a subagent (separate context) |

### Dynamic Content

Use `!` backticks to inject command output:

```markdown
Current branch: !`git rev-parse --abbrev-ref HEAD`
Recent changes: !`git log --oneline -5`
```

---

## Exercise

### Setup

```bash
mkdir -p /tmp/kata-05/src/.claude/skills && cd /tmp/kata-05
git init
cat > src/app.js << 'EOF'
function calculateTotal(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total = total + items[i].price * items[i].quantity;
  }
  return total;
}

function formatCurrency(amount) {
  return "$" + amount.toFixed(2);
}

module.exports = { calculateTotal, formatCurrency };
EOF
```

### Tasks

#### 1. Create a Code Review Skill

Create `.claude/skills/review/SKILL.md`:

```yaml
---
name: review
description: Review code for quality issues
allowed-tools: Read, Grep, Glob
argument-hint: [file-path]
---

Review the file at $ARGUMENTS for:

1. **Bugs** - Logic errors, edge cases, off-by-one errors
2. **Performance** - Unnecessary loops, memory leaks
3. **Readability** - Naming, structure, complexity
4. **Modern JS** - Could use modern syntax (map, reduce, arrow functions)?

Format your review as a numbered list of findings with severity (LOW/MEDIUM/HIGH).
```

Test it:

```bash
cd /tmp/kata-05
claude
```

Type: `/review src/app.js`

#### 2. Create a Skill with Dynamic Content

Create `.claude/skills/status/SKILL.md`:

```yaml
---
name: status
description: Show project status with git context
disable-model-invocation: true
---

## Current Project Status

Git branch: !`git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "no git"`
Recent commits: !`git log --oneline -3 2>/dev/null || echo "no commits"`
Changed files: !`git status --short 2>/dev/null || echo "clean"`

Based on this context, summarize what's happening in this project
and suggest what to work on next.
```

Test it:

```bash
cd /tmp/kata-05
claude
```

Type: `/status`

#### 3. Create a Personal Global Skill

Create `~/.claude/skills/explain/SKILL.md`:

```yaml
---
name: explain
description: Explain code like I'm a junior developer
argument-hint: [file-or-concept]
---

Explain $ARGUMENTS in simple terms:

1. What does it do? (one sentence)
2. How does it work? (step by step, plain language)
3. Why is it done this way? (design decisions)
4. What would break if we changed it? (risks)

Use analogies where helpful. Avoid jargon.
```

This skill is now available in **all** your projects:

```bash
cd /tmp/kata-05
claude
```

Type: `/explain src/app.js`

#### 4. Browse Available Skills

In Claude, type `/` and pause — see the autocomplete list of all available skills.

### Discussion Points

- What repetitive prompts could you turn into skills for your team?
- When would you use `disable-model-invocation: true`?
- How would you share skills across a team? (hint: `.claude/skills/` in git)
