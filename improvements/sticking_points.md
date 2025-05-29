# Hyperapp Development Sticking Points

This document outlines the major challenges encountered while developing a hyperapp (P2P poker) and proposed solutions.

## 1. WebSocket Handler Recognition

### Problem
The `kit` build tool doesn't recognize the `#[ws]` attribute for WebSocket handlers, despite:
- The TaskManager example clearly using `#[ws]` handlers
- The samchat example defining WebSocket endpoints
- Documentation suggesting WebSocket support exists

### Error
```
Error processing project: Method 'handle_websocket' in the #[hyperprocess] impl block is missing a required attribute ([remote], [local], [http], or [init])
```

### Proposed Solutions
1. **Update kit tool** - The error messages suggest kit is out of date
2. **Missing imports** - There might be a specific import or feature flag needed
3. **Macro version mismatch** - The hyperprocess_macro version might not match what kit expects
4. **Alternative pattern** - There might be a different way to implement WebSocket handlers

## 2. Type Definition Recognition in WIT Generation

### Problem
The kit build tool fails to recognize Rust struct types when generating WIT interfaces:
```
Found types used (directly or indirectly) in function signatures that are neither WIT built-ins nor defined locally: ["game-state"]
```

### Issues
- WIT expects kebab-case names (game-state) but Rust uses PascalCase (GameState)
- The type scanner might not be finding complex nested types
- Circular dependencies between types might confuse the scanner

### Proposed Solutions
1. **Simplify types** - Use only primitive types in HTTP endpoints
2. **Manual WIT files** - Create WIT files manually with proper type mappings
3. **Type annotations** - There might be attributes to help the scanner
4. **Flatten structures** - Avoid deeply nested types in API boundaries

## 3. Unclear Macro Import Requirements

### Problem
Examples show inconsistent import patterns:
- samchat imports `hyperprocess_macro::hyperprocess`
- TaskManager doesn't import it explicitly
- Both compile successfully

### Confusion
- Is the macro re-exported through another crate?
- Is it in a prelude?
- Does the order of dependencies matter?

### Proposed Solutions
1. **Check dependency re-exports** - Look for the macro in hyperware_app_common or process_macros
2. **Consistent imports** - Always use explicit imports
3. **Documentation** - Need clear documentation on required imports

## 4. Helper Methods in Hyperprocess Impl Block

### Problem
The build tool requires all methods in the `#[hyperprocess]` impl block to have handler attributes:
```
Method 'create_deck' in the #[hyperprocess] impl block is missing a required attribute
```

### Current Workaround
Move helper methods to a separate impl block

### Better Solutions
1. **Private methods** - There should be a way to mark private helper methods
2. **#[helper] attribute** - A specific attribute for non-handler methods
3. **Automatic detection** - The macro should detect which methods are handlers

## 5. Missing Build Tool Documentation

### Problem
Limited documentation on:
- What attributes are available (`#[ws]`, `#[http]`, etc.)
- How type generation works
- Build tool requirements and limitations
- Error message meanings

### Proposed Solutions
1. **Comprehensive attribute guide** - Document all available handler attributes
2. **Type system guide** - Explain how Rust types map to WIT
3. **Build troubleshooting guide** - Common errors and solutions
4. **More examples** - Examples showing edge cases and advanced patterns

## 6. WebSocket Client Implementation Unclear

### Problem
While the server-side WebSocket endpoint is defined, it's unclear:
- How to properly implement the WebSocket handler
- How to handle different message types
- How to manage connection state
- How to broadcast to multiple clients

### Proposed Solutions
1. **WebSocket example app** - A dedicated example showing WebSocket patterns
2. **Client library** - TypeScript/JavaScript client library for hyperapp WebSockets
3. **Handler templates** - Code generation for common WebSocket patterns

## 7. Development Workflow Friction

### Problem
The development cycle is slow:
- Compilation errors are cryptic
- No hot reload for UI changes
- Limited debugging information
- No clear way to test locally

### Proposed Solutions
1. **Better error messages** - Include suggested fixes in error output
2. **Development mode** - Fast compilation without optimization
3. **UI development server** - Separate UI development with mock data
4. **Local test environment** - Easy way to run hyperware node locally

## 8. Type Safety Between Rust and TypeScript

### Problem
No automatic type generation between Rust backend and TypeScript frontend:
- Manual type definitions in both places
- Easy to have mismatches
- No compile-time checking

### Proposed Solutions
1. **Type generator** - Generate TypeScript types from Rust structs
2. **Shared schema** - Define types in a schema language
3. **Runtime validation** - Automatic validation at API boundaries

## Summary

The hyperapp framework shows promise but needs:
1. **Better tooling** - More robust build tools with clear error messages
2. **Complete documentation** - Comprehensive guides for all features
3. **More examples** - Real-world examples showing best practices
4. **Type system improvements** - Better integration between Rust and WIT
5. **Developer experience** - Faster iteration cycles and better debugging

These improvements would significantly reduce the learning curve and development friction for hyperapp developers.