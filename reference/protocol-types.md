# MCP Protocol Types

**Version:** 1.4.1  
**Component:** Protocol Types  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) is built on JSON-RPC 2.0 and defines a comprehensive type system for client-server communication. This document details the core protocol types, their structure, and usage patterns.

## Core Types

### JSON-RPC Messages
```typescript
type JSONRPCMessage =
  | JSONRPCRequest
  | JSONRPCNotification
  | JSONRPCResponse
  | JSONRPCError;

type RequestId = string | number;
type ProgressToken = string | number;
type Cursor = string;

interface Request {
  method: string;
  params?: {
    _meta?: {
      progressToken?: ProgressToken;
    };
    [key: string]: unknown;
  };
}

interface Notification {
  method: string;
  params?: {
    _meta?: { [key: string]: unknown };
    [key: string]: unknown;
  };
}

interface Result {
  _meta?: { [key: string]: unknown };
  [key: string]: unknown;
}

interface JSONRPCRequest extends Request {
  jsonrpc: "2.0";
  id: RequestId;
}

interface JSONRPCNotification extends Notification {
  jsonrpc: "2.0";
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

### Protocol Constants
```typescript
const LATEST_PROTOCOL_VERSION = "2024-11-05";
const JSONRPC_VERSION = "2.0";

// Standard JSON-RPC error codes
const PARSE_ERROR = -32700;
const INVALID_REQUEST = -32600;
const METHOD_NOT_FOUND = -32601;
const INVALID_PARAMS = -32602;
const INTERNAL_ERROR = -32603;
```

### Initialization Types
```typescript
interface InitializeRequest extends Request {
  method: "initialize";
  params: {
    protocolVersion: string;
    capabilities: ClientCapabilities;
    clientInfo: Implementation;
  };
}

interface InitializeResult extends Result {
  protocolVersion: string;
  capabilities: ServerCapabilities;
  serverInfo: Implementation;
  instructions?: string;
}

interface InitializedNotification extends Notification {
  method: "notifications/initialized";
}
```

### Capability Types
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

### Utility Types
```typescript
interface CancelledNotification extends Notification {
  method: "notifications/cancelled";
  params: {
    requestId: RequestId;
    reason?: string;
  };
}

type EmptyResult = Result;
```

### Implementation Type
```typescript
interface Implementation {
  name: string;
  version: string;
}
```

### Ping Types
```typescript
interface PingRequest extends Request {
  method: "ping";
}
```

### Progress Types
```typescript
interface ProgressNotification extends Notification {
  method: "notifications/progress";
  params: {
    progressToken: ProgressToken;
    progress: number;
    total?: number;
  };
}
```

### Pagination Types
```typescript
interface PaginatedRequest extends Request {
  params?: {
    cursor?: Cursor;
  };
}

interface PaginatedResult extends Result {
  nextCursor?: Cursor;
}
```

### Resource Types
```typescript
interface ListResourcesRequest extends PaginatedRequest {
  method: "resources/list";
}

interface ListResourcesResult extends PaginatedResult {
  resources: Resource[];
}

interface ListResourceTemplatesRequest extends PaginatedRequest {
  method: "resources/templates/list";
}

interface ListResourceTemplatesResult extends PaginatedResult {
  resourceTemplates: ResourceTemplate[];
}

interface ReadResourceRequest extends Request {
  method: "resources/read";
  params: {
    uri: string;
  };
}

interface ReadResourceResult extends Result {
  contents: (TextResourceContents | BlobResourceContents)[];
}

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

interface ResourceContents {
  uri: string;
  mimeType?: string;
}
```

### Resource Notification Types
```typescript
interface ResourceListChangedNotification extends Notification {
  method: "notifications/resources/list_changed";
}

interface SubscribeRequest extends Request {
  method: "resources/subscribe";
  params: {
    uri: string;
  };
}

interface UnsubscribeRequest extends Request {
  method: "resources/unsubscribe";
  params: {
    uri: string;
  };
}

interface ResourceUpdatedNotification extends Notification {
  method: "notifications/resources/updated";
  params: {
    uri: string;
  };
}
```

### Resource Content Types
```typescript
interface TextResourceContents extends ResourceContents {
  text: string;
}

interface BlobResourceContents extends ResourceContents {
  blob: string; // base64
}
```

### Prompt Types
```typescript
interface ListPromptsRequest extends PaginatedRequest {
  method: "prompts/list";
}

interface ListPromptsResult extends PaginatedResult {
  prompts: Prompt[];
}

interface GetPromptRequest extends Request {
  method: "prompts/get";
  params: {
    name: string;
    arguments?: { [key: string]: string };
  };
}

interface GetPromptResult extends Result {
  description?: string;
  messages: PromptMessage[];
}

interface Prompt {
  name: string;
  description?: string;
  arguments?: PromptArgument[];
}

interface PromptArgument {
  name: string;
  description?: string;
  required?: boolean;
}

type Role = "user" | "assistant";

interface PromptMessage {
  role: Role;
  content: TextContent | ImageContent | EmbeddedResource;
}

interface EmbeddedResource {
  type: "resource";
  resource: TextResourceContents | BlobResourceContents;
}

interface PromptListChangedNotification extends Notification {
  method: "notifications/prompts/list_changed";
}
```

### Tool Types
```typescript
interface ListToolsRequest extends PaginatedRequest {
  method: "tools/list";
}

interface ListToolsResult extends PaginatedResult {
  tools: Tool[];
}

interface CallToolResult extends Result {
  content: (TextContent | ImageContent | EmbeddedResource)[];
  isError?: boolean;
}

interface CallToolRequest extends Request {
  method: "tools/call";
  params: {
    name: string;
    arguments?: { [key: string]: unknown };
  };
}

interface ToolListChangedNotification extends Notification {
  method: "notifications/tools/list_changed";
}

interface Tool {
  name: string;
  description?: string;
  inputSchema: {
    type: "object";
    properties?: { [key: string]: object };
    required?: string[];
  };
}
```

### Logging Types
```typescript
interface SetLevelRequest extends Request {
  method: "logging/setLevel";
  params: {
    level: LoggingLevel;
  };
}

interface LoggingMessageNotification extends Notification {
  method: "notifications/message";
  params: {
    level: LoggingLevel;
    logger?: string;
    data: unknown;
  };
}

type LoggingLevel = string; // Maps to syslog severities (RFC-5424)
```

### Logging Level Type
```typescript
type LoggingLevel =
  | "debug"
  | "info"
  | "notice"
  | "warning"
  | "error"
  | "critical"
  | "alert"
  | "emergency";
```

### Sampling Types
```typescript
interface CreateMessageRequest extends Request {
  method: "sampling/createMessage";
  params: {
    messages: SamplingMessage[];
    modelPreferences?: ModelPreferences;
    systemPrompt?: string;
    includeContext?: "none" | "thisServer" | "allServers";
    temperature?: number;
    maxTokens: number;
    stopSequences?: string[];
    metadata?: object;
  };
}

interface CreateMessageResult extends Result, SamplingMessage {
  model: string;
  stopReason?: "endTurn" | "stopSequence" | "maxTokens" | string;
}

interface SamplingMessage {
  role: Role;
  content: TextContent | ImageContent;
}
```

### Content Types
```typescript
interface Annotated {
  annotations?: {
    audience?: Role[];
    priority?: number; // 0-1
  }
}

interface TextContent extends Annotated {
  type: "text";
  text: string;
}

interface ImageContent extends Annotated {
  type: "image";
  data: string; // base64
  mimeType: string;
}
```

### Model Types
```typescript
interface ModelPreferences {
  hints?: ModelHint[];
  costPriority?: number; // 0-1
  speedPriority?: number; // 0-1
  intelligencePriority?: number; // 0-1
}

interface ModelHint {
  name?: string;
}
```

### Completion Types
```typescript
interface CompleteRequest extends Request {
  method: "completion/complete";
  params: {
    ref: PromptReference | ResourceReference;
    argument: {
      name: string;
      value: string;
    };
  };
}

interface CompleteResult extends Result {
  completion: {
    values: string[]; // Max 100 items
    total?: number;
    hasMore?: boolean;
  };
}
```

### Reference Types
```typescript
interface ResourceReference {
  type: "ref/resource";
  uri: string;
}

interface PromptReference {
  type: "ref/prompt";
  name: string;
}
```

### Root Types
```typescript
interface ListRootsRequest extends Request {
  method: "roots/list";
}

interface ListRootsResult extends Result {
  roots: Root[];
}

interface Root {
  uri: string; // Must start with file://
  name?: string;
}

interface RootsListChangedNotification extends Notification {
  method: "notifications/roots/list_changed";
}
```

### Message Type Groups
```typescript
type ClientRequest =
  | PingRequest
  | InitializeRequest
  | CompleteRequest
  | SetLevelRequest
  | GetPromptRequest
  | ListPromptsRequest
  | ListResourcesRequest
  | ListResourceTemplatesRequest
  | ReadResourceRequest
  | SubscribeRequest
  | UnsubscribeRequest
  | CallToolRequest
  | ListToolsRequest;

type ClientNotification =
  | CancelledNotification
  | ProgressNotification
  | InitializedNotification
  | RootsListChangedNotification;

type ClientResult = 
  | EmptyResult 
  | CreateMessageResult 
  | ListRootsResult;

type ServerRequest =
  | PingRequest
  | CreateMessageRequest
  | ListRootsRequest;

type ServerNotification =
  | CancelledNotification
  | ProgressNotification
  | LoggingMessageNotification
  | ResourceUpdatedNotification
  | ResourceListChangedNotification
  | ToolListChangedNotification
  | PromptListChangedNotification;

type ServerResult =
  | EmptyResult
  | InitializeResult
  | CompleteResult
  | GetPromptResult
  | ListPromptsResult
  | ListResourcesResult
  | ListResourceTemplatesResult
  | ReadResourceResult
  | CallToolResult
  | ListToolsResult;
```

## Features

### Message Types
- Request messages
- Notification messages
- Response messages
- Error messages

### Protocol Features
- Version negotiation
- Capability declaration
- Progress tracking
- Cancellation support
- Resource management
- Pagination support

### Metadata Support
- Request metadata
- Response metadata
- Notification metadata
- Error details
- Resource metadata

### Resource Features
- URI-based identification
- Template support
- Content types
- Subscription system
- Change notifications

### Error Handling
- Standard error codes
- Error messages
- Error details
- Cancellation

### Prompt Features
- Named prompts
- Argument support
- Message roles
- Content types
- Change notifications

### Tool Features
- Named tools
- Schema validation
- Error handling
- Content types
- Change notifications

### Logging Features
- Log levels
- Named loggers
- Structured data
- Level control

### Sampling Features
- Message creation
- Model preferences
- System prompts
- Context control
- Temperature control
- Token limits
- Stop sequences

### Content Features
- Text content
- Image content
- Annotations
- Audience targeting
- Priority levels

### Model Features
- Model hints
- Cost priority
- Speed priority
- Intelligence priority
- Model selection

### Completion Features
- Prompt completion
- Resource completion
- Value limits
- Pagination support

### Reference Features
- Resource references
- Prompt references
- URI templates
- Name-based lookup

### Root Features
- File system roots
- Named roots
- Root listing
- Change notifications

### Message Groups
- Client requests
- Client notifications
- Client results
- Server requests
- Server notifications
- Server results

## Best Practices

1. **Message Handling**
   - Validate message format
   - Check protocol version
   - Handle metadata
   - Process errors

2. **Protocol Usage**
   - Negotiate versions
   - Declare capabilities
   - Track progress
   - Handle cancellation

3. **Error Management**
   - Use standard codes
   - Provide clear messages
   - Include details
   - Clean up state

4. **State Management**
   - Track requests
   - Handle notifications
   - Manage progress
   - Clean up resources

## Error Handling

### Standard Error Codes
```typescript
enum StandardError {
  PARSE_ERROR = -32700,
  INVALID_REQUEST = -32600,
  METHOD_NOT_FOUND = -32601,
  INVALID_PARAMS = -32602,
  INTERNAL_ERROR = -32603
}

interface ErrorDetails {
  code: StandardError;
  message: string;
  data?: unknown;
}
```

### Error Handling Strategy
1. Validate message format
2. Check protocol version
3. Handle standard errors
4. Clean up resources

## Security Considerations

### Message Security
```typescript
class MessageSecurity {
  // Validate message
  private validateMessage(
    message: JSONRPCMessage
  ): boolean {
    return this.isValidFormat(message) &&
           this.isValidVersion(message);
  }

  // Check metadata
  private validateMetadata(
    meta: unknown
  ): boolean {
    return this.isSafeMetadata(meta) &&
           this.isWithinLimits(meta);
  }

  // Monitor messages
  private monitorMessages(
    message: JSONRPCMessage
  ): void {
    this.trackMessages();
    this.checkRateLimits();
    this.logMessage(message);
  }
}
```

### Best Practices
1. **Message Protection**
   - Validate format
   - Check versions
   - Monitor usage
   - Log messages

2. **Data Protection**
   - Validate metadata
   - Check size limits
   - Handle timeouts
   - Monitor usage

## Related Documentation
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Server API Reference](server.md)

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>