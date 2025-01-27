# MCP Client API Reference

**Version:** 1.4.1  
**Component:** Client API  
**Last Updated:** 2024

## Overview

The MCP Client API provides interfaces for implementing clients that connect to MCP servers and manage communication between hosts and servers. This document details the available APIs and their usage.

## Core Client API

### Client Configuration
```typescript
interface ClientConfig {
  clientInfo: {
    name: string;
    version: string;
  };
  options: {
    capabilities?: ClientCapabilities;
  };
}

interface ClientCapabilities {
  experimental?: { [key: string]: object };
  roots?: {
    listChanged?: boolean;
  };
  sampling?: object;
}
```

### Connection Management
```typescript
interface Client {
  // Initialize and connect
  connect(transport: Transport): Promise<void>;
  
  // Close connection
  close(): Promise<void>;
  
  // Get server capabilities
  getServerCapabilities(): ServerCapabilities;
  
  // Set notification handlers
  setNotificationHandler<T>(
    schema: Schema<T>,
    handler: (notification: T) => void
  ): void;
}
```

## Resource API

### Resource Operations
```typescript
interface ResourceClient {
  // List resources
  listResources(params?: {
    cursor?: string;
  }): Promise<{
    resources: Resource[];
    nextCursor?: string;
  }>;

  // Read resource
  readResource(params: {
    uri: string;
  }): Promise<{
    contents: (TextResourceContents | BlobResourceContents)[];
  }>;

  // Subscribe to updates
  subscribe(params: {
    uri: string;
  }): Promise<void>;

  // Unsubscribe from updates
  unsubscribe(params: {
    uri: string;
  }): Promise<void>;
}
```

## Tool API

### Tool Operations
```typescript
interface ToolClient {
  // List tools
  listTools(params?: {
    cursor?: string;
  }): Promise<{
    tools: Tool[];
    nextCursor?: string;
  }>;

  // Call tool
  callTool(params: {
    name: string;
    arguments?: { [key: string]: unknown };
  }): Promise<{
    content: (TextContent | ImageContent | EmbeddedResource)[];
    isError?: boolean;
  }>;
}
```

## Prompt API

### Prompt Operations
```typescript
interface PromptClient {
  // List prompts
  listPrompts(params?: {
    cursor?: string;
  }): Promise<{
    prompts: Prompt[];
    nextCursor?: string;
  }>;

  // Get prompt
  getPrompt(params: {
    name: string;
    arguments?: { [key: string]: string };
  }): Promise<{
    description?: string;
    messages: PromptMessage[];
  }>;
}
```

## Sampling API

### Sampling Operations
```typescript
interface SamplingClient {
  // Create message
  createMessage(params: {
    messages: SamplingMessage[];
    modelPreferences?: ModelPreferences;
    systemPrompt?: string;
    includeContext?: "none" | "thisServer" | "allServers";
    temperature?: number;
    maxTokens: number;
    stopSequences?: string[];
    metadata?: object;
  }): Promise<{
    model: string;
    role: Role;
    content: TextContent | ImageContent;
    stopReason?: string;
  }>;
}
```

## Root Management API

### Root Operations
```typescript
interface RootClient {
  // List roots
  listRoots(): Promise<{
    roots: Root[];
  }>;

  // Notify root changes
  notifyRootsChanged(): void;
}
```

## Transport API

### Transport Interface
```typescript
interface Transport {
  // Start transport
  start(): Promise<void>;
  
  // Close transport
  close(): Promise<void>;
  
  // Send message
  send(message: JSONRPCMessage): Promise<void>;
  
  // Set message handler
  onMessage?: (message: JSONRPCMessage) => void;
  
  // Set close handler
  onClose?: () => void;
}
```

## Event Handling

### Message Events
```typescript
interface MessageHandler {
  // Handle request
  onRequest(request: JSONRPCRequest): Promise<unknown>;
  
  // Handle notification
  onNotification(notification: JSONRPCNotification): void;
  
  // Handle response
  onResponse(response: JSONRPCResponse | JSONRPCError): void;
}
```

### Connection Events
```typescript
interface ConnectionHandler {
  // Handle connection close
  onClose(): void;
  
  // Handle connection error
  onError(error: Error): void;
}
```

## Best Practices

1. **Connection Management**
   - Handle reconnection gracefully
   - Implement timeout handling
   - Clean up resources on close
   - Monitor connection health

2. **Message Handling**
   - Validate message formats
   - Handle message ordering
   - Implement request timeouts
   - Support cancellation

3. **Event Handling**
   - Use typed notification handlers
   - Handle all event types
   - Implement error recovery
   - Log important events

4. **Resource Management**
   - Cache resource metadata
   - Handle subscription updates
   - Implement pagination
   - Track resource states

## Related Documentation
- [Client Implementation Guide](../guides/implementation-patterns.md)
- [Transport Layer Reference](../reference/transport.md)
- [Protocol Specification](../reference/protocol-spec.md)

---

<div align="center">

Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se) | Started: January 25, 2025 | Last Updated: January 27, 2025

</div> 