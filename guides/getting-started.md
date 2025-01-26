# Getting Started with MCP Development

**Version:** 1.4.1  
**Component:** Getting Started Guide  
**Last Updated:** 2024

## Overview

This guide will help you get started with Model Context Protocol (MCP) development, covering both client and server implementations. We'll walk through the basics of setting up your development environment, creating your first MCP server, and integrating with LLM applications.

## Prerequisites

1. **Node.js Environment**
   ```bash
   # Install Node.js (v18 or later recommended)
   nvm install 18
   nvm use 18
   ```

2. **TypeScript Setup**
   ```bash
   # Install TypeScript
   npm install -g typescript
   ```

3. **MCP SDK Installation**
   ```bash
   # Create a new project
   mkdir my-mcp-project
   cd my-mcp-project
   npm init -y
   
   # Install MCP SDK
   npm install @modelcontextprotocol/sdk
   ```

## Your First MCP Server

### 1. Basic Server Setup

```typescript
// src/server.ts
import { McpServer } from "@modelcontextprotocol/sdk";
import { z } from "zod";

class SimpleServer {
  private server: McpServer;
  
  constructor() {
    this.server = new McpServer({
      name: "simple-server",
      version: "1.0.0"
    });
    
    this.registerCapabilities();
    this.registerResources();
    this.registerTools();
  }
  
  private registerCapabilities() {
    this.server.registerCapabilities({
      resources: {
        subscribe: true,
        listChanged: true
      },
      tools: {
        listChanged: true
      }
    });
  }
  
  private registerResources() {
    this.server.resource("hello", "/hello", {
      read: async () => ({
        contents: [{
          type: "text",
          text: "Hello, World!"
        }]
      })
    });
  }
  
  private registerTools() {
    this.server.tool("greet", {
      description: "Generate a greeting",
      inputSchema: z.object({
        name: z.string()
      }),
      callback: async (args) => ({
        content: [{
          type: "text",
          text: `Hello, ${args.name}!`
        }]
      })
    });
  }
  
  async start() {
    // Use stdio transport for simplicity
    const transport = new StdioTransport();
    await this.server.connect(transport);
  }
}

// Start the server
const server = new SimpleServer();
server.start().catch(console.error);
```

### 2. Project Structure

```
my-mcp-project/
├── package.json
├── tsconfig.json
├── src/
│   ├── server.ts
│   ├── resources/
│   │   └── hello.ts
│   ├── tools/
│   │   └── greet.ts
│   └── types/
│       └── index.ts
└── README.md
```

### 3. Configuration Files

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "strict": true,
    "outDir": "dist"
  },
  "include": ["src"]
}
```

```json
// package.json
{
  "name": "my-mcp-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "tsc -w"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.4.1",
    "zod": "^3.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

## Adding Features

### 1. Resource Implementation

```typescript
// src/resources/file-system.ts
import { McpServer, Resource, ReadResourceResult } from "@modelcontextprotocol/sdk";
import { promises as fs } from "fs";
import { join } from "path";

export class FileSystemResource {
  constructor(private server: McpServer, private basePath: string) {
    this.register();
  }
  
  private register() {
    this.server.resource("files", "/files/{path}", {
      list: this.listFiles.bind(this),
      read: this.readFile.bind(this)
    });
  }
  
  private async listFiles(): Promise<Resource[]> {
    const files = await fs.readdir(this.basePath);
    return files.map(file => ({
      uri: `/files/${file}`,
      name: file
    }));
  }
  
  private async readFile(uri: URL): Promise<ReadResourceResult> {
    const path = uri.pathname.replace("/files/", "");
    const content = await fs.readFile(join(this.basePath, path), "utf-8");
    return {
      contents: [{
        type: "text",
        text: content
      }]
    };
  }
}
```

### 2. Tool Implementation

```typescript
// src/tools/calculator.ts
import { McpServer, CallToolResult } from "@modelcontextprotocol/sdk";
import { z } from "zod";

export class CalculatorTool {
  constructor(private server: McpServer) {
    this.register();
  }
  
  private register() {
    this.server.tool("calculate", {
      description: "Perform basic calculations",
      inputSchema: z.object({
        expression: z.string()
      }),
      callback: this.calculate.bind(this)
    });
  }
  
  private async calculate(
    args: { expression: string }
  ): Promise<CallToolResult> {
    try {
      const result = eval(args.expression);
      return {
        content: [{
          type: "text",
          text: String(result)
        }]
      };
    } catch (error) {
      return {
        content: [{
          type: "text",
          text: `Error: ${error.message}`
        }],
        isError: true
      };
    }
  }
}
```

## Testing Your Server

### 1. Manual Testing

```typescript
// src/test.ts
import { Client, StdioTransport } from "@modelcontextprotocol/sdk";

async function test() {
  const client = new Client({
    name: "test-client",
    version: "1.0.0"
  });
  
  const transport = new StdioTransport();
  await client.connect(transport);
  
  // Test resources
  const resources = await client.listResources();
  console.log("Resources:", resources);
  
  // Test tools
  const result = await client.callTool({
    name: "greet",
    arguments: { name: "Alice" }
  });
  console.log("Tool result:", result);
}

test().catch(console.error);
```

### 2. Automated Testing

```typescript
// src/__tests__/server.test.ts
import { Client, McpServer, InMemoryTransport } from "@modelcontextprotocol/sdk";

describe("SimpleServer", () => {
  let server: McpServer;
  let client: Client;
  
  beforeEach(async () => {
    server = new SimpleServer();
    client = new Client({
      name: "test-client",
      version: "1.0.0"
    });
    
    const transport = new InMemoryTransport();
    await Promise.all([
      server.connect(transport.server),
      client.connect(transport.client)
    ]);
  });
  
  afterEach(async () => {
    await Promise.all([
      server.close(),
      client.close()
    ]);
  });
  
  test("greet tool", async () => {
    const result = await client.callTool({
      name: "greet",
      arguments: { name: "Alice" }
    });
    
    expect(result.content).toEqual([{
      type: "text",
      text: "Hello, Alice!"
    }]);
  });
});
```

## Best Practices

1. **Project Organization**
   - Separate concerns (resources, tools, types)
   - Use meaningful file names
   - Implement proper error handling
   - Add comprehensive tests

2. **Resource Design**
   - Use clear URI structures
   - Implement proper validation
   - Handle edge cases
   - Document behavior

3. **Tool Implementation**
   - Provide clear descriptions
   - Validate inputs thoroughly
   - Return meaningful errors
   - Support progress tracking

4. **Error Handling**
   - Use appropriate error codes
   - Provide helpful messages
   - Implement logging
   - Handle cleanup

## Next Steps

1. **Learn More**
   - Read the [Protocol Specification](../reference/protocol-spec.md)
   - Explore [Example Implementations](../examples/implementations.md)
   - Study [Advanced Topics](../guides/advanced.md)

2. **Contribute**
   - Report issues
   - Submit pull requests
   - Share examples
   - Join discussions

## Related Documentation

- [Server Implementation](../core/server.md)
- [Client Implementation](../core/client.md)
- [Resource Management](../guides/resources.md)
- [Tool Development](../guides/tools.md)

---
<div align="center">
  <sub>Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se)</sub>
  <br>
  <sub>Started: January 25, 2025 | Last Updated: January 27, 2025</sub>
</div>