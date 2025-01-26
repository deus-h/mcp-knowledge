# MCP Prompt System

**Version:** 1.4.1  
**Component:** Prompt System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a standardized way for servers to expose prompt templates to clients, enabling structured interactions with language models. This document details the prompt system's architecture, implementation, and best practices.

## Core Components

### Prompt Types
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
  role: Role;
  content: TextContent | ImageContent | EmbeddedResource;
}

type Role = "user" | "assistant";

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
  resource: TextResourceContents | BlobResourceContents;
}
```

## Implementation

### Prompt Manager
```typescript
class PromptManager {
  private prompts = new Map<string, Prompt>();
  private templates = new Map<string, (args: { [key: string]: string }) => Promise<PromptMessage[]>>();
  private listChangedHandlers = new Set<() => void>();

  // List prompts with pagination
  async listPrompts(params?: {
    cursor?: string;
  }): Promise<{
    prompts: Prompt[];
    nextCursor?: string;
  }> {
    const prompts = Array.from(this.prompts.values());
    return this.paginate(prompts, params);
  }

  // Get prompt
  async getPrompt(params: {
    name: string;
    arguments?: { [key: string]: string };
  }): Promise<{
    description?: string;
    messages: PromptMessage[];
  }> {
    const prompt = this.prompts.get(params.name);
    if (!prompt) {
      throw new Error(`Prompt ${params.name} not found`);
    }

    this.validateArguments(prompt, params.arguments);
    const template = this.templates.get(params.name);
    
    if (!template) {
      throw new Error(`Template for prompt ${params.name} not found`);
    }

    const messages = await template(params.arguments ?? {});
    return {
      description: prompt.description,
      messages
    };
  }

  // Register prompt
  registerPrompt(
    prompt: Prompt,
    template: (args: { [key: string]: string }) => Promise<PromptMessage[]>
  ): void {
    if (this.prompts.has(prompt.name)) {
      throw new Error(`Prompt ${prompt.name} already registered`);
    }
    this.validatePrompt(prompt);
    this.prompts.set(prompt.name, prompt);
    this.templates.set(prompt.name, template);
    this.notifyListChanged();
  }

  // Subscribe to list changes
  onListChanged(handler: () => void): void {
    this.listChangedHandlers.add(handler);
  }

  // Validate arguments
  private validateArguments(
    prompt: Prompt,
    args?: { [key: string]: string }
  ): void {
    for (const arg of prompt.arguments ?? []) {
      if (arg.required && !args?.[arg.name]) {
        throw new Error(
          `Missing required argument: ${arg.name}`
        );
      }
    }
  }

  // Validate prompt
  private validatePrompt(prompt: Prompt): void {
    if (!prompt.name || typeof prompt.name !== 'string') {
      throw new Error('Invalid prompt name');
    }
    if (prompt.arguments) {
      for (const arg of prompt.arguments) {
        if (!arg.name || typeof arg.name !== 'string') {
          throw new Error('Invalid argument name');
        }
      }
    }
  }

  // Notify list changes
  private notifyListChanged(): void {
    for (const handler of this.listChangedHandlers) {
      handler();
    }
  }
}
```

## Features

### Prompt Management
- Name-based identification
- Argument validation
- Template support
- Change notifications
- Pagination support
- Duplicate prevention

### Content Types
- Text content
- Image content (base64)
- Resource embedding
- Mixed content support

### Template System
- Argument handling
- Message generation
- Content validation
- Error handling
- Async support

### Change Management
- List monitoring
- State tracking
- Change notifications
- Error handling
- Subscription management

## Best Practices

1. **Prompt Implementation**
   - Use descriptive names
   - Document all arguments
   - Validate inputs thoroughly
   - Handle errors gracefully
   - Support pagination
   - Implement change notifications

2. **Template Implementation**
   - Validate arguments before use
   - Handle async operations properly
   - Clean up resources after use
   - Report errors with context
   - Support mixed content types
   - Implement timeouts

3. **Content Management**
   - Support all content types
   - Handle resources efficiently
   - Validate content thoroughly
   - Monitor content size
   - Implement caching
   - Handle timeouts

4. **Security Management**
   - Validate all inputs
   - Check permissions
   - Monitor usage patterns
   - Log access attempts
   - Implement rate limiting
   - Sanitize outputs

## Error Handling

### Common Errors
```typescript
enum PromptError {
  PROMPT_NOT_FOUND = "prompt_not_found",
  INVALID_ARGUMENTS = "invalid_arguments",
  TEMPLATE_ERROR = "template_error",
  CONTENT_ERROR = "content_error",
  DUPLICATE_PROMPT = "duplicate_prompt",
  VALIDATION_ERROR = "validation_error"
}

interface PromptErrorResult {
  error: PromptError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate prompt and arguments thoroughly
2. Handle template errors with context
3. Manage content errors gracefully
4. Clean up resources on failure
5. Provide detailed error messages
6. Implement retry logic where appropriate

## Security Considerations

### Prompt Security
```typescript
class PromptSecurity {
  // Validate prompt
  private validatePrompt(
    prompt: Prompt
  ): boolean {
    return this.isValidName(prompt.name) &&
           this.areArgumentsSafe(prompt.arguments) &&
           this.isWithinSizeLimits(prompt);
  }

  // Check arguments
  private validateArguments(
    args: { [key: string]: string }
  ): boolean {
    return this.areArgumentsSafe(args) &&
           this.isWithinLimits(args) &&
           this.noInjectionRisks(args);
  }

  // Monitor usage
  private monitorUsage(
    promptName: string,
    user: string
  ): void {
    this.trackUsage(promptName, user);
    this.checkRateLimits(user);
    this.logAccess(promptName, user);
    this.alertOnAnomalies(promptName, user);
  }
}
```

### Best Practices
1. **Prompt Protection**
   - Validate names thoroughly
   - Check arguments for safety
   - Monitor usage patterns
   - Log all access attempts
   - Implement rate limiting
   - Alert on anomalies

2. **Content Protection**
   - Validate all content
   - Check size limits
   - Handle timeouts properly
   - Monitor usage patterns
   - Implement caching
   - Sanitize outputs

## Protocol Integration

### Capabilities
```typescript
interface PromptCapabilities {
  prompts: {
    listChanged: boolean;
  };
}
```

### Protocol Messages
1. **List Prompts Request**
   - Method: `prompts/list`
   - Supports pagination
   - Returns available prompts

2. **Get Prompt Request**
   - Method: `prompts/get`
   - Supports argument completion
   - Returns prompt messages

3. **List Changed Notification**
   - Method: `notifications/prompts/list_changed`
   - Sent when prompts change
   - No response expected

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

## Common Use Cases

### 1. Basic Text Prompt
```typescript
// Register a simple text prompt
promptManager.registerPrompt(
  {
    name: "greet",
    description: "Generate a greeting message",
    arguments: [
      {
        name: "name",
        description: "Name to greet",
        required: true
      },
      {
        name: "language",
        description: "Language for greeting",
        required: false
      }
    ]
  },
  async (args) => {
    const language = args.language || "English";
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: `Generate a greeting for ${args.name} in ${language}`
        }
      }
    ];
  }
);

// Use the prompt
const result = await promptManager.getPrompt({
  name: "greet",
  arguments: {
    name: "Alice",
    language: "Spanish"
  }
});
```

### 2. Multi-Turn Conversation
```typescript
// Register a conversation prompt
promptManager.registerPrompt(
  {
    name: "codeReview",
    description: "Review code changes",
    arguments: [
      {
        name: "code",
        description: "Code to review",
        required: true
      },
      {
        name: "language",
        description: "Programming language",
        required: true
      }
    ]
  },
  async (args) => {
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: `Please review this ${args.language} code:\n${args.code}`
        }
      },
      {
        role: "assistant",
        content: {
          type: "text",
          text: "I'll review the code for best practices and potential issues."
        }
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "Focus on security and performance aspects."
        }
      }
    ];
  }
);
```

### 3. Resource-Based Prompt
```typescript
// Register a prompt with embedded resource
promptManager.registerPrompt(
  {
    name: "analyzeLog",
    description: "Analyze log file contents",
    arguments: [
      {
        name: "logUri",
        description: "URI of the log file",
        required: true
      }
    ]
  },
  async (args) => {
    const resource = await resourceManager.readResource({ uri: args.logUri });
    return [
      {
        role: "user",
        content: {
          type: "resource",
          resource: resource
        }
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "Analyze this log file for errors and anomalies."
        }
      }
    ];
  }
);
```

### 4. Image Analysis Prompt
```typescript
// Register a prompt with image content
promptManager.registerPrompt(
  {
    name: "describeImage",
    description: "Generate description of an image",
    arguments: [
      {
        name: "imageData",
        description: "Base64 encoded image data",
        required: true
      },
      {
        name: "format",
        description: "Image format (e.g., 'png', 'jpeg')",
        required: true
      }
    ]
  },
  async (args) => {
    return [
      {
        role: "user",
        content: {
          type: "image",
          data: args.imageData,
          mimeType: `image/${args.format}`
        }
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "Describe what you see in this image in detail."
        }
      }
    ];
  }
);
```

### 5. Paginated Prompt Listing
```typescript
// List prompts with pagination
async function listAllPrompts(): Promise<Prompt[]> {
  const allPrompts: Prompt[] = [];
  let cursor: string | undefined;
  
  do {
    const result = await promptManager.listPrompts({ cursor });
    allPrompts.push(...result.prompts);
    cursor = result.nextCursor;
  } while (cursor);
  
  return allPrompts;
}

// Usage example
const prompts = await listAllPrompts();
for (const prompt of prompts) {
  console.log(`${prompt.name}: ${prompt.description}`);
}
```

### 6. Change Notification Handling
```typescript
// Subscribe to prompt list changes
promptManager.onListChanged(() => {
  console.log("Prompt list has changed");
  // Refresh UI or cache
  listAllPrompts().then(prompts => {
    updatePromptCache(prompts);
    refreshPromptUI(prompts);
  });
});

// Example prompt registration with notification
promptManager.registerPrompt(
  {
    name: "newPrompt",
    description: "A new prompt template"
  },
  async () => {
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: "This is a new prompt."
        }
      }
    ];
  }
); // This will trigger the onListChanged handler
```

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 