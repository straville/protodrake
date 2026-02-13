# Kata 0: Directory & Config Files Example (Reference)

## Overview

This is a reference example showing the full directory structure of an advanced Claude Code project. It demonstrates how all configuration features fit together: `CLAUDE.md` hierarchy, `.claude/` settings, rules, skills, hooks, MCP servers, and CI integration.

Use this as a blueprint when setting up Claude Code on a real codebase.

---

## Directory Structure

```
my-saas-app/
├── CLAUDE.md                              # Root project instructions
├── CLAUDE.local.md                        # Personal overrides (gitignored)
├── .mcp.json                              # Project-scoped MCP servers
│
├── .claude/
│   ├── settings.json                      # Project settings (shared via git)
│   ├── settings.local.json                # Local settings overrides (gitignored)
│   ├── rules/
│   │   ├── code-style.md                  # Unconditional style rules
│   │   ├── api-guidelines.md              # Path-scoped API rules
│   │   ├── testing.md                     # Testing conventions
│   │   └── security.md                    # Security rules
│   └── skills/
│       ├── deploy/
│       │   └── SKILL.md                   # /deploy slash command
│       ├── db-migrate/
│       │   └── SKILL.md                   # /db-migrate slash command
│       ├── gen-api-client/
│       │   ├── SKILL.md                   # /gen-api-client slash command
│       │   └── template.hbs              # Supporting template file
│       └── review-security/
│           └── SKILL.md                   # /review-security slash command
│
├── scripts/
│   ├── hooks/
│   │   ├── pre-commit-lint.sh             # Lint before commit
│   │   ├── validate-migrations.sh         # Check Prisma migrations
│   │   └── check-secrets.sh              # Detect leaked secrets
│   ├── mcp/
│   │   ├── package.json                   # MCP server deps
│   │   └── project-context-server.ts      # Custom MCP server
│   └── ci/
│       └── claude-review.sh               # CI integration for PR review
│
├── src/
│   ├── CLAUDE.md                          # Frontend-specific rules
│   ├── api/
│   │   └── CLAUDE.md                      # API-specific rules
│   ├── components/
│   └── lib/
│
├── docs/
│   ├── architecture.md
│   └── onboarding.md
│
├── .gitignore
└── README.md
```

---

## How It All Connects

| Feature | File(s) | Purpose |
|---------|---------|---------|
| **Project context** | `CLAUDE.md` | Commands, conventions, architecture |
| **Personal overrides** | `CLAUDE.local.md` | Your local env quirks (gitignored) |
| **Subfolder rules** | `src/CLAUDE.md`, `src/api/CLAUDE.md` | Auto-loaded when Claude touches those dirs |
| **Modular rules** | `.claude/rules/*.md` | Path-scoped style/security/testing rules |
| **Permissions** | `.claude/settings.json` | Allowlist/denylist for tool use |
| **Hooks** | `.claude/settings.json` → `scripts/hooks/` | PreToolUse (block secrets), PostToolUse (lint), Stop (check migrations) |
| **Skills** | `.claude/skills/*/SKILL.md` | `/deploy`, `/db-migrate`, `/gen-api-client`, `/review-security` |
| **MCP servers** | `.mcp.json` → `scripts/mcp/` | DB introspection, Linear issues, Sentry errors |
| **CI integration** | `scripts/ci/claude-review.sh` | Automated PR review in GitHub Actions |
