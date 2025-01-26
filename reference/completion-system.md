# MCP Completion System

**Version:** 1.4.1  
**Component:** Completion System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a robust completion system that enables IDE-like autocompletion for prompts and resource URIs. This document details the completion system's architecture, implementation, and best practices.

## Core Components

### Reference Types
```typescript
interface PromptReference {
  type: "ref/prompt";
  name: string;
}

interface ResourceReference {
  type: "ref/resource";
  uri: string; // URI or URI template
}

type CompletionReference = PromptReference | ResourceReference;
```

### Request/Response Types
```typescript
interface CompleteRequest {
  ref: CompletionReference;
  argument: {
    name: string;
    value: string;
  };
}

interface CompleteResult {
  completion: {
    values: string[]; // Max 100 items
    total?: number;
    hasMore?: boolean;
  };
}
```

## Implementation

### Completion Provider
```typescript
interface CompletionProvider {
  // Get completion suggestions
  complete(params: CompleteRequest): Promise<CompleteResult>;
}

class CompletionManager implements CompletionProvider {
  // Fuzzy matching
  private fuzzyMatch(
    value: string,
    candidates: string[]
  ): string[] {
    return candidates.filter(c => 
      c.toLowerCase().includes(value.toLowerCase())
    );
  }

  // Rate limiting
  private rateLimiter = new RateLimiter({
    maxRequests: 10,
    timeWindow: 1000 // ms
  });

  // Handle completion request
  async complete(params: CompleteRequest): Promise<CompleteResult> {
    await this.rateLimiter.checkLimit();
    
    const { ref, argument } = params;
    const matches = await this.findMatches(ref, argument);
    
    return {
      completion: {
        values: matches.slice(0, 100),
        total: matches.length,
        hasMore: matches.length > 100
      }
    };
  }
}
```

### Prompt Completion
```typescript
class PromptCompletionProvider {
  // Get prompt argument completions
  async getPromptCompletions(
    promptName: string,
    argument: string,
    value: string
  ): Promise<string[]> {
    const prompt = await this.getPrompt(promptName);
    if (!prompt) return [];
    
    const arg = prompt.arguments?.find(a => a.name === argument);
    if (!arg) return [];
    
    return this.getArgumentCompletions(arg, value);
  }

  // Get argument-specific completions
  private async getArgumentCompletions(
    argument: PromptArgument,
    value: string
  ): Promise<string[]> {
    switch (argument.type) {
      case "language":
        return this.getLanguageCompletions(value);
      case "file":
        return this.getFileCompletions(value);
      default:
        return [];
    }
  }
}
```

### Resource Completion
```typescript
class ResourceCompletionProvider {
  // Get resource URI completions
  async getResourceCompletions(
    uriTemplate: string,
    value: string
  ): Promise<string[]> {
    const template = this.parseTemplate(uriTemplate);
    const variables = this.extractVariables(template);
    
    return this.expandTemplate(template, variables, value);
  }

  // Parse URI template
  private parseTemplate(template: string): URITemplate {
    return new URITemplate(template);
  }

  // Extract template variables
  private extractVariables(
    template: URITemplate
  ): TemplateVariable[] {
    return template.variables.map(v => ({
      name: v.name,
      required: !v.optional
    }));
  }
}
```

## Features

### Fuzzy Matching
- Case-insensitive matching
- Substring matching
- Prefix matching
- Edit distance calculation

### Caching
- Result caching
- Cache invalidation
- Cache size limits
- TTL-based expiration

### Rate Limiting
- Request rate limits
- Time windows
- Burst allowance
- Client tracking

### Ranking
- Relevance scoring
- Prefix matching priority
- Usage frequency
- Recent selections

## Best Practices

1. **Performance**
   - Implement efficient matching
   - Use appropriate caching
   - Optimize response size
   - Handle timeouts

2. **User Experience**
   - Sort by relevance
   - Provide quick responses
   - Handle partial matches
   - Support keyboard navigation

3. **Resource Management**
   - Limit result size
   - Clean up caches
   - Monitor memory usage
   - Handle failures gracefully

4. **Security**
   - Validate inputs
   - Rate limit requests
   - Control access
   - Prevent information leaks

## Error Handling

### Common Errors
```typescript
enum CompletionError {
  INVALID_REFERENCE = "invalid_reference",
  INVALID_ARGUMENT = "invalid_argument",
  RATE_LIMIT = "rate_limit",
  TIMEOUT = "timeout"
}

interface CompletionErrorResult {
  error: CompletionError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate inputs early
2. Return clear error messages
3. Include error details when safe
4. Log errors appropriately

## Related Documentation
- [Server Utilities](server-utilities.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by Alex Johnson (alex.johnson@company.com)</sub>