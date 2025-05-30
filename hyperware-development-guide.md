# Hyperware Development Guide

This guide consolidates lessons learned from developing the Hyper Poker application on the Hyperware platform.

## Table of Contents
1. [HTTP Endpoint Requirements](#http-endpoint-requirements)
2. [Runtime Requirements](#runtime-requirements)
3. [P2P Communication](#p2p-communication)
4. [Common Pitfalls](#common-pitfalls)
5. [Best Practices](#best-practices)

## HTTP Endpoint Requirements

### The _request_body Parameter

**CRITICAL**: All HTTP endpoints in Hyperware MUST accept a `_request_body: String` parameter, even if unused.

```rust
// ❌ WRONG - Will cause deserialization errors
#[http]
async fn get_games(&self) -> Vec<GameSummary> {
    // ...
}

// ✅ CORRECT - Include _request_body even if unused
#[http]
async fn get_games(&self, _request_body: String) -> Vec<GameSummary> {
    // ...
}
```

### Multi-Parameter HTTP Endpoints

When an HTTP endpoint has multiple parameters, the frontend must send them as a **tuple (array)**, not an object:

```typescript
// ❌ WRONG - Object format
const response = await makeApiCall({
  JoinRemoteGame: {
    host_node: remoteHost,
    game_id: gameId,
    buy_in: buyInAmount,
    preferred_seat: seatNumber,
  }
});

// ✅ CORRECT - Tuple/array format
const response = await makeApiCall({
  JoinRemoteGame: [
    remoteHost,
    gameId,
    buyInAmount,
    seatNumber
  ]
});
```

Parameters must be in the exact order defined in the backend function signature.

## Runtime Requirements

### The /our.js Script

The Hyperware runtime provides node identity through the `/our.js` script. This MUST be included in your HTML:

```html
<!doctype html>
<html lang="en">
  <head>
    <!-- This sets window.our.node and window.our.process -->
    <script src="/our.js"></script>
    <!-- ... rest of head ... -->
  </head>
</html>
```

Without this script:
- `window.our` will be undefined
- The UI will show "node not connected"
- WebSocket connections will fail

## P2P Communication

### ProcessId Construction

For node-to-node communication, use the string format and parse it:

```rust
let publisher = "hpn-testing-beta.os";
let target_process_id_str = format!("process-name:package-name:{}", publisher);
let target_process_id = target_process_id_str.parse::<ProcessId>()
    .map_err(|e| format!("Failed to parse ProcessId: {}", e))?;

let remote_address = Address::new(remote_node_name, target_process_id);
```

### Remote RPC Calls

For reliable remote communication, use the Request API with proper timeouts:

```rust
let request_wrapper = json!({
    "RemoteMethodName": (param1, param2)  // Note: tuple format, not array
});

let result = Request::new()
    .target(remote_address)
    .body(serde_json::to_vec(&request_wrapper).unwrap())
    .expects_response(30)  // CRITICAL: Always set timeout
    .send_and_await_response(30).unwrap();
```

### Remote vs HTTP Endpoints

- **#[http]** endpoints: Called by the frontend UI
- **#[remote]** endpoints: Called by other nodes

Remote endpoints automatically handle request deserialization based on the WIT definitions.

## Common Pitfalls

### 1. React State Race Conditions

When updating state and immediately using it, the state may not be updated yet:

```typescript
// ❌ WRONG - State might not be updated
setRemoteHost(host);
await fetchData(); // fetchData uses remoteHost from state

// ✅ CORRECT - Pass value explicitly
setRemoteHost(host);
await fetchData(host); // Pass host directly
```

### 2. Homepage Icon

To add your app to the system homepage:

```rust
#[init]
async fn initialize(&mut self) {
    add_to_homepage("App Name", Some("🎮"), Some("/"), None);
}
```

Also ensure your manifest includes the capability:
```json
{
  "grant_capabilities": [
    {
      "process": "homepage:homepage:sys",
      "params": "\"messaging\""
    }
  ]
}
```

### 3. Parameter Format Mismatch

The generated code expects tuples `(...)` not arrays `[...]` for remote calls:

```rust
// Generated code expects:
json!({"HandleJoinRequest": (game_id, player_address, player_name, buy_in, preferred_seat)})

// Not:
json!({"HandleJoinRequest": [game_id, player_address, player_name, buy_in, preferred_seat]})
```

## Best Practices

### 1. Always Add Debugging

During development, add comprehensive logging:

```rust
println!("Function called with params: {:?}", params);
println!("Current state: {:?}", self.some_field);
```

### 2. Handle Errors Gracefully

Always return proper Result types with meaningful error messages:

```rust
match some_operation() {
    Ok(result) => Ok(result),
    Err(e) => Err(format!("Operation failed: {:?}", e))
}
```

### 3. Test P2P Features Early

Don't wait until the end to test node-to-node communication. Test with at least two nodes as soon as you add remote endpoints.

### 4. Use Type-Safe Communication

Define proper types for all requests and responses in your WIT file. This ensures type safety across the frontend/backend boundary.

### 5. Understand the Build Process

The `kit b --hyperapp` command:
1. Generates WIT files from your Rust code
2. Builds the UI (npm install && npm run build)
3. Generates caller-utils for type-safe RPC
4. Compiles the Rust process
5. Packages everything into a deployable app

## Debugging Tips

### Check Console Logs

Both frontend and backend logs are crucial:
- Frontend: Browser developer console
- Backend: Terminal where the node is running

### Common Error Messages

1. **"Failed to deserialize HTTP request into HPMRequest enum"**
   - Check parameter format (tuple vs object)
   - Ensure _request_body parameter exists

2. **"Node not connected"**
   - Verify /our.js is included in HTML
   - Check that window.our.node exists

3. **"Game not found" (or similar) on remote calls**
   - Check ProcessId format
   - Verify both nodes are running
   - Check network connectivity between nodes

### Testing Checklist

- [ ] Local single-player functionality works
- [ ] Multiple players can join locally
- [ ] Remote game discovery returns results
- [ ] Remote game joining works
- [ ] Game state updates properly for all players
- [ ] Players can perform actions in turn
- [ ] Game completes and starts new hands

Remember: The Hyperware platform is designed for peer-to-peer applications. Always think about how your app will behave across multiple nodes!