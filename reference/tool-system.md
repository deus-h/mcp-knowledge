# MCP Tool System

**Version:** 1.4.1  
**Component:** Tool System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a standardized way for servers to expose tools that can be invoked by language models, enabling interaction with external systems such as databases, APIs, or computations. This document details the tool system's architecture, implementation, and best practices.

## Core Components

### Tool Types
```typescript
interface Tool {
  name: string;
  description: string;
  inputSchema: {
    type: "object";
    properties: {
      [key: string]: {
        type: string;
        description: string;
      };
    };
    required?: string[];
  };
}

interface ToolCall {
  name: string;
  arguments: { [key: string]: unknown };
}

interface ToolResult {
  content: (TextContent | ImageContent | EmbeddedResource)[];
  isError: boolean;
}

interface TextContent {
  type: "text";
  text: string;
}

interface ImageContent {
  type: "image";
  data: string; // base64
  mimeType: string;
}

interface EmbeddedResource {
  type: "resource";
  resource: {
    uri: string;
    mimeType: string;
    text?: string;
    blob?: string; // base64
  };
}
```

## Implementation

### Tool Manager
```typescript
class ToolManager {
  private tools = new Map<string, Tool>();
  private handlers = new Map<string, ToolHandler>();
  private listChangedHandlers = new Set<() => void>();

  // List tools with pagination
  async listTools(params?: {
    cursor?: string;
  }): Promise<{
    tools: Tool[];
    nextCursor?: string;
  }> {
    const tools = Array.from(this.tools.values());
    return this.paginate(tools, params);
  }

  // Call tool
  async callTool(params: {
    name: string;
    arguments: { [key: string]: unknown };
  }): Promise<{
    content: (TextContent | ImageContent | EmbeddedResource)[];
    isError: boolean;
  }> {
    const tool = this.tools.get(params.name);
    if (!tool) {
      throw new Error("Tool not found");
    }

    this.validateArguments(tool, params.arguments);
    const handler = this.handlers.get(params.name);
    
    if (!handler) {
      throw new Error("Tool handler not found");
    }

    return handler.handle(params.arguments);
  }

  // Register tool
  registerTool(
    tool: Tool,
    handler: ToolHandler
  ): void {
    this.validateTool(tool);
    this.tools.set(tool.name, tool);
    this.handlers.set(tool.name, handler);
    this.notifyListChanged();
  }

  // Subscribe to list changes
  onListChanged(handler: () => void): void {
    this.listChangedHandlers.add(handler);
  }

  // Notify list changes
  private notifyListChanged(): void {
    for (const handler of this.listChangedHandlers) {
      handler();
    }
  }
}
```

### Tool Handler
```typescript
interface ToolHandler {
  handle(
    args: { [key: string]: unknown }
  ): Promise<ToolResult>;
}

class BaseToolHandler implements ToolHandler {
  constructor(
    private executor: (
      args: { [key: string]: unknown }
    ) => Promise<ToolResult>
  ) {}

  // Handle tool call
  async handle(
    args: { [key: string]: unknown }
  ): Promise<ToolResult> {
    try {
      return await this.executor(args);
    } catch (error) {
      return {
        content: [{
          type: "text",
          text: `Error: ${error.message}`
        }],
        isError: true
      };
    }
  }
}
```

## Features

### Tool Management
- Name-based identification
- Schema validation
- Handler registration
- Change notifications

### Content Types
- Text content
- Image content
- Resource embedding
- Mixed content

### Schema System
- Input validation
- Type checking
- Required fields
- Documentation

### Error Handling
- Protocol errors
- Execution errors
- Result handling
- Error reporting

## Best Practices

1. **Tool Implementation**
   - Use clear names
   - Document schemas
   - Validate inputs
   - Handle errors

2. **Handler Implementation**
   - Validate arguments
   - Handle failures
   - Clean up resources
   - Report errors

3. **Content Management**
   - Support mixed types
   - Handle resources
   - Validate content
   - Monitor size

4. **Security Management**
   - Validate inputs
   - Check permissions
   - Monitor usage
   - Log access

## Error Handling

### Common Errors
```typescript
enum ToolError {
  TOOL_NOT_FOUND = "tool_not_found",
  INVALID_ARGUMENTS = "invalid_arguments",
  EXECUTION_ERROR = "execution_error",
  HANDLER_ERROR = "handler_error"
}

interface ToolErrorResult {
  error: ToolError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate tool and arguments
2. Handle execution errors
3. Manage handler errors
4. Clean up resources

## Security Considerations

### Tool Security
```typescript
class ToolSecurity {
  // Validate tool
  private validateTool(
    tool: Tool
  ): boolean {
    return this.isValidName(tool.name) &&
           this.isValidSchema(tool.inputSchema);
  }

  // Check arguments
  private validateArguments(
    args: { [key: string]: unknown }
  ): boolean {
    return this.areArgumentsSafe(args) &&
           this.isWithinLimits(args);
  }

  // Monitor usage
  private monitorUsage(
    toolName: string,
    user: string
  ): void {
    this.trackUsage(toolName, user);
    this.checkRateLimits(user);
    this.logAccess(toolName, user);
  }
}
```

### Best Practices
1. **Tool Protection**
   - Validate names
   - Check schemas
   - Monitor usage
   - Log access

2. **Execution Protection**
   - Validate inputs
   - Check permissions
   - Handle timeouts
   - Monitor usage

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>