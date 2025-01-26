# MCP Implementation Patterns Guide

**Version:** 1.4.1  
**Component:** Implementation Patterns  
**Last Updated:** 2024

## Overview

This guide covers common implementation patterns and best practices for building MCP servers using the TypeScript SDK. These patterns are derived from examining real-world server implementations and the core SDK.

## Server Setup Patterns

### 1. Basic Server Setup

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
});

// Register capabilities
server.registerCapabilities({
  tools: {},
  resources: {},
  prompts: {}
});

// Connect transport
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 2. Configuration Management

```typescript
interface ServerConfig {
  allowedPaths: string[];
  maxRequestSize: number;
  timeoutMs: number;
  cacheEnabled: boolean;
}

class ConfigurableServer {
  private config: ServerConfig;
  
  constructor(config: ServerConfig) {
    this.config = {
      ...defaultConfig,
      ...config
    };
    
    this.server = new McpServer({
      name: "configurable-server",
      version: "1.0.0"
    });
  }
  
  async validateRequest(req: Request): Promise<void> {
    if (req.size > this.config.maxRequestSize) {
      throw new Error("Request too large");
    }
  }
}
```

## Resource Patterns

### 1. Resource Template Design

```typescript
// Dynamic path parameters
server.resource(
  "file",
  new ResourceTemplate("file://{path}", {
    list: async () => ({
      resources: await listAvailableFiles()
    })
  }),
  async (uri, { path }) => ({
    contents: [{
      uri: uri.href,
      text: await readFile(path)
    }]
  })
);

// Query parameters
server.resource(
  "search",
  new ResourceTemplate("search://{term}{?limit,offset}", {
    list: undefined
  }),
  async (uri, { term, limit, offset }) => ({
    contents: [{
      uri: uri.href,
      text: await performSearch(term, limit, offset)
    }]
  })
);
```

### 2. Resource Caching

```typescript
class CachedResource {
  private cache = new Map<string, {
    content: string;
    expiry: number;
  }>();

  constructor(
    private ttlMs: number = 60000
  ) {}

  async getContent(uri: string): Promise<string> {
    const cached = this.cache.get(uri);
    if (cached && Date.now() < cached.expiry) {
      return cached.content;
    }

    const content = await this.fetchContent(uri);
    this.cache.set(uri, {
      content,
      expiry: Date.now() + this.ttlMs
    });

    return content;
  }
}
```

## Tool Patterns

### 1. Input Validation

```typescript
server.tool(
  "process-data",
  {
    input: z.string(),
    options: z.object({
      format: z.enum(["json", "yaml", "xml"]),
      validate: z.boolean().default(true),
      maxSize: z.number().positive()
    }).partial()
  },
  async ({ input, options }) => {
    // Validate input size
    if (input.length > options?.maxSize ?? 1024) {
      throw new Error("Input too large");
    }

    // Process with options
    const result = await processData(input, options);
    return {
      content: [{
        type: "text",
        text: result
      }]
    };
  }
);
```

### 2. Progress Tracking

```typescript
server.tool(
  "long-operation",
  { data: z.string() },
  async ({ data }, { progressToken }) => {
    const steps = 10;
    for (let i = 0; i < steps; i++) {
      if (progressToken) {
        await server.notification({
          method: "progress",
          params: {
            progressToken,
            progress: {
              current: i,
              total: steps,
              message: `Processing step ${i + 1}`
            }
          }
        });
      }
      await processStep(i, data);
    }
    return { success: true };
  }
);
```

## Security Patterns

### 1. Path Validation

```typescript
class SecureFileSystem {
  constructor(
    private allowedPaths: string[]
  ) {}

  async validatePath(requestedPath: string): Promise<string> {
    const absolute = path.resolve(requestedPath);
    const normalized = path.normalize(absolute);

    const isAllowed = this.allowedPaths.some(
      allowed => normalized.startsWith(allowed)
    );

    if (!isAllowed) {
      throw new Error("Access denied");
    }

    return normalized;
  }
}
```

### 2. Request Sanitization

```typescript
class SecureServer {
  private sanitizeInput(input: unknown): string {
    if (typeof input !== "string") {
      throw new Error("Invalid input type");
    }

    // Remove dangerous characters
    return input.replace(/[<>]/g, "");
  }

  private validateRequest(req: Request): void {
    // Check authentication
    if (!this.isAuthenticated(req)) {
      throw new Error("Unauthorized");
    }

    // Rate limiting
    if (this.isRateLimited(req)) {
      throw new Error("Too many requests");
    }
  }
}
```

## Error Handling Patterns

### 1. Structured Error Response

```typescript
class ErrorHandler {
  handleError(error: unknown): JSONRPCError {
    if (error instanceof ValidationError) {
      return {
        code: -32602,
        message: "Invalid params",
        data: {
          details: error.details,
          path: error.path
        }
      };
    }

    if (error instanceof AuthError) {
      return {
        code: -32001,
        message: "Unauthorized",
        data: {
          reason: error.reason
        }
      };
    }

    // Default error
    return {
      code: -32603,
      message: "Internal error",
      data: {
        type: error?.constructor?.name
      }
    };
  }
}
```

### 2. Recovery Strategies

```typescript
class ResilientServer {
  async withRetry<T>(
    operation: () => Promise<T>,
    options: {
      maxAttempts: number;
      backoffMs: number;
    }
  ): Promise<T> {
    let lastError: Error;
    
    for (let i = 0; i < options.maxAttempts; i++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;
        await new Promise(resolve => 
          setTimeout(resolve, options.backoffMs * Math.pow(2, i))
        );
      }
    }
    
    throw lastError;
  }
}
```

## Performance Patterns

### 1. Connection Pooling

```typescript
class DatabasePool {
  private pool: Connection[] = [];
  private inUse = new Set<Connection>();

  async getConnection(): Promise<Connection> {
    let conn = this.pool.find(c => !this.inUse.has(c));
    
    if (!conn) {
      conn = await this.createConnection();
      this.pool.push(conn);
    }
    
    this.inUse.add(conn);
    return conn;
  }

  releaseConnection(conn: Connection): void {
    this.inUse.delete(conn);
  }
}
```

### 2. Request Batching

```typescript
class BatchProcessor {
  private queue: Request[] = [];
  private timer?: NodeJS.Timeout;

  queueRequest(req: Request): void {
    this.queue.push(req);
    
    if (!this.timer) {
      this.timer = setTimeout(() => {
        this.processBatch();
      }, 100);
    }
  }

  private async processBatch(): Promise<void> {
    const batch = this.queue;
    this.queue = [];
    this.timer = undefined;
    
    await Promise.all(
      batch.map(req => this.processRequest(req))
    );
  }
}
```

## Best Practices

1. **Server Design**
   - Use dependency injection
   - Implement graceful shutdown
   - Handle connection lifecycle
   - Monitor server health

2. **Resource Management**
   - Validate all paths
   - Implement proper caching
   - Handle large resources
   - Clean up resources

3. **Error Handling**
   - Use structured errors
   - Implement recovery
   - Log appropriately
   - Provide context

4. **Security**
   - Validate all input
   - Implement rate limiting
   - Use secure defaults
   - Handle authentication

## Related Documentation

- [Server Architecture](server-architecture.md)
- [Transport Layer](transports.md)
- [Error Handling](error-handling.md)
- [Security Guide](security.md) 

---
<div align="center">
  <sub>Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se)</sub>
  <br>
  <sub>Started: January 25, 2025 | Last Updated: January 27, 2025</sub>
</div>