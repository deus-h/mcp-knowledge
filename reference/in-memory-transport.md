# MCP In-Memory Transport

**Version:** 1.4.1  
**Component:** In-Memory Transport  
**Last Updated:** 2024

## Overview

The MCP SDK provides an in-memory transport implementation for creating clients and servers that communicate within the same process. This is particularly useful for testing, development, and scenarios where network transport is not needed.

## Features

### 1. Linked Transport Pairs

Create paired transports for client-server communication:

```typescript
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory";

const [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();
```

### 2. Message Queue

Messages are queued if received before transport is started:

```typescript
class InMemoryTransport implements Transport {
  private _messageQueue: JSONRPCMessage[] = [];

  async start(): Promise<void> {
    // Process queued messages
    while (this._messageQueue.length > 0) {
      const message = this._messageQueue.shift();
      if (message) {
        this.onmessage?.(message);
      }
    }
  }
}
```

### 3. Event Handlers

```typescript
interface Transport {
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

## Implementation Details

### 1. Transport Interface

```typescript
class InMemoryTransport implements Transport {
  private _otherTransport?: InMemoryTransport;
  private _messageQueue: JSONRPCMessage[] = [];

  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

### 2. Creating Transport Pairs

```typescript
static createLinkedPair(): [InMemoryTransport, InMemoryTransport] {
  const clientTransport = new InMemoryTransport();
  const serverTransport = new InMemoryTransport();
  clientTransport._otherTransport = serverTransport;
  serverTransport._otherTransport = clientTransport;
  return [clientTransport, serverTransport];
}
```

### 3. Message Handling

```typescript
async send(message: JSONRPCMessage): Promise<void> {
  if (!this._otherTransport) {
    throw new Error("Not connected");
  }

  if (this._otherTransport.onmessage) {
    this._otherTransport.onmessage(message);
  } else {
    this._otherTransport._messageQueue.push(message);
  }
}
```

### 4. Connection Management

```typescript
async close(): Promise<void> {
  const other = this._otherTransport;
  this._otherTransport = undefined;
  await other?.close();
  this.onclose?.();
}
```

## Usage Examples

### 1. Basic Setup

```typescript
import { Client } from "@modelcontextprotocol/sdk/client";
import { Server } from "@modelcontextprotocol/sdk/server";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory";

// Create transport pair
const [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();

// Setup client and server
const client = new Client({
  name: "test-client",
  version: "1.0.0"
});

const server = new Server({
  name: "test-server",
  version: "1.0.0"
});

// Connect transports
await client.connect(clientTransport);
await server.connect(serverTransport);
```

### 2. Testing Setup

```typescript
describe("MCP Communication", () => {
  let clientTransport: InMemoryTransport;
  let serverTransport: InMemoryTransport;
  
  beforeEach(() => {
    [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();
  });
  
  afterEach(async () => {
    await clientTransport.close();
    await serverTransport.close();
  });
  
  it("should handle messages", async () => {
    // Test implementation
  });
});
```

## Best Practices

1. **Transport Usage**
   - Use for testing
   - Use for development
   - Use for prototyping
   - Use for integration tests

2. **Message Handling**
   - Handle queue overflow
   - Process messages in order
   - Validate message format
   - Handle errors gracefully

3. **Resource Management**
   - Clean up connections
   - Clear message queues
   - Handle timeouts
   - Monitor memory usage

4. **Testing**
   - Test edge cases
   - Test error conditions
   - Test message ordering
   - Test connection states

## Related Documentation

- [Transport Layer](transports.md)
- [Testing Guide](../guides/testing.md)
- [Client Architecture](client-architecture.md)
- [Server Architecture](server-architecture.md)

<sub>Created and maintained by John Smith (john.smith@example.com)</sub>