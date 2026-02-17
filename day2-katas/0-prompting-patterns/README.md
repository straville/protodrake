# Prompting Patterns for AI-Assisted Development

Universal patterns that work across Claude Code, Claude.ai, Cursor, and Copilot — plus where each tool needs a different approach.

---

## Core Patterns

### 1. Be Specific About Output

"Make this better" gives you a coin flip. "Refactor this function to use async/await and remove the callback parameter" gives you exactly what you need. State the transformation, not the vibe. Include the format if it matters: "return a markdown table," "use bullet points," "just the shell command, no explanation."

### 2. Front-Load Context and Constraints

Put boundaries before the ask. Claude reads top-down and anchors on early instructions.

**Weak:** "Add pagination to the users endpoint. Oh and only use existing utils."
**Strong:** "Using only the existing utils in `src/lib/`, add cursor-based pagination to the users endpoint."

The constraint arrives before the creative work begins, so the model plans within bounds rather than backtracking.

### 3. Show the Shape You Want

A 3-line example of desired output beats a paragraph of description. If you want a specific format, show it:

```
Format each result like:
[STATUS] filename — reason
```

This works everywhere — chat, CLI, inline completions. Models are excellent at pattern-matching from examples.

### 4. Decompose, Don't Monologue

One clear task per message outperforms a wall of requirements. Let the model finish one step, verify it, then direct the next. This keeps the model focused and gives you checkpoints to course-correct. A 15-step prompt might get steps 1-8 right and hallucinate the rest. Five 3-step prompts rarely go sideways.

### 5. Constrain Scope Explicitly

AI models are eager — they'll "improve" surrounding code, add error handling you didn't ask for, and refactor imports while fixing a typo. Set explicit boundaries:

- "Only modify files in `src/auth/`"
- "Don't change the tests"
- "Fix the null check on line 42 — nothing else"

In tools with file access (Claude Code, Cursor), this prevents drift across your codebase. In chat (Claude.ai), it prevents the model from rewriting your entire snippet when you asked about one function.

### 6. Ask for Plans Before Code

"How would you approach this?" is the highest-leverage prompt in your toolkit. It surfaces misunderstandings, reveals the model's assumptions, and lets you steer before any code is written. This is especially valuable for multi-file changes, architectural decisions, or unfamiliar codebases.

### 7. Correct by Contrast

"Don't use class inheritance, use composition instead" is clearer than just "use composition." The contrast gives the model a negative example to avoid and a positive one to follow. This also works for style: "Don't add JSDoc comments. Use inline comments only where logic isn't obvious."

### 8. Iterate, Don't Restart

Build on the conversation. "Now add error handling to what you just wrote" preserves context and avoids re-explaining the setup. Starting a new conversation to refine output throws away everything the model learned about your intent. Use follow-ups aggressively — that's what the context window is for.

### 9. Use Persistent Instructions for Repeated Conventions

If you find yourself repeating the same instructions ("always use bun not npm," "we use tabs not spaces," "prefer named exports"), move them into persistent configuration. Every tool has a mechanism for this — use it instead of burning tokens repeating yourself.

### 10. Name the Task Type

"This is a bug fix" or "this is a refactor" primes the model differently. A bug fix should be surgical — minimal change, regression test. A refactor should preserve behavior. A new feature needs more creativity. Naming the type sets the right posture.

---

## Prompting by Tool: Use Cases and Examples

Each tool has a different interaction model, context window, and set of capabilities. The same intent often needs a different prompt shape.

### Claude Code (CLI / agentic)

Claude Code has full filesystem access, runs commands, and reads your `CLAUDE.md` files. Prompts should leverage this — point at files, ask for multi-step operations, and let it explore.

| Use Case | Prompt Example |
|----------|---------------|
| **Bug fix** | `"The /api/users endpoint returns 500 when email is null. Find the handler, fix the null check, and add a unit test."` |
| **Feature** | `"Add rate limiting to all POST routes. Use the existing redis client in src/cache/client.ts. 100 requests per minute per IP."` |
| **Refactor** | `"Move all raw SQL queries from src/services/ into src/db/queries/. Update all imports. Don't change behavior."` |
| **Explore** | `"How does the auth flow work? Trace from login route to session creation."` |
| **Plan first** | `"I need to add WebSocket support for live notifications. Enter plan mode and propose an approach before writing code."` |
| **Multi-step with sub-agents** | `"Add a new 'teams' feature: DB schema, API endpoints, and frontend page. Use sub-agents for backend and frontend in parallel."` |

**Key trait:** You can be ambitious. Claude Code handles multi-file changes, runs tests, and self-corrects. Tell it the outcome, let it figure out the path. Use `CLAUDE.md` so you don't re-explain your stack every session.

### Claude.ai (chat)

No file access, no execution. You bring the code as pasted snippets. Prompts need to be self-contained with all relevant context included in the message.

| Use Case | Prompt Example |
|----------|---------------|
| **Debug a snippet** | `"This function throws 'cannot read property of undefined' on line 12. Here's the function: [paste]. What's wrong and how do I fix it?"` |
| **Design / architecture** | `"I'm building a notification system. Users can get email, SMS, or push. I need it extensible for new channels. Suggest a pattern — no code yet, just the design."` |
| **Code review** | `"Review this PR diff for security issues. Focus on SQL injection and auth bypass: [paste diff]"` |
| **Explain** | `"Explain what this regex does step by step: /^(?=.*[A-Z])(?=.*\d)[A-Za-z\d@$!%*?&]{8,}$/"` |
| **Generate with constraints** | `"Write a PostgreSQL migration that adds a 'teams' table with id (uuid), name (varchar 255, not null, unique), created_at (timestamptz). Use IF NOT EXISTS."` |
| **Compare approaches** | `"What are the tradeoffs between cursor-based and offset-based pagination for a REST API with ~1M rows?"` |

**Key trait:** You are the context. Paste what matters, trim what doesn't. Claude.ai excels at design conversations, explanations, and working through a problem before you touch code. Use Projects to upload reference files so you don't re-paste every message.

### Cursor (IDE-integrated)

Cursor sees your open files and codebase. It can edit inline (Tab), chat in a sidebar, or run in agent mode (Composer). Prompts vary by mode.

| Use Case | Prompt Example |
|----------|---------------|
| **Inline edit (Cmd+K)** | Select a function, then: `"Convert to async/await"` or `"Add input validation with zod"` — short, surgical. |
| **Chat with codebase** | `"@codebase How is authentication handled? Which middleware checks the JWT?"` — use `@` references to pull in context. |
| **Composer (agent)** | `"Add a dark mode toggle to the settings page. Use the existing theme context in src/contexts/theme.tsx. Update the header component to include the toggle."` |
| **Fix error** | `"@file:src/api/users.ts This throws a type error on line 34. The 'role' field is optional but I'm accessing it without a null check."` |
| **Generate test** | `"@file:src/utils/format.ts Write unit tests for all exported functions. Use vitest. Cover edge cases."` |

**Key trait:** Use `@` references heavily — `@file`, `@codebase`, `@docs`, `@web`. Cursor's power is contextual awareness within the IDE. Short prompts with precise references outperform long descriptions. For Composer (agent mode), prompts resemble Claude Code — outcome-oriented, multi-step.

### GitHub Copilot / VS Code

Copilot is primarily an autocomplete engine (inline suggestions) with a chat sidebar. It excels at continuing patterns and generating boilerplate. Prompts are often implicit (the code you're already writing) or brief comments.

| Use Case | Prompt Example |
|----------|---------------|
| **Inline completion** | Type a function signature and let Copilot finish the body. Write a descriptive function name — `calculateDiscountedPrice(items, couponCode)` — and Tab to accept. |
| **Comment-driven** | `// Parse CSV file, skip header row, return array of {name, email, role} objects` — then let Copilot generate the function below. |
| **Chat sidebar** | `"Explain what #selection does"` or `"Write a unit test for #file:utils.ts #selection"` — use `#` to reference context. |
| **Fix suggestion** | `"/fix"` in chat with an error selected — Copilot proposes a fix inline. |
| **Generate from pattern** | Write one test case manually, then start the next `it()` block — Copilot picks up the pattern and generates similar tests. |

**Key trait:** Copilot works best when you lead by example. Write one instance of the pattern you want, and it'll replicate it. The chat is useful but less capable for multi-step reasoning than Claude Code or Cursor. Think of it as a fast autocomplete with a chat fallback, not an agent.

---

## Quick Comparison

| Capability | Claude Code | Claude.ai | Cursor | Copilot |
|-----------|-------------|-----------|--------|---------|
| File system access | Full | None | Full (workspace) | Limited (open files) |
| Command execution | Yes | No | Yes (agent mode) | No |
| Multi-file edits | Native | Manual paste | Native | Limited |
| Persistent config | CLAUDE.md | Projects | .cursorrules | Instructions file |
| Best for | Multi-step tasks, refactors, full features | Design, review, explanation, brainstorming | In-IDE edits, codebase Q&A | Autocomplete, boilerplate, pattern continuation |
| Prompt style | Outcome-oriented, ambitious | Self-contained, context-rich | Reference-heavy (`@`), surgical | Implicit (code context), comment-driven |
