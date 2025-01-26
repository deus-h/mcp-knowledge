# MCP Client Utilities API Reference

**Version:** 1.4.1  
**Component:** Client Utilities  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides several utility APIs for client implementations, including sampling support for language model interactions. This document details these utilities and their implementation.

## Sampling API

### Core Types
```typescript
type Role = "user" | "assistant";

interface SamplingProvider {
  createMessage(params: {
    messages: SamplingMessage[];
    modelPreferences?: ModelPreferences;
    systemPrompt?: string;
    includeContext?: "none" | "thisServer" | "allServers";
    temperature?: number;
    maxTokens: number;
    stopSequences?: string[];
    metadata?: object;
  }): Promise<SamplingMessage & {
    model: string;
    stopReason?: "endTurn" | "stopSequence" | "maxTokens" | string;
  }>;
}

interface SamplingMessage {
  role: Role;
  content: TextContent | ImageContent;
}

interface ModelPreferences {
  hints?: ModelHint[];
  costPriority?: number; // 0-1
  speedPriority?: number; // 0-1
  intelligencePriority?: number; // 0-1
}

interface ModelHint {
  name?: string;
}

interface TextContent extends Annotated {
  type: "text";
  text: string;
}

interface ImageContent extends Annotated {
  type: "image";
  data: string; // base64
  mimeType: string;
}

interface Annotated {
  annotations?: {
    audience?: Role[];
    priority?: number; // 0-1
  };
}
```

### Implementation
```typescript
class SamplingManager implements SamplingProvider {
  private modelSelector: ModelSelector;
  private userApproval: UserApprovalHandler;
  private rateLimiter: RateLimiter;

  constructor(
    modelSelector: ModelSelector,
    userApproval: UserApprovalHandler,
    rateLimiter: RateLimiter
  ) {
    this.modelSelector = modelSelector;
    this.userApproval = userApproval;
    this.rateLimiter = rateLimiter;
  }

  // Create message
  async createMessage(params: {
    messages: SamplingMessage[];
    modelPreferences?: ModelPreferences;
    systemPrompt?: string;
    includeContext?: "none" | "thisServer" | "allServers";
    temperature?: number;
    maxTokens: number;
    stopSequences?: string[];
    metadata?: object;
  }): Promise<SamplingMessage & {
    model: string;
    stopReason?: string;
  }> {
    await this.rateLimiter.checkLimit();

    // Select model
    const model = await this.modelSelector.selectModel(
      params.modelPreferences
    );

    // Get user approval
    const approved = await this.userApproval.approve({
      messages: params.messages,
      model,
      systemPrompt: params.systemPrompt
    });

    if (!approved) {
      throw new Error("User rejected sampling request");
    }

    // Generate response
    const response = await this.generateResponse(
      model,
      params
    );

    // Get response approval
    const approvedResponse = await this.userApproval.approveResponse(
      response
    );

    return approvedResponse;
  }

  // Generate response
  private async generateResponse(
    model: string,
    params: {
      messages: SamplingMessage[];
      systemPrompt?: string;
      temperature?: number;
      maxTokens: number;
      stopSequences?: string[];
    }
  ): Promise<SamplingMessage & {
    model: string;
    stopReason?: string;
  }> {
    // Call language model API
    return {
      role: "assistant",
      content: {
        type: "text",
        text: "Generated response"
      },
      model,
      stopReason: "endTurn"
    };
  }
}

interface ModelSelector {
  selectModel(
    preferences?: ModelPreferences
  ): Promise<string>;
}

interface UserApprovalHandler {
  approve(params: {
    messages: SamplingMessage[];
    model: string;
    systemPrompt?: string;
  }): Promise<boolean>;

  approveResponse(
    response: SamplingMessage & {
      model: string;
      stopReason?: string;
    }
  ): Promise<SamplingMessage & {
    model: string;
    stopReason?: string;
  }>;
}
```

### Features
- Message generation
- Model selection
- User approval
- Rate limiting
- Context inclusion
- Response review

### Best Practices
1. **Model Selection**
   - Honor preferences
   - Map model hints
   - Consider costs
   - Monitor usage

2. **User Approval**
   - Clear UI/UX
   - Show context
   - Allow edits
   - Track decisions

3. **Performance**
   - Rate limit requests
   - Cache responses
   - Monitor usage
   - Handle timeouts

4. **Security**
   - Validate inputs
   - Review content
   - Control access
   - Log usage

// ... more utilities to be added ...

## Root Management API

### Core Types
```typescript
interface RootProvider {
  listRoots(): Promise<Root[]>;
  onListChanged(handler: () => void): void;
}

interface Root {
  uri: string; // Must start with file://
  name?: string;
}
```

### Implementation
```typescript
class RootManager implements RootProvider {
  private roots = new Map<string, Root>();
  private listChangedHandlers = new Set<() => void>();
  private validator: RootValidator;

  constructor(validator: RootValidator) {
    this.validator = validator;
  }

  // List roots
  async listRoots(): Promise<Root[]> {
    const roots = Array.from(this.roots.values());
    return roots.filter(root => 
      this.validator.isAccessible(root.uri)
    );
  }

  // Add root
  async addRoot(root: Root): Promise<void> {
    await this.validator.validateRoot(root);
    this.roots.set(root.uri, root);
    this.notifyListChanged();
  }

  // Remove root
  removeRoot(uri: string): void {
    if (this.roots.delete(uri)) {
      this.notifyListChanged();
    }
  }

  // Subscribe to list changes
  onListChanged(handler: () => void): void {
    this.listChangedHandlers.add(handler);
  }

  // Notify list changes
  private notifyListChanged(): void {
    for (const handler of this.listChangedHandlers) {
      handler();
    }
  }
}

interface RootValidator {
  validateRoot(root: Root): Promise<void>;
  isAccessible(uri: string): boolean;
}

class FileSystemRootValidator implements RootValidator {
  // Validate root
  async validateRoot(root: Root): Promise<void> {
    if (!root.uri.startsWith("file://")) {
      throw new Error("Root URI must start with file://");
    }

    const path = this.uriToPath(root.uri);
    if (!await this.isDirectory(path)) {
      throw new Error("Root must be a directory");
    }

    if (!await this.hasPermissions(path)) {
      throw new Error("Insufficient permissions");
    }
  }

  // Check accessibility
  isAccessible(uri: string): boolean {
    const path = this.uriToPath(uri);
    return this.exists(path) && this.hasPermissions(path);
  }

  // Convert URI to path
  private uriToPath(uri: string): string {
    return uri.replace("file://", "");
  }

  // Check if path is directory
  private async isDirectory(path: string): Promise<boolean> {
    // Check if path is directory
    return true;
  }

  // Check permissions
  private async hasPermissions(path: string): Promise<boolean> {
    // Check read/write permissions
    return true;
  }

  // Check if path exists
  private exists(path: string): boolean {
    // Check if path exists
    return true;
  }
}
```

### Features
- Root management
- URI validation
- Permission checks
- Change notifications
- Root filtering
- Path validation

### Best Practices
1. **Root Management**
   - Validate URIs
   - Check permissions
   - Monitor changes
   - Clean up state

2. **Security**
   - Validate paths
   - Check access
   - Monitor usage
   - Log changes

3. **Performance**
   - Cache results
   - Batch updates
   - Monitor usage
   - Clean up regularly

4. **User Experience**
   - Clear UI/UX
   - Show status
   - Allow edits
   - Track changes

## Related Documentation
- [Protocol Specification](../reference/protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Client API Reference](client.md)

## Model Selection API

### Model Selection Interface
```typescript
interface ModelSelector {
  // Select model based on preferences
  selectModel(preferences: ModelPreferences): string;
  
  // Get available models
  getAvailableModels(): Model[];
  
  // Check model compatibility
  isModelCompatible(
    model: string,
    requirements: ModelRequirements
  ): boolean;
}

interface Model {
  name: string;
  provider: string;
  capabilities: {
    cost: number;      // 0-1
    speed: number;     // 0-1
    intelligence: number; // 0-1
  };
}
```

### Model Selection Implementation
```typescript
class ModelSelectionManager implements ModelSelector {
  private models = new Map<string, Model>();

  // Find matching model from hints
  private findMatchingModel(hints: ModelHint[]): string | undefined {
    for (const hint of hints) {
      const model = Array.from(this.models.values()).find(m =>
        m.name.includes(hint.name ?? "")
      );
      if (model) return model.name;
    }
  }

  // Select model by priorities
  private selectModelByPriorities(
    priorities: Required<ModelPreferences>
  ): string {
    return Array.from(this.models.values())
      .sort((a, b) => this.calculateScore(b, priorities) - 
                      this.calculateScore(a, priorities))
      [0].name;
  }

  private calculateScore(
    model: Model,
    priorities: Required<ModelPreferences>
  ): number {
    return (
      model.capabilities.cost * priorities.costPriority +
      model.capabilities.speed * priorities.speedPriority +
      model.capabilities.intelligence * priorities.intelligencePriority
    );
  }
}
```

## Best Practices

1. **Sampling Implementation**
   - Require user approval
   - Validate messages
   - Handle model selection
   - Implement rate limits

2. **Root Management**
   - Validate URIs
   - Handle notifications
   - Track changes
   - Clean up resources

3. **Model Selection**
   - Honor model hints
   - Balance priorities
   - Handle fallbacks
   - Consider costs

4. **Security Considerations**
   - Get user consent
   - Validate content
   - Rate limit requests
   - Handle sensitive data

## Related Documentation
- [Client API Reference](client.md)
- [Implementation Guide](../guides/implementation-patterns.md)
- [Protocol Specification](../reference/protocol-spec.md)

---

<div align="center">

Created and maintained by [Amadeus Samiel H.](mailto:amadeus.hritani@simhop.se) | Started: January 25, 2025 | Last Updated: January 27, 2025

</div> 