# MCP Example Implementations Guide

**Version:** 1.4.1  
**Component:** Example Implementations  
**Last Updated:** 2024

## Overview

This guide provides detailed examples of MCP server implementations, demonstrating various patterns and use cases. Each example includes complete code and explanations of key concepts.

## Basic Examples

### 1. Echo Server

A simple server that demonstrates core MCP concepts:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "echo-server",
  version: "1.0.0"
});

// Simple echo tool
server.tool(
  "echo",
  { message: z.string() },
  async ({ message }) => ({
    content: [{
      type: "text",
      text: message
    }]
  })
);

// Echo resource
server.resource(
  "message",
  new ResourceTemplate("message://{text}", { list: undefined }),
  async (uri, { text }) => ({
    contents: [{
      uri: uri.href,
      text: text
    }]
  })
);

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 2. Calculator Server

A server implementing mathematical operations:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "calculator",
  version: "1.0.0"
});

// Basic operations
server.tool(
  "add",
  { a: z.number(), b: z.number() },
  async ({ a, b }) => ({
    content: [{ type: "text", text: String(a + b) }]
  })
);

server.tool(
  "multiply",
  { a: z.number(), b: z.number() },
  async ({ a, b }) => ({
    content: [{ type: "text", text: String(a * b) }]
  })
);

// History resource
const history: string[] = [];
server.resource(
  "history",
  "calc://history",
  async (uri) => ({
    contents: [{
      uri: uri.href,
      text: history.join("\n")
    }]
  })
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Integration Examples

### 1. Database Explorer

A server that provides access to a SQLite database:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import { z } from "zod";
import sqlite3 from "sqlite3";
import { open } from "sqlite";

const server = new McpServer({
  name: "sqlite-explorer",
  version: "1.0.0"
});

// Open database
const db = await open({
  filename: "database.sqlite",
  driver: sqlite3.Database
});

// List tables
server.resource(
  "tables",
  "db://tables",
  async (uri) => {
    const tables = await db.all(
      "SELECT name FROM sqlite_master WHERE type='table'"
    );
    return {
      contents: [{
        uri: uri.href,
        text: tables.map(t => t.name).join("\n")
      }]
    };
  }
);

// Query tool
server.tool(
  "query",
  { sql: z.string() },
  async ({ sql }) => {
    const results = await db.all(sql);
    return {
      content: [{
        type: "text",
        text: JSON.stringify(results, null, 2)
      }]
    };
  }
);

// Start HTTP server
const app = express();
app.get("/mcp", async (req, res) => {
  const transport = new SSEServerTransport("/messages", res);
  await server.connect(transport);
});
```

### 2. File System Server

A server that provides access to a file system:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import * as fs from "fs/promises";
import * as path from "path";

const server = new McpServer({
  name: "fs-server",
  version: "1.0.0"
});

// List directory contents
server.resource(
  "directory",
  new ResourceTemplate("fs://{path}", {
    list: async () => {
      const entries = await fs.readdir(".", { withFileTypes: true });
      return {
        resources: entries.map(entry => ({
          name: entry.name,
          uri: `fs://${entry.name}`
        }))
      };
    }
  }),
  async (uri, { path: dirPath }) => {
    const entries = await fs.readdir(dirPath, { withFileTypes: true });
    return {
      contents: [{
        uri: uri.href,
        text: entries.map(e => 
          `${e.isDirectory() ? "d" : "-"} ${e.name}`
        ).join("\n")
      }]
    };
  }
);

// Read file contents
server.resource(
  "file",
  new ResourceTemplate("file://{path}", { list: undefined }),
  async (uri, { path: filePath }) => {
    const content = await fs.readFile(filePath, "utf-8");
    return {
      contents: [{
        uri: uri.href,
        text: content
      }]
    };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Advanced Examples

### 1. API Gateway

A server that proxies requests to external APIs:

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import { z } from "zod";

const server = new McpServer({
  name: "api-gateway",
  version: "1.0.0"
});

// API endpoints resource
server.resource(
  "endpoints",
  "api://endpoints",
  async (uri) => ({
    contents: [{
      uri: uri.href,
      text: [
        "weather: Get weather data",
        "news: Get news articles",
        "stocks: Get stock prices"
      ].join("\n")
    }]
  })
);

// Weather API
server.tool(
  "weather",
  { city: z.string() },
  async ({ city }) => {
    const response = await fetch(
      `https://api.weather.com/v1/city/${city}`
    );
    const data = await response.json();
    return {
      content: [{
        type: "text",
        text: JSON.stringify(data, null, 2)
      }]
    };
  }
);

// News API
server.tool(
  "news",
  { query: z.string() },
  async ({ query }) => {
    const response = await fetch(
      `https://api.news.com/v2/search?q=${query}`
    );
    const data = await response.json();
    return {
      content: [{
        type: "text",
        text: JSON.stringify(data, null, 2)
      }]
    };
  }
);

const app = express();
app.get("/mcp", async (req, res) => {
  const transport = new SSEServerTransport("/messages", res);
  await server.connect(transport);
});
```

## Best Practices

1. **Server Design**
   - Keep responsibilities focused
   - Handle errors gracefully
   - Validate inputs thoroughly
   - Document capabilities

2. **Resource Design**
   - Use meaningful URIs
   - Implement list when appropriate
   - Keep responses concise
   - Cache when possible

3. **Tool Design**
   - Validate parameters strictly
   - Handle async operations
   - Provide clear feedback
   - Rate limit if needed

4. **Security**
   - Validate all inputs
   - Sanitize outputs
   - Implement authentication
   - Rate limit requests

## Related Documentation

- [Server Architecture](server-architecture.md)
- [Resource Management](resources.md)
- [Tool Development](tools.md)
- [Security Guide](security.md) 

---
<div align="center">
  <sub>Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se)</sub>
  <br>
  Started: January 25, 2025 | Last Updated: January 27, 2025
</div>