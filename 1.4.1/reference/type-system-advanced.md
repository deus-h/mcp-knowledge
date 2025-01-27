# MCP Advanced Type System

**Version:** 1.4.1  
**Component:** Type System - Advanced Features  
**Last Updated:** 2024

## Overview

This document covers advanced features of the MCP type system, including type validation, schema definitions, and utility types.

## Schema Validation

The MCP SDK uses Zod for runtime type validation. All protocol types have corresponding schema definitions:

```typescript
import { z } from "zod";

// Base schema definitions
export const JSONRPC_VERSION = "2.0";
export const LATEST_PROTOCOL_VERSION = "2024-11-05";

// Progress tracking
export const ProgressTokenSchema = z.union([
  z.string(),
  z.number().int()
]);

// Pagination
export const CursorSchema = z.string();
```

## Type Utilities

### 1. Type Flattening

```typescript
type Primitive = string | number | boolean | bigint | null | undefined;

type Flatten<T> = T extends Primitive
  ? T
  : T extends Array<infer U>
  ? Array<Flatten<U>>
  : T extends Set<infer U>
  ? Set<Flatten<U>>
  : T extends Map<infer K, infer V>
  ? Map<Flatten<K>, Flatten<V>>
  : T extends object
  ? { [K in keyof T]: Flatten<T[K]> }
  : T;
```

### 2. Schema Inference

```typescript
type Infer<Schema extends ZodTypeAny> = Flatten<z.infer<Schema>>;

// Example usage:
export type ProgressToken = Infer<typeof ProgressTokenSchema>;
export type Cursor = Infer<typeof CursorSchema>;
```

## Message Types

### 1. Base Message Types

```typescript
export type JSONRPCMessage =
  | JSONRPCRequest
  | JSONRPCNotification
  | JSONRPCResponse
  | JSONRPCError;

export interface JSONRPCRequest {
  jsonrpc: "2.0";
  id: RequestId;
  method: string;
  params?: unknown;
}
```

### 2. Content Types

```typescript
export interface TextContent {
  type: "text";
  text: string;
  mimeType?: string;
}

export interface ImageContent {
  type: "image";
  data: string;
  mimeType: string;
}

export interface EmbeddedResource {
  type: "resource";
  uri: string;
}
```

## Advanced Features

### 1. Resource References

```typescript
export interface ResourceReference {
  type: "resource";
  uri: string;
  range?: {
    start: number;
    end: number;
  };
}

export interface PromptReference {
  type: "prompt";
  name: string;
  arguments?: Record<string, unknown>;
}
```

### 2. Sampling Support

```typescript
export interface SamplingMessage {
  role: "user" | "assistant";
  content: TextContent | ImageContent | EmbeddedResource;
}

export interface CreateMessageRequest {
  messages: SamplingMessage[];
  modelPreferences?: {
    model?: string;
    provider?: string;
  };
  systemPrompt?: string;
  temperature?: number;
  maxTokens: number;
  stopSequences?: string[];
}
```

### 3. Completion Support

```typescript
export interface CompleteRequest {
  text: string;
  references?: (ResourceReference | PromptReference)[];
  options?: {
    maxLength?: number;
    stopAt?: string[];
  };
}

export interface CompleteResult {
  completion: string;
  references?: (ResourceReference | PromptReference)[];
}
```

## Error Types

### 1. Error Codes

```typescript
export enum ErrorCode {
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

### 2. Error Classes

```typescript
export class McpError extends Error {
  constructor(
    public readonly code: number,
    message: string,
    public readonly data?: unknown,
  ) {
    super(message);
  }
}
```

## Client/Server Types

### 1. Client Types

```typescript
export type ClientRequest = 
  | InitializeRequest
  | ListResourcesRequest
  | ReadResourceRequest
  | ListToolsRequest
  | CallToolRequest;

export type ClientNotification =
  | InitializedNotification
  | CancelledNotification;

export type ClientResult =
  | InitializeResult
  | ListResourcesResult
  | ReadResourceResult
  | ListToolsResult
  | CallToolResult;
```

### 2. Server Types

```typescript
export type ServerRequest =
  | PingRequest;

export type ServerNotification =
  | ResourceListChangedNotification
  | ResourceUpdatedNotification
  | ToolListChangedNotification
  | LoggingMessageNotification;

export type ServerResult =
  | EmptyResult;
```

## Best Practices

1. **Type Safety**
   - Use schema validation
   - Leverage type inference
   - Handle unknown types
   - Validate at boundaries

2. **Schema Design**
   - Keep schemas flat
   - Use composition
   - Document constraints
   - Version carefully

3. **Error Handling**
   - Use typed errors
   - Include context
   - Handle gracefully
   - Log appropriately

4. **Type Extensions**
   - Follow conventions
   - Document changes
   - Maintain compatibility
   - Test thoroughly

## Related Documentation

- [Protocol Specification](protocol-spec.md)
- [Implementation Patterns](../guides/implementation-patterns.md)
- [Error Handling](error-handling.md)
- [Cross-Language Guide](../guides/cross-language.md)

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>