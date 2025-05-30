# Hyperware P2P Communication Guide

This guide explains how to implement peer-to-peer (node-to-node) communication in Hyperware applications.

## IMPORTANT: Real-time Updates in Hyperware

**DO NOT use WebSockets for node-to-node real-time updates.** Hyperware has its own internal P2P messaging system. WebSocket endpoints in the hyperprocess macro are not yet functional and should not be relied upon.

For real-time game updates (e.g., player joins, moves, state changes):
- Use Hyperware's Request API to send messages between nodes
- Implement polling on the client side if needed
- Use the fire-and-forget pattern (shown below) for broadcasting updates

## Key Patterns from Samchat

### 1. ProcessId Construction
When making remote calls, use the string format and parse it:
```rust
let publisher = "hpn-testing-beta.os";
let target_process_id_str = format!("process_name:package_name:{}", publisher);
let target_process_id = target_process_id_str.parse::<ProcessId>()
    .map_err(|e| format!("Failed to parse ProcessId: {}", e))?;
```

### 2. Address Construction
```rust
let target_address = Address::new(remote_node_name, target_process_id);
```

### 3. Remote Request Pattern
For remote calls, use the Request API directly with JSON-wrapped payloads:
```rust
let request_wrapper = json!({
    "RemoteMethodName": parameters
});

let result = Request::new()
    .target(target_address)
    .body(serde_json::to_vec(&request_wrapper).unwrap())
    .expects_response(30)  // CRITICAL: Always set timeout for remote calls
    .send_and_await_response(30).unwrap();
```

### 4. Fire-and-Forget Pattern
For notifications that don't need responses:
```rust
let _ = Request::new()
    .target(target_address)
    .body(request_body)
    .expects_response(30)  // Still set this for reliability
    .send();
```

## Key Differences from Generated Code

The generated caller_utils use a `send` function that may not properly set `expects_response`. For reliable remote communication, consider using the direct Request API as shown above.

## Common Issues

1. **"Failed to deserialize" errors**: Usually means the ProcessId format is incorrect
2. **No response from remote node**: Ensure `expects_response` is set
3. **Connection failures**: Verify both nodes are running and the node names are correct (e.g., "alice.os", "bob.os")

## Testing P2P Features

1. Run two instances of your app on different nodes
2. Use the actual node names (not "placeholder.os") when testing
3. Check logs on both nodes to debug communication issues