# Hyperapp Quickstart Guide

**ALWAYS copy samchat, not TaskManager**

TaskManager is broken. Samchat works. Start from samchat every time.

## Step-by-Step Guide

### 1. Copy samchat
```bash
cp -r /path/to/samchat myapp
cd myapp
```

### 2. Rename Everything
```bash
# Update package name in metadata.json
sed -i '' 's/samchat/myapp/g' metadata.json

# Update Cargo.toml files
sed -i '' 's/samchat/myapp/g' Cargo.toml
sed -i '' 's/samchat/myapp/g' samchat/Cargo.toml

# Rename directory
mv samchat myapp
```

### 3. Files to Modify

#### `myapp/src/lib.rs`
- Your backend logic
- Keep the imports as-is (from hyperprocess_macro)
- Keep the #[hyperprocess] macro
- Modify the AppState struct for your needs
- Add your HTTP handlers with #[http]

#### `metadata.json`
```json
{
  "name": "myapp",
  "publisher": "yourname"
}
```

#### `ui/src/store/samchat.ts`
- Rename to match your app
- Update the API endpoints
- Keep the WebSocket structure intact

#### `ui/src/types/samchat.ts`
- Rename to match your app
- Update types to match your WIT output

#### `ui/index.html`
**CRITICAL**: Keep this script tag:
```html
<script src="/our.js"></script>
```

### 4. Build Commands
```bash
# First time - installs UI dependencies
kit bs --hyperapp

# Subsequent builds
kit b --hyperapp

# Start
kit s
```

### 5. Common Fixes

**PartialEq error**: Add `#[derive(PartialEq)]` to your structs

**WIT type error**: Return JSON strings instead of complex types

**WebSocket not connecting**: Check you have `/our.js` script tag

**Import errors**: Use `hyperprocess_macro::*` not separate crates

### 6. What NOT to Do
- Don't copy TaskManager
- Don't remove the `/our.js` script
- Don't use HashMap in WIT types
- Don't import serde separately
- Don't try WebSocket handlers (not implemented yet)

### 7. Minimal Working Example
```rust
use hyperprocess_macro::*;

#[hyperprocess]
#[derive(Default, Serialize, Deserialize)]
pub struct AppState {
    counter: u32,
}

#[http]
async fn increment(&mut self) -> u32 {
    self.counter += 1;
    self.counter
}

#[http]
async fn get_count(&self) -> u32 {
    self.counter
}
```

That's it. Start from samchat, modify incrementally, test often.