# Kata 08: Expert Topics — Plugin Marketplaces & Subagent Architecture (Info/Reference)

> This section is a reference guide, not a hands-on kata. It covers advanced Claude Code internals and ecosystem patterns for developer-expert audiences.

---

## Topic A: Private Plugin Marketplaces

### The Problem

Teams want to share Claude Code extensions (skills, MCP servers, hooks, CLAUDE.md templates) internally without publishing to the world. There's no official "plugin store" — so you build your own distribution layer.

### Architecture Patterns

#### 1. Git-Based Marketplace (Simplest)

A monorepo or multi-repo structure where each plugin is a directory:

```
company-claude-plugins/
  skills/
    code-review/SKILL.md
    security-scan/SKILL.md
    deploy-staging/SKILL.md
  mcp-servers/
    jira-bridge/
    internal-api/
  hooks/
    pre-commit-lint.sh
    pre-commit-lint.ps1
    block-secrets.sh
    block-secrets.ps1
  templates/
    CLAUDE.md.backend
    CLAUDE.md.frontend
```

Distribution: teams clone/submodule the repo and symlink into `~/.claude/` or `.claude/`.

**macOS / Linux install script:**

```bash
#!/bin/bash
PLUGIN_REPO="$HOME/.claude-plugins"
git clone git@github.com:yourco/claude-plugins.git "$PLUGIN_REPO"

# Symlink global skills
ln -sf "$PLUGIN_REPO/skills/"* "$HOME/.claude/skills/"

# Symlink hooks (project-level, run from project root)
# ln -sf "$PLUGIN_REPO/hooks/"*.sh ".claude/hooks/"
```

**Windows (PowerShell) install script:**

```powershell
$PluginRepo = "$env:USERPROFILE\.claude-plugins"
git clone git@github.com:yourco/claude-plugins.git $PluginRepo

# Symlink global skills
Get-ChildItem "$PluginRepo\skills" -Directory | ForEach-Object {
    New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\$($_.Name)" -Target $_.FullName -Force
}
```

#### 2. NPM/Registry-Based Marketplace

Publish MCP servers and tooling as scoped npm packages to a private registry (Verdaccio, GitHub Packages, Artifactory):

```bash
npm install --save-dev @yourco/claude-mcp-jira @yourco/claude-skill-review
```

Then reference in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "jira": {
      "command": "npx",
      "args": ["@yourco/claude-mcp-jira"],
      "env": {
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

#### 3. OCI/Container-Based Distribution

For MCP servers that need isolated environments:

```json
{
  "mcpServers": {
    "secure-db": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "registry.yourco.com/claude-mcp-db:latest"],
      "env": {
        "DB_CONNECTION_STRING": "${DB_CONN}"
      }
    }
  }
}
```

### Governance Considerations

| Concern | Solution |
|---------|----------|
| **Versioning** | Pin plugin versions in `package.json` or git submodule commits |
| **Approval** | PR-based plugin submissions with security review |
| **Auditing** | Hooks log all tool usage; MCP servers can log internally |
| **Rollback** | Git tags/npm versions allow instant rollback |
| **Access control** | Private repos/registries with team-scoped permissions |

---

## Topic B: Subagent Flow & Architecture

### What Are Subagents?

Claude Code can spawn **subagents** — independent Claude instances that run in isolated context windows. The parent agent delegates work and receives results without polluting its own context.

### How Subagents Work Internally

```
User Prompt
    |
    v
Main Agent (Opus/Sonnet)
    |
    |-- Task tool call: subagent_type="Explore"
    |       |
    |       v
    |   Subagent (inherits or overrides model)
    |       |- Has own context window
    |       |- Has restricted tool access based on type
    |       |- Returns single message to parent
    |       |
    |       v
    |   Result returned to Main Agent
    |
    |-- Task tool call: subagent_type="Bash"
    |       |
    |       v
    |   Another independent subagent
    |       |- Separate context
    |       |- Separate tool permissions
    |       v
    |   Result returned to Main Agent
    |
    v
Main Agent synthesizes results
```

### Built-in Subagent Types

| Type | Tools Available | Use Case |
|------|----------------|----------|
| **Explore** | Read, Glob, Grep, WebFetch, WebSearch | Codebase exploration, research |
| **Bash** | Bash only | Command execution, git ops |
| **Plan** | Read, Glob, Grep (no writes) | Architecture planning |
| **general-purpose** | All tools | Complex multi-step tasks |

### Subagent Patterns for Expert Users

#### Pattern 1: Parallel Research

The main agent spawns multiple Explore subagents simultaneously to research different aspects:

```
Main: "Refactor the auth system"
  |
  |-- Explore: "Find all auth-related files"
  |-- Explore: "Find all session management code"
  |-- Explore: "Find all middleware patterns"
  |
  v (all 3 return in parallel)
Main: Synthesizes findings, creates plan
```

#### Pattern 2: Background Workers

Long-running tasks (builds, tests) run in background subagents:

```
Main: Editing code
  |
  |-- Bash (background): "npm run test:watch"
  |-- Bash (background): "npm run build"
  |
  v
Main: Continues working, checks results later
```

#### Pattern 3: Context Window Protection

Subagents protect the main context from large outputs:

```
Main: needs to analyze a 5000-line file
  |
  |-- Explore: reads the file, summarizes key findings
  |       (5000 lines stay in subagent context, only summary returns)
  |
  v
Main: receives compact summary, context stays clean
```

### Subagent Communication Model

- **Input**: The parent sends a text prompt. If the subagent type has "access to current context", it also sees the full conversation history.
- **Output**: The subagent returns a single text message. No structured data, no streaming — just text.
- **Isolation**: Each subagent has its own context window. It cannot see other subagents or modify the parent's state.
- **Resumption**: Subagents can be resumed by ID, preserving their full context from previous invocations.

### Skills as Subagents

Skills with `context: fork` in their frontmatter run as subagents:

```yaml
---
name: deep-review
context: fork
allowed-tools: Read, Grep, Glob
---

Perform an exhaustive code review of $ARGUMENTS...
```

This keeps the review's large context isolated from the main conversation.

### Advanced: Claude Code SDK & Custom Agents

The `@anthropic-ai/claude-code` SDK lets you build custom agent workflows programmatically:

**macOS / Linux:**

```bash
npm install @anthropic-ai/claude-code
```

**Windows (PowerShell):**

```powershell
npm install @anthropic-ai/claude-code
```

```typescript
import { query } from "@anthropic-ai/claude-code";

// Simple query
const result = await query({
  prompt: "Analyze the auth system and suggest improvements",
  options: {
    maxTurns: 10,
    model: "claude-sonnet-4-5-20250929",
    permissionMode: "plan",
  },
});

// Multi-agent orchestration
const analysis = await query({
  prompt: "Find all security vulnerabilities",
  options: { maxTurns: 5 },
});

const fix = await query({
  prompt: `Fix these issues: ${analysis.result}`,
  options: { maxTurns: 15 },
});
```

### Performance Considerations

| Factor | Impact |
|--------|--------|
| **Context inheritance** | "Access to current context" agents receive full history — costs more tokens |
| **Model selection** | Use `haiku` for fast, simple subagent tasks; `opus` for complex reasoning |
| **Parallelism** | Independent subagents run truly in parallel — significant time savings |
| **Max turns** | Set `max_turns` to prevent runaway subagents |
| **Background mode** | Background agents don't block the main conversation |

---

## Further Reading

- [Model Context Protocol specification](https://modelcontextprotocol.io/)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code SDK on npm](https://www.npmjs.com/package/@anthropic-ai/claude-code)
- [Building MCP Servers guide](https://modelcontextprotocol.io/quickstart/server)
