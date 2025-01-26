# MCP Transport Layer

**Version:** 1.4.1  
**Component:** Transport Layer  
**Last Updated:** 2024

## Overview

The MCP Transport Layer provides the communication foundation for MCP clients and servers. It abstracts the underlying transport mechanisms (stdio, SSE, WebSocket) behind a common interface while handling message serialization, connection management, and error handling.

## Transport Interface

```typescript
interface Transport {
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
  
  start(): Promise<void>;
  close(): Promise<void>;
  send(message: JSONRPCMessage): Promise<void>;
}
```

## Available Transports

### 1. stdio Transport

Provides communication through standard input/output streams.

```typescript
class StdioServerTransport implements Transport {
  constructor(
    private _stdin: Readable = process.stdin,
    private _stdout: Writable = process.stdout
  ) {}
  
  // Implementation details
  private _readBuffer: ReadBuffer;
  private _started: boolean;
  
  // Event handlers
  _ondata = (chunk: Buffer) => { /* ... */ };
  _onerror = (error: Error) => { /* ... */ };
}
```

Key Features:
- Buffer management for message framing
- Non-blocking write operations
- Error handling for stream events
- Automatic cleanup on close

### 2. SSE Transport

Server-Sent Events transport with HTTP POST for client messages.

```typescript
class SSEServerTransport implements Transport {
  constructor(
    private _endpoint: string,
    private res: ServerResponse
  ) {}
  
  // Implementation details
  private _sseResponse?: ServerResponse;
  private _sessionId: string;
  
  // Message handling
  async handlePostMessage(
    req: IncomingMessage,
    res: ServerResponse,
    parsedBody?: unknown
  ): Promise<void>;
}
```

Key Features:
- Session management
- Bidirectional communication
- Message size limits
- Content type validation

### 3. WebSocket Transport

WebSocket-based transport for full-duplex communication.

```typescript
class WebSocketTransport implements Transport {
  constructor(
    private url: string,
    private options?: WebSocketOptions
  ) {}
  
  // Implementation details
  private _ws?: WebSocket;
  private _connected: boolean;
}
```

Key Features:
- Full-duplex communication
- Automatic reconnection
- Binary message support
- Connection state management

## Message Handling

### 1. Message Format

```typescript
interface JSONRPCMessage {
  jsonrpc: "2.0";
  id?: string | number;
  method?: string;
  params?: unknown;
  result?: unknown;
  error?: {
    code: number;
    message: string;
    data?: unknown;
  };
}
```

### 2. Message Processing

```typescript
class ReadBuffer {
  append(chunk: Buffer): void;
  readMessage(): JSONRPCMessage | null;
  clear(): void;
}

function serializeMessage(message: JSONRPCMessage): string {
  return JSON.stringify(message) + "\n";
}
```

## Implementation Guidelines

### 1. Creating Custom Transports

```typescript
class CustomTransport implements Transport {
  // Required event handlers
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
  
  // Required methods
  async start(): Promise<void> {
    // Initialize connection
  }
  
  async close(): Promise<void> {
    // Clean up resources
  }
  
  async send(message: JSONRPCMessage): Promise<void> {
    // Send message
  }
}
```

### 2. Error Handling

```typescript
class TransportBase implements Transport {
  protected handleError(error: Error): void {
    if (this.onerror) {
      this.onerror(error);
    } else {
      console.error("Unhandled transport error:", error);
    }
  }
  
  protected handleClose(): void {
    this.onclose?.();
  }
}
```

## Best Practices

1. **Connection Management**
   - Implement proper connection lifecycle
   - Handle reconnection scenarios
   - Clean up resources on close

2. **Message Processing**
   - Validate message format
   - Handle partial messages
   - Implement proper framing

3. **Error Handling**
   - Provide detailed error information
   - Handle network errors
   - Implement timeout handling

4. **Performance**
   - Use appropriate buffer sizes
   - Implement message batching
   - Handle backpressure

## Example Usage

### 1. stdio Transport

```typescript
import { StdioServerTransport } from "@mcp/sdk";

const transport = new StdioServerTransport();
transport.onmessage = (message) => {
  console.log("Received:", message);
};

await transport.start();
```

### 2. SSE Transport

```typescript
import { SSEServerTransport } from "@mcp/sdk";

const transport = new SSEServerTransport("/mcp", response);
transport.onmessage = (message) => {
  console.log("Received:", message);
};

await transport.start();

// Handle POST requests
app.post("/mcp", async (req, res) => {
  await transport.handlePostMessage(req, res);
});
```

### 3. WebSocket Transport

```typescript
import { WebSocketTransport } from "@mcp/sdk";

const transport = new WebSocketTransport("ws://localhost:3000/mcp");
transport.onmessage = (message) => {
  console.log("Received:", message);
};

await transport.start();
```

## Related Documentation

- [Protocol Specification](../reference/protocol-spec.md)
- [Client Implementation](./client.md)
- [Server Implementation](./server.md)
- [Error Handling](../guides/error-handling.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 