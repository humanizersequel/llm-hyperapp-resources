# Hyperware Resources

This directory contains guides and documentation for developing applications on the Hyperware platform.

## Available Guides

### 1. [Hyperware Development Guide](./hyperware-development-guide.md)
Comprehensive guide covering:
- HTTP endpoint requirements
- Runtime requirements (/our.js)
- P2P communication patterns
- Common pitfalls and best practices
- Debugging tips

### 2. [Hyperware Troubleshooting Checklist](./hyperware-troubleshooting-checklist.md)
Quick reference for:
- Common build errors
- Runtime errors
- State management issues
- P2P communication problems
- Quick fixes and debugging steps

### 3. [HTTP Endpoints Guide](./hyperprocess-http-endpoints-guide.md)
Specific guidance on:
- The `_request_body` parameter requirement
- Parameter passing formats
- Error handling

### 4. [Runtime Requirements Guide](./hyperware-runtime-requirements-guide.md)
Details about:
- The `/our.js` script
- Node identity management
- WebSocket connections

### 5. [P2P Communication Guide](./hyperware-p2p-communication-guide.md)
Explains:
- ProcessId construction
- Remote RPC patterns
- Request/response handling
- Common P2P issues

## Key Concepts

### The Hyperware Platform

Hyperware is a peer-to-peer application platform where:
- Each user runs their own node
- Apps communicate directly between nodes
- No central servers required
- State is managed locally on each node

### Development Workflow

1. **Write Rust backend** using `#[hyperprocess]` macro
2. **Define interfaces** in WIT files
3. **Build frontend** with React/TypeScript
4. **Compile with** `kit b --hyperapp`
5. **Test locally** then with multiple nodes

### Critical Requirements

1. **ALL HTTP endpoints** must have `_request_body: String` parameter
2. **Frontend HTML** must include `<script src="/our.js"></script>`
3. **Multi-param endpoints** use tuple format `[param1, param2]` not objects
4. **Remote calls** need `.expects_response(timeout)`
5. **ProcessId format**: `"app-name:package-name:publisher"`

## Learning Path

For new developers:
1. Start with the [Development Guide](./hyperware-development-guide.md)
2. Keep the [Troubleshooting Checklist](./hyperware-troubleshooting-checklist.md) handy
3. Refer to specific guides as needed
4. Study the samchat example app for patterns

## Example Apps

### Samchat (in `important code/samchat/`)
A working P2P chat application demonstrating:
- Text messaging between nodes
- Group chat functionality
- File sharing
- Voice calls
- Proper error handling

Study this app to understand:
- Remote endpoint patterns
- State management
- UI/backend communication
- P2P architecture

## Common Patterns

### Remote Method Calls
```rust
// Backend
#[remote]
async fn handle_request(&mut self, param: String) -> Result<String, String> {
    // Process request from another node
}

// Calling from another node
let request_wrapper = json!({
    "HandleRequest": (param,)
});
Request::new()
    .target(remote_address)
    .body(serde_json::to_vec(&request_wrapper).unwrap())
    .expects_response(30)
    .send_and_await_response(30).unwrap()
```

### State Updates with Race Condition Prevention
```typescript
// Don't rely on React state immediately after setting
const doSomething = async (value: string) => {
  setValue(value);
  await processValue(value); // Pass explicitly, don't use state
};
```

## Debugging Commands

```bash
# Full rebuild
kit b --hyperapp

# Check built package
ls -la pkg/

# View generated WIT
cat api/*.wit

# Check UI build
cd ui && npm run build
```

## Getting Help

1. Check the troubleshooting checklist first
2. Look for similar patterns in samchat
3. Add debugging output (println!/console.log)
4. Test with minimal examples
5. Verify all requirements are met

Remember: Most issues come from missing requirements or format mismatches!