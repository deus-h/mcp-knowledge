# MCP Server Primitives

**Version:** 1.4.1  
**Component:** Server Primitives  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) defines three fundamental primitives that servers can expose to provide context and capabilities to language models:

1. **Prompts**: User-controlled templates for model interactions
2. **Resources**: Application-controlled contextual data
3. **Tools**: Model-controlled executable functions

## Prompts

### Overview
Prompts are user-controlled templates that guide language model interactions. They provide structured messages and instructions that users can explicitly select and customize.

### Implementation
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

interface PromptMessage {
  role: "user" | "assistant";
  content: TextContent | ImageContent | EmbeddedResource;
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

### Features
- User-initiated invocation
- Argument customization
- Support for text, images, and embedded resources
- Auto-completion for arguments
- Change notifications

## Resources

### Overview
Resources are application-controlled data sources that provide contextual information to the model. They represent structured data or content managed by the client.

### Implementation
```typescript
interface Resource {
  uri: string;
  name?: string;
  mimeType: string;
  metadata?: { [key: string]: unknown };
}

interface TextResourceContents {
  type: "text";
  text: string;
}

interface BlobResourceContents {
  type: "blob";
  data: string; // base64
  mimeType: string;
}

interface ResourceTemplate {
  name: string;
  description?: string;
  variables: TemplateVariable[];
}

interface TemplateVariable {
  name: string;
  description?: string;
  required?: boolean;
}
```

### Features
- URI-based identification
- Content streaming
- Subscription for updates
- Template-based creation
- Metadata support

## Tools

### Overview
Tools are model-controlled functions that enable language models to perform actions or retrieve information. They provide a way for models to interact with external systems.

### Implementation
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

interface ToolCall {
  name: string;
  arguments?: { [key: string]: unknown };
}

interface ToolResult {
  content: (TextContent | ImageContent | EmbeddedResource)[];
  isError?: boolean;
}
```

### Features
- Schema-based validation
- Progress tracking
- Error handling
- Rate limiting
- Access control

## Control Hierarchy

| Primitive | Control      | Description                                  | Example Use Cases                    |
|-----------|-------------|----------------------------------------------|-------------------------------------|
| Prompts   | User        | Interactive templates for model interactions  | Slash commands, menu options         |
| Resources | Application | Contextual data managed by the client        | File contents, git history           |
| Tools     | Model       | Executable functions for model actions        | API requests, file operations        |

## Best Practices

1. **Prompt Implementation**
   - Use clear, descriptive names
   - Provide helpful argument descriptions
   - Support argument completion
   - Handle template variables

2. **Resource Implementation**
   - Use consistent URI schemes
   - Implement efficient streaming
   - Handle concurrent access
   - Support partial reads

3. **Tool Implementation**
   - Validate inputs thoroughly
   - Provide clear error messages
   - Track progress for long operations
   - Implement timeouts

4. **Security Considerations**
   - Validate all inputs
   - Control access to sensitive data
   - Rate limit operations
   - Log important events

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by John Smith (john.smith@example.com)</sub>