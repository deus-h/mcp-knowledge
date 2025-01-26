# MCP Example Implementations

**Version:** 1.4.1  
**Component:** Example Implementations  
**Last Updated:** 2024

## Overview

This document provides an overview of example MCP server implementations, demonstrating common patterns and best practices for different use cases.

## Server Categories

### 1. File System Access

#### Filesystem Server
Provides access to local file system resources:
- File reading/writing
- Directory listing
- File metadata
- Path manipulation

#### Git Server
Git repository integration:
- Repository browsing
- Commit history
- File diffs
- Branch management

### 2. Database Access

#### PostgreSQL Server
SQL database integration:
- Query execution
- Schema inspection
- Data exploration
- Result formatting

#### SQLite Server
Lightweight database access:
- In-memory databases
- File-based storage
- Query tools
- Schema management

### 3. External Services

#### GitHub/GitLab Servers
Source code hosting integration:
- Repository access
- Issue tracking
- Pull requests
- Code search

#### Slack Server
Team communication integration:
- Message history
- Channel management
- User information
- File sharing

#### Google Services
Google service integration:
- Drive file access
- Maps location data
- Calendar events
- Email integration

### 4. Web Integration

#### Fetch Server
HTTP request handling:
- GET/POST requests
- Header management
- Response parsing
- Error handling

#### Puppeteer Server
Web automation:
- Page scraping
- Screenshot capture
- PDF generation
- DOM manipulation

### 5. Specialized Tools

#### AWS KB Retrieval
Knowledge base integration:
- Document search
- Content retrieval
- Metadata access
- Version tracking

#### Time Server
Time-related utilities:
- Current time
- Time zones
- Date formatting
- Calendar operations

## Implementation Patterns

### 1. Resource Management

```typescript
class ExampleServer {
  constructor() {
    this.server = new McpServer({
      name: "example-server",
      version: "1.0.0"
    });
    
    // Register resources
    this.server.resource("files", "/files/{path}", {
      list: this.listFiles.bind(this),
      read: this.readFile.bind(this)
    });
  }
  
  private async listFiles(): Promise<ListResourcesResult> {
    // Implementation
  }
  
  private async readFile(uri: URL): Promise<ReadResourceResult> {
    // Implementation
  }
}
```

### 2. Tool Implementation

```typescript
class DatabaseServer {
  constructor() {
    this.server = new McpServer({
      name: "database-server",
      version: "1.0.0"
    });
    
    // Register tools
    this.server.tool("query", {
      description: "Execute SQL query",
      inputSchema: z.object({
        query: z.string(),
        params: z.array(z.unknown()).optional()
      }),
      callback: this.executeQuery.bind(this)
    });
  }
  
  private async executeQuery(
    args: { query: string; params?: unknown[] }
  ): Promise<CallToolResult> {
    // Implementation
  }
}
```

### 3. Prompt Templates

```typescript
class AIServer {
  constructor() {
    this.server = new McpServer({
      name: "ai-server",
      version: "1.0.0"
    });
    
    // Register prompts
    this.server.prompt("analyze", {
      description: "Analyze data",
      argsSchema: z.object({
        data: z.string(),
        format: z.string().optional()
      }),
      callback: this.analyzeData.bind(this)
    });
  }
  
  private async analyzeData(
    args: { data: string; format?: string }
  ): Promise<GetPromptResult> {
    // Implementation
  }
}
```

## Common Patterns

### 1. Error Handling

```typescript
class RobustServer {
  private handleError(error: unknown): CallToolResult {
    if (error instanceof DatabaseError) {
      return {
        content: [{
          type: "text",
          text: `Database error: ${error.message}`
        }],
        isError: true
      };
    }
    
    return {
      content: [{
        type: "text",
        text: error instanceof Error ? error.message : String(error)
      }],
      isError: true
    };
  }
}
```

### 2. Resource Caching

```typescript
class CachingServer {
  private cache = new Map<string, CacheEntry>();
  
  private async getCachedResource(
    uri: string
  ): Promise<ReadResourceResult> {
    const cached = this.cache.get(uri);
    if (cached && !this.isExpired(cached)) {
      return cached.data;
    }
    
    const data = await this.fetchResource(uri);
    this.cache.set(uri, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
}
```

### 3. Progress Tracking

```typescript
class LongRunningServer {
  private async operationWithProgress(
    token: string
  ): Promise<void> {
    const total = 100;
    for (let i = 0; i < total; i++) {
      await this.server.sendNotification({
        method: "notifications/progress",
        params: {
          progressToken: token,
          progress: i,
          total
        }
      });
      
      await this.doWork();
    }
  }
}
```

## Best Practices

1. **Resource Organization**
   - Use meaningful URI structures
   - Implement proper pagination
   - Handle concurrent access
   - Cache when appropriate

2. **Tool Design**
   - Clear descriptions
   - Validate inputs thoroughly
   - Return meaningful errors
   - Support cancellation

3. **Prompt Management**
   - Template reusability
   - Clear documentation
   - Argument validation
   - Context management

4. **Error Handling**
   - Detailed error messages
   - Proper error types
   - Recovery strategies
   - Logging support

## Related Documentation

- [Server Implementation](../core/server.md)
- [Resource Management](../guides/resources.md)
- [Tool Development](../guides/tools.md)
- [Error Handling](../guides/error-handling.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 