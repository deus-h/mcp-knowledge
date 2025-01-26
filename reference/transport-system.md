# MCP Transport System

**Version:** 1.4.1  
**Component:** Transport System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a flexible transport system that enables communication between clients and servers through various channels. This document details the transport system's architecture, implementation, and best practices.

## Core Components

### Transport Interface
```typescript
interface Transport {
  // Start transport
  start(): Promise<void>;
  
  // Close transport
  close(): Promise<void>;
  
  // Send message
  send(message: JSONRPCMessage): Promise<void>;
  
  // Set message handler
  onMessage?: (message: JSONRPCMessage) => void;
  
  // Set close handler
  onClose?: () => void;
}
```

## Standard Transports

### Standard I/O Transport
```typescript
class StdioTransport implements Transport {
  private process: ChildProcess;
  private buffer: string = "";

  constructor(command: string, args: string[]) {
    this.process = spawn(command, args);
  }

  // Start transport
  async start(): Promise<void> {
    this.process.stdout.on("data", this.handleData);
    this.process.on("close", this.handleClose);
    this.process.on("error", this.handleError);
  }

  // Close transport
  async close(): Promise<void> {
    this.process.stdin.end();
    await new Promise<void>(resolve => {
      this.process.on("exit", () => resolve());
      setTimeout(() => {
        this.process.kill("SIGTERM");
      }, 5000);
    });
  }

  // Send message
  async send(message: JSONRPCMessage): Promise<void> {
    const data = JSON.stringify(message) + "\n";
    return new Promise((resolve, reject) => {
      this.process.stdin.write(data, err => {
        if (err) reject(err);
        else resolve();
      });
    });
  }

  // Handle incoming data
  private handleData = (data: Buffer): void => {
    this.buffer += data.toString();
    let newlineIndex: number;
    while ((newlineIndex = this.buffer.indexOf("\n")) !== -1) {
      const line = this.buffer.slice(0, newlineIndex);
      this.buffer = this.buffer.slice(newlineIndex + 1);
      try {
        const message = JSON.parse(line);
        this.onMessage?.(message);
      } catch (error) {
        this.handleError(error);
      }
    }
  };
}
```

### SSE Transport
```typescript
class SSETransport implements Transport {
  private eventSource?: EventSource;
  private endpoint?: string;

  constructor(private sseUrl: string) {}

  // Start transport
  async start(): Promise<void> {
    this.eventSource = new EventSource(this.sseUrl);
    
    return new Promise((resolve, reject) => {
      this.eventSource!.addEventListener("endpoint", event => {
        this.endpoint = (event as MessageEvent).data;
        resolve();
      });
      
      this.eventSource!.addEventListener("error", () => {
        reject(new Error("SSE connection failed"));
      });
      
      this.eventSource!.addEventListener("message", event => {
        try {
          const message = JSON.parse(
            (event as MessageEvent).data
          );
          this.onMessage?.(message);
        } catch (error) {
          this.handleError(error);
        }
      });
    });
  }

  // Close transport
  async close(): Promise<void> {
    this.eventSource?.close();
  }

  // Send message
  async send(message: JSONRPCMessage): Promise<void> {
    if (!this.endpoint) {
      throw new Error("Transport not initialized");
    }
    
    const response = await fetch(this.endpoint, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(message)
    });
    
    if (!response.ok) {
      throw new Error("Failed to send message");
    }
  }
}
```

## Custom Transport Implementation

### Transport Base
```typescript
abstract class BaseTransport implements Transport {
  protected onMessage?: (message: JSONRPCMessage) => void;
  protected onClose?: () => void;

  // Abstract methods
  abstract start(): Promise<void>;
  abstract close(): Promise<void>;
  abstract send(message: JSONRPCMessage): Promise<void>;

  // Common validation
  protected validateMessage(
    message: JSONRPCMessage
  ): void {
    if (!this.isValidMessage(message)) {
      throw new Error("Invalid message format");
    }
  }

  // Common error handling
  protected handleError(error: Error): void {
    console.error("Transport error:", error);
    this.close().catch(console.error);
  }
}
```

### Custom Transport Example
```typescript
class WebSocketTransport extends BaseTransport {
  private ws?: WebSocket;

  constructor(private url: string) {
    super();
  }

  // Start transport
  async start(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket(this.url);
      
      this.ws.onopen = () => resolve();
      this.ws.onerror = () => reject();
      
      this.ws.onmessage = event => {
        try {
          const message = JSON.parse(event.data);
          this.onMessage?.(message);
        } catch (error) {
          this.handleError(error);
        }
      };
      
      this.ws.onclose = () => this.onClose?.();
    });
  }

  // Implementation-specific methods
  async send(message: JSONRPCMessage): Promise<void> {
    if (!this.ws) {
      throw new Error("Transport not initialized");
    }
    
    this.validateMessage(message);
    this.ws.send(JSON.stringify(message));
  }
}
```

## Features

### Transport Management
- Connection handling
- Message formatting
- Error handling
- State tracking

### Message Handling
- Message parsing
- Message validation
- Message queuing
- Error recovery

### Connection Management
- Connection setup
- Connection teardown
- Connection monitoring
- Reconnection handling

### Error Handling
- Transport errors
- Message errors
- Connection errors
- Recovery strategies

## Best Practices

1. **Transport Implementation**
   - Handle connection states
   - Validate messages
   - Implement timeouts
   - Clean up resources

2. **Message Handling**
   - Parse carefully
   - Handle partial data
   - Queue messages
   - Handle backpressure

3. **Error Handling**
   - Handle disconnects
   - Implement retries
   - Log errors
   - Clean up state

4. **Security**
   - Validate input
   - Handle timeouts
   - Protect credentials
   - Monitor usage

## Error Handling

### Common Errors
```typescript
enum TransportError {
  CONNECTION_FAILED = "connection_failed",
  SEND_FAILED = "send_failed",
  INVALID_MESSAGE = "invalid_message",
  TIMEOUT = "timeout"
}

interface TransportErrorResult {
  error: TransportError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Handle connection failures
2. Implement timeouts
3. Validate messages
4. Clean up resources

## Security Considerations

### Transport Security
```typescript
class TransportSecurity {
  // Validate connection
  private validateConnection(
    url: string
  ): boolean {
    return this.isSecureProtocol(url) &&
           this.isAllowedHost(url);
  }

  // Check message security
  private validateMessageSecurity(
    message: JSONRPCMessage
  ): boolean {
    return this.isWithinSizeLimit(message) &&
           this.isRateLimitOk(message);
  }

  // Monitor transport
  private monitorTransport(
    transport: Transport
  ): void {
    this.trackBandwidth(transport);
    this.checkLatency(transport);
    this.monitorErrors(transport);
  }
}
```

### Best Practices
1. **Connection Security**
   - Use secure protocols
   - Validate endpoints
   - Monitor connections
   - Handle timeouts

2. **Message Security**
   - Validate sizes
   - Rate limit
   - Check content
   - Monitor usage

## Related Documentation
- [Server API Reference](server.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)

<sub>Created and maintained by Transport System Team (transport-team@company.com)</sub>