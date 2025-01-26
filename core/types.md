# MCP Type System

**Version:** 1.4.1  
**Component:** Core Types  
**Last Updated:** 2024

## Overview

The MCP TypeScript SDK uses a comprehensive type system built on [Zod](https://github.com/colinhacks/zod) for runtime validation and TypeScript for static typing. This document outlines the core types that form the foundation of the MCP protocol.

## Protocol Version

```typescript
const LATEST_PROTOCOL_VERSION = "2024-11-05"
const SUPPORTED_PROTOCOL_VERSIONS = [
  LATEST_PROTOCOL_VERSION,
  "2024-10-07",
]
```

## Core Message Types

### JSON-RPC Base Types

1. **Request**
   ```typescript
   interface Request {
     method: string;
     params?: {
       _meta?: {
         progressToken?: string | number;
       };
     };
   }
   ```

2. **Notification**
   ```typescript
   interface Notification {
     method: string;
     params?: {
       _meta?: Record<string, unknown>;
     };
   }
   ```

3. **Result**
   ```typescript
   interface Result {
     _meta?: Record<string, unknown>;
   }
   ```

### JSON-RPC Messages

1. **Request Message**
   ```typescript
   interface JSONRPCRequest {
     jsonrpc: "2.0";
     id: string | number;
     method: string;
     params?: object;
   }
   ```

2. **Notification Message**
   ```typescript
   interface JSONRPCNotification {
     jsonrpc: "2.0";
     method: string;
     params?: object;
   }
   ```

3. **Response Message**
   ```typescript
   interface JSONRPCResponse {
     jsonrpc: "2.0";
     id: string | number;
     result: Result;
   }
   ```

## Error Handling

```typescript
enum ErrorCode {
  // SDK error codes
  ConnectionClosed = -32000,
  RequestTimeout = -32001,

  // Standard JSON-RPC error codes
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603,
}
```

### Error Response
```typescript
interface JSONRPCError {
  jsonrpc: "2.0";
  id: string | number;
  error: {
    code: number;
    message: string;
    data?: unknown;
  };
}
```

## Protocol-Specific Types

### Implementation Info
```typescript
interface Implementation {
  name: string;
  version: string;
}
```

### Capabilities

1. **Client Capabilities**
   ```typescript
   interface ClientCapabilities {
     experimental?: Record<string, unknown>;
     sampling?: Record<string, unknown>;
     roots?: {
       listChanged?: boolean;
     };
   }
   ```

2. **Server Capabilities**
   ```typescript
   interface ServerCapabilities {
     experimental?: Record<string, unknown>;
     logging?: Record<string, unknown>;
   }
   ```

## Resource Types

1. **Resource Contents**
   - Text Resources
   - Blob Resources
   - Template Resources

2. **Resource Operations**
   - List Resources
   - Read Resource
   - Subscribe/Unsubscribe
   - Resource Updates

## Tool Types

1. **Tool Definition**
   ```typescript
   interface Tool {
     name: string;
     description: string;
     parameters: object;
   }
   ```

2. **Tool Operations**
   - List Tools
   - Call Tool
   - Tool Updates

## Type Utilities

The SDK provides several utility types for working with the type system:

1. **Flatten<T>**
   - Recursively flattens complex types
   - Handles primitives, arrays, sets, maps, and objects

2. **Infer<Schema>**
   - Extracts TypeScript types from Zod schemas
   - Used throughout the SDK for type inference

## Best Practices

1. **Type Validation**
   - Always validate incoming messages using Zod schemas
   - Use TypeScript types for compile-time checking
   - Handle validation errors appropriately

2. **Error Handling**
   - Use the provided ErrorCode enum for standard errors
   - Include detailed error messages
   - Add relevant error data when available

3. **Type Extensions**
   - Follow the schema extension pattern for custom types
   - Maintain backward compatibility
   - Document any extensions thoroughly

## Related Documentation

- [Protocol Specification](../reference/protocol-spec.md)
- [Server Implementation](../guides/server-implementation.md)
- [Client Development](../guides/client-development.md)
- [Error Handling Guide](../guides/error-handling.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 