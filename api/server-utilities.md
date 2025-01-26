# MCP Server Utilities API Reference

**Version:** 1.4.1  
**Component:** Server Utilities  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides several utility APIs for server implementations, including completion support, logging, and pagination. This document details these utilities and their implementation.

## Completion API

### Core Types
```typescript
interface CompletionProvider {
  complete(params: {
    ref: PromptReference | ResourceReference;
    argument: {
      name: string;
      value: string;
    };
  }): Promise<{
    values: string[];
    total?: number;
    hasMore?: boolean;
  }>;
}

interface PromptReference {
  type: "ref/prompt";
  name: string;
}

interface ResourceReference {
  type: "ref/resource";
  uri: string;
}
```

### Implementation
```typescript
class CompletionManager implements CompletionProvider {
  private promptCompletions = new Map<string, (value: string) => Promise<string[]>>();
  private resourceCompletions = new Map<string, (value: string) => Promise<string[]>>();

  // Register prompt completion
  registerPromptCompletion(
    name: string,
    handler: (value: string) => Promise<string[]>
  ): void {
    this.promptCompletions.set(name, handler);
  }

  // Register resource completion
  registerResourceCompletion(
    uriPattern: string,
    handler: (value: string) => Promise<string[]>
  ): void {
    this.resourceCompletions.set(uriPattern, handler);
  }

  // Complete request
  async complete(params: {
    ref: PromptReference | ResourceReference;
    argument: {
      name: string;
      value: string;
    };
  }): Promise<{
    values: string[];
    total?: number;
    hasMore?: boolean;
  }> {
    const { ref, argument } = params;

    if (ref.type === "ref/prompt") {
      return this.completePrompt(ref.name, argument);
    } else {
      return this.completeResource(ref.uri, argument);
    }
  }

  // Complete prompt argument
  private async completePrompt(
    name: string,
    argument: { name: string; value: string }
  ): Promise<{
    values: string[];
    total?: number;
    hasMore?: boolean;
  }> {
    const handler = this.promptCompletions.get(name);
    if (!handler) {
      return { values: [] };
    }

    const matches = await handler(argument.value);
    return {
      values: matches.slice(0, 100),
      total: matches.length,
      hasMore: matches.length > 100
    };
  }

  // Complete resource URI
  private async completeResource(
    uri: string,
    argument: { name: string; value: string }
  ): Promise<{
    values: string[];
    total?: number;
    hasMore?: boolean;
  }> {
    const handler = this.resourceCompletions.get(uri);
    if (!handler) {
      return { values: [] };
    }

    const matches = await handler(argument.value);
    return {
      values: matches.slice(0, 100),
      total: matches.length,
      hasMore: matches.length > 100
    };
  }
}
```

### Features
- Prompt argument completion
- Resource URI completion
- Fuzzy matching support
- Pagination support
- Rate limiting
- Caching support

### Best Practices
1. **Completion Implementation**
   - Sort by relevance
   - Implement fuzzy matching
   - Rate limit requests
   - Cache results

2. **Input Handling**
   - Validate all inputs
   - Handle partial matches
   - Support case sensitivity
   - Clean input data

3. **Performance**
   - Implement caching
   - Use rate limiting
   - Optimize matching
   - Monitor usage

4. **Security**
   - Validate inputs
   - Control access
   - Monitor usage
   - Log access

## Logging API

### Core Types
```typescript
type LoggingLevel =
  | "debug"
  | "info"
  | "notice"
  | "warning"
  | "error"
  | "critical"
  | "alert"
  | "emergency";

interface LogProvider {
  setLevel(level: LoggingLevel): void;
  log(params: {
    level: LoggingLevel;
    logger?: string;
    data: unknown;
  }): void;
}
```

### Implementation
```typescript
class LogManager implements LogProvider {
  private currentLevel: LoggingLevel = "info";
  private loggers = new Map<string, Logger>();
  private rateLimiter = new RateLimiter({
    maxRequests: 100,
    timeWindow: 1000 // ms
  });

  // Set minimum log level
  setLevel(level: LoggingLevel): void {
    this.currentLevel = level;
  }

  // Log message
  async log(params: {
    level: LoggingLevel;
    logger?: string;
    data: unknown;
  }): Promise<void> {
    if (!this.shouldLog(params.level)) {
      return;
    }

    await this.rateLimiter.checkLimit();
    
    const logger = this.getLogger(params.logger);
    logger.log(params.level, params.data);
  }

  // Check if level should be logged
  private shouldLog(level: LoggingLevel): boolean {
    const levels: LoggingLevel[] = [
      "debug",
      "info",
      "notice",
      "warning",
      "error",
      "critical",
      "alert",
      "emergency"
    ];

    const currentIndex = levels.indexOf(this.currentLevel);
    const messageIndex = levels.indexOf(level);
    return messageIndex >= currentIndex;
  }

  // Get or create logger
  private getLogger(name = "default"): Logger {
    let logger = this.loggers.get(name);
    if (!logger) {
      logger = new Logger(name);
      this.loggers.set(name, logger);
    }
    return logger;
  }
}

class Logger {
  constructor(private name: string) {}

  // Log message
  log(level: LoggingLevel, data: unknown): void {
    this.sanitizeData(data);
    this.sendNotification({
      level,
      logger: this.name,
      data
    });
  }

  // Remove sensitive data
  private sanitizeData(data: unknown): void {
    // Remove passwords, tokens, etc.
  }

  // Send notification
  private sendNotification(params: {
    level: LoggingLevel;
    logger: string;
    data: unknown;
  }): void {
    // Send notifications/message notification
  }
}
```

### Features
- Log levels (RFC 5424)
- Named loggers
- Rate limiting
- Data sanitization
- Level filtering
- Structured data

### Best Practices
1. **Logger Implementation**
   - Use consistent levels
   - Rate limit messages
   - Sanitize data
   - Monitor usage

2. **Message Content**
   - Include context
   - Use structured data
   - Remove sensitive info
   - Be concise

3. **Performance**
   - Implement rate limiting
   - Filter by level
   - Batch messages
   - Monitor volume

4. **Security**
   - Remove secrets
   - Validate data
   - Control access
   - Monitor content

## Pagination API

### Core Types
```typescript
type Cursor = string;

interface PaginationProvider {
  createCursor(state: unknown): Cursor;
  parseCursor(cursor: Cursor): unknown;
  cleanupCursors(): void;
}

interface PaginatedRequest {
  cursor?: Cursor;
}

interface PaginatedResult<T> {
  items: T[];
  nextCursor?: Cursor;
}
```

### Implementation
```typescript
class PaginationManager implements PaginationProvider {
  private cursors = new Map<Cursor, CursorState>();
  private expiryTime = 30 * 60 * 1000; // 30 minutes

  // Create cursor
  createCursor(state: unknown): Cursor {
    const cursor = this.generateCursor();
    this.cursors.set(cursor, {
      state,
      expires: Date.now() + this.expiryTime
    });
    return cursor;
  }

  // Parse cursor
  parseCursor(cursor: Cursor): unknown {
    const cursorState = this.cursors.get(cursor);
    if (!cursorState) {
      throw new Error("Invalid cursor");
    }

    if (cursorState.expires < Date.now()) {
      this.cursors.delete(cursor);
      throw new Error("Cursor expired");
    }

    return cursorState.state;
  }

  // Clean up expired cursors
  cleanupCursors(): void {
    const now = Date.now();
    for (const [cursor, state] of this.cursors) {
      if (state.expires < now) {
        this.cursors.delete(cursor);
      }
    }
  }

  // Generate cursor
  private generateCursor(): Cursor {
    return Buffer.from(
      crypto.randomBytes(32)
    ).toString('base64');
  }
}

interface CursorState {
  state: unknown;
  expires: number;
}
```

### Features
- Cursor-based pagination
- State management
- Cursor expiration
- Automatic cleanup
- Error handling
- Secure cursors

### Best Practices
1. **Cursor Management**
   - Use secure cursors
   - Handle expiration
   - Clean up state
   - Monitor usage

2. **State Management**
   - Store minimal state
   - Handle errors
   - Validate state
   - Clean up resources

3. **Performance**
   - Optimize page size
   - Cache results
   - Monitor memory
   - Clean up regularly

4. **Security**
   - Validate cursors
   - Secure state
   - Monitor usage
   - Log access

## Best Practices

1. **Completion Implementation**
   - Use fuzzy matching
   - Implement rate limiting
   - Cache results
   - Sort by relevance

2. **Logging Implementation**
   - Use appropriate levels
   - Include context
   - Support multiple loggers
   - Handle structured data

3. **Pagination Implementation**
   - Use cursor-based pagination
   - Clean up old cursors
   - Handle invalid cursors
   - Support variable page sizes

4. **Security Considerations**
   - Validate all inputs
   - Rate limit requests
   - Control access
   - Prevent information leaks

## Related Documentation
- [Server API Reference](server.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Protocol Specification](../reference/protocol-spec.md)

---

<div align="center">

Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se) | Started: January 25, 2025 | Last Updated: January 27, 2025

</div> 