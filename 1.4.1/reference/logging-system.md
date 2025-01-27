# MCP Logging System

**Version:** 1.4.1  
**Component:** Logging System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a standardized logging system that enables servers to send structured log messages to clients. This document details the logging system's architecture, implementation, and best practices.

## Core Components

### Log Levels
```typescript
type LoggingLevel =
  | "debug"    // Detailed debugging information
  | "info"     // General informational messages
  | "notice"   // Normal but significant events
  | "warning"  // Warning conditions
  | "error"    // Error conditions
  | "critical" // Critical conditions
  | "alert"    // Action must be taken immediately
  | "emergency"; // System is unusable

interface LogMessage {
  level: LoggingLevel;
  logger?: string;
  data: unknown;
}

interface SetLevelRequest {
  level: LoggingLevel;
}
```

## Implementation

### Logging Provider
```typescript
interface LogProvider {
  // Set logging level
  setLevel(params: SetLevelRequest): Promise<void>;

  // Log message
  log(params: LogMessage): void;
}

class LogManager implements LogProvider {
  private currentLevel: LoggingLevel = "info";
  private loggers = new Map<string, Logger>();

  // Set logging level
  async setLevel(params: SetLevelRequest): Promise<void> {
    this.currentLevel = params.level;
    this.loggers.forEach(logger => 
      logger.setLevel(params.level)
    );
  }

  // Send log message
  log(params: LogMessage): void {
    const logger = this.getLogger(params.logger);
    if (this.shouldLog(params.level)) {
      logger.log(params.level, params.data);
    }
  }

  private shouldLog(level: LoggingLevel): boolean {
    const levels: LoggingLevel[] = [
      "debug", "info", "notice", "warning",
      "error", "critical", "alert", "emergency"
    ];
    return levels.indexOf(level) >= 
           levels.indexOf(this.currentLevel);
  }
}
```

### Logger Implementation
```typescript
class Logger {
  private name: string;
  private level: LoggingLevel;
  private rateLimiter: RateLimiter;

  constructor(name: string, level: LoggingLevel) {
    this.name = name;
    this.level = level;
    this.rateLimiter = new RateLimiter({
      maxRequests: 100,
      timeWindow: 1000 // ms
    });
  }

  // Set logger level
  setLevel(level: LoggingLevel): void {
    this.level = level;
  }

  // Log message with rate limiting
  async log(
    level: LoggingLevel,
    data: unknown
  ): Promise<void> {
    if (!this.shouldLog(level)) return;
    
    try {
      await this.rateLimiter.checkLimit();
      await this.sendNotification({
        level,
        logger: this.name,
        data: this.sanitizeData(data)
      });
    } catch (error) {
      // Handle rate limit or send errors
    }
  }

  // Sanitize log data
  private sanitizeData(data: unknown): unknown {
    return this.removeSecrets(
      this.removePII(
        this.validateData(data)
      )
    );
  }
}
```

## Features

### Log Levels
- **Debug**: Function entry/exit, variable values
- **Info**: Operation progress, state changes
- **Notice**: Configuration changes, normal events
- **Warning**: Deprecated features, potential issues
- **Error**: Operation failures, handled errors
- **Critical**: Component failures, data loss
- **Alert**: Immediate action required
- **Emergency**: Complete system failure

### Rate Limiting
- Message rate limits
- Time windows
- Burst allowance
- Per-logger limits

### Data Handling
- JSON serialization
- Data validation
- Secret removal
- PII protection

### Logger Management
- Named loggers
- Level inheritance
- Dynamic configuration
- Logger cleanup

## Best Practices

1. **Message Content**
   - Use appropriate levels
   - Include context
   - Structure data
   - Be concise

2. **Performance**
   - Implement rate limiting
   - Buffer messages
   - Batch updates
   - Handle backpressure

3. **Security**
   - Remove secrets
   - Protect PII
   - Validate data
   - Control access

4. **Resource Management**
   - Clean up loggers
   - Monitor memory
   - Handle failures
   - Implement timeouts

## Error Handling

### Common Errors
```typescript
enum LoggingError {
  INVALID_LEVEL = "invalid_level",
  RATE_LIMIT = "rate_limit",
  SEND_FAILED = "send_failed",
  INVALID_DATA = "invalid_data"
}

interface LoggingErrorResult {
  error: LoggingError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate levels and data
2. Handle rate limits gracefully
3. Retry failed sends
4. Log errors appropriately

## Security Considerations

### Data Protection
1. **Never Log**:
   - Passwords or secrets
   - API keys or tokens
   - Personal information
   - Internal system details

2. **Always**:
   - Validate input
   - Sanitize output
   - Control access
   - Monitor usage

### Implementation
```typescript
class LogSecurity {
  // Check for secrets
  private hasSecrets(data: unknown): boolean {
    return this.secretPatterns.some(pattern =>
      this.matchesPattern(data, pattern)
    );
  }

  // Check for PII
  private hasPII(data: unknown): boolean {
    return this.piiPatterns.some(pattern =>
      this.matchesPattern(data, pattern)
    );
  }

  // Sanitize data
  sanitize(data: unknown): unknown {
    if (this.hasSecrets(data)) {
      return this.redactSecrets(data);
    }
    if (this.hasPII(data)) {
      return this.anonymizePII(data);
    }
    return data;
  }
}
```

## Related Documentation
- [Server Utilities](server-utilities.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by DevOps Team (devops@company.com)</sub>