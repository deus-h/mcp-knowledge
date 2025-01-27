# MCP Protocol Utilities

**Version:** 1.4.1  
**Component:** Protocol Utilities  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides several utility features to support common protocol operations like cancellation, progress tracking, and connection health monitoring.

## Cancellation

### 1. Cancellation Flow

```typescript
interface CancelNotification {
  method: "notifications/cancelled";
  params: {
    requestId: RequestId;
    reason?: string;
  };
}

// Example
const notification = {
  jsonrpc: "2.0",
  method: "notifications/cancelled",
  params: {
    requestId: "123",
    reason: "User requested cancellation"
  }
};
```

### 2. Behavior Requirements

```typescript
class CancellationHandler {
  // Track active requests
  private activeRequests = new Map<RequestId, {
    startTime: number;
    cancellable: boolean;
  }>();

  // Handle cancellation
  async handleCancellation(requestId: RequestId): Promise<void> {
    const request = this.activeRequests.get(requestId);
    if (!request || !request.cancellable) {
      return; // Ignore invalid or uncancellable requests
    }

    // Stop processing
    await this.stopProcessing(requestId);
    
    // Clean up resources
    this.activeRequests.delete(requestId);
  }
}
```

## Progress Tracking

### 1. Progress Flow

```typescript
interface ProgressRequest {
  params: {
    _meta?: {
      progressToken?: ProgressToken;
    };
  };
}

interface ProgressNotification {
  method: "notifications/progress";
  params: {
    progressToken: ProgressToken;
    progress: number;
    total?: number;
  };
}

// Example
const request = {
  jsonrpc: "2.0",
  id: 1,
  method: "some_method",
  params: {
    _meta: {
      progressToken: "abc123"
    }
  }
};

const progress = {
  jsonrpc: "2.0",
  method: "notifications/progress",
  params: {
    progressToken: "abc123",
    progress: 50,
    total: 100
  }
};
```

### 2. Progress Implementation

```typescript
class ProgressTracker {
  private activeTokens = new Map<ProgressToken, {
    startTime: number;
    lastUpdate: number;
    progress: number;
    total?: number;
  }>();

  // Report progress
  async reportProgress(
    token: ProgressToken,
    progress: number,
    total?: number
  ): Promise<void> {
    const tracking = this.activeTokens.get(token);
    if (!tracking) return;

    // Update progress
    tracking.progress = progress;
    tracking.total = total;
    tracking.lastUpdate = Date.now();

    // Send notification
    await this.sendProgressNotification(token, progress, total);
  }
}
```

## Ping Mechanism

### 1. Ping Messages

```typescript
interface PingRequest {
  method: "ping";
}

interface PingResponse {
  result: Record<string, never>;
}

// Example
const ping = {
  jsonrpc: "2.0",
  id: "123",
  method: "ping"
};

const pong = {
  jsonrpc: "2.0",
  id: "123",
  result: {}
};
```

### 2. Ping Implementation

```typescript
class PingManager {
  private lastPing: number = 0;
  private pingInterval: number = 30000; // 30 seconds
  private pingTimeout: number = 5000;   // 5 seconds

  // Send ping
  async sendPing(): Promise<boolean> {
    this.lastPing = Date.now();
    
    try {
      const response = await this.sendRequest({
        method: "ping"
      }, this.pingTimeout);
      
      return true;
    } catch (error) {
      return false;
    }
  }

  // Start ping cycle
  async startPingCycle(): Promise<void> {
    while (this.isConnected) {
      await this.sleep(this.pingInterval);
      
      if (!await this.sendPing()) {
        await this.handleConnectionFailure();
      }
    }
  }
}
```

## Best Practices

### 1. Cancellation

```typescript
// Best practices for cancellation
class CancellationBestPractices {
  // Track request state
  private requestState = new Map<RequestId, {
    startTime: number;
    status: "active" | "cancelling" | "completed";
  }>();

  // Handle race conditions
  async handleResponse(id: RequestId, response: unknown): Promise<void> {
    const state = this.requestState.get(id);
    if (!state || state.status === "completed") {
      return; // Ignore late responses
    }

    if (state.status === "cancelling") {
      // Log but don't process
      this.logger.debug(`Received response for cancelled request ${id}`);
      return;
    }

    // Process response normally
    await this.processResponse(id, response);
  }
}
```

### 2. Progress

```typescript
// Best practices for progress tracking
class ProgressBestPractices {
  // Rate limiting
  private lastUpdate = new Map<ProgressToken, number>();
  private minUpdateInterval = 100; // ms

  async reportProgress(
    token: ProgressToken,
    progress: number,
    total?: number
  ): Promise<void> {
    const now = Date.now();
    const lastTime = this.lastUpdate.get(token) ?? 0;

    if (now - lastTime < this.minUpdateInterval) {
      return; // Skip update (too frequent)
    }

    this.lastUpdate.set(token, now);
    await this.sendProgressNotification(token, progress, total);
  }
}
```

### 3. Ping

```typescript
// Best practices for ping management
class PingBestPractices {
  // Adaptive intervals
  private baseInterval = 30000;
  private maxInterval = 300000;
  private currentInterval: number;

  // Adjust interval based on latency
  private adjustInterval(latency: number): void {
    if (latency > 1000) {
      this.currentInterval = Math.min(
        this.currentInterval * 2,
        this.maxInterval
      );
    } else {
      this.currentInterval = Math.max(
        this.currentInterval / 2,
        this.baseInterval
      );
    }
  }
}
```

## Related Documentation

- [Basic Protocol](basic-protocol.md)
- [Type System](type-system-advanced.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Error Handling](error-handling.md)

<sub>Created and maintained by John Smith (john.smith@company.com)</sub>