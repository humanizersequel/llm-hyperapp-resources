# Hyperware Imports Guide

This guide covers common import issues when building Hyperware apps and how to resolve them.

## The Golden Rule

**DO NOT** add `hyperware_process_lib` as a direct dependency in your `Cargo.toml`. The hyperprocess macro provides it automatically.

## Common Import Errors and Solutions

### Error: `hyperware_process_lib` is ambiguous

```
error[E0659]: `hyperware_process_lib` is ambiguous
 --> src/lib.rs:2:5
  |
2 | use hyperware_process_lib::{our, homepage::add_to_homepage};
  |     ^^^^^^^^^^^^^^^^^^^^^ ambiguous name
```

**Cause**: You have `hyperware_process_lib` in your Cargo.toml dependencies.

**Solution**: Remove it from Cargo.toml. The hyperprocess macro already provides access to it.

### Correct Import Pattern

Follow the samchat example:

```rust
use hyperprocess_macro::hyperprocess;
use hyperware_process_lib::{our, homepage::add_to_homepage};
use serde::{Deserialize, Serialize};
```

### Correct Cargo.toml Dependencies

```toml
[dependencies]
anyhow = "1.0"
chrono = { version = "0.4", features = ["serde"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
uuid = { version = "1.4.1", features = ["v4", "serde"] }
wit-bindgen = "0.36.0"
# ... other dependencies

# DO NOT ADD hyperware_process_lib here!

[dependencies.hyperprocess_macro]
git = "https://github.com/hyperware-ai/hyperprocess-macro"
rev = "47400ab"

[dependencies.hyperware_app_common]
git = "https://github.com/hyperware-ai/hyperprocess-macro"
rev = "47400ab"
```

## Why This Happens

The hyperprocess macro expands to include imports from `hyperware_process_lib`. When you also add it as a direct dependency, Rust sees two different ways to access the same crate:
1. Through your direct dependency
2. Through the macro expansion

This creates ambiguity that prevents compilation.

## Best Practices

1. **Copy working examples**: Look at samchat or other working apps for the correct pattern
2. **Keep dependencies minimal**: Only add what you actually need
3. **Use consistent versions**: Ensure git dependencies use the same revision across your project
4. **Trust the macro**: The hyperprocess macro handles most imports for you

## Debugging Tips

If you see import errors:
1. Check if you have `hyperware_process_lib` in Cargo.toml - remove it
2. Ensure you're using the import pattern from working examples
3. Run `cargo clean` and rebuild if you've made dependency changes
4. Check that all git dependencies use consistent revisions

## What the Macro Provides

The hyperprocess macro automatically makes available:
- `our()` - access to process identity
- `homepage::add_to_homepage()` - homepage registration
- Request/Response types through generated code
- All necessary trait implementations

You don't need to manually import the process lib to access these.
