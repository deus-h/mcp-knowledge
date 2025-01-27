# MCP Cross-Language Implementation Guide

**Version:** 1.4.1  
**Component:** Cross-Language Patterns  
**Last Updated:** 2024

## Overview

The Model Context Protocol is language-agnostic, with implementations available in multiple languages including TypeScript and Python. This guide covers patterns and practices for implementing MCP servers across different languages.

## Core Concepts

### 1. Common Protocol Types

All implementations share common JSON-RPC message types:

```typescript
// TypeScript
interface JSONRPCMessage {
  jsonrpc: "2.0";
  id?: string | number;
  method?: string;
  params?: unknown;
  result?: unknown;
  error?: {
    code: number;
    message: string;
    data?: unknown;
  };
}

# Python
class JSONRPCMessage(TypedDict):
    jsonrpc: Literal["2.0"]
    id: Optional[Union[str, int]]
    method: Optional[str]
    params: Optional[Any]
    result: Optional[Any]
    error: Optional[JSONRPCError]
```

### 2. Server Implementation

Basic server setup across languages:

```typescript
// TypeScript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";

const server = new McpServer({
  name: "example-server",
  version: "1.0.0"
});

// Python
from mcp.server.lowlevel import Server

app = Server("example-server")
```

## Transport Patterns

### 1. Standard I/O Transport

```typescript
// TypeScript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const transport = new StdioServerTransport();
await server.connect(transport);

# Python
from mcp.server.stdio import stdio_server

async def run():
    async with stdio_server() as streams:
        await app.run(
            streams[0], 
            streams[1], 
            app.create_initialization_options()
        )
```

### 2. Server-Sent Events (SSE)

```typescript
// TypeScript
import { SSEServerTransport } from "@modelcontextprotocol/sdk/server/sse.js";
import express from "express";

const app = express();
app.get("/mcp", async (req, res) => {
  const transport = new SSEServerTransport("/messages", res);
  await server.connect(transport);
});

# Python
from mcp.server.sse import SseServerTransport
from starlette.applications import Starlette
from starlette.routing import Mount, Route

sse = SseServerTransport("/messages/")

async def handle_sse(request):
    async with sse.connect_sse(
        request.scope, 
        request.receive, 
        request._send
    ) as streams:
        await app.run(
            streams[0],
            streams[1],
            app.create_initialization_options()
        )
```

## Tool Implementation

### 1. Basic Tool Definition

```typescript
// TypeScript
server.tool(
  "example",
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

# Python
@app.call_tool()
async def example_tool(
    name: str,
    arguments: dict
) -> list[types.TextContent]:
    if name != "example":
        raise ValueError(f"Unknown tool: {name}")
    return [
        types.TextContent(
            type="text",
            text=f"Processed: {arguments['input']}"
        )
    ]
```

### 2. Tool Registration

```typescript
// TypeScript
server.registerCapabilities({
  tools: {
    example: {
      inputSchema: {
        type: "object",
        properties: {
          input: { type: "string" }
        }
      }
    }
  }
});

# Python
@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="example",
            description="Example tool",
            inputSchema={
                "type": "object",
                "properties": {
                    "input": {
                        "type": "string"
                    }
                }
            }
        )
    ]
```

## Resource Implementation

### 1. Resource Definition

```typescript
// TypeScript
server.resource(
  "example",
  new ResourceTemplate("example://{id}", {
    list: async () => ({
      resources: [
        { name: "Example", uri: "example://1" }
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

# Python
@app.read_resource()
async def read_resource(
    uri: str
) -> types.Resource:
    return types.Resource(
        contents=[
            types.TextContent(
                type="text",
                text=f"Resource {uri}"
            )
        ]
    )
```

## Error Handling

### 1. Standard Errors

```typescript
// TypeScript
class McpError extends Error {
  constructor(
    public code: number,
    message: string,
    public data?: unknown
  ) {
    super(message);
  }
}

throw new McpError(-32602, "Invalid params");

# Python
class McpError(Exception):
    def __init__(
        self,
        code: int,
        message: str,
        data: Optional[Any] = None
    ):
        self.code = code
        self.message = message
        self.data = data
        super().__init__(message)

raise McpError(-32602, "Invalid params")
```

### 2. Error Translation

```typescript
// TypeScript
function translateError(error: unknown): JSONRPCError {
  if (error instanceof McpError) {
    return {
      code: error.code,
      message: error.message,
      data: error.data
    };
  }
  return {
    code: -32603,
    message: "Internal error"
  };
}

# Python
def translate_error(error: Exception) -> dict:
    if isinstance(error, McpError):
        return {
            "code": error.code,
            "message": error.message,
            "data": error.data
        }
    return {
        "code": -32603,
        "message": "Internal error"
    }
```

## Type Validation

### 1. Schema Validation

```typescript
// TypeScript
import { z } from "zod";

const schema = z.object({
  input: z.string()
});

// Python
from pydantic import BaseModel

class InputModel(BaseModel):
    input: str
```

### 2. Runtime Type Checking

```typescript
// TypeScript
function validateType<T>(
  value: unknown,
  schema: z.ZodType<T>
): T {
  return schema.parse(value);
}

# Python
def validate_type(
    value: Any,
    model: Type[BaseModel]
) -> BaseModel:
    return model.parse_obj(value)
```

## Best Practices

1. **Protocol Compatibility**
   - Use standard JSON-RPC messages
   - Follow common error codes
   - Implement standard methods
   - Validate message formats

2. **Type Safety**
   - Use strong typing
   - Validate inputs/outputs
   - Define clear interfaces
   - Handle type coercion

3. **Error Handling**
   - Use standard error codes
   - Provide error context
   - Handle language specifics
   - Maintain consistency

4. **Transport Handling**
   - Abstract transport details
   - Handle connection lifecycle
   - Manage message flow
   - Support standard transports

## Related Documentation

- [Protocol Specification](../reference/protocol-spec.md)
- [Type System](types.md)
- [Error Handling](error-handling.md)
- [Transport Layer](transports.md) 

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div>