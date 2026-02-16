```
                                                        \ | /
                                                       --\*/--
                          ___                            / | \
                         /   \
                        | ^ ^ |
                         \_u_/
                      _____|_____
          .---.     /     |     \     .---.
         | O  |---/   /---|---\  \---| O  |
         | /\ |  |   /    |    \  |  | /\ |
         |/  \|  |  |     |     | |  |/  \|
         '----'  |  |     |     | |  '----'
                  \  \    |    /  /
                   \  ----|---  /
                    |    / \   |    how Claude perceives me
                    |   /   \  |    Authored by Claude
                    |  /     \ |
                    | /       \|
                    |/         '
                    '
```

# Protodrake — Claude Code Day 2 Workshop

A hands-on workshop for developers learning Claude Code's most powerful features. 8 katas, 1.5 hours, zero fluff.

## Who Is This For?

Developers who have used Claude Code at least once and want to go deeper. You should be comfortable with a terminal and have Claude Code installed (`npm install -g @anthropic-ai/claude-code` or `brew install claude-code`).

## Workshop Schedule (~90 min)

| # | Kata | Time | Description |
|---|------|------|-------------|
| 0 | [Directory & Config Files Example](day2-katas/0-directory-and-config-files-example/) | Reference | Full project structure blueprint with all Claude Code config features |
| 00 | [CLAUDE.md Files & Sub-Agents](day2-katas/00-CLAUDE.mds-subagents-example/) | Reference | Layered CLAUDE.md hierarchy with reusable sub-agent architecture |
| 01 | [Agent Modes](day2-katas/01-agent-modes/) | ~15 min | Permission modes, Plan mode, Shift+Tab cycling |
| 02 | [Security & Containment](day2-katas/02-security-containment/) | ~15 min | Permission rules, sandboxing, file boundaries |
| 03 | [Variables & Configuration](day2-katas/03-variables/) | ~15 min | Settings layers, CLAUDE.md, environment variables |
| 04 | [Hotkeys & Shortcuts](day2-katas/04-hotkeys/) | ~15 min | Keyboard shortcuts, Vim mode, custom keybindings |
| 05 | [Skills (Custom Slash Commands)](day2-katas/05-skills/) | ~15 min | Creating reusable prompt templates as `/commands` |
| 06 | [Hooks (Lifecycle Automation)](day2-katas/06-hooks/) | ~15 min | PreToolUse/PostToolUse hooks, blocking, logging |
| 07 | [Custom MCP Server](day2-katas/07-custom-mcp-server/) | ~20 min | *(Optional)* Building an MCP server from scratch |
| 08 | [Expert Topics](day2-katas/08-expert-topics/) | Reference | Private plugin marketplaces, subagent architecture |

## Prerequisites

- Claude Code installed and authenticated (`claude` command works)
- Node.js 18+ (for kata 07)
- Git
- A terminal (works on macOS, Linux, and Windows via PowerShell)

## Cross-Platform Support

All katas include setup instructions for:
- **macOS** / **Linux** (bash)
- **Windows** (PowerShell)

## How to Use

1. Clone this repo
2. Work through katas 01-06 in order (each builds on prior concepts)
3. Kata 07 is optional if you have extra time or interest in MCP
4. Kata 08 is a reference doc — read at your own pace

Each kata has:
- **Theory** — concise overview of the feature (2-3 min read)
- **Exercise** — hands-on tasks to run locally (10-12 min)
- **Discussion Points** — team conversation starters

## Running the Workshop

**As a facilitator:**
- Budget 15 min per kata for katas 01-06
- Demo each kata briefly before participants start
- Use discussion points for group debriefs between katas
- Kata 07 (MCP) works best as a live-coding demo if time is short

**Self-paced:**
- Work through at your own speed
- Each kata is self-contained with its own setup/teardown
