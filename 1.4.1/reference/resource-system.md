# MCP Resource System

**Version:** 1.4.1  
**Component:** Resource System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a standardized way for servers to expose resources to clients, enabling access to contextual data such as files, database schemas, or application-specific information. This document details the resource system's architecture, implementation, and best practices.

## Core Components

### Resource Types
```typescript
interface Resource {
  uri: string;
  name: string;
  description?: string;
  mimeType: string;
}

interface ResourceContents {
  uri: string;
  mimeType: string;
  text?: string;
  blob?: string; // base64
}

interface ResourceTemplate {
  uriTemplate: string;
  name: string;
  description?: string;
  mimeType: string;
  variables?: TemplateVariable[];
}

interface TemplateVariable {
  name: string;
  description?: string;
  required?: boolean;
}
```

## Implementation

### Resource Manager
```typescript
class ResourceManager {
  private resources = new Map<string, Resource>();
  private templates = new Map<string, ResourceTemplate>();
  private subscribers = new Map<string, Set<() => void>>();
  private listChangedHandlers = new Set<() => void>();

  // List resources with pagination
  async listResources(params?: {
    cursor?: string;
  }): Promise<{
    resources: Resource[];
    nextCursor?: string;
  }> {
    const resources = Array.from(this.resources.values());
    return this.paginate(resources, params);
  }

  // Read resource contents
  async readResource(params: {
    uri: string;
  }): Promise<{
    contents: ResourceContents[];
  }> {
    const resource = this.resources.get(params.uri);
    if (!resource) {
      throw new Error("Resource not found");
    }

    return {
      contents: await this.loadContents(resource)
    };
  }

  // List resource templates
  async listTemplates(): Promise<{
    resourceTemplates: ResourceTemplate[];
  }> {
    return {
      resourceTemplates: Array.from(
        this.templates.values()
      )
    };
  }

  // Subscribe to resource changes
  subscribeToResource(
    uri: string,
    handler: () => void
  ): void {
    let handlers = this.subscribers.get(uri);
    if (!handlers) {
      handlers = new Set();
      this.subscribers.set(uri, handlers);
    }
    handlers.add(handler);
  }

  // Subscribe to list changes
  onListChanged(handler: () => void): void {
    this.listChangedHandlers.add(handler);
  }

  // Notify resource update
  private notifyResourceUpdated(uri: string): void {
    const handlers = this.subscribers.get(uri);
    if (handlers) {
      for (const handler of handlers) {
        handler();
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

### Template Manager
```typescript
class TemplateManager {
  private templates = new Map<string, ResourceTemplate>();

  // Register template
  registerTemplate(template: ResourceTemplate): void {
    this.validateTemplate(template);
    this.templates.set(template.uriTemplate, template);
  }

  // Create resource from template
  async createResource(
    template: ResourceTemplate,
    variables: { [key: string]: string }
  ): Promise<Resource> {
    this.validateVariables(template, variables);
    const uri = this.expandTemplate(
      template.uriTemplate,
      variables
    );
    
    return {
      uri,
      name: template.name,
      description: template.description,
      mimeType: template.mimeType
    };
  }

  // Validate template variables
  private validateVariables(
    template: ResourceTemplate,
    variables: { [key: string]: string }
  ): void {
    for (const variable of template.variables ?? []) {
      if (variable.required && !variables[variable.name]) {
        throw new Error(
          `Missing required variable: ${variable.name}`
        );
      }
    }
  }
}
```

## Features

### Resource Management
- URI-based identification
- Content retrieval
- Template support
- Change notifications

### Content Types
- Text content
- Binary content
- MIME type support
- Mixed content

### Template System
- URI templates
- Variable validation
- Template expansion
- Resource creation

### Subscription System
- Resource monitoring
- List monitoring
- Change notifications
- State tracking

## Best Practices

1. **Resource Implementation**
   - Use clear URIs
   - Document resources
   - Validate content
   - Handle errors

2. **Template Implementation**
   - Document variables
   - Validate inputs
   - Handle expansion
   - Clean up resources

3. **Content Management**
   - Support mixed types
   - Handle large content
   - Validate content
   - Monitor size

4. **Change Management**
   - Track changes
   - Notify subscribers
   - Handle races
   - Clean up state

## Error Handling

### Common Errors
```typescript
enum ResourceError {
  RESOURCE_NOT_FOUND = "resource_not_found",
  INVALID_URI = "invalid_uri",
  CONTENT_ERROR = "content_error",
  TEMPLATE_ERROR = "template_error"
}

interface ResourceErrorResult {
  error: ResourceError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate URIs and templates
2. Handle content errors
3. Manage state errors
4. Clean up resources

## Security Considerations

### Resource Security
```typescript
class ResourceSecurity {
  // Validate URI
  private validateUri(uri: string): boolean {
    return this.isValidUri(uri) &&
           this.isAllowedScheme(uri);
  }

  // Check access
  private checkAccess(
    uri: string,
    user: string
  ): boolean {
    return this.hasPermission(user, uri) &&
           this.isWithinLimits(user);
  }

  // Monitor access
  private monitorAccess(
    uri: string,
    user: string
  ): void {
    this.trackAccess(uri, user);
    this.checkRateLimits(user);
    this.logAccess(uri, user);
  }
}
```

### Best Practices
1. **Resource Protection**
   - Validate URIs
   - Check permissions
   - Monitor access
   - Log usage

2. **Content Protection**
   - Validate content
   - Check size limits
   - Handle timeouts
   - Monitor usage

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>