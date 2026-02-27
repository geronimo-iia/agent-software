---
title: "WASM CI/CD Pipeline"
summary: "Build pipeline, Docker builder, and testing strategies for WASM modules"
read_when:
  - Setting up automated builds for WASM modules
  - Configuring Docker-based WASM builds
  - Implementing WASM module testing
status: active
last_updated: "2025-01-16"
---

# WASM CI/CD Pipeline

## GitHub Actions Workflow

### Build and Test Pipeline
```yaml
# .github/workflows/wasm-build.yml
name: WASM Build and Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
        target: wasm32-wasip2
        override: true
    
    - name: Install WASM tools
      run: |
        cargo install wasm-opt
        cargo install wasm-tools
        cargo install wit-bindgen-cli
    
    - name: Build WASM module
      run: |
        cargo build --target wasm32-wasip2 --release
        wasm-opt -Oz --enable-bulk-memory \
          target/wasm32-wasip2/release/*.wasm \
          -o dist/module.wasm
    
    - name: Test WASM module
      run: |
        cargo test
        wasm-tools validate dist/module.wasm
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: wasm-module
        path: dist/module.wasm
```

## Docker Builder

### Multi-stage Dockerfile
```dockerfile
# Dockerfile.wasm-builder
FROM rust:1.75 as builder

# Install WASM target and tools
RUN rustup target add wasm32-wasip2
RUN cargo install wasm-opt wasm-tools wit-bindgen-cli

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src/ src/
COPY wit/ wit/

# Build WASM module
RUN cargo build --target wasm32-wasip2 --release
RUN mkdir -p dist
RUN wasm-opt -Oz --enable-bulk-memory \
    target/wasm32-wasip2/release/*.wasm \
    -o dist/module.wasm

# Runtime stage
FROM scratch
COPY --from=builder /app/dist/module.wasm /module.wasm
```

### Build Script
```bash
#!/bin/bash
# scripts/build-wasm.sh

set -e

echo "Building WASM module..."

# Build with Docker for consistent environment
docker build -f Dockerfile.wasm-builder -t wasm-builder .
docker create --name temp-container wasm-builder
docker cp temp-container:/module.wasm ./dist/
docker rm temp-container

# Validate output
wasm-tools validate dist/module.wasm
echo "Build complete: dist/module.wasm"
```

## Testing Strategy

### Unit Tests
```rust
// src/lib.rs
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_execute_success() {
        let input = Input {
            parameters: "test-param".to_string(),
            context: None,
        };
        
        let result = execute(input).unwrap();
        assert_eq!(result.status, Status::Success);
        assert!(result.result.contains("test-param"));
    }
    
    #[test]
    fn test_execute_error_handling() {
        let input = Input {
            parameters: "invalid".to_string(),
            context: None,
        };
        
        let result = execute(input);
        assert!(result.is_err());
    }
}
```

### Integration Tests
```bash
#!/bin/bash
# tests/integration-test.sh

# Build module
cargo build --target wasm32-wasip2 --release

# Test with wasmtime
wasmtime run --wasi preview2 target/wasm32-wasip2/release/module.wasm

# Validate interface compliance
wasm-tools component wit target/wasm32-wasip2/release/module.wasm
```

## Release Pipeline

### Automated Releases
```yaml
# .github/workflows/release.yml
name: Release WASM Module

on:
  push:
    tags: ['v*']

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Build optimized WASM
      run: |
        # Build and optimize
        cargo build --target wasm32-wasip2 --release
        wasm-opt -Oz target/wasm32-wasip2/release/*.wasm -o module.wasm
    
    - name: Create release
      uses: softprops/action-gh-release@v1
      with:
        files: module.wasm
        generate_release_notes: true
```

## Quality Gates

- **Size limits** - WASM modules should be < 1MB optimized
- **Interface validation** - All WIT interfaces must validate
- **Performance tests** - Execution time benchmarks
- **Security scan** - Static analysis of WASM bytecode
- **Resource limits** - Test fuel and epoch deadline enforcement

## Runtime Security Testing

### Sandbox Validation
```bash
#!/bin/bash
# tests/security-test.sh

# Test resource limits
echo "Testing fuel exhaustion..."
wasmtime run --fuel 1000 target/wasm32-wasip2/release/module.wasm

# Test capability isolation
echo "Testing file access restrictions..."
wasmtime run --dir /tmp::/ target/wasm32-wasip2/release/module.wasm

# Validate bytecode integrity
echo "Validating bytecode..."
wasm-tools validate target/wasm32-wasip2/release/module.wasm
```

### Performance Benchmarks
```yaml
# .github/workflows/benchmark.yml
- name: Benchmark WASM execution
  run: |
    # Measure startup latency (should be ~1ms)
    hyperfine 'wasmtime run module.wasm'
    
    # Measure memory usage (should be ~1MB per instance)
    /usr/bin/time -v wasmtime run module.wasm
```