# MCP Messaging System

**Version:** 1.4.1  
**Component:** Messaging System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) uses JSON-RPC 2.0 as its messaging protocol, providing a robust foundation for client-server communication. This document details the messaging system's architecture, implementation, and best practices.

## Core Components

### Message Types
```typescript
interface JSONRPCMessage {
  jsonrpc: "2.0";
}

interface JSONRPCRequest extends JSONRPCMessage {
  id: string | number;
  method: string;
  params?: { [key: string]: unknown };
}

interface JSONRPCResponse extends JSONRPCMessage {
  id: string | number;
  result?: { [key: string]: unknown };
  error?: JSONRPCError;
}

interface JSONRPCNotification extends JSONRPCMessage {
  method: string;
  params?: { [key: string]: unknown };
}

interface JSONRPCError {
  code: number;
  message: string;
  data?: unknown;
}
```

## Implementation

### Message Manager
```typescript
class MessageManager {
  private nextRequestId: number = 1;
  private pendingRequests = new Map<
    RequestId,
    {
      resolve: (result: unknown) => void;
      reject: (error: Error) => void;
      method: string;
      timestamp: number;
    }
  >();

  // Send request
  async sendRequest<T>(
    method: string,
    params?: unknown
  ): Promise<T> {
    const id = this.generateRequestId();
    const request: JSONRPCRequest = {
      jsonrpc: "2.0",
      id,
      method,
      params
    };

    return new Promise((resolve, reject) => {
      this.pendingRequests.set(id, {
        resolve,
        reject,
        method,
        timestamp: Date.now()
      });
      this.transport.send(request).catch(reject);
    });
  }

  // Send notification
  async sendNotification(
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

  // Handle incoming message
  async handleMessage(message: JSONRPCMessage): Promise<void> {
    if (this.isRequest(message)) {
      await this.handleRequest(message);
    } else if (this.isNotification(message)) {
      await this.handleNotification(message);
    } else if (this.isResponse(message)) {
      await this.handleResponse(message);
    }
  }
}
```

### Message Validation
```typescript
class MessageValidator {
  // Validate request
  validateRequest(request: JSONRPCRequest): void {
    if (request.jsonrpc !== "2.0") {
      throw new Error("Invalid JSON-RPC version");
    }
    
    if (!request.id || !request.method) {
      throw new Error("Invalid request format");
    }
    
    if (typeof request.id !== "string" &&
        typeof request.id !== "number") {
      throw new Error("Invalid request ID type");
    }
  }

  // Validate response
  validateResponse(response: JSONRPCResponse): void {
    if (response.jsonrpc !== "2.0") {
      throw new Error("Invalid JSON-RPC version");
    }
    
    if (!response.id) {
      throw new Error("Invalid response format");
    }
    
    if (response.result && response.error) {
      throw new Error("Response cannot have both result and error");
    }
  }

  // Validate notification
  validateNotification(
    notification: JSONRPCNotification
  ): void {
    if (notification.jsonrpc !== "2.0") {
      throw new Error("Invalid JSON-RPC version");
    }
    
    if (!notification.method) {
      throw new Error("Invalid notification format");
    }
    
    if ("id" in notification) {
      throw new Error("Notification cannot have ID");
    }
  }
}
```

## Features

### Message Types
- Requests
- Responses
- Notifications
- Error handling

### Message Management
- Request tracking
- Response matching
- Notification routing
- Error handling

### Message Validation
- Format validation
- Type checking
- Schema validation
- Error reporting

### Message Routing
- Method routing
- Handler registration
- Response routing
- Error routing

## Best Practices

1. **Message Handling**
   - Validate all messages
   - Track requests
   - Handle timeouts
   - Clean up state

2. **Error Handling**
   - Use standard codes
   - Include context
   - Handle timeouts
   - Clean up state

3. **Message Routing**
   - Register handlers
   - Validate methods
   - Handle unknown methods
   - Track performance

4. **Message Validation**
   - Check formats
   - Validate types
   - Handle malformed
   - Log issues

## Error Handling

### Standard Error Codes
```typescript
enum JSONRPCErrorCode {
  PARSE_ERROR = -32700,
  INVALID_REQUEST = -32600,
  METHOD_NOT_FOUND = -32601,
  INVALID_PARAMS = -32602,
  INTERNAL_ERROR = -32603
}

interface StandardError {
  code: JSONRPCErrorCode;
  message: string;
  data?: unknown;
}
```

### Error Creation
```typescript
class ErrorCreator {
  // Create standard error
  createError(
    code: JSONRPCErrorCode,
    message: string,
    data?: unknown
  ): JSONRPCError {
    return {
      code,
      message,
      data
    };
  }

  // Create parse error
  createParseError(data?: unknown): JSONRPCError {
    return this.createError(
      JSONRPCErrorCode.PARSE_ERROR,
      "Parse error",
      data
    );
  }

  // Create invalid request error
  createInvalidRequestError(
    data?: unknown
  ): JSONRPCError {
    return this.createError(
      JSONRPCErrorCode.INVALID_REQUEST,
      "Invalid request",
      data
    );
  }
}
```

## Security Considerations

### Message Security
```typescript
class MessageSecurity {
  // Validate message size
  private validateSize(
    message: JSONRPCMessage
  ): boolean {
    const size = JSON.stringify(message).length;
    return size <= this.maxMessageSize;
  }

  // Check message rate
  private checkMessageRate(
    sender: string
  ): boolean {
    return this.rateLimiter.checkLimit(sender);
  }

  // Validate content
  private validateContent(
    message: JSONRPCMessage
  ): boolean {
    return !this.containsSensitiveData(message) &&
           this.isWellFormed(message);
  }
}
```

### Best Practices
1. **Message Protection**
   - Validate sizes
   - Rate limit
   - Check content
   - Monitor usage

2. **Data Protection**
   - Remove sensitive data
   - Validate content
   - Control access
   - Log security events

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by John Smith (john.smith@company.com)</sub>