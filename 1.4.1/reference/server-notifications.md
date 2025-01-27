# MCP Server Notifications

**Version:** 1.4.1  
**Component:** Server Notifications  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a standardized way for servers to send notifications to clients about various events and state changes. This document details the server notification system's architecture, implementation, and best practices.

## Core Components

### Notification Types
```typescript
interface ServerNotification {
  jsonrpc: "2.0";
  method: string;
  params?: unknown;
}

// Resource Notifications
interface ResourceUpdatedNotification extends Notification {
  method: "notifications/resources/updated";
  params: {
    uri: string; // URI of updated resource
  };
}

interface ResourceListChangedNotification extends Notification {
  method: "notifications/resources/list_changed";
}

// Tool Notifications
interface ToolListChangedNotification extends Notification {
  method: "notifications/tools/list_changed";
}

// Prompt Notifications
interface PromptListChangedNotification extends Notification {
  method: "notifications/prompts/list_changed";
}

// Progress Notifications
interface ProgressNotification extends Notification {
  method: "notifications/progress";
  params: {
    progressToken: string | number;
    progress: number;
    total?: number;
  };
}

// Logging Notifications
interface LoggingMessageNotification extends Notification {
  method: "notifications/message";
  params: {
    level: LoggingLevel;
    logger?: string;
    data: unknown;
  };
}

type LoggingLevel =
  | "debug"
  | "info"
  | "notice"
  | "warning"
  | "error"
  | "critical"
  | "alert"
  | "emergency";
```

## Implementation

### Notification Manager
```typescript
class NotificationManager {
  private listChangedHandlers = new Map<string, Set<() => void>>();
  private transport: Transport;

  constructor(transport: Transport) {
    this.transport = transport;
  }

  // Send resource update notification
  async sendResourceUpdated(uri: string): Promise<void> {
    await this.sendNotification("notifications/resources/updated", { uri });
  }

  // Send resource list changed notification
  async sendResourceListChanged(): Promise<void> {
    await this.sendNotification("notifications/resources/list_changed");
  }

  // Send tool list changed notification
  async sendToolListChanged(): Promise<void> {
    await this.sendNotification("notifications/tools/list_changed");
  }

  // Send prompt list changed notification
  async sendPromptListChanged(): Promise<void> {
    await this.sendNotification("notifications/prompts/list_changed");
  }

  // Send progress notification
  async sendProgress(
    progressToken: string | number,
    progress: number,
    total?: number
  ): Promise<void> {
    await this.sendNotification("notifications/progress", {
      progressToken,
      progress,
      total
    });
  }

  // Send log message notification
  async sendLogMessage(
    level: LoggingLevel,
    data: unknown,
    logger?: string
  ): Promise<void> {
    await this.sendNotification("notifications/message", {
      level,
      data,
      logger
    });
  }

  // Generic notification sender
  private async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const notification: ServerNotification = {
      jsonrpc: "2.0",
      method,
      params
    };
    await this.transport.send(notification);
  }

  // Subscribe to list changes
  onListChanged(type: string, handler: () => void): void {
    if (!this.listChangedHandlers.has(type)) {
      this.listChangedHandlers.set(type, new Set());
    }
    this.listChangedHandlers.get(type)!.add(handler);
  }

  // Unsubscribe from list changes
  offListChanged(type: string, handler: () => void): void {
    this.listChangedHandlers.get(type)?.delete(handler);
  }
}
```

## Advanced Implementation

### Transport Layer Integration
```typescript
interface TransportConfig {
  reconnectInterval: number;
  maxReconnectAttempts: number;
  bufferSize: number;
  timeout: number;
}

class NotificationTransport {
  private transport: Transport;
  private config: TransportConfig;
  private buffer: ServerNotification[] = [];
  private isConnected = false;
  private reconnectAttempts = 0;
  
  constructor(transport: Transport, config: TransportConfig) {
    this.transport = transport;
    this.config = config;
    this.setupTransport();
  }

  private setupTransport(): void {
    this.transport.onDisconnect(() => {
      this.isConnected = false;
      this.handleDisconnect();
    });

    this.transport.onConnect(() => {
      this.isConnected = true;
      this.reconnectAttempts = 0;
      this.flushBuffer();
    });
  }

  private async handleDisconnect(): Promise<void> {
    if (this.reconnectAttempts >= this.config.maxReconnectAttempts) {
      throw new Error('Max reconnection attempts reached');
    }

    await new Promise(resolve => 
      setTimeout(resolve, this.config.reconnectInterval)
    );
    
    this.reconnectAttempts++;
    await this.transport.connect();
  }

  private async flushBuffer(): Promise<void> {
    while (this.buffer.length > 0 && this.isConnected) {
      const notification = this.buffer.shift();
      if (notification) {
        try {
          await this.transport.send(notification);
        } catch (error) {
          this.buffer.unshift(notification);
          break;
        }
      }
    }
  }

  async send(notification: ServerNotification): Promise<void> {
    if (!this.isConnected) {
      if (this.buffer.length >= this.config.bufferSize) {
        throw new Error('Notification buffer full');
      }
      this.buffer.push(notification);
      return;
    }

    try {
      await Promise.race([
        this.transport.send(notification),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Send timeout')), this.config.timeout)
        )
      ]);
    } catch (error) {
      this.buffer.push(notification);
      throw error;
    }
  }
}
```

### Notification Batching
```typescript
interface NotificationBatch {
  notifications: ServerNotification[];
  timestamp: number;
}

class BatchingNotificationManager extends NotificationManager {
  private batchSize: number;
  private batchTimeout: number;
  private currentBatch: NotificationBatch;
  private batchTimer: NodeJS.Timeout | null = null;

  constructor(
    transport: Transport, 
    batchSize: number = 10,
    batchTimeout: number = 1000
  ) {
    super(transport);
    this.batchSize = batchSize;
    this.batchTimeout = batchTimeout;
    this.currentBatch = this.createNewBatch();
  }

  private createNewBatch(): NotificationBatch {
    return {
      notifications: [],
      timestamp: Date.now()
    };
  }

  private async flushBatch(): Promise<void> {
    if (this.currentBatch.notifications.length === 0) {
      return;
    }

    try {
      // Send all notifications in batch
      await Promise.all(
        this.currentBatch.notifications.map(notification =>
          super.sendNotification(notification.method, notification.params)
        )
      );
    } catch (error) {
      console.error('Failed to send batch:', error);
      // Implement retry logic here
    }

    this.currentBatch = this.createNewBatch();
  }

  private scheduleBatchFlush(): void {
    if (this.batchTimer) {
      clearTimeout(this.batchTimer);
    }

    this.batchTimer = setTimeout(
      () => this.flushBatch(),
      this.batchTimeout
    );
  }

  async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const notification: ServerNotification = {
      jsonrpc: "2.0",
      method,
      params
    };

    this.currentBatch.notifications.push(notification);

    if (this.currentBatch.notifications.length >= this.batchSize) {
      await this.flushBatch();
    } else {
      this.scheduleBatchFlush();
    }
  }
}
```

### Priority Queue Implementation
```typescript
enum NotificationPriority {
  HIGH = 0,
  MEDIUM = 1,
  LOW = 2
}

interface PrioritizedNotification {
  notification: ServerNotification;
  priority: NotificationPriority;
  timestamp: number;
}

class PriorityNotificationManager extends NotificationManager {
  private queues: Map<NotificationPriority, PrioritizedNotification[]>;
  private isProcessing: boolean = false;

  constructor(transport: Transport) {
    super(transport);
    this.queues = new Map();
    Object.values(NotificationPriority)
      .filter(p => typeof p === 'number')
      .forEach(priority => {
        this.queues.set(priority as NotificationPriority, []);
      });
  }

  private getPriorityForMethod(method: string): NotificationPriority {
    if (method.startsWith('notifications/error')) {
      return NotificationPriority.HIGH;
    }
    if (method.includes('list_changed')) {
      return NotificationPriority.MEDIUM;
    }
    return NotificationPriority.LOW;
  }

  async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const notification: ServerNotification = {
      jsonrpc: "2.0",
      method,
      params
    };

    const priority = this.getPriorityForMethod(method);
    const prioritizedNotification: PrioritizedNotification = {
      notification,
      priority,
      timestamp: Date.now()
    };

    this.queues.get(priority)!.push(prioritizedNotification);
    
    if (!this.isProcessing) {
      await this.processQueues();
    }
  }

  private async processQueues(): Promise<void> {
    this.isProcessing = true;

    try {
      while (this.hasNotifications()) {
        const notification = this.getNextNotification();
        if (notification) {
          await super.sendNotification(
            notification.notification.method,
            notification.notification.params
          );
        }
      }
    } finally {
      this.isProcessing = false;
    }
  }

  private hasNotifications(): boolean {
    return Array.from(this.queues.values())
      .some(queue => queue.length > 0);
  }

  private getNextNotification(): PrioritizedNotification | undefined {
    for (const priority of Object.values(NotificationPriority)) {
      const queue = this.queues.get(priority as NotificationPriority);
      if (queue && queue.length > 0) {
        return queue.shift();
      }
    }
    return undefined;
  }
}
```

### Advanced Security Features
```typescript
interface SecurityConfig {
  maxNotificationsPerSecond: number;
  maxPayloadSize: number;
  sensitivePatterns: RegExp[];
  allowedMethods: string[];
  blockedIPs: string[];
}

class EnhancedNotificationSecurity extends NotificationSecurity {
  private config: SecurityConfig;
  private notificationCounts: Map<string, {
    count: number;
    firstNotification: number;
  }>;

  constructor(config: SecurityConfig) {
    super();
    this.config = config;
    this.notificationCounts = new Map();
  }

  validateNotification(
    notification: ServerNotification,
    source: string
  ): boolean {
    return this.isMethodAllowed(notification.method) &&
           this.isSourceAllowed(source) &&
           this.isWithinRateLimit(source) &&
           this.isPayloadSizeValid(notification) &&
           this.noSensitiveData(notification);
  }

  private isMethodAllowed(method: string): boolean {
    return this.config.allowedMethods.includes(method);
  }

  private isSourceAllowed(source: string): boolean {
    return !this.config.blockedIPs.includes(source);
  }

  private isWithinRateLimit(source: string): boolean {
    const now = Date.now();
    const sourceStats = this.notificationCounts.get(source);

    if (!sourceStats) {
      this.notificationCounts.set(source, {
        count: 1,
        firstNotification: now
      });
      return true;
    }

    if (now - sourceStats.firstNotification >= 1000) {
      // Reset counter for new second
      sourceStats.count = 1;
      sourceStats.firstNotification = now;
      return true;
    }

    if (sourceStats.count >= this.config.maxNotificationsPerSecond) {
      return false;
    }

    sourceStats.count++;
    return true;
  }

  private isPayloadSizeValid(notification: ServerNotification): boolean {
    const size = new TextEncoder().encode(
      JSON.stringify(notification)
    ).length;
    return size <= this.config.maxPayloadSize;
  }

  private noSensitiveData(notification: ServerNotification): boolean {
    const notificationStr = JSON.stringify(notification);
    return !this.config.sensitivePatterns.some(pattern =>
      pattern.test(notificationStr)
    );
  }

  handleSecurityViolation(
    violation: {
      type: string;
      source: string;
      details: unknown;
    }
  ): void {
    // Log violation
    console.error('Security violation:', violation);

    // Update security metrics
    this.updateSecurityMetrics(violation);

    // Take action based on violation type
    switch (violation.type) {
      case 'rate_limit':
        this.handleRateLimitViolation(violation.source);
        break;
      case 'sensitive_data':
        this.handleSensitiveDataLeak(violation.details);
        break;
      case 'payload_size':
        this.handleOversizedPayload(violation.source);
        break;
      default:
        this.handleUnknownViolation(violation);
    }
  }

  private updateSecurityMetrics(violation: {
    type: string;
    source: string;
    details: unknown;
  }): void {
    // Implementation for updating security metrics
  }

  private handleRateLimitViolation(source: string): void {
    // Implementation for handling rate limit violations
  }

  private handleSensitiveDataLeak(details: unknown): void {
    // Implementation for handling sensitive data leaks
  }

  private handleOversizedPayload(source: string): void {
    // Implementation for handling oversized payloads
  }

  private handleUnknownViolation(violation: {
    type: string;
    source: string;
    details: unknown;
  }): void {
    // Implementation for handling unknown violations
  }
}
```

### Advanced Monitoring
```typescript
interface NotificationMetrics {
  totalSent: number;
  totalFailed: number;
  averageLatency: number;
  methodCounts: Map<string, number>;
  errorCounts: Map<string, number>;
}

class MonitoredNotificationManager extends NotificationManager {
  private metrics: NotificationMetrics;
  private latencyHistory: number[] = [];
  private readonly maxLatencyHistory = 1000;

  constructor(transport: Transport) {
    super(transport);
    this.metrics = {
      totalSent: 0,
      totalFailed: 0,
      averageLatency: 0,
      methodCounts: new Map(),
      errorCounts: new Map()
    };
  }

  async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const startTime = performance.now();
    
    try {
      await super.sendNotification(method, params);
      this.recordSuccess(method, performance.now() - startTime);
    } catch (error) {
      this.recordError(method, error);
      throw error;
    }
  }

  private recordSuccess(method: string, latency: number): void {
    this.metrics.totalSent++;
    this.metrics.methodCounts.set(
      method,
      (this.metrics.methodCounts.get(method) || 0) + 1
    );
    
    this.updateLatencyMetrics(latency);
  }

  private recordError(method: string, error: unknown): void {
    this.metrics.totalFailed++;
    const errorType = error instanceof Error ? error.name : 'Unknown';
    this.metrics.errorCounts.set(
      errorType,
      (this.metrics.errorCounts.get(errorType) || 0) + 1
    );
  }

  private updateLatencyMetrics(latency: number): void {
    this.latencyHistory.push(latency);
    if (this.latencyHistory.length > this.maxLatencyHistory) {
      this.latencyHistory.shift();
    }
    
    this.metrics.averageLatency = this.latencyHistory.reduce(
      (sum, val) => sum + val, 0
    ) / this.latencyHistory.length;
  }

  getMetrics(): NotificationMetrics {
    return { ...this.metrics };
  }

  getMethodDistribution(): Map<string, number> {
    const total = this.metrics.totalSent;
    const distribution = new Map<string, number>();
    
    this.metrics.methodCounts.forEach((count, method) => {
      distribution.set(method, count / total);
    });
    
    return distribution;
  }

  getErrorDistribution(): Map<string, number> {
    const total = this.metrics.totalFailed;
    const distribution = new Map<string, number>();
    
    this.metrics.errorCounts.forEach((count, error) => {
      distribution.set(error, count / total);
    });
    
    return distribution;
  }

  getLatencyPercentiles(): {
    p50: number;
    p90: number;
    p99: number;
  } {
    const sorted = [...this.latencyHistory].sort((a, b) => a - b);
    const len = sorted.length;
    
    return {
      p50: sorted[Math.floor(len * 0.5)],
      p90: sorted[Math.floor(len * 0.9)],
      p99: sorted[Math.floor(len * 0.99)]
    };
  }
}
```

## Features

### Notification Types
- Resource updates
- List changes (resources, tools, prompts)
- Progress updates
- Log messages
- Cancellation notices

### Notification Management
- Type-safe notifications
- Transport abstraction
- Handler registration
- Subscription management

### Message Validation
- Format validation
- Type checking
- Parameter validation
- Error handling

### Transport Support
- Standard I/O
- Server-Sent Events (SSE)
- WebSocket
- Custom transports

## Best Practices

1. **Notification Implementation**
   - Use appropriate notification types
   - Include relevant context
   - Handle transport errors
   - Validate before sending
   - Implement rate limiting
   - Monitor notification volume

2. **Handler Implementation**
   - Register handlers early
   - Handle errors gracefully
   - Clean up subscriptions
   - Validate notifications
   - Implement timeouts
   - Log handling errors

3. **Transport Management**
   - Handle disconnections
   - Implement reconnection
   - Buffer notifications
   - Monitor health
   - Handle backpressure
   - Log transport issues

4. **Security Management**
   - Validate notification sources
   - Monitor notification patterns
   - Implement rate limits
   - Log suspicious activity
   - Handle DoS attempts
   - Sanitize data

## Error Handling

### Common Errors
```typescript
enum NotificationError {
  INVALID_FORMAT = "invalid_format",
  TRANSPORT_ERROR = "transport_error",
  RATE_LIMIT = "rate_limit",
  VALIDATION_ERROR = "validation_error",
  HANDLER_ERROR = "handler_error"
}

interface NotificationErrorResult {
  error: NotificationError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate notifications before sending
2. Handle transport errors gracefully
3. Implement retry logic where appropriate
4. Log errors with context
5. Monitor error patterns
6. Clean up on failures

## Security Considerations

### Notification Security
```typescript
class NotificationSecurity {
  // Validate notification
  private validateNotification(
    notification: ServerNotification
  ): boolean {
    return this.isValidFormat(notification) &&
           this.isWithinRateLimit(notification) &&
           this.noSensitiveData(notification);
  }

  // Monitor patterns
  private monitorPatterns(
    method: string,
    source: string
  ): void {
    this.trackVolume(method, source);
    this.checkRateLimits(source);
    this.detectAnomalies(method, source);
  }

  // Handle security events
  private handleSecurityEvent(
    event: SecurityEvent
  ): void {
    this.logEvent(event);
    this.alertIfNeeded(event);
    this.applyMitigation(event);
  }
}
```

### Best Practices
1. **Notification Protection**
   - Validate all notifications
   - Monitor notification patterns
   - Implement rate limiting
   - Log security events
   - Handle DoS attempts
   - Alert on anomalies

2. **Data Protection**
   - Sanitize notification data
   - Check data volumes
   - Monitor patterns
   - Implement timeouts
   - Handle failures
   - Log security events

## Protocol Integration

### Capabilities
```typescript
interface NotificationCapabilities {
  resources?: {
    listChanged?: boolean;
    subscribe?: boolean;
  };
  tools?: {
    listChanged?: boolean;
  };
  prompts?: {
    listChanged?: boolean;
  };
}
```

### Protocol Messages
1. **Resource Update**
   - Method: `notifications/resources/updated`
   - Requires subscription
   - Includes resource URI

2. **List Changes**
   - Methods: `notifications/*/list_changed`
   - Requires capability
   - No parameters needed

3. **Progress Updates**
   - Method: `notifications/progress`
   - Includes progress token
   - Optional total value

4. **Log Messages**
   - Method: `notifications/message`
   - Includes severity level
   - Optional logger name

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

## Common Use Cases

### 1. Resource Update Tracking
```typescript
class ResourceTracker {
  private notificationManager: NotificationManager;
  
  constructor(transport: Transport) {
    this.notificationManager = new NotificationManager(transport);
  }

  // Track resource changes
  async handleResourceChange(uri: string): Promise<void> {
    try {
      // Notify about specific resource update
      await this.notificationManager.sendResourceUpdated(uri);
      
      // Notify about list change if needed
      await this.notificationManager.sendResourceListChanged();
      
      console.log(`Resource ${uri} update notifications sent`);
    } catch (error) {
      console.error(`Failed to send resource notifications: ${error}`);
    }
  }
}

// Usage example
const tracker = new ResourceTracker(transport);
await tracker.handleResourceChange("file:///workspace/example.txt");
```

### 2. Progress Reporting
```typescript
class LongRunningOperation {
  private notificationManager: NotificationManager;
  private progressToken: string;
  
  constructor(transport: Transport) {
    this.notificationManager = new NotificationManager(transport);
    this.progressToken = crypto.randomUUID();
  }

  // Execute operation with progress
  async execute(): Promise<void> {
    try {
      const totalSteps = 100;
      
      for (let step = 0; step <= totalSteps; step++) {
        // Perform work
        await this.doWork(step);
        
        // Report progress
        await this.notificationManager.sendProgress(
          this.progressToken,
          step,
          totalSteps
        );
        
        // Simulate work
        await new Promise(resolve => setTimeout(resolve, 100));
      }
    } catch (error) {
      console.error(`Operation failed: ${error}`);
      throw error;
    }
  }

  private async doWork(step: number): Promise<void> {
    // Actual work implementation
  }
}

// Usage example
const operation = new LongRunningOperation(transport);
await operation.execute();
```

### 3. Logging Integration
```typescript
class LoggingService {
  private notificationManager: NotificationManager;
  
  constructor(transport: Transport) {
    this.notificationManager = new NotificationManager(transport);
  }

  // Log with different levels
  async log(
    level: LoggingLevel,
    message: string,
    data?: unknown
  ): Promise<void> {
    try {
      await this.notificationManager.sendLogMessage(
        level,
        {
          message,
          timestamp: new Date().toISOString(),
          ...data
        },
        "main"
      );
    } catch (error) {
      console.error(`Failed to send log message: ${error}`);
    }
  }
}

// Usage example
const logger = new LoggingService(transport);

// Log different types of messages
await logger.log("info", "Operation started");
await logger.log("warning", "Resource not found", { uri: "example.txt" });
await logger.log("error", "Operation failed", { 
  error: "Connection timeout",
  retryCount: 3
});
```

### 4. List Change Monitoring
```typescript
class ListChangeMonitor {
  private notificationManager: NotificationManager;
  private changeHandlers: Map<string, Set<() => void>>;
  
  constructor(transport: Transport) {
    this.notificationManager = new NotificationManager(transport);
    this.changeHandlers = new Map();
  }

  // Monitor specific list type
  monitorList(type: string, handler: () => void): void {
    this.notificationManager.onListChanged(type, handler);
    
    if (!this.changeHandlers.has(type)) {
      this.changeHandlers.set(type, new Set());
    }
    this.changeHandlers.get(type)!.add(handler);
  }

  // Stop monitoring
  stopMonitoring(type: string, handler: () => void): void {
    this.notificationManager.offListChanged(type, handler);
    this.changeHandlers.get(type)?.delete(handler);
  }

  // Notify about changes
  async notifyChange(type: string): Promise<void> {
    switch (type) {
      case "resources":
        await this.notificationManager.sendResourceListChanged();
        break;
      case "tools":
        await this.notificationManager.sendToolListChanged();
        break;
      case "prompts":
        await this.notificationManager.sendPromptListChanged();
        break;
    }
  }
}

// Usage example
const monitor = new ListChangeMonitor(transport);

// Monitor resources
monitor.monitorList("resources", () => {
  console.log("Resource list changed");
  refreshResourceList();
});

// Monitor tools
monitor.monitorList("tools", () => {
  console.log("Tool list changed");
  refreshToolList();
});
```

### 5. Rate-Limited Notifications
```typescript
class RateLimitedNotifier {
  private notificationManager: NotificationManager;
  private rateLimits: Map<string, {
    lastNotification: number;
    minimumInterval: number;
  }>;
  
  constructor(transport: Transport) {
    this.notificationManager = new NotificationManager(transport);
    this.rateLimits = new Map();
  }

  // Send rate-limited notification
  async sendNotification(
    type: string,
    method: string,
    params?: unknown,
    minimumInterval: number = 1000
  ): Promise<void> {
    const now = Date.now();
    const limit = this.rateLimits.get(type);
    
    if (limit && (now - limit.lastNotification) < limit.minimumInterval) {
      console.log(`Skipping notification due to rate limit: ${type}`);
      return;
    }
    
    try {
      await this.notificationManager.sendNotification(method, params);
      this.rateLimits.set(type, {
        lastNotification: now,
        minimumInterval
      });
    } catch (error) {
      console.error(`Failed to send notification: ${error}`);
    }
  }
}

// Usage example
const notifier = new RateLimitedNotifier(transport);

// Send rate-limited notifications
await notifier.sendNotification(
  "resource-update",
  "notifications/resources/updated",
  { uri: "example.txt" },
  5000 // minimum 5 seconds between notifications
);
```

### 6. Error Recovery
```typescript
class NotificationRecovery {
  private notificationManager: NotificationManager;
  private pendingNotifications: Array<{
    method: string;
    params?: unknown;
    retries: number;
  }>;
  private maxRetries: number;
  
  constructor(transport: Transport, maxRetries: number = 3) {
    this.notificationManager = new NotificationManager(transport);
    this.pendingNotifications = [];
    this.maxRetries = maxRetries;
  }

  // Send with retry
  async sendWithRetry(
    method: string,
    params?: unknown
  ): Promise<void> {
    try {
      await this.notificationManager.sendNotification(method, params);
    } catch (error) {
      console.error(`Failed to send notification: ${error}`);
      this.pendingNotifications.push({
        method,
        params,
        retries: 0
      });
      await this.processRetries();
    }
  }

  // Process retries
  private async processRetries(): Promise<void> {
    for (const notification of [...this.pendingNotifications]) {
      if (notification.retries >= this.maxRetries) {
        console.error(
          `Failed to send notification after ${this.maxRetries} retries:`,
          notification
        );
        this.pendingNotifications = this.pendingNotifications
          .filter(n => n !== notification);
        continue;
      }
      
      try {
        await this.notificationManager.sendNotification(
          notification.method,
          notification.params
        );
        this.pendingNotifications = this.pendingNotifications
          .filter(n => n !== notification);
      } catch (error) {
        notification.retries++;
        console.error(
          `Retry ${notification.retries} failed:`,
          error
        );
        await new Promise(resolve => 
          setTimeout(resolve, 1000 * Math.pow(2, notification.retries))
        );
      }
    }
  }
}

// Usage example
const recovery = new NotificationRecovery(transport);

// Send with automatic retry
await recovery.sendWithRetry(
  "notifications/resources/updated",
  { uri: "example.txt" }
);
```

## Advanced Patterns

### Notification Middleware
```typescript
type NotificationMiddleware = (
  notification: ServerNotification,
  next: () => Promise<void>
) => Promise<void>;

class MiddlewareNotificationManager extends NotificationManager {
  private middlewares: NotificationMiddleware[] = [];

  use(middleware: NotificationMiddleware): void {
    this.middlewares.push(middleware);
  }

  async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    const notification: ServerNotification = {
      jsonrpc: "2.0",
      method,
      params
    };

    let index = 0;
    const executeMiddleware = async (): Promise<void> => {
      if (index < this.middlewares.length) {
        const middleware = this.middlewares[index++];
        await middleware(notification, executeMiddleware);
      } else {
        await super.sendNotification(method, params);
      }
    };

    await executeMiddleware();
  }
}

// Example middlewares
const loggingMiddleware: NotificationMiddleware = async (notification, next) => {
  console.log('Sending notification:', notification);
  const startTime = performance.now();
  try {
    await next();
    console.log(
      'Notification sent in',
      performance.now() - startTime,
      'ms'
    );
  } catch (error) {
    console.error('Notification failed:', error);
    throw error;
  }
};

const validationMiddleware: NotificationMiddleware = async (notification, next) => {
  if (!notification.method) {
    throw new Error('Missing method');
  }
  if (typeof notification.method !== 'string') {
    throw new Error('Invalid method type');
  }
  await next();
};

const retryMiddleware: NotificationMiddleware = async (notification, next) => {
  const maxRetries = 3;
  let retries = 0;
  
  while (true) {
    try {
      await next();
      break;
    } catch (error) {
      if (++retries >= maxRetries) {
        throw error;
      }
      await new Promise(resolve => 
        setTimeout(resolve, 1000 * Math.pow(2, retries))
      );
    }
  }
};
```

### Notification Decorators
```typescript
function logNotification() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      console.log('Notification args:', args);
      const result = await originalMethod.apply(this, args);
      console.log('Notification sent');
      return result;
    };
    
    return descriptor;
  };
}

function retryNotification(maxRetries: number = 3) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      let retries = 0;
      
      while (true) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          if (++retries >= maxRetries) {
            throw error;
          }
          await new Promise(resolve => 
            setTimeout(resolve, 1000 * Math.pow(2, retries))
          );
        }
      }
    };
    
    return descriptor;
  };
}

class DecoratedNotificationManager extends NotificationManager {
  @logNotification()
  @retryNotification(3)
  async sendNotification(
    method: string,
    params?: unknown
  ): Promise<void> {
    await super.sendNotification(method, params);
  }
}
```

### Integration Examples

#### Express Integration
```typescript
import express from 'express';
import { NotificationManager } from './notifications';

const app = express();
const notifications = new NotificationManager(transport);

// Middleware to track requests
app.use(async (req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', async () => {
    await notifications.sendNotification(
      'notifications/request',
      {
        method: req.method,
        path: req.path,
        status: res.statusCode,
        duration: Date.now() - startTime
      }
    );
  });
  
  next();
});

// Resource change endpoint
app.post('/resources/:id', async (req, res) => {
  try {
    // Update resource
    await updateResource(req.params.id, req.body);
    
    // Send notifications
    await notifications.sendResourceUpdated(req.params.id);
    await notifications.sendResourceListChanged();
    
    res.sendStatus(200);
  } catch (error) {
    await notifications.sendLogMessage(
      'error',
      'Failed to update resource',
      { id: req.params.id, error }
    );
    res.sendStatus(500);
  }
});
```

#### WebSocket Integration
```typescript
import WebSocket from 'ws';
import { NotificationManager } from './notifications';

const wss = new WebSocket.Server({ port: 8080 });
const notifications = new NotificationManager(transport);

wss.on('connection', (ws) => {
  const progressToken = crypto.randomUUID();
  let progress = 0;
  
  // Send progress updates
  const progressInterval = setInterval(async () => {
    progress += 10;
    if (progress <= 100) {
      await notifications.sendProgress(
        progressToken,
        progress,
        100
      );
    } else {
      clearInterval(progressInterval);
    }
  }, 1000);
  
  ws.on('message', async (message) => {
    try {
      const data = JSON.parse(message.toString());
      
      // Handle message
      await handleMessage(data);
      
      // Send notification
      await notifications.sendNotification(
        'notifications/message',
        { type: 'success', data }
      );
    } catch (error) {
      await notifications.sendLogMessage(
        'error',
        'Failed to handle message',
        { error }
      );
    }
  });
  
  ws.on('close', () => {
    clearInterval(progressInterval);
  });
});
```

#### React Integration
```typescript
import { useEffect, useState } from 'react';
import { NotificationManager } from './notifications';

const notifications = new NotificationManager(transport);

function ResourceList() {
  const [resources, setResources] = useState([]);
  
  useEffect(() => {
    // Subscribe to resource changes
    notifications.onListChanged('resources', async () => {
      const updatedResources = await fetchResources();
      setResources(updatedResources);
    });
    
    // Initial fetch
    fetchResources().then(setResources);
    
    return () => {
      // Cleanup subscription
      notifications.offListChanged('resources');
    };
  }, []);
  
  return (
    <ul>
      {resources.map(resource => (
        <li key={resource.id}>{resource.name}</li>
      ))}
    </ul>
  );
}

function ProgressBar() {
  const [progress, setProgress] = useState(0);
  
  useEffect(() => {
    const progressToken = crypto.randomUUID();
    let mounted = true;
    
    // Start operation
    async function runOperation() {
      try {
        const operation = new LongRunningOperation(
          transport,
          progressToken
        );
        await operation.execute();
      } catch (error) {
        console.error('Operation failed:', error);
      }
    }
    
    // Listen for progress
    notifications.onProgress(progressToken, (value, total) => {
      if (mounted) {
        setProgress(total ? (value / total) * 100 : value);
      }
    });
    
    runOperation();
    
    return () => {
      mounted = false;
      notifications.offProgress(progressToken);
    };
  }, []);
  
  return (
    <div className="progress-bar">
      <div 
        className="progress-bar-fill" 
        style={{ width: `${progress}%` }}
      />
    </div>
  );
}

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 