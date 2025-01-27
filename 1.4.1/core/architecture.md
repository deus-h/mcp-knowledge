# MCP Architecture Overview

**Version:** 1.4.1  
**Component:** Core Architecture  
**Last Updated:** 2024

## Core Components

The MCP TypeScript SDK is organized into several key components:

### 1. Client
The client implementation for connecting to MCP servers, handling:
- Connection management
- Protocol message handling
- Resource and tool interactions

### 2. Server
Server-side implementation including:
- Resource management
- Tool execution
- Protocol handlers
- Transport layers (stdio, HTTP/SSE)

### 3. Shared Components
Common utilities and types used by both client and server:
- Protocol messages
- Data structures
- Utility functions

### 4. Types
Comprehensive TypeScript type definitions for:
- Protocol messages
- Server capabilities
- Resource definitions
- Tool specifications

### 5. CLI
Command-line interface for:
- Server management
- Testing
- Development utilities

## Directory Structure

```
src/
├── client/       # Client implementation
├── server/       # Server implementation
├── shared/       # Shared utilities
├── types.ts      # Type definitions
└── cli.ts        # CLI implementation
```

## Key Concepts

1. **Protocol Flow**
   - Client initiates connection to server
   - Server advertises capabilities
   - Resources and tools are discovered
   - Interactions occur through defined protocol messages

2. **Resource Management**
   - Resources represent data or context
   - Can be static or dynamic
   - Support versioning and updates

3. **Tool Integration**
   - Tools provide actionable capabilities
   - Defined interface for execution
   - Support for async operations

4. **Transport Layer**
   - Supports multiple transport mechanisms
   - stdio for direct integration
   - HTTP/SSE for network communication

## Implementation Notes

- Written in TypeScript for type safety
- Modular architecture for extensibility
- Test coverage for core functionality
- Support for both Node.js and browser environments

## Next Steps

- [Client Core Concepts](./client.md)
- [Server Core Concepts](./server.md)
- [Transport Layer](./transport.md)
- [Type System](./types.md)

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 