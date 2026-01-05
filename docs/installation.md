# Installation

Multiple ways to install Gemini MCP Tool, depending on your needs.

## Prerequisites

- Node.js v16.0.0 or higher
- Claude Desktop or Claude Code with MCP support
- Gemini CLI installed (`npm install -g @google/gemini-cli`)

## Method 1: NPX (Recommended)

No installation needed - runs directly:

```json
{
  "mcpServers": {
    "gemini-cli": {
      "command": "npx",
      "args": ["-y", "gemini-mcp-tool-patched"]
    }
  }
}
```

## Method 2: Global Installation

```bash
npm install -g gemini-mcp-tool-patched
```

Then configure:
```json
{
  "mcpServers": {
    "gemini-cli": {
      "command": "gemini-mcp-patched"
    }
  }
}
```

## Method 3: Local Project

```bash
npm install gemini-mcp-tool-patched
```

See [Getting Started](/getting-started) for full setup instructions.


