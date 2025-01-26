# MCP Testing Guide

**Version:** 1.4.1  
**Component:** Testing Patterns  
**Last Updated:** 2024

## Overview

This guide covers testing patterns and best practices for the MCP TypeScript SDK, including unit tests, integration tests, and end-to-end testing strategies.

## Test Categories

### 1. Unit Tests

Test individual components in isolation:

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory";

describe("Server", () => {
  let server: Server;
  let transport: InMemoryTransport;

  beforeEach(() => {
    server = new Server({
      name: "test-server",
      version: "1.0.0"
    });
    [transport] = InMemoryTransport.createLinkedPair();
  });

  afterEach(async () => {
    await transport.close();
  });

  it("should initialize with capabilities", async () => {
    server.registerCapabilities({
      resources: {},
      tools: {}
    });

    await server.connect(transport);
    expect(server.capabilities).toBeDefined();
  });
});
```

### 2. Integration Tests

Test component interactions:

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";

describe("Process cleanup", () => {
  jest.setTimeout(5000); // 5 second timeout

  it("should exit cleanly after closing transport", async () => {
    const server = new Server(
      {
        name: "test-server",
        version: "1.0.0"
      },
      {
        capabilities: {}
      }
    );

    const transport = new StdioServerTransport();
    await server.connect(transport);
    await transport.close();

    // Test passes if process doesn't hang
    expect(true).toBe(true);
  });
});
```

### 3. End-to-End Tests

Test complete workflows:

```typescript
import { Client } from "@modelcontextprotocol/sdk/client";
import { Server } from "@modelcontextprotocol/sdk/server";
import { InMemoryTransport } from "@modelcontextprotocol/sdk/inMemory";

describe("End-to-End Communication", () => {
  let client: Client;
  let server: Server;
  let clientTransport: InMemoryTransport;
  let serverTransport: InMemoryTransport;

  beforeEach(() => {
    client = new Client({
      name: "test-client",
      version: "1.0.0"
    });

    server = new Server({
      name: "test-server",
      version: "1.0.0"
    });

    [clientTransport, serverTransport] = InMemoryTransport.createLinkedPair();
  });

  afterEach(async () => {
    await client.close();
    await server.close();
  });

  it("should complete resource request workflow", async () => {
    // Setup
    server.resource("test", {
      uri: "test://resource",
      name: "Test Resource",
      content: "Test Content"
    });

    // Connect
    await client.connect(clientTransport);
    await server.connect(serverTransport);

    // Test workflow
    const resources = await client.listResources();
    expect(resources).toHaveLength(1);

    const content = await client.readResource("test://resource");
    expect(content).toBe("Test Content");
  });
});
```

## Testing Patterns

### 1. Transport Testing

```typescript
describe("Transport", () => {
  it("should handle message queuing", async () => {
    const [client, server] = InMemoryTransport.createLinkedPair();
    
    // Send before start
    await client.send({ type: "test" });
    
    // Start and verify queued message
    let received: unknown;
    server.onmessage = msg => { received = msg; };
    await server.start();
    
    expect(received).toEqual({ type: "test" });
  });
});
```

### 2. Resource Testing

```typescript
describe("Resources", () => {
  it("should handle resource templates", async () => {
    const server = new Server({
      name: "test-server",
      version: "1.0.0"
    });

    server.resource(
      "template",
      new ResourceTemplate("test://{id}", {
        list: async () => ({
          resources: [
            { name: "Test", uri: "test://1" }
          ]
        })
      }),
      async (uri, { id }) => ({
        contents: [{
          uri: uri.href,
          text: `Resource ${id}`
        }]
      })
    );

    // Test template resolution
    const resource = await server.readResource("test://123");
    expect(resource.contents[0].text).toBe("Resource 123");
  });
});
```

### 3. Tool Testing

```typescript
describe("Tools", () => {
  it("should handle tool execution", async () => {
    const server = new Server({
      name: "test-server",
      version: "1.0.0"
    });

    server.tool(
      "test",
      {
        input: z.string()
      },
      async ({ input }) => ({
        content: [{
          type: "text",
          text: `Processed: ${input}`
        }]
      })
    );

    const result = await server.callTool("test", {
      input: "test data"
    });

    expect(result.content[0].text).toBe("Processed: test data");
  });
});
```

## Best Practices

1. **Test Organization**
   - Group by feature
   - Use descriptive names
   - Follow AAA pattern
   - Keep tests focused

2. **Test Coverage**
   - Test edge cases
   - Test error paths
   - Test async behavior
   - Test cleanup

3. **Test Isolation**
   - Clean up resources
   - Reset state
   - Mock externals
   - Handle timeouts

4. **Test Maintenance**
   - Keep tests simple
   - Avoid test coupling
   - Document patterns
   - Update regularly

## Testing Tools

1. **Test Runners**
   - Jest
   - Mocha
   - Vitest
   - Node Test Runner

2. **Assertion Libraries**
   - Jest Expect
   - Chai
   - Node Assert
   - Custom Matchers

3. **Mocking Tools**
   - Jest Mock
   - Sinon
   - Test Doubles
   - Spy Functions

4. **Coverage Tools**
   - Istanbul
   - Jest Coverage
   - V8 Coverage
   - SonarQube

## Related Documentation

- [Implementation Patterns](implementation-patterns.md)
- [In-Memory Transport](../reference/in-memory-transport.md)
- [Error Handling](../reference/error-handling.md)
- [Development Guide](development.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 