# Kata 07: Building a Custom MCP Server (Optional, ~20 min)

## Theory

The **Model Context Protocol (MCP)** is an open standard that lets AI tools like Claude Code connect to external data sources and services through a unified interface. Instead of building custom integrations, you build an MCP server that any MCP-compatible client can use.

### How MCP Works

```
Claude Code (client)  <--stdio/SSE-->  MCP Server  <-->  Your Data/API
```

An MCP server exposes three primitives:

| Primitive | Purpose | Example |
|-----------|---------|---------|
| **Tools** | Actions Claude can call | `create_ticket`, `query_db`, `deploy` |
| **Resources** | Read-only data endpoints | `file://config`, `db://users` |
| **Prompts** | Reusable prompt templates | `summarize-pr`, `review-code` |

### Configuring MCP Servers in Claude Code

In `.claude/settings.json` or `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["./mcp-server/index.js"],
      "cwd": "/path/to/project"
    }
  }
}
```

For remote SSE servers:

```json
{
  "mcpServers": {
    "remote-api": {
      "url": "https://my-mcp-server.example.com/sse"
    }
  }
}
```

### Transport Options

| Transport | Use case |
|-----------|----------|
| **stdio** | Local servers, runs as a subprocess |
| **SSE** | Remote servers, HTTP-based streaming |

### The MCP TypeScript SDK

```bash
npm install @modelcontextprotocol/sdk
```

Minimal server structure:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool("my-tool", { param: z.string() }, async ({ param }) => {
  return { content: [{ type: "text", text: `Result: ${param}` }] };
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## Exercise

### Setup

**macOS / Linux:**

```bash
mkdir -p /tmp/kata-07/mcp-server && cd /tmp/kata-07
git init
npm init -y
npm install @modelcontextprotocol/sdk zod
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path $env:TEMP\kata-07\mcp-server | Set-Location
Set-Location $env:TEMP\kata-07
git init
npm init -y
npm install @modelcontextprotocol/sdk zod
```

### Tasks

#### 1. Build a Simple Tool Server

Create `mcp-server/index.js`:

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "workshop-tools",
  version: "1.0.0",
});

// Tool: Generate a UUID
server.tool("generate_uuid", "Generate a random UUID v4", {}, async () => {
  const uuid = crypto.randomUUID();
  return {
    content: [{ type: "text", text: uuid }],
  };
});

// Tool: Count words in text
server.tool(
  "count_words",
  "Count words in a given text",
  { text: z.string().describe("The text to count words in") },
  async ({ text }) => {
    const count = text.trim().split(/\s+/).filter(Boolean).length;
    return {
      content: [{ type: "text", text: `Word count: ${count}` }],
    };
  }
);

// Tool: Format JSON
server.tool(
  "format_json",
  "Pretty-print a JSON string",
  { json: z.string().describe("JSON string to format") },
  async ({ json }) => {
    try {
      const formatted = JSON.stringify(JSON.parse(json), null, 2);
      return {
        content: [{ type: "text", text: formatted }],
      };
    } catch (e) {
      return {
        content: [{ type: "text", text: `Invalid JSON: ${e.message}` }],
        isError: true,
      };
    }
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

Add `"type": "module"` to your `package.json`.

#### 2. Register with Claude Code

Create `.claude/settings.json`:

```json
{
  "mcpServers": {
    "workshop-tools": {
      "command": "node",
      "args": ["mcp-server/index.js"]
    }
  }
}
```

#### 3. Test Your MCP Server

```bash
cd /tmp/kata-07
claude
```

Try these prompts:

1. `"Generate a UUID for me"` — Claude should use your `generate_uuid` tool
2. `"Count the words in: The quick brown fox jumps over the lazy dog"` — uses `count_words`
3. `"Format this JSON: {\"name\":\"test\",\"value\":42}"` — uses `format_json`

Check that Claude lists your tools: type `/mcp` in the session to see connected servers.

#### 4. Add a Resource (Bonus)

Extend `mcp-server/index.js` — add a resource that serves project info:

```javascript
server.resource("project-info", "project://info", async (uri) => {
  const pkg = JSON.parse(
    await import("fs").then((fs) =>
      fs.readFileSync("./package.json", "utf-8")
    )
  );
  return {
    contents: [
      {
        uri: uri.href,
        mimeType: "application/json",
        text: JSON.stringify(pkg, null, 2),
      },
    ],
  };
});
```

#### 5. Add Error Handling and Validation (Bonus)

Extend one of your tools with proper input validation using Zod schemas:

```javascript
server.tool(
  "calculate",
  "Perform basic arithmetic",
  {
    a: z.number().describe("First number"),
    b: z.number().describe("Second number"),
    operation: z.enum(["add", "subtract", "multiply", "divide"]),
  },
  async ({ a, b, operation }) => {
    let result;
    switch (operation) {
      case "add": result = a + b; break;
      case "subtract": result = a - b; break;
      case "multiply": result = a * b; break;
      case "divide":
        if (b === 0) {
          return { content: [{ type: "text", text: "Error: Division by zero" }], isError: true };
        }
        result = a / b;
        break;
    }
    return { content: [{ type: "text", text: `${a} ${operation} ${b} = ${result}` }] };
  }
);
```

### Discussion Points

- What internal APIs or databases at your company would benefit from an MCP server?
- When would you use stdio vs SSE transport?
- How would you handle authentication for an MCP server that accesses sensitive data?
- How do MCP tools compare to Claude Code skills for extending functionality?
