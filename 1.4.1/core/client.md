# MCP Client Architecture

**Version:** 1.4.1  
**Component:** Client Implementation  
**Last Updated:** 2024

## Overview

The MCP Client implementation provides a type-safe, feature-rich interface for connecting to and interacting with MCP servers. It handles protocol negotiation, capability management, and provides high-level APIs for all MCP operations.

## Core Components

### Client Class

```typescript
class Client<RequestT, NotificationT, ResultT> extends Protocol<
  ClientRequest | RequestT,
  ClientNotification | NotificationT,
  ClientResult | ResultT
> {
  constructor(
    clientInfo: Implementation,
    options?: ClientOptions
  );
}
```

### Key Features

1. **Type Safety**
   - Generic type parameters for custom extensions
   - Runtime validation with Zod schemas
   - Compile-time type checking

2. **Capability Management**
   - Automatic capability negotiation
   - Runtime capability checking
   - Extensible capability system

3. **Transport Layer**
   - stdio support
   - SSE (Server-Sent Events)
   - WebSocket support
   - Custom transport options

4. **Protocol Handling**
   - Automatic initialization
   - Version negotiation
   - Message serialization
   - Error handling

## Client Configuration

### Options

```typescript
interface ClientOptions {
  capabilities?: ClientCapabilities;
  timeout?: number;
  logger?: Logger;
}

interface ClientCapabilities {
  experimental?: Record<string, unknown>;
  sampling?: Record<string, unknown>;
  roots?: {
    listChanged?: boolean;
  };
}
```

### Implementation Info

```typescript
interface Implementation {
  name: string;
  version: string;
}
```

## Core Functionality

### 1. Connection Management

```typescript
class Client {
  async connect(transport: Transport): Promise<void> {
    // 1. Connect transport
    // 2. Initialize protocol
    // 3. Negotiate capabilities
    // 4. Send initialized notification
  }

  async close(): Promise<void> {
    // Clean shutdown
  }
}
```

### 2. Resource Operations

```typescript
class Client {
  async listResources(params?: ListResourcesRequest["params"]): Promise<ListResourcesResult>;
  async readResource(params: ReadResourceRequest["params"]): Promise<ReadResourceResult>;
  async subscribeResource(params: SubscribeRequest["params"]): Promise<void>;
  async unsubscribeResource(params: UnsubscribeRequest["params"]): Promise<void>;
}
```

### 3. Tool Operations

```typescript
class Client {
  async listTools(params?: ListToolsRequest["params"]): Promise<ListToolsResult>;
  async callTool(params: CallToolRequest["params"]): Promise<CallToolResult>;
}
```

### 4. Prompt Operations

```typescript
class Client {
  async listPrompts(params?: ListPromptsRequest["params"]): Promise<ListPromptsResult>;
  async getPrompt(params: GetPromptRequest["params"]): Promise<GetPromptResult>;
  async complete(params: CompleteRequest["params"]): Promise<CompleteResult>;
}
```

## Transport Implementations

### 1. stdio Transport

```typescript
import { StdioTransport } from "@mcp/sdk";

const transport = new StdioTransport();
const client = new Client({ name: "my-client", version: "1.0.0" });
await client.connect(transport);
```

### 2. SSE Transport

```typescript
import { SSETransport } from "@mcp/sdk";

const transport = new SSETransport("http://localhost:3000/mcp");
const client = new Client({ name: "my-client", version: "1.0.0" });
await client.connect(transport);
```

### 3. WebSocket Transport

```typescript
import { WebSocketTransport } from "@mcp/sdk";

const transport = new WebSocketTransport("ws://localhost:3000/mcp");
const client = new Client({ name: "my-client", version: "1.0.0" });
await client.connect(transport);
```

## Error Handling

### 1. Connection Errors

```typescript
try {
  await client.connect(transport);
} catch (error) {
  if (error instanceof McpError) {
    switch (error.code) {
      case ErrorCode.ConnectionClosed:
        // Handle connection closure
        break;
      case ErrorCode.RequestTimeout:
        // Handle timeout
        break;
    }
  }
}
```

### 2. Operation Errors

```typescript
try {
  await client.callTool({ name: "my-tool", arguments: {} });
} catch (error) {
  if (error instanceof McpError) {
    switch (error.code) {
      case ErrorCode.InvalidParams:
        // Handle invalid parameters
        break;
      case ErrorCode.MethodNotFound:
        // Handle unknown tool
        break;
    }
  }
}
```

## Best Practices

1. **Connection Management**
   - Always handle connection errors
   - Implement proper cleanup
   - Monitor connection state

2. **Type Safety**
   - Use generic type parameters
   - Define custom types when needed
   - Validate all inputs

3. **Error Handling**
   - Catch and handle specific errors
   - Provide meaningful error messages
   - Implement proper fallbacks

4. **Resource Usage**
   - Unsubscribe from resources when done
   - Clean up event listeners
   - Handle resource updates properly

## Example Usage

### Basic Client

```typescript
import { Client, StdioTransport } from "@mcp/sdk";

async function main() {
  const client = new Client({
    name: "example-client",
    version: "1.0.0",
    capabilities: {
      sampling: {},
      roots: { listChanged: true }
    }
  });

  const transport = new StdioTransport();
  await client.connect(transport);

  // List available tools
  const { tools } = await client.listTools();

  // Call a tool
  const result = await client.callTool({
    name: "example-tool",
    arguments: { input: "test" }
  });

  await client.close();
}
```

### Extended Client

```typescript
import { Client, SSETransport } from "@mcp/sdk";

// Custom request types
interface CustomRequest extends Request {
  method: "custom/method";
  params: {
    data: string;
  };
}

// Create typed client
const client = new Client<CustomRequest>({
  name: "custom-client",
  version: "1.0.0"
});

const transport = new SSETransport("http://localhost:3000/mcp");
await client.connect(transport);
```

## Related Documentation

- [Protocol Specification](../reference/protocol-spec.md)
- [Transport Layer](../guides/transport.md)
- [Type System](./types.md)
- [Error Handling](../guides/error-handling.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 