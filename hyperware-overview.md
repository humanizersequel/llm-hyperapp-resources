# Hyperware Platform Overview

## What is Hyperware?

Hyperware is a peer-to-peer (P2P) application platform that enables developers to build decentralized applications where each user runs their own node. Unlike traditional client-server architectures, Hyperware applications communicate directly between nodes without central servers, creating a truly distributed computing environment.

### Core Philosophy
- **No central servers** - Each user runs their own node
- **Direct P2P communication** - Nodes communicate directly with each other
- **Local state management** - Each node maintains its own state
- **Process-based architecture** - Applications are built as processes that handle messages

## Key Concepts

### 1. Nodes
A node is an instance of the Hyperware runtime that:
- Has a unique identity (e.g., `alice.os`, `bob.os`)
- Runs multiple processes (applications)
- Can communicate with other nodes over the network
- Manages local state and resources

### 2. Processes
Processes are the fundamental building blocks of Hyperware applications:
- Each app is a process with a unique identifier: `process-name:package-name:publisher`
- Processes handle messages asynchronously
- State is maintained within each process
- Processes can communicate locally (same node) or remotely (different nodes)

### 3. P2P Communication
Hyperware uses a message-passing architecture:
- **Request/Response pattern** - Send a request and await a response
- **Fire-and-forget pattern** - Send a message without expecting a response
- **Address format** - Messages are sent to addresses combining node and process IDs
- **Correlation system** - Tracks request/response pairs using unique IDs

### 4. The Hyperapp Framework
Hyperapp is a macro-driven framework that simplifies Hyperware development by:
- Providing async/await support through a custom runtime
- Automatically generating WebAssembly Interface Types (WIT) files
- Creating type-safe RPC-style function calls between processes
- Handling state persistence automatically

## Development Workflow

### 1. Setup
```bash
# Create a new app by copying the samchat example (DO NOT use TaskManager - it's broken)
cp -r /path/to/samchat myapp
cd myapp

# Update metadata.json with your app details
{
  "name": "MyApp",
  "publisher": "yourname.os"
}
```

### 2. Write Your Backend (Rust)
```rust
use hyperprocess_macro::*;

#[hyperprocess(
    name = "My App",
    ui = Some(HttpBindingConfig::default()),
    endpoints = vec![
        Binding::Http { 
            path: "/api", 
            config: HttpBindingConfig::new(false, false, false, None) 
        }
    ],
    save_config = SaveOptions::EveryMessage,
    wit_world = "myapp-dot-os-v0"
)]
#[derive(Default, Serialize, Deserialize)]
pub struct AppState {
    // Your state fields here
}

impl AppState {
    #[init]
    async fn initialize(&mut self) {
        // Runs on process startup
        add_to_homepage("My App", Some("🚀"), Some("/"), None);
    }
    
    #[http]
    async fn my_endpoint(&mut self, _request_body: String) -> String {
        // ALL HTTP endpoints MUST have _request_body parameter
        "Response".to_string()
    }
    
    #[remote]
    async fn handle_remote_call(&mut self, data: String) -> Result<String, String> {
        // Handle calls from other nodes
        Ok("Processed".to_string())
    }
}
```

### 3. Build Your App
```bash
# First build (installs UI dependencies)
kit bs --hyperapp

# Subsequent builds
kit b --hyperapp

# Start the app
kit s
```

### 4. Frontend Integration
Your UI must include the Hyperware runtime script:
```html
<!DOCTYPE html>
<html>
<head>
    <!-- CRITICAL: This provides window.our.node identity -->
    <script src="/our.js"></script>
    <!-- Rest of your head -->
</head>
</html>
```

## App Structure

### Directory Layout
```
myapp/
├── Cargo.toml           # Workspace configuration
├── metadata.json        # App metadata
├── myapp/              # Main process directory
│   ├── Cargo.toml      # Process dependencies
│   └── src/
│       └── lib.rs      # Process implementation
├── ui/                 # Frontend code
│   ├── index.html      # Entry point (must include /our.js)
│   └── src/            # React/TypeScript code
├── api/                # Generated WIT files
└── pkg/               # Built package output
```

### Critical Requirements

#### 1. HTTP Endpoints
**ALL HTTP endpoints MUST accept a `_request_body: String` parameter**, even if unused:
```rust
// ❌ WRONG - Will cause deserialization errors
#[http]
async fn get_data(&self) -> Vec<Data> { }

// ✅ CORRECT
#[http]
async fn get_data(&self, _request_body: String) -> Vec<Data> { }
```

#### 2. Multi-Parameter Endpoints
Frontend must send parameters as **tuples (arrays)**, not objects:
```typescript
// ❌ WRONG
await fetch('/api', {
  body: JSON.stringify({ 
    MyMethod: { param1: "a", param2: "b" }
  })
});

// ✅ CORRECT
await fetch('/api', {
  body: JSON.stringify({ 
    MyMethod: ["a", "b"]  // Tuple format
  })
});
```

#### 3. Runtime Script
The `/our.js` script is **mandatory** and provides:
```javascript
window.our = {
    node: "yournode.os",        // Node identity
    process: "app:package:publisher"  // Process identity
}
```

#### 4. Import Rules
**NEVER** add `hyperware_process_lib` to Cargo.toml dependencies. The hyperprocess macro provides it automatically:
```rust
// ✅ CORRECT - Use what the macro provides
use hyperprocess_macro::*;
use hyperware_process_lib::{our, homepage::add_to_homepage};

// ❌ WRONG - Don't add to Cargo.toml
[dependencies]
hyperware_process_lib = "..."  // DON'T DO THIS
```

#### 5. WIT Type Compatibility
Only use WIT-compatible types in handler signatures:
- ✅ Basic types: `String`, `bool`, `u8`-`u64`, `i8`-`i64`, `f32`, `f64`
- ✅ `Vec<T>`, `Option<T>` where T is supported
- ✅ Simple structs with public fields
- ❌ `HashMap` - use `Vec<(K,V)>` instead
- ❌ Fixed arrays `[T; N]` - use `Vec<T>`
- ❌ Complex enums with data

**Escape hatch**: Return JSON strings for complex data:
```rust
#[http]
async fn get_complex_data(&self, _request_body: String) -> String {
    serde_json::to_string(&self.complex_data).unwrap()
}
```

## P2P Communication Patterns

### Remote Process Calls
```rust
// Construct the target address
let publisher = "hpn-testing-beta.os";
let target_process_id = format!("app-name:package-name:{}", publisher)
    .parse::<ProcessId>()?;
let target_address = Address::new(remote_node_name, target_process_id);

// Make the remote call
let request_wrapper = json!({
    "RemoteMethodName": (param1, param2)  // Note: tuple format
});

let result = Request::new()
    .target(target_address)
    .body(serde_json::to_vec(&request_wrapper).unwrap())
    .expects_response(30)  // CRITICAL: Always set timeout
    .send_and_await_response(30)?;
```

### Fire-and-Forget Messages
```rust
let _ = Request::new()
    .target(target_address)
    .body(message_body)
    .expects_response(30)  // Still set for reliability
    .send();
```

## Important Limitations

### 1. WebSocket Handlers Not Implemented
Despite being defined in the macro, `#[ws]` handlers don't work yet. Use HTTP polling instead:
```rust
// ❌ This won't work
#[ws]
async fn handle_ws(&self, data: String) { }

// ✅ Use HTTP polling
#[http]
async fn poll_updates(&self, _request_body: String) -> Vec<Update> { }
```

### 2. No Real-time Updates Between Nodes
Don't use WebSockets for node-to-node real-time updates. Instead:
- Use Hyperware's Request API for node communication
- Implement client-side polling if needed
- Use fire-and-forget pattern for notifications

### 3. Type Generation Limitations
The WIT generator has strict requirements:
- All types must be directly referenced in handler signatures
- Nested types might not be discovered automatically
- Complex types often need to be simplified or serialized as JSON

## Development Tools

### The `kit` Command
```bash
kit b --hyperapp    # Build the app
kit bs --hyperapp   # Build and start
kit s              # Start a built app
kit r              # Remove running app
kit i <process> <json>  # Inject message for testing
```

### Testing Workflow
1. Test locally with single node
2. Test with multiple browser tabs (same node)
3. Run two local nodes for P2P testing
4. Test with remote nodes

### Debugging Tips
- Add `println!` in Rust backend
- Use `console.log` in frontend
- Check both node consoles for P2P issues
- Use browser DevTools network tab
- Common errors usually involve:
  - Missing `_request_body` parameter
  - Wrong parameter format (object vs tuple)
  - Missing `/our.js` script
  - ProcessId format issues

## Best Practices

### 1. Start from Samchat
Always copy the samchat example, not TaskManager. Samchat is a working example that demonstrates:
- Proper P2P communication
- File sharing between nodes
- Group messaging
- State management
- Error handling

### 2. Handle Node Identity Gracefully
```typescript
if (window.our?.node) {
    // Running in Hyperware
    setNodeId(window.our.node);
} else {
    // Development environment
    console.warn("Not in Hyperware environment");
}
```

### 3. Design for P2P
- Think about state synchronization
- Handle network failures gracefully
- Design for eventual consistency
- Consider offline scenarios

### 4. Keep Types Simple
- Use primitive types at API boundaries
- Serialize complex data as JSON
- Avoid deeply nested structures
- Test WIT generation frequently

## Summary

Hyperware enables truly decentralized applications where users own their data and compute. The Hyperapp framework makes development easier with async support and automatic code generation, but requires following specific patterns and requirements. Success comes from understanding the P2P architecture, following the established patterns, and designing applications that embrace decentralization rather than fighting it.

Key takeaways:
- Every user runs their own node
- Apps communicate through message passing
- State lives on individual nodes
- Follow the required patterns exactly
- Start from working examples (samchat)
- Test P2P features early and often