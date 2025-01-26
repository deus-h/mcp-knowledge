# MCP Protocol Internals

**Version:** 1.4.1  
**Component:** Protocol Implementation  
**Last Updated:** 2024

## Overview

This document details the internal implementation of the Model Context Protocol (MCP), focusing on the core protocol mechanisms, message handling, and state management.

## Protocol Implementation

### Message Processing
```typescript
class Protocol {
  private nextRequestId: number = 1;
  private pendingRequests = new Map<
    RequestId,
    {
      resolve: (result: unknown) => void;
      reject: (error: Error) => void;
      method: string;
    }
  >();

  // Send request and wait for response
  protected async sendRequest<T>(
    method: string,
    params?: unknown
  ): Promise<T> {
    const id = this.nextRequestId++;
    const request: JSONRPCRequest = {
      jsonrpc: "2.0",
      id,
      method,
      params
    };

    return new Promise((resolve, reject) => {
      this.pendingRequests.set(id, { resolve, reject, method });
      this.transport.send(request).catch(reject);
    });
  }

  // Send notification (no response expected)
  protected async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const notification: JSONRPCNotification = {
      jsonrpc: "2.0",
      method,
      params
    };
    await this.transport.send(notification);
  }
}
```

### Message Routing
```typescript
class MessageRouter {
  private handlers = new Map<string, (params: unknown) => Promise<unknown>>();

  // Register request handler
  registerHandler(
    method: string,
    handler: (params: unknown) => Promise<unknown>
  ): void {
    this.handlers.set(method, handler);
  }

  // Route incoming message
  async routeMessage(message: JSONRPCMessage): Promise<void> {
    if (isRequest(message)) {
      await this.handleRequest(message);
    } else if (isNotification(message)) {
      await this.handleNotification(message);
    } else if (isResponse(message)) {
      await this.handleResponse(message);
    }
  }
}
```

## State Management

### Connection State
```typescript
class ConnectionState {
  private state: "disconnected" | "connecting" | "connected" | "closing";
  private capabilities?: ServerCapabilities;
  private serverInfo?: Implementation;

  // Initialize connection
  async initialize(): Promise<void> {
    this.state = "connecting";
    const response = await this.sendInitialize();
    this.capabilities = response.capabilities;
    this.serverInfo = response.serverInfo;
    this.state = "connected";
  }

  // Handle connection close
  async close(): Promise<void> {
    this.state = "closing";
    await this.cleanup();
    this.state = "disconnected";
  }
}
```

### Request Tracking
```typescript
class RequestTracker {
  private activeRequests = new Map<RequestId, {
    startTime: number;
    method: string;
    params: unknown;
    progressToken?: ProgressToken;
  }>();

  // Track new request
  trackRequest(request: JSONRPCRequest): void {
    this.activeRequests.set(request.id, {
      startTime: Date.now(),
      method: request.method,
      params: request.params,
      progressToken: request.params?._meta?.progressToken
    });
  }

  // Complete request tracking
  completeRequest(id: RequestId): void {
    this.activeRequests.delete(id);
  }
}
```

## Transport Layer

### Transport Management
```typescript
class TransportManager {
  private transport?: Transport;
  private reconnectAttempts: number = 0;
  private maxReconnectAttempts: number = 3;

  // Initialize transport
  async initializeTransport(transport: Transport): Promise<void> {
    this.transport = transport;
    transport.onMessage = this.handleMessage.bind(this);
    transport.onClose = this.handleClose.bind(this);
    await transport.start();
  }

  // Handle transport failure
  private async handleTransportFailure(): Promise<void> {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      await this.reconnect();
    } else {
      await this.close();
    }
  }
}
```

## Error Handling

### Error Processing
```typescript
class ErrorProcessor {
  // Create protocol error
  createError(
    code: number,
    message: string,
    data?: unknown
  ): JSONRPCError {
    return {
      jsonrpc: "2.0",
      id: null,
      error: { code, message, data }
    };
  }

  // Handle protocol error
  handleError(error: JSONRPCError): void {
    const { code, message, data } = error.error;
    switch (code) {
      case PARSE_ERROR:
        this.handleParseError(message, data);
        break;
      case INVALID_REQUEST:
        this.handleInvalidRequest(message, data);
        break;
      case METHOD_NOT_FOUND:
        this.handleMethodNotFound(message, data);
        break;
      default:
        this.handleUnknownError(code, message, data);
    }
  }
}
```

## Utility Functions

### Message Validation
```typescript
function validateMessage(message: unknown): JSONRPCMessage {
  if (!isJSONRPCMessage(message)) {
    throw new Error("Invalid JSON-RPC message");
  }
  return message;
}

function isJSONRPCMessage(value: unknown): value is JSONRPCMessage {
  return (
    isRequest(value) ||
    isNotification(value) ||
    isResponse(value) ||
    isError(value)
  );
}
```

### Request Helpers
```typescript
function createRequest(
  method: string,
  params?: unknown,
  id?: RequestId
): JSONRPCRequest {
  return {
    jsonrpc: "2.0",
    id: id ?? generateRequestId(),
    method,
    params
  };
}

function createNotification(
  method: string,
  params?: unknown
): JSONRPCNotification {
  return {
    jsonrpc: "2.0",
    method,
    params
  };
}
```

## Best Practices

1. **Message Processing**
   - Validate all messages
   - Track request state
   - Handle timeouts
   - Process in order

2. **State Management**
   - Track connection state
   - Monitor request lifecycle
   - Handle transitions
   - Clean up resources

3. **Transport Handling**
   - Implement reconnection
   - Handle failures
   - Buffer messages
   - Monitor health

4. **Error Processing**
   - Use standard codes
   - Include context
   - Log details
   - Handle recovery

## Related Documentation
- [Protocol Specification](../reference/protocol-spec.md)
- [Type System](../reference/type-system-advanced.md)
- [Implementation Guide](../guides/implementation-patterns.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 