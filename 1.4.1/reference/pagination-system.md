# MCP Pagination System

**Version:** 1.4.1  
**Component:** Pagination System  
**Last Updated:** 2024

## Overview

The Model Context Protocol (MCP) provides a robust pagination system that enables efficient handling of large result sets. This document details the pagination system's architecture, implementation, and best practices.

## Core Components

### Pagination Types
```typescript
interface PaginatedRequest {
  cursor?: string;
  pageSize?: number;
}

interface PaginatedResult<T> {
  items: T[];
  nextCursor?: string;
  total?: number;
}

interface CursorState {
  offset: number;
  total: number;
  filters?: { [key: string]: unknown };
}
```

## Implementation

### Pagination Provider
```typescript
interface PaginationProvider {
  // Get next page of results
  getNextPage<T>(params: {
    cursor?: string;
    pageSize?: number;
  }): Promise<{
    items: T[];
    nextCursor?: string;
  }>;
}

class PaginationManager implements PaginationProvider {
  private pageSize: number = 50;
  private cursors = new Map<string, CursorState>();

  // Create cursor
  createCursor(total: number): string {
    const cursor = this.generateCursor();
    this.cursors.set(cursor, {
      offset: 0,
      total
    });
    return cursor;
  }

  // Get next page
  async getNextPage<T>(
    items: T[],
    params?: PaginatedRequest
  ): Promise<PaginatedResult<T>> {
    const { cursor, state } = this.getCursorState(params?.cursor);
    const start = state.offset;
    const end = Math.min(
      start + (params?.pageSize ?? this.pageSize),
      state.total
    );
    
    const pageItems = items.slice(start, end);
    const nextCursor = end < state.total ? cursor : undefined;
    
    if (nextCursor) {
      this.cursors.set(cursor, {
        ...state,
        offset: end
      });
    } else {
      this.cursors.delete(cursor);
    }
    
    return {
      items: pageItems,
      nextCursor,
      total: state.total
    };
  }

  // Clean up expired cursors
  private cleanupCursors(): void {
    const now = Date.now();
    for (const [cursor, state] of this.cursors) {
      if (this.isExpired(state, now)) {
        this.cursors.delete(cursor);
      }
    }
  }
}
```

### Cursor Management
```typescript
class CursorManager {
  private readonly expirationTime = 30 * 60 * 1000; // 30 minutes

  // Generate cursor
  private generateCursor(): string {
    return Buffer.from(
      JSON.stringify({
        id: crypto.randomUUID(),
        timestamp: Date.now()
      })
    ).toString('base64');
  }

  // Parse cursor
  private parseCursor(cursor: string): CursorData {
    try {
      return JSON.parse(
        Buffer.from(cursor, 'base64').toString()
      );
    } catch {
      throw new Error("Invalid cursor");
    }
  }

  // Check cursor expiration
  private isExpired(
    cursor: CursorData,
    now: number
  ): boolean {
    return now - cursor.timestamp > this.expirationTime;
  }
}
```

## Features

### Cursor-Based Pagination
- Opaque cursor tokens
- Stateful cursor tracking
- Cursor expiration
- Cursor cleanup

### Page Management
- Dynamic page sizes
- Total count tracking
- Progress tracking
- Result caching

### State Management
- Filter persistence
- Sort order tracking
- Query parameters
- Search context

### Performance
- Memory optimization
- Cache management
- Batch processing
- Resource cleanup

## Best Practices

1. **Cursor Management**
   - Use opaque tokens
   - Implement expiration
   - Clean up regularly
   - Handle invalid cursors

2. **Performance**
   - Optimize page size
   - Cache results
   - Batch operations
   - Monitor memory

3. **Resource Management**
   - Track cursor state
   - Clean expired cursors
   - Handle timeouts
   - Manage memory

4. **Error Handling**
   - Validate cursors
   - Handle expiration
   - Return clear errors
   - Log issues

## Error Handling

### Common Errors
```typescript
enum PaginationError {
  INVALID_CURSOR = "invalid_cursor",
  EXPIRED_CURSOR = "expired_cursor",
  INVALID_PAGE_SIZE = "invalid_page_size",
  RESOURCE_NOT_FOUND = "resource_not_found"
}

interface PaginationErrorResult {
  error: PaginationError;
  message: string;
  details?: unknown;
}
```

### Error Handling Strategy
1. Validate cursors and page sizes
2. Handle expired cursors gracefully
3. Clean up invalid state
4. Return clear error messages

## Security Considerations

### Cursor Security
```typescript
class CursorSecurity {
  // Validate cursor format
  private validateCursor(cursor: string): boolean {
    try {
      const decoded = this.decodeCursor(cursor);
      return this.isValidFormat(decoded) &&
             !this.isExpired(decoded) &&
             this.hasValidSignature(decoded);
    } catch {
      return false;
    }
  }

  // Sign cursor data
  private signCursor(data: CursorData): string {
    const payload = JSON.stringify(data);
    return this.hmac(payload, this.secret);
  }

  // Verify cursor signature
  private verifyCursor(
    cursor: string,
    signature: string
  ): boolean {
    const computed = this.signCursor(
      this.parseCursor(cursor)
    );
    return this.compareSignatures(signature, computed);
  }
}
```

### Best Practices
1. **Cursor Protection**
   - Sign cursors
   - Validate signatures
   - Prevent tampering
   - Expire old cursors

2. **Resource Protection**
   - Validate access
   - Rate limit requests
   - Monitor usage
   - Clean up resources

## Related Documentation
- [Server Utilities](server-utilities.md)
- [Protocol Specification](protocol-spec.md)
- [Implementation Guide](../guides/implementation-patterns.md) 

<sub>Created and maintained by Alex Chen (alex.chen@company.com)</sub>