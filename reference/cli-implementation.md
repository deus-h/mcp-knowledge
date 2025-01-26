# MCP CLI Implementation

**Version:** 1.4.1  
**Component:** Command Line Interface Implementation  
**Last Updated:** 2024

## Overview

The MCP CLI provides a command-line interface for interacting with MCP servers. It supports multiple connection methods and offers an interactive interface for exploring server capabilities.

## Core Components

### 1. CLI Entry Point

```typescript
// CLI configuration
const cli = meow(`
  Usage
    $ mcp-cli
    $ mcp-cli --config [config.json]
    $ mcp-cli npx <package-name> <args>
    $ mcp-cli node path/to/server/index.js args...
    $ mcp-cli --sse http://localhost:8000/sse

  Options
    --config, -c  Path to the config file
    --sse, -s  SSE endpoint
`);

// Command handling
if (cli.input.length > 0) {
  const [command, ...args] = cli.input;
  await runWithCommand(command, args);
} else if (cli.flags.sse) {
  await runWithSSE(cli.flags.sse);
} else {
  await runWithConfig(cli.flags.config);
}
```

### 2. Client Setup

```typescript
async function createClient() {
  const client = new Client(
    { name: "mcp-cli", version: "1.0.0" },
    { capabilities: {} }
  );
  
  // Handle server logs
  client.setNotificationHandler(
    LoggingMessageNotificationSchema,
    (notification) => {
      logger.debug("[server log]:", notification.params.data);
    }
  );
  
  return client;
}
```

### 3. Server Connection

```typescript
async function connectServer(transport) {
  // Initialize connection
  const spinner = createSpinner("Connecting to server...");
  const client = await createClient();
  await client.connect(transport);
  
  // List available primitives
  const primitives = await listPrimitives(client);
  
  // Interactive loop
  while (true) {
    const { primitive } = await prompts({
      name: "primitive",
      type: "autocomplete",
      message: "Pick a primitive",
      choices: primitives.map(p => ({
        title: colors.bold(`${p.type}(${p.value.name})`),
        description: p.value.description,
        value: p
      }))
    });
    
    // Handle primitive execution
    await executePrimitive(client, primitive);
  }
}
```

## Transport Implementations

### 1. Standard I/O

```typescript
export async function runWithCommand(command, args) {
  const transport = new StdioClientTransport({
    command,
    args
  });
  await connectServer(transport);
}
```

### 2. Server-Sent Events

```typescript
export async function runWithSSE(uri) {
  const transport = new SSEClientTransport(
    new URL(uri)
  );
  await connectServer(transport);
}
```

### 3. Configuration-based

```typescript
export async function runWithConfig(configPath) {
  // Load config
  const config = await readConfig(configPath);
  if (!config.mcpServers) {
    throw new Error("No MCP servers found in config");
  }
  
  // Select server
  const server = await pickServer(config);
  const serverConfig = config.mcpServers[server];
  
  // Connect
  const transport = new StdioClientTransport(serverConfig);
  await connectServer(transport);
}
```

## Utility Functions

### 1. Primitive Management

```typescript
async function listPrimitives(client) {
  const capabilities = client.getServerCapabilities();
  const primitives = [];
  
  // List resources
  if (capabilities.resources) {
    const { resources } = await client.listResources();
    resources.forEach(item => 
      primitives.push({ type: "resource", value: item })
    );
  }
  
  // List tools
  if (capabilities.tools) {
    const { tools } = await client.listTools();
    tools.forEach(item => 
      primitives.push({ type: "tool", value: item })
    );
  }
  
  // List prompts
  if (capabilities.prompts) {
    const { prompts } = await client.listPrompts();
    prompts.forEach(item => 
      primitives.push({ type: "prompt", value: item })
    );
  }
  
  return primitives;
}
```

### 2. Input Handling

```typescript
async function readJSONSchemaInputs(schema) {
  if (!schema || isEmpty(schema)) return {};
  
  // Parse schema
  const questions = [];
  traverse.default(schema, (s, _isCycle, path, parent) => {
    const key = path.replace("$.properties.", "");
    const required = parent?.required?.includes(
      key.split(".").at(-1)
    );
    
    // Build questions based on type
    if (s.type === "string") {
      questions.push({
        key,
        type: "text",
        required,
        initial: s.default
      });
    } else if (s.type === "number") {
      questions.push({
        key,
        type: "number",
        required,
        initial: s.default,
        max: s.maximum,
        min: s.minimum
      });
    }
  });
  
  // Collect answers
  const results = {};
  for (const q of questions) {
    const { value } = await prompts({
      name: "value",
      message: `${q.required ? "* " : ""}${q.key}`,
      ...q
    });
    if (value !== "") {
      setPath(results, q.key, value);
    }
  }
  
  return results;
}
```

### 3. UI Components

```typescript
// Spinner
function createSpinner(text) {
  return yoctoSpinner({
    text,
    stream: process.stderr
  }).start();
}

// Pretty printing
function prettyPrint(obj) {
  logger.dir(obj, {
    depth: null,
    colors: true
  });
}

// Config path
function getClaudeConfigPath() {
  if (process.platform === "win32") {
    return join(homedir(), "AppData", "Roaming", "Claude");
  }
  if (process.platform === "darwin") {
    return join(homedir(), "Library", "Application Support", "Claude");
  }
}
```

## Best Practices

1. **User Interface**
   - Use spinners for feedback
   - Provide clear prompts
   - Show progress
   - Handle errors gracefully

2. **Configuration**
   - Support multiple sources
   - Validate inputs
   - Handle defaults
   - Platform-specific paths

3. **Transport**
   - Handle disconnects
   - Support multiple types
   - Clean up resources
   - Validate connections

4. **Primitives**
   - Type validation
   - Input validation
   - Error handling
   - Progress tracking

## Related Documentation

- [Basic Protocol](basic-protocol.md)
- [Transport Layer](transports.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Type System](type-system-advanced.md) 

<sub>Created and maintained by Alex Johnson (alex.johnson@company.com)</sub>