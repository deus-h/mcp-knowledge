# MCP Protocol Specification

**Version:** 1.4.1  
**Component:** Protocol Specification  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) is a JSON-RPC based protocol that enables standardized communication between LLM applications and context providers. This document details the protocol specification, message formats, and core concepts.

## Protocol Version

The latest protocol version is `2024-11-05`. The protocol version is negotiated during initialization:

```typescript
interface InitializeRequest {
  method: "initialize";
  params: {
    protocolVersion: string;
    capabilities: ClientCapabilities;
    clientInfo: Implementation;
  };
}

interface InitializeResult {
  protocolVersion: string;
  capabilities: ServerCapabilities;
  serverInfo: Implementation;
  instructions?: string;
}
```

## Message Types

### 1. Base Message Types

All messages follow the JSON-RPC 2.0 specification:

```typescript
type JSONRPCMessage =
  | JSONRPCRequest
  | JSONRPCNotification
  | JSONRPCResponse
  | JSONRPCError;

interface JSONRPCRequest {
  jsonrpc: "2.0";
  id: RequestId;
  method: string;
  params?: unknown;
}

interface JSONRPCNotification {
  jsonrpc: "2.0";
  method: string;
  params?: unknown;
}

interface JSONRPCResponse {
  jsonrpc: "2.0";
  id: RequestId;
  result: Result;
}

interface JSONRPCError {
  jsonrpc: "2.0";
  id: RequestId;
  error: {
    code: number;
    message: string;
    data?: unknown;
  };
}
```

### 2. Request Types

```typescript
interface Request {
  method: string;
  params?: {
    _meta?: {
      progressToken?: ProgressToken;
    };
    [key: string]: unknown;
  };
}

interface PaginatedRequest extends Request {
  params?: {
    cursor?: Cursor;
  };
}
```

### 3. Response Types

```typescript
interface Result {
  _meta?: { [key: string]: unknown };
  [key: string]: unknown;
}

interface PaginatedResult extends Result {
  nextCursor?: Cursor;
}
```

## Core Features

### 1. Initialization

The initialization process establishes protocol version and capabilities:

```typescript
interface ClientCapabilities {
  experimental?: { [key: string]: object };
  roots?: {
    listChanged?: boolean;
  };
  sampling?: object;
}

interface ServerCapabilities {
  experimental?: { [key: string]: object };
  logging?: object;
  prompts?: {
    listChanged?: boolean;
  };
  resources?: {
    subscribe?: boolean;
    listChanged?: boolean;
  };
  tools?: {
    listChanged?: boolean;
  };
}
```

### 2. Progress Tracking

Long-running operations can report progress:

```typescript
interface ProgressNotification {
  method: "notifications/progress";
  params: {
    progressToken: ProgressToken;
    progress: number;
    total?: number;
  };
}
```

### 3. Resource Management

Resources are identified by URIs and can be dynamic:

```typescript
interface Resource {
  uri: string;
  name: string;
  description?: string;
  mimeType?: string;
  size?: number;
}

interface ResourceTemplate {
  uriTemplate: string;
  name: string;
  description?: string;
  mimeType?: string;
}
```

### 4. Resource Operations

```typescript
interface ListResourcesRequest {
  method: "resources/list";
  params?: {
    cursor?: Cursor;
  };
}

interface ReadResourceRequest {
  method: "resources/read";
  params: {
    uri: string;
  };
}

interface SubscribeRequest {
  method: "resources/subscribe";
  params: {
    uri: string;
  };
}
```

## Error Handling

### 1. Standard Error Codes

```typescript
const PARSE_ERROR = -32700;
const INVALID_REQUEST = -32600;
const METHOD_NOT_FOUND = -32601;
const INVALID_PARAMS = -32602;
const INTERNAL_ERROR = -32603;
```

### 2. Error Response Format

```typescript
interface JSONRPCError {
  jsonrpc: "2.0";
  id: RequestId;
  error: {
    code: number;
    message: string;
    data?: unknown;
  };
}
```

## Protocol Flow

### 1. Connection Establishment

1. Client connects to server
2. Client sends `initialize` request
3. Server responds with capabilities
4. Client sends `initialized` notification
5. Normal operation begins

### 2. Resource Access

1. Client lists available resources
2. Client reads specific resources
3. Client can subscribe to resource updates
4. Server sends update notifications

### 3. Tool Usage

1. Client lists available tools
2. Client calls tools with parameters
3. Server executes tools and returns results
4. Progress notifications if requested

## Security Considerations

1. **URI Validation**
   - Validate all URIs
   - Check path traversal
   - Verify protocols
   - Handle escaping

2. **Resource Access**
   - Implement authentication
   - Check permissions
   - Rate limit requests
   - Validate content

3. **Error Handling**
   - Sanitize error messages
   - Handle timeouts
   - Implement retries
   - Log security events

4. **Data Protection**
   - Validate input size
   - Check content types
   - Handle sensitive data
   - Implement encryption

## Best Practices

1. **Protocol Usage**
   - Follow JSON-RPC spec
   - Handle all message types
   - Implement required methods
   - Support cancellation

2. **Resource Design**
   - Use clear URI schemes
   - Implement templates
   - Handle pagination
   - Support updates

3. **Error Management**
   - Use standard codes
   - Provide context
   - Handle recovery
   - Log appropriately

4. **Performance**
   - Implement caching
   - Use pagination
   - Handle streaming
   - Monitor resources

## Related Documentation

- [Type System](types.md)
- [Transport Layer](transports.md)
- [Error Handling](error-handling.md)
- [Security Guide](security.md) 

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>