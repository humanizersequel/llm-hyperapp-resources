# WIT Compatibility Guide for Hyperware Apps

This guide helps you avoid common WIT (WebAssembly Interface Types) generation issues when building Hyperware applications using the hyperprocess macro framework.

## Key Constraints

### 1. Supported Types
✅ **DO use these types:**
- Basic types: `String`, `bool`, `u8`, `u16`, `u32`, `u64`, `i8`, `i16`, `i32`, `i64`, `f32`, `f64`
- `Vec<T>` where T is a supported type
- `Option<T>` where T is a supported type
- Simple structs with public fields
- Simple enums (unit variants only)

❌ **DON'T use these types:**
- `HashMap<K, V>` - use `Vec<(K, V)>` instead
- Fixed-size arrays `[T; N]` - use `Vec<T>` instead
- Complex enums with data (e.g., `Variant { field: Type }`) - use separate structs
- Trait objects or generics
- References or lifetimes
- `HashSet` - use `Vec<T>` instead

### 2. Type Visibility Rules

**Critical:** Types must be directly referenced in handler function signatures to be included in WIT generation.

```rust
// ❌ BAD: SidePot only used inside GameView, not in any function signature
pub struct SidePot { ... }
pub struct GameView {
    side_pots: Vec<SidePot>, // WIT won't find SidePot!
}

// ✅ GOOD: Add a dummy function if needed
#[http]
async fn get_side_pot(&self) -> SidePot { ... }
```

### 3. Enum Limitations

```rust
// ❌ BAD: Complex enum variants
pub enum GameEvent {
    PlayerJoined { player: Player },
    PotWon { winner: String, amount: u64 },
}

// ✅ GOOD: Simple enum with separate data struct
pub enum EventType {
    PlayerJoined,
    PotWon,
}

pub struct EventData {
    pub event_type: EventType,
    pub player: Option<Player>,
    pub winner: Option<String>,
    pub amount: Option<u64>,
}
```

## Design Patterns

### 1. Internal State Pattern
Store complex internal state as JSON to avoid WIT limitations:

```rust
#[derive(Default, Serialize, Deserialize)]
pub struct AppState {
    // Instead of: games: HashMap<String, Game>
    games_json: Vec<String>, // Serialized Game objects
}

// Internal complex type (not exposed via WIT)
struct Game {
    players: HashMap<String, Player>,
    // ... complex fields
}

// Simplified type for API responses
#[derive(Serialize, Deserialize)]
pub struct GameSummary {
    pub id: String,
    pub player_count: u8,
    // ... only simple fields
}
```

### 2. JSON String Pattern for Complex Data
For complex responses, return JSON strings:

```rust
#[http]
async fn get_game_state(&self, game_id: String) -> Result<String, String> {
    let game = self.find_game(&game_id)?;
    let state = create_game_view(game);
    Ok(serde_json::to_string(&state).unwrap())
}
```

### 3. Remote Handler Pattern
Serialize complex types for remote handlers:

```rust
// ❌ BAD: Complex type in remote handler
#[remote]
async fn handle_update(&mut self, game: Game) -> Result<bool, String>

// ✅ GOOD: Use JSON strings
#[remote]
async fn handle_update(&mut self, game_json: String) -> Result<bool, String> {
    let game: Game = serde_json::from_str(&game_json)?;
    // ...
}
```

## Debugging WIT Issues

### 1. Common Error Messages
- `"Found types used... that are neither WIT built-ins nor defined locally"` - Type isn't directly used in any handler function
- `"Error processing... Type::Array"` - Fixed-size array detected
- `"Skipping complex enum variant"` - Enum has non-unit variants

### 2. Debugging Steps
1. **Start minimal**: Begin with simple types and one handler
2. **Check samchat example**: Compare your types with the working example
3. **Add dummy handlers**: If a type isn't recognized, add a handler that returns it
4. **Simplify incrementally**: Remove complex types one by one to isolate issues
5. **Use strings as escape hatch**: When in doubt, serialize to JSON string

### 3. Build Command
Always use `kit b --hyperapp` to check WIT generation before UI issues.

## Quick Checklist

Before building, ensure:
- [ ] All struct fields are `pub`
- [ ] No HashMap, use Vec<(K,V)>
- [ ] No fixed arrays, use Vec
- [ ] No complex enum variants
- [ ] All types used in nested structs also appear in some handler signature
- [ ] Complex data is either simplified or returned as JSON strings
- [ ] Remote handlers use primitive types or JSON strings

## Example: Fixing a Complex Type

```rust
// Original (causes WIT errors):
pub struct Game {
    players: HashMap<String, Player>,
    seats: [Option<String>; 9],
    event: GameEvent::PlayerJoined { player: Player },
}

// Fixed version:
pub struct Game {
    players: Vec<Player>,        // Changed from HashMap
    seats: Vec<Option<String>>,  // Changed from array
    event_type: String,          // Simplified from complex enum
    event_data: String,          // JSON string for complex data
}
```

Remember: When facing WIT generation errors, the solution is almost always to simplify your types or serialize complex data as JSON strings.
