# Hyperprocess HTTP Endpoints Guide

This guide explains how to properly define HTTP endpoints when using the hyperprocess macro.

## The Key Rule

**ALL HTTP endpoints must accept a request body parameter**, even if they don't use it.

## Common Error

```
Failed to deserialize HTTP request into HPMRequest enum: invalid type: unit variant, expected struct variant
```

This error occurs when your HTTP endpoint doesn't accept the expected parameters.

## Correct Pattern

### Backend (Rust)

```rust
#[http]
async fn get_games(&self, _request_body: String) -> Vec<GameSummary> {
    // Even though we don't use the request body, we MUST accept it
    // The underscore prefix tells Rust we intentionally don't use it
    self.games.iter().collect()
}

#[http] 
async fn create_game(&self, request_body: String) -> Result<String, String> {
    // For endpoints that need data, parse the request body
    let request: CreateGameRequest = serde_json::from_str(&request_body)?;
    // ... handle request
}
```

### Frontend (TypeScript)

```typescript
// For endpoints with no real parameters, send empty string
const response = await fetch(`${BASE_URL}api`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ GetGames: "" })
});

// For endpoints with parameters, send the actual data
const response = await fetch(`${BASE_URL}api`, {
    method: 'POST', 
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
        CreateGame: {
            game_name: "My Game",
            config: gameConfig
        }
    })
});
```

## Why This Happens

The hyperprocess macro generates code that expects all HTTP endpoints to receive a request body. This is a consistent pattern that simplifies the generated code, even if some endpoints don't need any input data.

## WIT Generation

Note that parameters starting with underscore (like `_request_body`) will have the underscore stripped in the generated WIT file. You'll see warnings like:

```
Warning: Field name starts with an underscore ('_'), which is invalid in WIT. Stripping the underscore from WIT definition.
```

This is expected and harmless.

## Best Practices

1. Always include the `_request_body: String` parameter for parameter-less endpoints
2. Use the underscore prefix to indicate unused parameters
3. Send empty string `""` from frontend for parameter-less calls
4. Parse the request body for endpoints that need data
