# Kata 02: Security & Containment (~15 min)

## Theory

Claude Code has multiple security layers that prevent it from doing harmful things — even if prompted to.

### Permission System

Tools have different trust levels:

| Tool Type | Needs Approval? |
|-----------|----------------|
| Read, Grep, Glob (read-only) | Never |
| Edit, Write (file modification) | Yes |
| Bash (shell commands) | Yes |
| WebFetch (network) | Yes |

### Permission Rules

Rules are evaluated in order: **deny > ask > allow**. Deny always wins.

Configure in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": ["Bash(npm run test)", "Bash(git status)"],
    "deny": ["Bash(rm -rf *)", "Bash(curl *)"]
  }
}
```

### Sandboxing

OS-level isolation for bash commands:
- **macOS**: Uses Seatbelt sandbox profiles
- **Linux**: Uses bubblewrap (bwrap)

Enable with `/sandbox` command. Restricts filesystem access and network connections.

### File Boundaries

- Claude can **read** files anywhere on your system
- Claude can only **write** to the current working directory and subdirectories
- Use `--add-dir` flag to grant access to additional directories

---

## Exercise

### Setup

```bash
mkdir -p /tmp/kata-02/src && cd /tmp/kata-02
git init
echo "DB_PASSWORD=supersecret123" > .env
echo "console.log('app');" > src/app.js
echo '{ "name": "kata-02" }' > package.json
```

### Tasks

#### 1. Create Permission Rules

Create a `.claude/settings.json` that:
- Allows `git status` and `git diff` without prompting
- Denies any `rm -rf` commands
- Denies reading `.env` files

```bash
mkdir -p /tmp/kata-02/.claude
```

Create `/tmp/kata-02/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Read(.env)"
    ]
  }
}
```

#### 2. Test the Rules

```bash
cd /tmp/kata-02
claude
```

Try these prompts and observe the behavior:

1. `"Run git status"` — should auto-approve
2. `"Read the .env file"` — should be denied
3. `"Delete the src folder with rm -rf"` — should be denied
4. `"Read src/app.js"` — should work (read-only, always allowed)

#### 3. Explore the Permission Menu

While in a Claude session:

- Type `/permissions` to see the interactive permission manager
- Review current rules
- Try adding a new allow rule interactively

#### 4. Test File Boundaries

```bash
cd /tmp/kata-02
claude
```

Ask Claude: `"Create a file at /tmp/outside-project/test.txt"`

- Observe: Claude should not be able to write outside the project directory
- Then ask: `"Create a file at src/new-file.js"` — this should work

### Discussion Points

- How would you configure permissions for a CI/CD pipeline?
- What's the difference between sandboxing and permissions?
- Why can Claude read system files but not write to them?
