# Hyperapp Framework Improvements TODO

## Build Tool Fixes
- Fix `#[ws]` attribute recognition for WebSocket handlers
- Make WIT type generation work with complex Rust structs
- Support helper methods in hyperprocess impl blocks without requiring attributes
- Clear error messages that explain what's wrong and how to fix it

## Documentation
- Complete guide on all available attributes (`#[http]`, `#[ws]`, `#[local]`, `#[remote]`)
- WebSocket implementation examples that actually work
- Type mapping guide: Rust types → WIT types
- Quick start template with all the imports and structure correct

## Developer Experience
- Hot reload for UI development
- Working local test environment setup
- Automatic TypeScript type generation from Rust types
- Template generator: `kit new --hyperapp myapp`

## Framework Features
- WebSocket handler that actually compiles
- Built-in state synchronization helpers
- Client-side TypeScript/React SDK
- Standard patterns for common tasks (auth, real-time updates, etc.)

## Examples Needed
- Working WebSocket chat app
- Multi-player game with real-time state sync
- File upload/download example
- Cross-process communication example

## Testing Tools
- Mock hyperware runtime for unit tests
- Integration test helpers
- WebSocket testing utilities
- State snapshot testing

## Type System
- Fix kebab-case vs PascalCase issues in WIT generation
- Support for generic types
- Better error messages for type mismatches
- Automatic serialization for complex types

## Performance
- Efficient state diff broadcasting
- Message batching for WebSocket
- Built-in caching layer
- Optimized serialization

## Essential Missing Features
- Session management
- Authentication helpers
- Rate limiting middleware
- CORS configuration that works
- Request validation middleware