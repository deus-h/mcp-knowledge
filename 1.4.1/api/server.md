# MCP Server API Reference

**Version:** 1.4.1  
**Component:** Server API  
**Last Updated:** 2024

## Overview

The MCP Server API provides interfaces for implementing servers that expose resources, tools, and prompts to clients. This document details the available APIs and their usage.

## Resource API

### Resource Management
```typescript
interface ResourceManager {
  // List available resources
  listResources(params?: {
    cursor?: string;
  }): Promise<{
    resources: Resource[];
    nextCursor?: string;
  }>;

  // Read resource contents
  readResource(params: {
    uri: string;
  }): Promise<{
    contents: (TextResourceContents | BlobResourceContents)[];
  }>;

  // Subscribe to resource updates
  subscribe(params: {
    uri: string;
  }): Promise<void>;

  // Unsubscribe from updates
  unsubscribe(params: {
    uri: string;
  }): Promise<void>;
}
```

### Resource Templates
```typescript
interface ResourceTemplateManager {
  // List available templates
  listTemplates(): Promise<{
    templates: ResourceTemplate[];
  }>;

  // Create resource from template
  createFromTemplate(params: {
    template: string;
    variables: { [key: string]: string };
  }): Promise<Resource>;
}
```

## Tool API

### Tool Management
```typescript
interface ToolManager {
  // List available tools
  listTools(params?: {
    cursor?: string;
  }): Promise<{
    tools: Tool[];
    nextCursor?: string;
  }>;

  // Call a tool
  callTool(params: {
    name: string;
    arguments?: { [key: string]: unknown };
  }): Promise<{
    content: (TextContent | ImageContent | EmbeddedResource)[];
    isError?: boolean;
  }>;
}
```

### Tool Definition
```typescript
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

## Prompt API

### Prompt Management
```typescript
interface PromptManager {
  // List available prompts
  listPrompts(params?: {
    cursor?: string;
  }): Promise<{
    prompts: Prompt[];
    nextCursor?: string;
  }>;

  // Get prompt template
  getPrompt(params: {
    name: string;
    arguments?: { [key: string]: string };
  }): Promise<{
    description?: string;
    messages: PromptMessage[];
  }>;
}
```

### Prompt Definition
```typescript
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
```

## Utility APIs

### Logging
```typescript
interface LogManager {
  // Set logging level
  setLevel(params: {
    level: LoggingLevel;
  }): Promise<void>;

  // Send log message
  log(params: {
    level: LoggingLevel;
    logger?: string;
    data: unknown;
  }): void;
}
```

### Completion
```typescript
interface CompletionManager {
  // Get completion suggestions
  complete(params: {
    ref: PromptReference | ResourceReference;
    argument: {
      name: string;
      value: string;
    };
  }): Promise<{
    completion: {
      values: string[];
      total?: number;
      hasMore?: boolean;
    };
  }>;
}
```

### Pagination
```typescript
interface PaginationManager {
  // Get next page of results
  getNextPage<T>(params: {
    cursor: string;
    pageSize?: number;
  }): Promise<{
    items: T[];
    nextCursor?: string;
  }>;
}
```

## Event Handling

### Notification Events
```typescript
interface NotificationHandler {
  // Resource events
  onResourceUpdated(uri: string): void;
  onResourceListChanged(): void;

  // Tool events
  onToolListChanged(): void;

  // Prompt events
  onPromptListChanged(): void;
}
```

### Progress Events
```typescript
interface ProgressHandler {
  // Report progress
  onProgress(params: {
    progressToken: string | number;
    progress: number;
    total?: number;
  }): void;
}
```

## Best Practices

1. **Resource Implementation**
   - Use URI templates for consistent resource identification
   - Implement efficient content streaming
   - Support partial resource reads
   - Handle concurrent access

2. **Tool Implementation**
   - Validate input schemas
   - Provide detailed error messages
   - Support progress tracking
   - Handle timeouts

3. **Prompt Implementation**
   - Document argument requirements
   - Support template variables
   - Handle embedded resources
   - Validate message formats

4. **Error Handling**
   - Use standard error codes
   - Provide detailed error messages
   - Include debugging information
   - Handle edge cases

## Related Documentation
- [Server Implementation Guide](../guides/implementation-patterns.md)
- [Type System Reference](../reference/type-system-advanced.md)
- [Protocol Specification](../reference/protocol-spec.md)

---

<div align="center">

Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se) | Started: January 25, 2025 | Last Updated: January 27, 2025

</div> 