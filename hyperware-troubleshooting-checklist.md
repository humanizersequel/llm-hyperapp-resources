# Hyperware Troubleshooting Checklist

Quick reference for common issues and their solutions.

## Build Errors

### ❌ "Field name starts with an underscore ('_'), which is invalid in WIT"
**Solution**: This is just a warning. The build process automatically strips the underscore.

### ❌ "Failed to deserialize HTTP request into HPMRequest enum"
**Causes & Solutions**:
1. Missing `_request_body` parameter in HTTP endpoint
2. Wrong parameter format (object vs tuple) in frontend
3. Mismatched parameter types

## Runtime Errors

### ❌ "Node not connected" in UI
**Checklist**:
- [ ] Is `<script src="/our.js"></script>` in your HTML `<head>`?
- [ ] Is it BEFORE your main app script?
- [ ] Check browser console for `window.our.node`
- [ ] Verify the node is running

### ❌ "Failed to query games from [node]: SendError"
**Checklist**:
- [ ] Is the remote node running?
- [ ] Is the node name spelled correctly?
- [ ] Check ProcessId format: `"app-name:package-name:publisher"`
- [ ] Verify both nodes have the same publisher

### ❌ Remote function calls not working
**Checklist**:
- [ ] Using `.expects_response(30)` on Request?
- [ ] Using tuple format `(...)` not array `[...]` in JSON?
- [ ] Remote endpoint marked with `#[remote]`?
- [ ] Both nodes running the same app version?

## State Management Issues

### ❌ "Game not found" after successfully joining
**Common Cause**: Race condition between state updates
**Solution**: Pass values explicitly instead of relying on React state

### ❌ UI not updating after actions
**Checklist**:
- [ ] Is polling set up correctly?
- [ ] Are you fetching from the right source (local vs remote)?
- [ ] Check WebSocket connection in browser console

## P2P Communication Issues

### ❌ Can't see games on remote host
**Debug Steps**:
1. Check host node console for "get_remote_games called"
2. Verify response format matches WIT definitions
3. Check for type conversion errors (caller_utils types vs local types)

### ❌ Can join but can't interact with remote game
**Common Issues**:
1. Remote action endpoints not implemented
2. Missing game state synchronization
3. Turn validation failing

## Quick Fixes

### Frontend Not Loading
```bash
# Rebuild everything
kit b --hyperapp

# If that fails, clean build
rm -rf target/
rm -rf ui/node_modules ui/dist
kit b --hyperapp
```

### Type Mismatches
1. Check WIT file for latest definitions
2. Ensure frontend types match WIT exactly
3. Rebuild to regenerate caller-utils

### WebSocket Issues
```javascript
// Check connection in browser console
console.log(window.our);
// Should show: { node: "yournode.os", process: "app-name:package:publisher" }
```

## Performance Tips

### Slow Game Updates
- Reduce polling interval (but not too low)
- Batch state updates
- Use WebSocket for real-time updates

### Large State Objects
- Only send changed data
- Use pagination for game lists
- Compress large payloads

## Development Workflow

### Recommended Testing Order
1. Test locally with single player
2. Test locally with multiple browser tabs
3. Test with two local nodes
4. Test with remote nodes

### Debugging P2P
1. Add console.log in frontend
2. Add println! in backend
3. Check both node consoles
4. Use browser network tab to inspect requests

Remember: Most issues are either:
- Parameter format mismatches
- Missing configuration (like /our.js)
- Race conditions in state management
- ProcessId format issues