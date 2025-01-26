# MCP Command Line Interface

**Version:** 1.4.1  
**Component:** Command Line Interface  
**Last Updated:** 2024

## Overview

The MCP SDK provides a command-line interface for running both clients and servers. This document details the CLI features and usage patterns.

## Installation

```bash
npm install -g @modelcontextprotocol/sdk
```

## Commands

### 1. Client Mode

Run an MCP client that connects to a server:

```bash
mcp client <server_url_or_command> [args...]
```

#### Connection Options:

1. **HTTP/HTTPS (SSE)**
   ```bash
   mcp client http://localhost:3000/sse
   mcp client https://example.com/sse
   ```

2. **WebSocket**
   ```bash
   mcp client ws://localhost:3000
   mcp client wss://example.com/mcp
   ```

3. **Standard I/O**
   ```bash
   mcp client ./server.js --arg1 --arg2
   ```

### 2. Server Mode

Run an MCP server that accepts client connections:

```bash
mcp server [port]
```

#### Server Options:

1. **HTTP Server (SSE)**
   ```bash
   mcp server 3000
   # Server runs on http://localhost:3000/sse
   ```

2. **Standard I/O**
   ```bash
   mcp server
   # Server runs on stdio
   ```

## Implementation Details

### 1. Client Implementation

```typescript
async function runClient(url_or_command: string, args: string[]) {
  const client = new Client(
    {
      name: "mcp-typescript test client",
      version: "0.1.0",
    },
    {
      capabilities: {
        sampling: {},
      },
    },
  );

  // Choose transport based on URL or command
  let clientTransport;
  if (url?.protocol === "http:" || url?.protocol === "https:") {
    clientTransport = new SSEClientTransport(url);
  } else if (url?.protocol === "ws:" || url?.protocol === "wss:") {
    clientTransport = new WebSocketClientTransport(url);
  } else {
    clientTransport = new StdioClientTransport({
      command: url_or_command,
      args,
    });
  }

  await client.connect(clientTransport);
}
```

### 2. Server Implementation

```typescript
async function runServer(port: number | null) {
  if (port !== null) {
    // HTTP/SSE Server
    const app = express();
    app.get("/sse", async (req, res) => {
      const transport = new SSEServerTransport("/message", res);
      const server = new Server(
        {
          name: "mcp-typescript test server",
          version: "0.1.0",
        },
        {
          capabilities: {},
        },
      );
      await server.connect(transport);
    });
  } else {
    // Standard I/O Server
    const server = new Server(
      {
        name: "mcp-typescript test server",
        version: "0.1.0",
      },
      {
        capabilities: {
          prompts: {},
          resources: {},
          tools: {},
          logging: {},
        },
      },
    );
    const transport = new StdioServerTransport();
    await server.connect(transport);
  }
}
```

## Transport Options

### 1. Server-Sent Events (SSE)

```typescript
// Client
const transport = new SSEClientTransport(new URL("http://localhost:3000/sse"));

// Server
const transport = new SSEServerTransport("/message", res);
```

### 2. WebSocket

```typescript
// Client
const transport = new WebSocketClientTransport(new URL("ws://localhost:3000"));

// Server
// WebSocket server implementation is handled by the ws package
```

### 3. Standard I/O

```typescript
// Client
const transport = new StdioClientTransport({
  command: "./server.js",
  args: ["--debug"],
});

// Server
const transport = new StdioServerTransport();
```

## Best Practices

1. **Transport Selection**
   - Use SSE for HTTP/HTTPS
   - Use WebSocket for real-time
   - Use stdio for local processes
   - Consider security requirements

2. **Error Handling**
   - Handle connection failures
   - Implement reconnection
   - Log transport errors
   - Provide user feedback

3. **Resource Management**
   - Clean up connections
   - Handle timeouts
   - Monitor memory usage
   - Implement graceful shutdown

4. **Security**
   - Validate URLs
   - Check permissions
   - Sanitize arguments
   - Use secure protocols

## Related Documentation

- [Transport Layer](transports.md)
- [Server Architecture](server-architecture.md)
- [Client Architecture](client-architecture.md)
- [Security Guide](security.md) 

<sub>Created and maintained by John Smith (john.smith@company.com)</sub>