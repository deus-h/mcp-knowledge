# MCP Completable Types Guide

**Version:** 1.4.1  
**Component:** Completable Types  
**Last Updated:** 2024

## Overview

The MCP SDK provides a powerful type extension system called "Completable Types" that enables autocompletion and validation for dynamic values. This feature is particularly useful for implementing intelligent prompt arguments and dynamic resource paths.

## Core Concepts

### 1. Completable Type Definition

A completable type wraps a Zod schema with completion capabilities:

```typescript
export type CompleteCallback<T extends ZodTypeAny> = (
  value: T["_input"]
) => T["_input"][] | Promise<T["_input"][]>;

export interface CompletableDef<T extends ZodTypeAny> extends ZodTypeDef {
  type: T;
  complete: CompleteCallback<T>;
  typeName: McpZodTypeKind.Completable;
}
```

### 2. Creating Completable Types

```typescript
// Basic usage
const completableString = completable(
  z.string(),
  async (value) => {
    // Return completion suggestions
    return ["option1", "option2", "option3"];
  }
);

// With validation
const completableNumber = completable(
  z.number().min(0).max(100),
  async (value) => {
    // Generate suggestions based on input
    return [10, 20, 30, 40, 50];
  }
);
```

## Usage Examples

### 1. Resource Path Completion

```typescript
const pathCompletable = completable(
  z.string(),
  async (partial) => {
    // Implement path completion logic
    const suggestions = await fs.readdir(partial);
    return suggestions.map(s => path.join(partial, s));
  }
);

// Usage in resource definition
const resource = {
  uri: pathCompletable,
  type: "file"
};
```

### 2. Enum Value Completion

```typescript
const colorCompletable = completable(
  z.enum(["red", "green", "blue"]),
  async (value) => {
    // Filter suggestions based on input
    const colors = ["red", "green", "blue"];
    return colors.filter(c => c.startsWith(value));
  }
);

// Usage in prompt definition
const prompt = {
  color: colorCompletable,
  intensity: z.number()
};
```

### 3. Dynamic Suggestions

```typescript
const userCompletable = completable(
  z.string(),
  async (partial) => {
    // Fetch suggestions from API
    const users = await api.searchUsers(partial);
    return users.map(u => u.username);
  }
);

// Usage in tool definition
const tool = {
  name: "assign",
  params: {
    assignee: userCompletable
  }
};
```

## Implementation Patterns

### 1. Caching Suggestions

```typescript
class CachedCompletable<T extends ZodTypeAny> {
  private cache = new Map<string, T["_input"][]>();

  constructor(
    private schema: T,
    private ttl: number = 60000
  ) {}

  async complete(value: T["_input"]): Promise<T["_input"][]> {
    const cached = this.cache.get(String(value));
    if (cached) return cached;

    const suggestions = await this.fetchSuggestions(value);
    this.cache.set(String(value), suggestions);
    
    setTimeout(() => {
      this.cache.delete(String(value));
    }, this.ttl);

    return suggestions;
  }
}
```

### 2. Fuzzy Matching

```typescript
class FuzzyCompletable<T extends ZodTypeAny> {
  constructor(
    private options: T["_input"][],
    private threshold: number = 0.3
  ) {}

  async complete(value: T["_input"]): Promise<T["_input"][]> {
    return this.options.filter(opt => 
      this.calculateSimilarity(String(value), String(opt)) > this.threshold
    );
  }

  private calculateSimilarity(a: string, b: string): number {
    // Implement fuzzy matching logic
    return similarity(a, b);
  }
}
```

### 3. Progressive Loading

```typescript
class ProgressiveCompletable<T extends ZodTypeAny> {
  constructor(
    private pageSize: number = 10
  ) {}

  async complete(
    value: T["_input"],
    page: number = 0
  ): Promise<T["_input"][]> {
    const allSuggestions = await this.fetchSuggestions(value);
    return allSuggestions.slice(
      page * this.pageSize,
      (page + 1) * this.pageSize
    );
  }
}
```

## Best Practices

1. **Performance**
   - Cache frequently used suggestions
   - Implement pagination for large sets
   - Use debouncing for API calls
   - Optimize search algorithms

2. **User Experience**
   - Provide meaningful suggestions
   - Sort by relevance
   - Handle partial matches
   - Support keyboard navigation

3. **Error Handling**
   - Validate input values
   - Handle network failures
   - Provide fallback options
   - Clear error messages

4. **Resource Management**
   - Clean up cached data
   - Handle memory constraints
   - Monitor API usage
   - Implement timeouts

## Advanced Features

### 1. Context-Aware Completion

```typescript
interface CompletionContext {
  user: string;
  workspace: string;
  permissions: string[];
}

const contextAwareCompletable = <T extends ZodTypeAny>(
  schema: T,
  context: CompletionContext
) => completable(
  schema,
  async (value) => {
    // Use context for suggestions
    return generateSuggestions(value, context);
  }
);
```

### 2. Composite Completion

```typescript
const compositeCompletable = <T extends ZodTypeAny>(
  schemas: Completable<T>[]
) => completable(
  schemas[0].unwrap(),
  async (value) => {
    // Combine suggestions from multiple sources
    const suggestions = await Promise.all(
      schemas.map(s => s.complete(value))
    );
    return unique(suggestions.flat());
  }
);
```

## Related Documentation

- [Type System](types.md)
- [Resource Management](resources.md)
- [Tool Development](tools.md)
- [Performance Guide](performance.md)

---
<div align="center">
  <sub>Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se)</sub>
  <br>
  <sub>Started: January 25, 2025 | Last Updated: January 27, 2025</sub>
</div>