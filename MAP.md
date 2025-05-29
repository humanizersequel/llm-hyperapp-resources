# LLM Hyperapp Resources Code Map

## Overview
This directory contains guides and resources for building Hyperware applications using the Hyperapp macro-driven framework.

## File Directory

### http-endpoints-guide.md
- **Purpose**: Explains proper HTTP endpoint definition with hyperprocess macro
- **Helpful for**: Fixing "Failed to deserialize HTTP request" errors
- **Key rule**: ALL HTTP endpoints must accept a request body parameter (even if unused)
- **Common fix**: Add `_request_body: String` parameter to parameter-less endpoints

### hyperware-imports-guide.md
- **Purpose**: Resolves common import ambiguity errors
- **Helpful for**: Fixing "hyperware_process_lib is ambiguous" errors
- **Key rule**: NEVER add `hyperware_process_lib` as direct dependency in Cargo.toml
- **Solution**: The hyperprocess macro provides it automatically

### runtime-requirements-guide.md
- **Purpose**: Covers essential runtime requirements for Hyperware apps
- **Helpful for**: Fixing "Node not connected" and undefined `window.our` issues
- **Key requirement**: Must include `<script src="/our.js"></script>` in UI index.html
- **Provides**: Node identity (window.our.node) and process info

### wit-compatibility-guide.md
- **Purpose**: Avoids WIT (WebAssembly Interface Types) generation issues
- **Helpful for**: Fixing type compatibility errors during build
- **Key constraints**: 
  - Use only WIT-compatible types (no HashMap, fixed arrays, complex enums)
  - All types must be referenced in handler function signatures
  - Use JSON strings for complex data as escape hatch
- **Common fixes**: Replace HashMap with Vec<(K,V)>, fixed arrays with Vec<T>

## Quick Reference

| Problem | Guide to Check |
|---------|---------------|
| HTTP deserialization errors | http-endpoints-guide.md |
| Import ambiguity errors | hyperware-imports-guide.md |
| UI shows "Node not connected" | runtime-requirements-guide.md |
| WIT generation/type errors | wit-compatibility-guide.md |