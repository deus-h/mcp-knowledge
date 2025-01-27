# MCP Progress System

**Version:** 1.4.1  
**Component:** Progress System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a robust progress tracking system that enables either party to report progress updates for long-running operations through notification messages. This document details the progress system's architecture, implementation, and best practices.

## Core Components

### Progress Types
```typescript
interface ProgressMeta {
  progressToken: string | number;
}

interface ProgressNotification {
  jsonrpc: "2.0";
  method: "notifications/progress";
  params: {
    progressToken: string | number;
    progress: number;
    total?: number;
  };
}

interface ProgressHandler {
  onProgress(
    token: string | number,
    progress: number,
    total?: number
  ): void;
}
```

## Implementation

### Progress Manager
```typescript
class ProgressManager {
  private handlers = new Map<string | number, ProgressHandler>();
  private activeTokens = new Set<string | number>();
  private rateLimiter: RateLimiter;

  constructor(config?: {
    maxUpdatesPerSecond?: number;
  }) {
    this.rateLimiter = new RateLimiter({
      maxRequests: config?.maxUpdatesPerSecond ?? 10,
      timeWindow: 1000
    });
  }

  // Register progress token
  registerToken(
    token: string | number,
    handler: ProgressHandler
  ): void {
    this.activeTokens.add(token);
    this.handlers.set(token, handler);
  }

  // Handle progress update
  async handleProgress(
    token: string | number,
    progress: number,
    total?: number
  ): Promise<void> {
    if (!this.activeTokens.has(token)) {
      return; // Token unknown or operation completed
    }

    await this.rateLimiter.checkLimit();
    
    const handler = this.handlers.get(token);
    if (handler) {
      handler.onProgress(token, progress, total);
      
      if (total && progress >= total) {
        this.cleanup(token);
      }
    }
  }

  // Clean up token
  private cleanup(token: string | number): void {
    this.activeTokens.delete(token);
    this.handlers.delete(token);
  }
}
```

### Progress Reporter
```typescript
class ProgressReporter {
  private lastProgress = 0;
  private token: string | number;
  private transport: Transport;

  constructor(
    token: string | number,
    transport: Transport
  ) {
    this.token = token;
    this.transport = transport;
  }

  // Report progress
  async reportProgress(
    progress: number,
    total?: number
  ): Promise<void> {
    if (progress <= this.lastProgress) {
      return; // Progress must increase
    }

    this.lastProgress = progress;
    
    const notification: ProgressNotification = {
      jsonrpc: "2.0",
      method: "notifications/progress",
      params: {
        progressToken: this.token,
        progress,
        total
      }
    };
    
    await this.transport.send(notification);
    
    if (total && progress >= total) {
      this.cleanup();
    }
  }

  // Clean up reporter
  private cleanup(): void {
    this.lastProgress = 0;
  }
}
```

## Features

### Progress Tracking
- Token management
- Progress updates
- Total tracking
- Rate limiting

### Progress Reporting
- Incremental updates
- Total estimation
- Completion detection
- State tracking

### Token Management
- Token generation
- Token validation
- Token cleanup
- State tracking

### Rate Control
- Update limiting
- Burst handling
- Backpressure
- Resource control

## Best Practices

1. **Progress Management**
   - Track tokens
   - Validate updates
   - Clean up completed
   - Handle rate limits

2. **Progress Reporting**
   - Increase monotonically
   - Estimate totals
   - Report completion
   - Handle failures

3. **Token Management**
   - Generate unique tokens
   - Track active tokens
   - Clean up expired
   - Monitor usage

4. **Rate Control**
   - Limit update rates
   - Handle bursts
   - Implement backoff
   - Monitor overhead

## Error Handling

### Common Errors
```typescript
enum ProgressError {
  INVALID_TOKEN = "invalid_token",
  INVALID_PROGRESS = "invalid_progress",
  RATE_LIMIT = "rate_limit",
  STATE_ERROR = "state_error"
}

interface ProgressErrorResult {
  error: ProgressError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate tokens and progress
2. Handle rate limits
3. Manage state errors
4. Clean up resources

## Security Considerations

### Progress Security
```typescript
class ProgressSecurity {
  // Validate token
  private validateToken(
    token: string | number
  ): boolean {
    return this.isValidToken(token) &&
           this.isActiveToken(token);
  }

  // Check progress update
  private validateUpdate(
    progress: number,
    lastProgress: number
  ): boolean {
    return progress > lastProgress &&
           this.isWithinLimits(progress);
  }

  // Monitor progress
  private monitorProgress(
    token: string | number
  ): void {
    this.trackUpdates(token);
    this.checkRateLimits(token);
    this.logProgress(token);
  }
}
```

### Best Practices
1. **Token Protection**
   - Validate tokens
   - Track usage
   - Monitor updates
   - Log anomalies

2. **Resource Protection**
   - Rate limit updates
   - Handle overload
   - Track resources
   - Monitor usage

## Related Documentation
- [Server Utilities](server-utilities.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by Jane Smith (jane.smith@company.com)</sub>