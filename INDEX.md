# 🗺️ MCP Knowledge Base Index

> *"A map is only as good as its legend. This index is our legend through the MCP landscape."* - Amadeus Samiel H.

**Created and Maintained by:**
- **Author:** Amadeus Samiel H.
- **Contact:** amadeus.hritani@simhop.se
- **Started:** January 25, 2025
- **Last Updated:** January 27, 2025

## 🔗 Quick Navigation

- [README](../README.md) - Start here for an overview
- [Getting Started Guide](guides/getting-started.md) - New to MCP?
- [Protocol Specification](reference/protocol-spec.md) - Technical details
- [Implementation Examples](examples/implementations.md) - Code patterns

## 📂 Directory Structure

> *"Structure brings clarity, clarity brings understanding."* - Amadeus Samiel H.

```
mcp-knowledge/
├── README.md                 # Main documentation overview
├── 1.4.1/                   # Version 1.4.1 documentation
    ├── core/                # Core protocol documentation
    │   ├── architecture.md  # System architecture
    │   ├── client.md       # Client core concepts
    │   ├── server.md       # Server core concepts
    │   ├── transport.md    # Transport layer concepts
    │   └── types.md        # Core type system
    ├── api/                 # API documentation
    │   ├── client.md       # Client API reference
    │   ├── server.md       # Server API reference
    │   ├── server-utilities.md  # Server utilities API
    │   └── client-utilities.md  # Client utilities API
    ├── reference/          # Technical reference
    │   ├── architecture.md # Detailed architecture
    │   ├── basic-protocol.md # Protocol basics
    │   ├── cli.md         # CLI reference
    │   ├── protocol-spec.md # Protocol specification
    │   ├── protocol-types.md # Protocol types
    │   ├── protocol-utilities.md # Protocol utilities
    │   ├── in-memory-transport.md # In-memory transport
    │   └── ... (additional reference files)
    ├── guides/             # Implementation guides
    │   ├── getting-started.md # Getting started guide
    │   ├── advanced.md    # Advanced features
    │   ├── cross-language.md # Cross-language patterns
    │   ├── implementation-patterns.md # Common patterns
    │   ├── testing.md    # Testing guide
    │   ├── transports.md # Transport guide
    │   └── uri-templating.md # URI templating guide
    ├── examples/          # Example implementations
    │   └── implementations.md # Example code & patterns
    └── internals/        # Internal documentation
        └── protocol.md  # Protocol internals
```

## 📚 Documentation Categories

> *"Each category is a chapter in our technical saga."* - Amadeus Samiel H.

### Core Documentation
- **Purpose**: Fundamental concepts and architecture
- **Key Files**: 
  - [`core/architecture.md`](core/architecture.md): System design & components
  - [`core/types.md`](core/types.md): Type system foundations
  - [`core/client.md`](core/client.md) & [`core/server.md`](core/server.md): Core concepts
  - [`core/transport.md`](core/transport.md): Transport layer concepts

### API Documentation
- **Purpose**: Interface specifications and usage
- **Key Files**:
  - [`api/server.md`](api/server.md): Server API reference
  - [`api/client.md`](api/client.md): Client API reference
  - [`api/server-utilities.md`](api/server-utilities.md): Server-side utilities
  - [`api/client-utilities.md`](api/client-utilities.md): Client-side utilities

### Technical Reference
- **Purpose**: Detailed technical specifications
- **Key Files**:
  - [`reference/protocol-spec.md`](reference/protocol-spec.md): Protocol details
  - [`reference/protocol-types.md`](reference/protocol-types.md): Protocol types
  - [`reference/basic-protocol.md`](reference/basic-protocol.md): Protocol basics
  - [`reference/cli.md`](reference/cli.md): CLI documentation
  - [`reference/protocol-utilities.md`](reference/protocol-utilities.md): Protocol utilities
  - [`reference/in-memory-transport.md`](reference/in-memory-transport.md): In-memory transport

### Implementation Guides
- **Purpose**: Practical implementation guidance
- **Key Files**:
  - [`guides/getting-started.md`](guides/getting-started.md): Onboarding guide
  - [`guides/implementation-patterns.md`](guides/implementation-patterns.md): Common patterns
  - [`guides/testing.md`](guides/testing.md): Testing strategies
  - [`guides/advanced.md`](guides/advanced.md): Advanced features
  - [`guides/cross-language.md`](guides/cross-language.md): Cross-language patterns
  - [`guides/transports.md`](guides/transports.md): Transport implementations
  - [`guides/uri-templating.md`](guides/uri-templating.md): URI templating guide

### Examples
- **Purpose**: Real-world implementation examples
- **Key Files**:
  - [`examples/implementations.md`](examples/implementations.md): Example code & patterns

### Internal Documentation
- **Purpose**: Internal architecture and details
- **Key Files**:
  - [`internals/protocol.md`](internals/protocol.md): Protocol internals

## 🔍 Quick Reference

> *"Knowledge at your fingertips, power at your command."* - Amadeus Samiel H.

| Category | Primary Use Case | Key Entry Point |
|----------|-----------------|-----------------|
| Core | Understanding fundamentals | [`core/architecture.md`](core/architecture.md) |
| API | Interface implementation | [`api/server.md`](api/server.md) |
| Reference | Technical details | [`reference/protocol-spec.md`](reference/protocol-spec.md) |
| Guides | Implementation help | [`guides/getting-started.md`](guides/getting-started.md) |
| Examples | Code patterns | [`examples/implementations.md`](examples/implementations.md) |
| Internals | Deep dive | [`internals/protocol.md`](internals/protocol.md) |

## 📖 Documentation Conventions

> *"Consistency is the foundation of clarity."* - Amadeus Samiel H.

1. **File Organization**
   - Each category has its own directory
   - Related files are grouped together
   - Consistent naming conventions

2. **File Structure**
   - Title and version info
   - Overview section
   - Detailed content
   - Examples where applicable
   - Related documentation links

3. **Content Standards**
   - Clear and concise language
   - Code examples in TypeScript
   - Practical implementation tips
   - Security considerations
   - Best practices

## 🔄 Maintenance

This knowledge base is actively maintained by Amadeus Samiel H. (amadeus.hritani@simhop.se) through:
- Regular updates from source materials
- Version-specific documentation
- Cross-referencing between documents
- Consistent formatting and style

## 🔗 Related Resources

> *"We stand on the shoulders of giants, but we forge our own path forward."* - Amadeus Samiel H.

- [MCP TypeScript SDK](https://github.com/cursor-io/mcp-typescript-sdk)
- [MCP Specification](https://github.com/cursor-io/mcp-specification)
- [MCP Examples](https://github.com/cursor-io/mcp-examples)

## 🎯 Common Tasks

> *"Every journey begins with a single step. Let these paths guide yours."* - Amadeus Samiel H.

1. **New to MCP?**
   - Start with [`README.md`](../README.md)
   - Then read [`guides/getting-started.md`](guides/getting-started.md)
   - Follow with [`core/architecture.md`](core/architecture.md)

2. **Implementing a Server?**
   - Begin with [`core/server.md`](core/server.md)
   - Check [`api/server.md`](api/server.md)
   - Review [`examples/implementations.md`](examples/implementations.md)

3. **Need API Details?**
   - Start with [`reference/protocol-spec.md`](reference/protocol-spec.md)
   - Check relevant API docs in [`api/`](api/)
   - Review type system in [`reference/protocol-types.md`](reference/protocol-types.md)

## 📂 File Quick Links

### Core Files
- [README](../README.md) - Main documentation overview
- [Architecture](core/architecture.md) - System architecture
- [Client Core](core/client.md) - Client core concepts
- [Server Core](core/server.md) - Server core concepts
- [Transport Layer](core/transport.md) - Transport layer concepts
- [Type System](core/types.md) - Core type system

### API Documentation
- [Server API](api/server.md) - Server API reference
- [Client API](api/client.md) - Client API reference
- [Server Utilities](api/server-utilities.md) - Server utilities API
- [Client Utilities](api/client-utilities.md) - Client utilities API

### Reference Documentation
- [Protocol Specification](reference/protocol-spec.md) - Protocol specification
- [Protocol Types](reference/protocol-types.md) - Protocol types
- [Basic Protocol](reference/basic-protocol.md) - Protocol basics
- [CLI Documentation](reference/cli.md) - CLI reference
- [Protocol Utilities](reference/protocol-utilities.md) - Protocol utilities
- [In-Memory Transport](reference/in-memory-transport.md) - In-memory transport

### Implementation Guides
- [Getting Started](guides/getting-started.md) - Getting started guide
- [Implementation Patterns](guides/implementation-patterns.md) - Common patterns
- [Testing Guide](guides/testing.md) - Testing strategies
- [Advanced Features](guides/advanced.md) - Advanced features
- [Cross-Language Patterns](guides/cross-language.md) - Cross-language patterns
- [Transport Guide](guides/transports.md) - Transport implementations
- [URI Templating](guides/uri-templating.md) - URI templating guide

### Examples & Internals
- [Implementation Examples](examples/implementations.md) - Example code & patterns
- [Protocol Internals](internals/protocol.md) - Protocol internals

---
<div align="center">
<sub>
Created and maintained by <a href="mailto:amadeus.hritani@simhop.se">Amadeus Samiel H.</a><br>
Started: January 25, 2025 | Last Updated: January 27, 2025
</sub>
</div> 