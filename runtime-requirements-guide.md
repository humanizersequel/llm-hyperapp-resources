# Hyperware Runtime Requirements Guide

This guide covers essential runtime requirements for Hyperware apps to function properly.

## The Missing Script Problem

### Symptoms
- UI shows "Node not connected"
- UI shows "Your ID: Unknown"
- `window.our` is undefined
- WebSocket connections fail to establish

### Root Cause
Missing the Hyperware runtime script that provides node identity.

## The Solution

Add this to your `ui/index.html` file's `<head>` section:

```html
<head>
    <!-- This sets window.our.node -->
    <script src="/our.js"></script>
    
    <!-- Your other head elements -->
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Your App</title>
</head>
```

## What `/our.js` Provides

The Hyperware runtime serves this script automatically. It sets:

```javascript
window.our = {
    node: "your-node-name.os",    // The node's identity
    process: "app-name:publisher"   // Set by your app
}
```

## How Apps Use This

### Frontend Code
```typescript
// Check if running in Hyperware environment
if (window.our?.node && window.our?.process) {
    // Initialize WebSocket with node identity
    const api = new HyperwareClientApi({
        uri: WEBSOCKET_URL,
        nodeId: window.our.node,
        processId: window.our.process,
        // ... handlers
    });
    
    // Display node ID in UI
    setMyNodeId(window.our.node);
} else {
    console.warn("Not running in Hyperware environment");
    setNodeConnected(false);
}
```

### Backend Code
```rust
// Backend can get node identity using our()
self.my_node_id = Some(our().node.clone());
```

## Build Process

When you run `kit b`, the build process:
1. Builds your UI with Vite
2. Copies everything to `pkg/ui/`
3. The runtime serves both your files AND `/our.js`

## Common Mistakes

❌ **Don't** try to include your own our.js file  
❌ **Don't** hardcode node identities  
❌ **Don't** assume window.our exists without checking  

✅ **Do** include the script tag in index.html  
✅ **Do** check if window.our exists before using it  
✅ **Do** handle the case where it's undefined (dev environment)

## Development vs Production

- **Development**: window.our might be undefined when running locally
- **Production**: window.our will always be set when running in Hyperware

This is why apps should gracefully handle both cases.

## Build Troubleshooting

### Common Build Errors and Fixes

**1. "cannot find crate"**
```rust
// ❌ Wrong
use serde::{Serialize, Deserialize};
use tokio;

// ✅ Right - hyperprocess_macro re-exports what you need
use hyperprocess_macro::{Serialize, Deserialize};
```

**2. "PartialEq not satisfied"**
```rust
// Add PartialEq to derives
#[derive(Serialize, Deserialize, PartialEq)]
pub struct YourType { }
```

**3. "WIT generation failed"**
- Return JSON strings for complex types
- No HashMap - use Vec<(K,V)>
- No fixed arrays - use Vec<T>

**4. "Module not found" in UI**
```bash
# Clean install
rm -rf ui/node_modules ui/package-lock.json
kit bs --hyperapp
```

### Actually Needed Imports

In your Rust code, you only need:
```rust
use hyperprocess_macro::*;
```

This gives you:
- Serialize, Deserialize
- #[hyperprocess], #[http], #[remote]
- our() function
- All WIT-compatible types

Don't import:
- serde separately
- wit_bindgen separately
- tokio (async is built-in)

### WebSocket Handlers Don't Work Yet

**Important**: #[ws] handlers are not implemented. Use HTTP polling or server-sent events instead.

```rust
// ❌ This won't work
#[ws]
async fn handle_ws(&self, data: String) { }

// ✅ Use HTTP instead
#[http]
async fn poll_updates(&self) -> Vec<Update> { }
```
