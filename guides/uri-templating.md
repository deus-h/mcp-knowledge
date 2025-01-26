# URI Templating in MCP

**Version:** 1.4.1  
**Component:** URI Template Guide  
**Last Updated:** 2024

## Overview

The MCP SDK provides a robust implementation of RFC 6570 URI Templates, enabling dynamic resource addressing and flexible URI pattern matching. This guide covers the usage of URI templates for resource identification, pattern matching, and variable expansion.

## Core Concepts

### 1. Template Syntax

URI templates use curly braces `{}` to denote variable expressions:

```typescript
// Basic variable
const template = new UriTemplate("/api/users/{userId}");

// Query parameters
const template = new UriTemplate("/api/users{?page,limit}");

// Multiple variables
const template = new UriTemplate("/api/{org}/{repo}/issues/{number}");
```

### 2. Operators

The template system supports several operators for different expansion styles:

| Operator | Description | Example |
|----------|-------------|---------|
| none | Simple expansion | `{var}` |
| + | Reserved expansion | `{+var}` |
| # | Fragment identifier | `{#var}` |
| . | Label expansion | `{.var}` |
| / | Path segments | `{/var}` |
| ? | Query expansion | `{?var}` |
| & | Query continuation | `{&var}` |

## Usage Examples

### 1. Basic Resource Addressing

```typescript
const template = new UriTemplate("/api/resources/{resourceId}");

// Expand with variables
const uri = template.expand({
  resourceId: "123"
}); // "/api/resources/123"
```

### 2. Query Parameters

```typescript
const template = new UriTemplate("/api/search{?q,page,limit}");

const uri = template.expand({
  q: "typescript",
  page: "1",
  limit: "10"
}); // "/api/search?q=typescript&page=1&limit=10"
```

### 3. Multiple Variables

```typescript
const template = new UriTemplate("/api/{version}/users/{userId}/posts/{postId}");

const uri = template.expand({
  version: "v1",
  userId: "user123",
  postId: "post456"
}); // "/api/v1/users/user123/posts/post456"
```

### 4. Optional Parameters

```typescript
const template = new UriTemplate("/api/users{/userId}{?fields,include}");

// With all parameters
const uri1 = template.expand({
  userId: "123",
  fields: "name,email",
  include: "posts"
}); // "/api/users/123?fields=name,email&include=posts"

// With some parameters omitted
const uri2 = template.expand({
  userId: "123"
}); // "/api/users/123"
```

## Pattern Matching

The URI template system also supports matching URIs against templates:

```typescript
const template = new UriTemplate("/api/users/{userId}/posts/{postId}");

const match = template.match("/api/users/123/posts/456");
if (match) {
  console.log(match.userId); // "123"
  console.log(match.postId); // "456"
}
```

## Best Practices

1. **Template Design**
   - Keep templates simple and readable
   - Use meaningful variable names
   - Group related variables logically
   - Consider optional parameters

2. **Variable Handling**
   - Validate variable values before expansion
   - Handle missing variables gracefully
   - Use appropriate operators for different contexts
   - Consider URL encoding requirements

3. **Pattern Matching**
   - Validate matched values
   - Handle no-match cases
   - Consider case sensitivity
   - Test with various input patterns

4. **Security**
   - Validate template input
   - Sanitize variables before expansion
   - Consider length limits
   - Handle special characters properly

## Implementation Details

### 1. Template Validation

```typescript
class UriTemplate {
  static isTemplate(str: string): boolean {
    return /\{[^}\s]+\}/.test(str);
  }

  private static validateLength(
    str: string,
    max: number,
    context: string
  ): void {
    if (str.length > max) {
      throw new Error(
        `${context} exceeds maximum length of ${max} characters`
      );
    }
  }
}
```

### 2. Variable Expansion

```typescript
class UriTemplate {
  private encodeValue(value: string, operator: string): string {
    if (operator === "+" || operator === "#") {
      return encodeURI(value);
    }
    return encodeURIComponent(value);
  }

  expand(variables: Variables): string {
    // Implementation handles different operators
    // and combines parts appropriately
    return expandedUri;
  }
}
```

## Error Handling

1. **Template Validation Errors**
   - Invalid template syntax
   - Unclosed expressions
   - Invalid operators
   - Length exceeded

2. **Expansion Errors**
   - Missing required variables
   - Invalid variable values
   - Encoding errors
   - Length limits exceeded

3. **Pattern Matching Errors**
   - No match found
   - Invalid input URI
   - Regex compilation errors
   - Group extraction errors

## Related Documentation

- [Resource Management](resource-management.md)
- [Protocol Specification](../reference/protocol-spec.md)
- [Security Guide](security.md)
- [Error Handling](error-handling.md) 

---
<div align="center">
  <sub>Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se)</sub>
  <br>
  <sub>Started: January 25, 2025 | Last Updated: January 27, 2025</sub>
</div>