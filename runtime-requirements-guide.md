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
