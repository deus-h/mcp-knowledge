# MCP Server Architecture

**Version:** 1.4.1  
**Component:** Server Implementation  
**Last Updated:** 2024

## Overview

The MCP Server implementation provides a high-level abstraction for creating MCP-compliant servers. It handles protocol messaging, resource management, tool execution, and prompt handling through a clean, type-safe API.

## Core Components

### McpServer Class

The main server class that provides high-level APIs for MCP functionality:

```typescript
class McpServer {
  public readonly server: Server;
  
  constructor(serverInfo: Implementation, options?: ServerOptions) {
    this.server = new Server(serverInfo, options);
  }
}
```

### Key Features

1. **Resource Management**
   - Static resources
   - Resource templates
   - URI-based access
   - Content streaming

2. **Tool Integration**
   - Named tools with schemas
   - Input validation
   - Async execution
   - Error handling

3. **Prompt Handling**
   - Template management
   - Argument completion
   - Dynamic generation

4. **Transport Layer**
   - stdio support
   - SSE (Server-Sent Events)
   - Custom transport options

## Component Architecture

### 1. Resource System

```typescript
interface RegisteredResource {
  name: string;
  metadata?: ResourceMetadata;
  readCallback: ReadResourceCallback;
}

interface ResourceTemplate {
  uriTemplate: UriTemplate;
  listCallback?: ListResourcesCallback;
  completeCallback?: Record<string, CompleteResourceTemplateCallback>;
}
```

### 2. Tool System

```typescript
interface RegisteredTool {
  description?: string;
  inputSchema?: AnyZodObject;
  callback: ToolCallback<undefined | ZodRawShape>;
}

type ToolCallback<Args> = (
  args: Args,
  extra: RequestHandlerExtra
) => Promise<CallToolResult>;
```

### 3. Prompt System

```typescript
interface RegisteredPrompt {
  description?: string;
  argsSchema?: ZodObject<PromptArgsRawShape>;
  callback: PromptCallback<undefined | PromptArgsRawShape>;
}
```

## Request Handling

The server implements several key request handlers:

1. **Tool Requests**
   - List available tools
   - Execute tool calls
   - Handle tool errors

2. **Resource Requests**
   - List resources
   - Read resource content
   - Handle templates

3. **Prompt Requests**
   - List prompts
   - Get prompt content
   - Handle completions

## Implementation Details

### 1. Initialization Flow

```typescript
async function initializeServer() {
  const server = new McpServer({
    name: "my-server",
    version: "1.0.0"
  });
  
  // Register capabilities
  server.registerResource(...);
  server.registerTool(...);
  server.registerPrompt(...);
  
  // Connect transport
  await server.connect(transport);
}
```

### 2. Request Processing

1. Message Reception
2. Schema Validation
3. Handler Execution
4. Response Generation

### 3. Error Handling

```typescript
enum ErrorCode {
  ConnectionClosed = -32000,
  RequestTimeout = -32001,
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603
}
```

## Best Practices

1. **Resource Management**
   - Use templates for dynamic resources
   - Implement proper cleanup
   - Handle concurrent access

2. **Tool Implementation**
   - Validate inputs thoroughly
   - Provide clear error messages
   - Handle async operations properly

3. **Prompt Handling**
   - Design reusable templates
   - Implement completion where useful
   - Handle context appropriately

4. **Error Handling**
   - Use appropriate error codes
   - Provide detailed messages
   - Handle edge cases

## Transport Options

### 1. stdio Transport

```typescript
import { StdioTransport } from "@mcp/sdk";

const transport = new StdioTransport();
await server.connect(transport);
```

### 2. SSE Transport

```typescript
import { SSETransport } from "@mcp/sdk";

const transport = new SSETransport({
  port: 3000
});
await server.connect(transport);
```

## Example Implementations

1. **Basic Resource Server**
   ```typescript
   server.resource("files", "/files/{path}", {
     list: async () => ({ resources: [] }),
     read: async (uri) => ({ content: await readFile(uri) })
   });
   ```

2. **Tool Provider**
   ```typescript
   server.tool("calculator", {
     description: "Performs calculations",
     input: z.object({
       expression: z.string()
     }),
     callback: async (args) => ({
       result: evaluate(args.expression)
     })
   });
   ```

## Related Documentation

- [Protocol Specification](../reference/protocol-spec.md)
- [Resource Management](../guides/resources.md)
- [Tool Development](../guides/tools.md)
- [Transport Layer](../guides/transport.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 